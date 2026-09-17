# Microsoft RD-Agent：优先参考实验迭代框架

查阅日期：2026-09-17。本篇基于官方 v0.8.0 文档站；项目为微软开源研发 Agent。[仓库](https://github.com/microsoft/RD-Agent)

## 1、背景

RD-Agent 将研发活动组织成提出假设、设计实验、实现代码、运行获取反馈、继续修改的循环。它覆盖多个应用场景，不是专门面向销量预测的成品。[框架说明](https://rdagent.readthedocs.io/en/latest/project_framework_introduction.html)

**入选理由（本项目分析）**：跨品牌探索容易积累大量“试过但忘了”的配置。比起只自动调用模型，RD-Agent 的思路更适合研究如何保留实验动机、实现和反馈。

## 2、问题

Finance Model Agent 围绕 Qlib 自动构造模型并回测；Data Science Agent 则支持特征工程和模型调优，并提供自定义数据任务的准备流程。[金融模型场景](https://rdagent.readthedocs.io/en/latest/scens/model_agent_fin.html)、[数据科学场景](https://rdagent.readthedocs.io/en/latest/scens/data_science.html)

**映射到本项目**：当已有 baseline 的性能不够时，Agent 需要明确“下一步为什么要试这个”，再把建议变成实际可比较的实验。单纯生成一段建议文字不算完成这个循环。

## 3、难点

**本项目分析**：实验循环中至少有四类失败需要分别记录：

| 失败类型 | 例子 | 下一步应关注什么 |
|---|---|---|
| 假设不清楚 | 同时改模型、窗口和特征 | 缩小改动，使结果可归因 |
| 实现不正确 | 所谓新特征没有进入训练 | 检查实际执行与数据流 |
| 实验不公平 | 新方案拥有更多预算 | 对齐预算、切分和种子 |
| 改善不稳定 | 只在一个时间窗口较好 | 检查其他窗口和品牌 |

官方 Data Science 页面的自定义示例是医疗时序分类，包含样本级随机划分，不能直接照搬为销售未来预测的时间切分。[数据科学场景](https://rdagent.readthedocs.io/en/latest/scens/data_science.html)

## 4、实现方法

### 文档确认的设计

金融模型场景给出了假设、任务转换、代码实现、Qlib 回测、反馈和再迭代的步骤；示例使用 CSI300 与 Alpha158 中的 20 个因子，并明确列出训练、验证和测试时间范围。[金融模型场景](https://rdagent.readthedocs.io/en/latest/scens/model_agent_fin.html)

通用数据科学场景要求准备任务描述、训练／预测输入、提交格式以及独立评分所需材料。接入新数据并不是只填写一个模型名称。[数据科学场景](https://rdagent.readthedocs.io/en/latest/scens/data_science.html)

### 如何借鉴（建议）

围绕现有 baseline 定义一张实验卡，而不必立刻引入整个框架：

```text
假设：某品牌的长历史窗口可能混入旧销售阶段，较短窗口可能更好。
证据：固定诊断工具生成的趋势变化和分窗口误差。
动作：仅修改允许的 input_size 候选，其他配置保持一致。
执行：运行原训练脚本，并分配固定训练预算。
评估：使用相同的滚动窗口，记录各窗口与总指标。
结论：接受、拒绝或证据不足；保留全部结果和下一步建议。
```

这只是拟议的实验形式，不是已经确认用户数据存在阶段变化。它的价值在于让“Agent 反思”对应可观察事实。

如果之后需要更自由的模型结构实验，可以再研究 RD-Agent 中场景、编码器、运行器与反馈组件的接口。已有脚本可作为运行器后端；评分逻辑仍由项目固定控制。

## 5、效果

官方 Benchmark 页面提供实现能力的评测流程，并指向 RD2Bench；这类基准关注 Agent 将因子或模型说明实现成代码的能力。[Benchmark 文档](https://rdagent.readthedocs.io/en/latest/research/benchmark.html)

本次查阅的这些页面没有提供可直接归因到“跨品牌销售预测”的量化提升。因此本篇不填写一个看似可比的精度百分比。场景示例证明有可参考的实现路径，不能代替实际收益评估。

本项目可以检验：同样时间和费用下，系统完成多少有效实验、重复失败比例多高、能否追溯每次选择，以及相对固定调参是否获得更好的候选。若只是增加叙述而没有提高有效实验比例，就不足以说明研发 Agent 有价值。

## 出处与阅读顺序

1. [框架设计](https://rdagent.readthedocs.io/en/latest/project_framework_introduction.html)。
2. [Finance Model Agent](https://rdagent.readthedocs.io/en/latest/scens/model_agent_fin.html)。
3. [Data Science Agent](https://rdagent.readthedocs.io/en/latest/scens/data_science.html)。
4. [Benchmark](https://rdagent.readthedocs.io/en/latest/research/benchmark.html)。
5. [官方仓库](https://github.com/microsoft/RD-Agent)。

核验范围：文档所述结构和示例；没有复现量化策略、竞赛结果或本项目训练。
