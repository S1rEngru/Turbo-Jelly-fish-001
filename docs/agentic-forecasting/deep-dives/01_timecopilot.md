# TimeCopilot：优先参考模型工具接口

查阅日期：2026-09-17。定位：开源时序预测 Agent；本篇重点是与 NeuralForecast 的连接方式。

## 1、背景

TimeCopilot 把语言模型与已有预测工具结合，提供数据分析、预测和结果问答入口。它同时面向预测执行和预测解释。[官方概览](https://timecopilot.dev/)

**入选理由（本项目分析）**：当前已经有训练 baseline，最需要先解决的是如何让 Agent 稳定调用它。这个项目能够提供接口层面的参考，而不要求先把预测模型替换成大语言模型。

## 2、问题

官方接口将不同预测器放进统一工作流，输出数据特征、候选比较、选择理由和预测结果。数据采用 `unique_id / ds / y` 长表形式。[官方概览](https://timecopilot.dev/)

**映射到本项目**：新品牌接入会重复发生三件事：准备数据、组织候选实验、解释为何选择某个结果。统一接口可以减少重复代码；但字段与业务口径能否统一，仍取决于品牌数据本身。

## 3、难点

文档暴露出实际接入约束：AutoNHITS 对部分频率有最短长度要求；概率预测使用 `quantiles`，不能直接用 `level`。这说明“模型都能预测”并不代表所有模型具有完全相同的输入与能力。[神经模型 API](https://timecopilot.dev/api/models/neural/)

**本项目分析**：需要特别区分三个问题。

- 接口兼容：训练脚本是否允许外部传入参数，并稳定返回预测与指标？
- 任务兼容：现有模型是否使用静态品牌特征、历史协变量或未来已知变量？简单三列表不能表达全部业务约束。
- 评估兼容：各候选是否使用相同预测长度、时间切分和指标？如果不一致，Agent 的选择理由再完整也不具备比较意义。

这些是迁移时需要核实的事项，不是断言 TimeCopilot 已完整解决跨品牌建模。

## 4、实现方法

### 文档确认的设计

`TimeCopilot` 接受 LLM 与预测器列表，提供分析、预测和查询能力；预测器是独立的可配置组件。[Agent API](https://timecopilot.dev/api/agent/)

NeuralForecast 接入的具体例子是 `AutoNHITS`：参数包括 `num_samples`、`backend` 和 `config`，文档默认搜索后端为 Optuna。底层工具执行模型拟合和数值搜索，LLM 的工具调用与这些优化步骤是不同层次。[神经模型 API](https://timecopilot.dev/api/models/neural/)

可以将其概括为：输入数据 → 工具计算特征和候选结果 → Agent 组织分析与选择 → 返回预测及说明。这里是调研者对接口职责的整理，不表示每个运行都严格按固定顺序调用所有工具。

### 如何借鉴到现有 baseline（建议）

先做一个适配层，让原训练脚本接收明确配置，输出结构化结果，例如：

| 输入 | 输出 |
|---|---|
| 数据版本、品牌、预测起点、预测长度 | 对应版本的逐时点预测 |
| 模型名、参数、随机种子、预算 | 模型产物位置、训练时间、退出状态 |
| 固定的回测协议编号 | 各窗口指标、聚合指标、错误摘要 |

Agent 第一阶段只在允许的配置中选择，不需要获得修改整个项目的权限。数值参数可以由现有 Auto 模型或搜索器处理。只有当常规配置无法表达实验时，再考虑增加有限的代码修改能力。

阅读代码时优先追踪 `timecopilot/models/neural.py` 的预测器封装，再看 Agent API 如何接收预测器；不要从大量模型实现逐个读起。[官方仓库](https://github.com/TimeCopilot/timecopilot)

## 5、效果

官方 GIFT-Eval 页面报告：对 24 个数据集、超过 14.4 万条序列和 1.77 亿数据点进行评估，计算成本低于 30 美元；该页面版本使用 Chronos-2、TimesFM-2.5、TiRex 的中位数集成，报告其当时的概率预测排名。[实验配置与结果](https://timecopilot.dev/experiments/gift-eval/)

**应如何解释**：这支持底层集成方案在指定基准上的表现。该实验不等价于“LLM 根据数据选择模型，比固定选择更好”，也不是完整企业运行成本。不能将其成本和排名直接移植到 NeuralForecast 训练或销售场景。

本项目最值得验证的收益是：是否减少人工调用、整理回测和撰写说明的时间。在尚未做相同预算对照前，不应给出 Agent 能提升多少预测精度的承诺。

## 出处与阅读顺序

1. [官方概览](https://timecopilot.dev/)：输入、输出与使用方式。
2. [神经模型 API](https://timecopilot.dev/api/models/neural/)：AutoNHITS 接口与约束。
3. [Agent API](https://timecopilot.dev/api/agent/)：上层调用接口。
4. [GIFT-Eval](https://timecopilot.dev/experiments/gift-eval/)：效果所对应的真实实验对象。
5. [仓库](https://github.com/TimeCopilot/timecopilot)；[论文](https://arxiv.org/abs/2509.00616)：进一步追踪入口。

核验范围：文档和其中展示的接口／源代码；未运行项目，未验证现有 baseline 可直接接入。
