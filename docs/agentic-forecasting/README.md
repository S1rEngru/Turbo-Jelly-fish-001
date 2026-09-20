# Agent辅助时序预测建模调研

调研始于2026-09-16，更新至2026-09-18。围绕已有NeuralForecast销售预测baseline，以及跨品牌适配的人力成本展开。当前重点为实验Harness与时序任务特化，先读09和10。

**首轮调研结论（历史）：已有相近实例，曾优先研究TimeCopilot、Microsoft Finn、金融训练TS-Agent和TimeSeriesScientist。此后经代码核查和任务重定位调整了推荐顺序，最新结论见09和10；企业来源不等于已验证生产收益。**

## 阅读入口

**最新方向（2026-09-18）：从算法选型转向可控、可恢复的实验 Harness。**

- [09 开源 Harness 架构调研](09_open_source_harness_architectures.md)：公开生态筛选，Deep Agents、AgentScope、AIDE、SWE-agent、mini-swe-agent 的代码机制、权限与上下文、失败恢复及验收。
- [10 时序预测训练 Agent 特化模块](10_forecasting_agent_specialization.md)：Finn 与 TimeCopilot 的实际实现，映射现有训练脚本的工具、诊断、评估、实验记忆和验证案例。
- [11 固定版本源码清单](11_harness_source_manifest.json)：7 个仓库、23 份源码/测试/许可文件的 commit、链接与 SHA-256；测试仅阅读，未执行。

**[模型、特征、超参联合优化与 Agent 工作流](08_joint_optimization_and_agent_workflow.md)**：2026-09-18 源码调研，重点分析 RD-Agent 决策闭环、FLAML 成本调度与现有训练脚本的结合方式。

**[降低重训成本与有依据的参数搜索](07_retraining_cost_and_search_spaces.md)**：NeuralForecast 续训边界、搜索空间、预算分配及多指标生产约束，附原始出处。

**[结合现有滚动训练逻辑的 Top 5 深入调研](06_top5_for_existing_training_workflow.md)**：场景映射、方法和源码核查、更新排名、后续修订项。TSci 因代码核查发现下调为第五。

0. **[六个项目详细总结与阅读指南](04_deep_dive_guide.md)**：2026-09-17 新增，每个项目按背景、问题、难点、实现方法、效果展开，并分析对现有 baseline 的参考价值。
1. [调研报告](01_report.md)：企业案例、核心项目、相关论文、比较矩阵，以及对现有baseline的启示。
2. [来源目录](02_sources.md)：56条来源记录，包含原始链接、机构／作者、日期／版本、查阅程度和证据限制。
3. [检索记录](03_search_log.md)：检索关键词、追溯路径、未能访问的内容、暂缓采信线索和下一阶段核实任务。

## 建议先读

- 报告第1节：方向是否有已有实例。
- 第3节：哪些属于企业实践，哪些只能算研发或产品能力。
- 第4节：最贴近NeuralForecast和训练脚本的参考。
- 第7–8节：怎样验证Agent价值，以及如何接上baseline。

本目录保存调研文档和出处，不包含第三方项目完整代码、论文全文或任何已执行的训练实验。首轮来源截止日为2026-09-16；详细总结补充核验至2026-09-17，见[本轮来源记录](05_deep_dive_sources.md)。动态页面后续可能变化。
