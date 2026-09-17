# Microsoft Finn：优先参考持续预测和更新机制

查阅日期：2026-09-17。定位：微软开源预测项目 finnts 的 Agent 工作流，主要使用 R。

## 1、背景

Finn 的 Agent 建立在现有预测流程之上，提供首次迭代、使用新数据更新、结果问答三个入口。官方示例使用 M4 月度序列，LLM 通过 ellmer 的工具调用接口参与流程。[Agent 文档](https://microsoft.github.io/finnts/articles/ai-agent.html)

**入选理由（本项目分析）**：跨品牌工作量不仅来自第一次训练，也来自每次上新数据后的重训、检查和异常处理。Finn 为这一持续运行问题提供了比单次演示更具体的参考。

## 2、问题

系统需要管理每次运行的版本、每条序列的最佳结果、汇总预测及诊断材料。文档明确列出 `agent_version`、`run_id`、最佳运行表、EDA 和模型摘要等产物。[Agent 文档](https://microsoft.github.io/finnts/articles/ai-agent.html)

**映射到本项目**：如果品牌数量增加，仅保存一个最终模型文件会难以回答“为什么本月换了模型”“哪个品牌变差了”“新方案是否真的优于上次”。实验与预测版本应该是一等产物。

## 3、难点

`update_forecast` 文档区分正常更新和重新开启搜索：默认不允许自动迭代；启用后，退化条件涉及超过 40% 的序列、WMAPE 比前次恶化超过 20%。新序列或部分失败序列可以走默认本地模型，但数量过多时要求重新迭代。[更新 API](https://microsoft.github.io/finnts/reference/update_forecast.html)

**本项目分析**：这体现了三个值得借鉴的问题。

1. 每次更新都搜索，训练和 LLM 成本可能快速增加；完全不搜索又可能错过数据变化。
2. 少量异常品牌应能局部处理，不能拖垮整批预测。
3. 大品牌和小品牌的误差重要性不同，聚合指标可能掩盖局部问题。

上述阈值只是该版本的规则，不能直接作为销售场景的推荐阈值。需要按实际回测波动、品牌规模和业务容忍度设置。

## 4、实现方法

### 文档确认的流程

先定义项目、序列组合字段、目标列、频率和预测长度，再把数据交给 Agent。系统产生预测与诊断，并允许查询运行结果。[Agent 文档](https://microsoft.github.io/finnts/articles/ai-agent.html)

首次优化使用 `iterate_forecast`，其中 `max_iter` 限制轮数，`weighted_mape_goal` 指定目标；支持本机或 Spark 并行。[迭代 API](https://microsoft.github.io/finnts/reference/iterate_forecast.html)

之后调用 `update_forecast` 复用以往方案处理最新数据，在允许且满足条件时重新优化；局部失败存在回退路径。[更新 API](https://microsoft.github.io/finnts/reference/update_forecast.html)

### 如何迁移到 Python / NeuralForecast（建议）

无需为了参考 Finn 把现有项目重写成 R。可以提取以下状态：

| 状态 | 程序行为 | Agent 是否需要参与 |
|---|---|---|
| 新品牌首次接入 | 诊断并运行候选实验 | 可以参与配置建议 |
| 常规更新 | 复用已确认配置，生成新预测 | 可按需参与解释 |
| 指标持续恶化 | 发起有预算上限的搜索 | 可以分析并提出调整 |
| 数据校验失败 | 标记失败并定位原因 | 可生成说明，不能掩盖错误 |
| 候选未改善 | 保留已确认方案 | 总结失败原因 |

再定义一个“最佳方案登记表”：数据版本、品牌、模型配置、回测窗口、分窗口指标、选中理由、启用时间。比较必须发生在可比数据与窗口上；不能把旧数据上的旧分数直接与新数据上的新分数混为模型改进。

## 5、效果

本次核查的 Agent 文档提供了可跟随的使用流程、输出结构和更新规则，但没有提供足以单独量化“增加 LLM Agent 后收益”的严格对照。[Agent 文档](https://microsoft.github.io/finnts/articles/ai-agent.html)

微软财务预测材料提到历史流程的时间和成本收益，但这些数字不能自动归到这里的新 Agent 功能。引用企业效果时应保持这一区别。[微软财务预测材料](https://cdn-dynmedia-1.microsoft.com/is/content/microsoftcorp/microsoft/mcaps/documents/fy26/Financial-forecasting.pdf)

对本项目，最有价值的验收问题是：定期更新时需要人工处理的品牌比例是否下降；失败是否能局部恢复；预测变差是否能被可靠发现。即使平均误差没有显著改善，这些指标也可能证明自动化价值。

## 出处与阅读顺序

1. [AI Agent 完整指南](https://microsoft.github.io/finnts/articles/ai-agent.html)。
2. [iterate_forecast](https://microsoft.github.io/finnts/reference/iterate_forecast.html)。
3. [update_forecast](https://microsoft.github.io/finnts/reference/update_forecast.html)。
4. [官方仓库](https://github.com/microsoft/finnts)。
5. [微软财务预测材料](https://cdn-dynmedia-1.microsoft.com/is/content/microsoftcorp/microsoft/mcaps/documents/fy26/Financial-forecasting.pdf)：仅作业务背景。

核验范围：官方文档与公开业务材料；未复现 Agent，也未审计微软内部运行。
