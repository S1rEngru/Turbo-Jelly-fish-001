# 六项目深读：来源与核验记录

查阅日期：2026-09-17。本文补充首轮的 [56 条来源目录](02_sources.md)，不替换原记录。以下记录对应本轮详细总结的正文；同一论文的 HTML 与摘要页属于同一来源，不计为独立证据。

## 来源索引

| 编号 | 官方来源 | 查阅位置／用途 | 限制 |
|---|---|---|---|
| D01 | [TimeCopilot 概览](https://timecopilot.dev/) | 输入、输出、Agent 与预测器关系 | 动态文档 |
| D02 | [TimeCopilot Agent API](https://timecopilot.dev/api/agent/) | 构造器、方法和预测器参数 | 未运行 |
| D03 | [TimeCopilot neural API](https://timecopilot.dev/api/models/neural/) | AutoNHITS 签名、后端、分位数与频率约束 | 接口不能证明业务适配 |
| D04 | [TimeCopilot GIFT-Eval](https://timecopilot.dev/experiments/gift-eval/) | 数据规模、集成成员、成本与结果说明 | 固定基础模型集成；历史排名 |
| D05 | [TimeCopilot 仓库](https://github.com/TimeCopilot/timecopilot) | 项目与代码入口 | 没有锁定提交 |
| D06 | [TimeCopilot 论文](https://arxiv.org/abs/2509.00616) | 研究背景补充入口 | 不用摘要替代 API 核查 |
| D07 | [Finn AI Agent](https://microsoft.github.io/finnts/articles/ai-agent.html) | 首次流程、产物、数据约定、问答 | 文档演示，非独立收益评测 |
| D08 | [Finn iterate_forecast](https://microsoft.github.io/finnts/reference/iterate_forecast.html) | 轮数、目标、并行选项 | 默认参数不是业务推荐 |
| D09 | [Finn update_forecast](https://microsoft.github.io/finnts/reference/update_forecast.html) | 退化触发、默认关闭搜索、失败回退 | 页面部分百分号显示异常；未引用含混的新序列数量表达 |
| D10 | [Finn 仓库](https://github.com/microsoft/finnts) | 官方实现入口 | 未执行 R 环境 |
| D11 | [微软财务预测材料](https://cdn-dynmedia-1.microsoft.com/is/content/microsoftcorp/microsoft/mcaps/documents/fy26/Financial-forecasting.pdf) | 沿用首轮业务背景核查 | 不能归因到新 LLM Agent |
| D12 | [TSci 仓库](https://github.com/Y-Research-SBU/TimeSeriesScientist) | README 五模块、配置、产物、许可证 | README 能力未逐项运行 |
| D13 | [TSci 论文 v2](https://arxiv.org/html/2510.01538v2) | 方法 3.1–3.5、实验与表 1 | 作者实验；与当前代码不必完全一致 |
| D14 | [MLZero 论文 v1](https://arxiv.org/html/2505.13941v1) | 数据理解、语义／情景记忆、迭代编码 | 原始方法版本 |
| D15 | [Amazon Science 介绍](https://www.amazon.science/blog/autogluon-assistant-zero-code-automl-through-multiagent-collaboration) | MAAB、MLE-bench Lite 的成功率和定义 | 官方自报；综合 ML 任务 |
| D16 | [AutoGluon Assistant 仓库](https://github.com/autogluon/autogluon-assistant) | Linux-only、研究代码声明、版本消息 | 后续公告不等于代码已发布 |
| D17 | [RD-Agent 框架](https://rdagent.readthedocs.io/en/latest/project_framework_introduction.html) | 假设、实现、执行、反馈 | 架构说明 |
| D18 | [RD-Agent 金融模型场景](https://rdagent.readthedocs.io/en/latest/scens/model_agent_fin.html) | Qlib 示例、模型实验、时间切分 | 金融示例，不是销售验证 |
| D19 | [RD-Agent 数据科学场景](https://rdagent.readthedocs.io/en/latest/scens/data_science.html) | 自定义任务文件及医疗分类示例 | 随机样本划分不直接适用未来销量预测 |
| D20 | [RD-Agent Benchmark](https://rdagent.readthedocs.io/en/latest/research/benchmark.html) | 实现能力评测与 RD2Bench 入口 | 没有据此提取销量提升数字 |
| D21 | [RD-Agent 仓库](https://github.com/microsoft/RD-Agent) | 官方实现入口 | 未复现竞赛或量化效果 |
| D22 | [金融 TS-Agent 论文 v1](https://arxiv.org/html/2508.13915v1) | 资源、训练模板、阶段搜索、实验表 1 | 官方代码未确认；需审计反馈集隔离 |

## 核对过的关键口径

- **TimeCopilot**：GIFT-Eval 页面对应基础模型中位数集成。没有把集成成绩归为 LLM 选择策略的独立贡献。
- **Finn**：`allow_iterate_forecast` 默认 FALSE；退化条件中的 40% 和 20% 来自更新 API；未将目标 WMAPE 参数写成已取得的精度。
- **TSci**：论文四角色，README 五模块。表 1 的 ECL 中 MAE 和 MAPE 的优劣方向不同；正文保留这个反例。
- **MLZero**：86% 对应 18/21 有效提交；92% 对应 MAAB 任务成功率。2025 论文与后来 MCTS/ExTS 更新分开记录。
- **RD-Agent**：医疗自定义数据示例属于分类任务，避免因为输入含时间维度就当作销量外推示例。
- **TS-Agent**：股票表选定行 MAE 相对变化为 `(5.258-4.912)/5.258 ≈ 6.58%`；标明这是本次计算、单个设置，不是论文整体结论。

## 访问与取舍记录

1. 先尝试的 RD-Agent `/scenarios/data_science.html` 未成功访问；之后从官方目录导航到有效地址 `/scens/data_science.html`，正文仅引用后者。
2. Finn 部分参考页初次只返回页面元信息；再次按行打开，读取参数及正文后再记录规则。
3. TimeCopilot API 页面包含较长源代码与行号块，采用定位 AutoNHITS 的方式核查签名，没有声称完成全仓库审计。
4. TSci 消融段落中部分文字方向值得进一步核对，本轮没有引用可疑的百分比；采用表 1 的明确数值。
5. 金融 TS-Agent 未确认官方代码，因此没有将它列入已确认开源项目，也没有提供猜测的 GitHub 地址。
6. 未下载论文全文或镜像第三方仓库；本地保存的是中文总结、出处链接、访问日期与证据限制。

## 交付检查

每篇保留五个指定维度；项目事实与迁移建议分开；结果标注作者自报且未独立复现；阅读入口和本地相对链接进行存在性检查。最终需要运行验证的事项集中在各篇“核验范围”，没有以安装成功或代码存在替代实验效果。
