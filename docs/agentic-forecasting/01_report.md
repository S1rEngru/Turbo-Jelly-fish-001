# Agent 辅助时序预测建模／Agentic AutoML 调研报告

研究日期：2026-09-16  
研究场景：已有基于 NeuralForecast 的销售预测 baseline，希望降低迁移到不同品牌时的数据处理、调用模型、训练、调参和分析成本。  
范围：公开可检索的一手企业资料、官方代码、论文与基准；不假设已经拿到或读懂用户的 baseline。当前项目目录未发现训练脚本，本次仅进行资料调研。

## 1. 结论

**这个方向已经有具体系统、开源实现和研究实验，不只是概念。与当前任务最贴近的路线，是在现有预测模型上增加一个能调用工具、管理实验、读取验证结果并调整配置的 Agent。**

但“有例子”需要拆开回答：

| 需要证明的事情 | 本次结论 | 最直接的资料 |
| --- | --- | --- |
| 有 Agent 自动参与时序建模、选模、调参吗？ | 有，已有文档和研究实现 | Finn、TimeCopilot、TimeSeriesScientist、金融训练 TS-Agent [S03](02_sources.md#s03)[S14–S20](02_sources.md#s14) |
| 有直接连接 NeuralForecast 的实现吗？ | 有；TimeCopilot 论文明确接入 AutoNHITS、AutoTFT | [TimeCopilot论文§2.2](https://arxiv.org/html/2509.00616) |
| 有企业参与研发和商业使用吗？ | 有；但公开的生产部署细节不均衡 | Salesforce MoiraiAgent 官方声明商业用途；微软、AWS有开源研发项目 [S03–S13](02_sources.md#s03) |
| 已证明 Agent 能在多个销售品牌上稳定降本增效吗？ | 本次没有找到同时公开品牌迁移、训练闭环、同预算对照及长期收益的一手完整证据 | 属于当前证据缺口，不代表不存在内部项目 |
| 必须先训练 Agent 或使用强化学习吗？ | 不需要；工具调用与实验反馈足以构成第一版 | Finn、TimeCopilot、AIDE 等路线；RL只作为延伸 [S03](02_sources.md#s03)[S15](02_sources.md#s15)[S25](02_sources.md#s25)[S40–S41](02_sources.md#s40) |

**推荐阅读优先级（研究者判断）：**

1. **TimeCopilot**：先确认怎样把预测模型封装为 Agent 可调用接口，与 NeuralForecast 的距离最近。
2. **Microsoft Finn**：看迭代训练、复用历史配置、更新数据、退化后重新优化的工程闭环。
3. **金融建模 TS-Agent**：看怎样围绕现成的 `train.py` 组织有边界的代码与配置修改。
4. **TimeSeriesScientist**：看数据诊断、选模、验证、集成和报告的完整流程。
5. **DCATS**：看能否通过选择合适的训练序列，帮助跨品牌迁移。
6. **TimeSeriesGym**：看如何证明 Agent 确实提升了建模能力。
7. **AutoGluon Assistant、RD-Agent、AIDE、ERA**：补充通用实验自动化和代码搜索方法。

## 2. 分类标准：别把三种“预测 Agent”混成一种

| 类型 | Agent 实际做什么 | 是否正中本项目需求 |
| --- | --- | --- |
| 预测服务编排 | 取数、调用已经训练好的模型、解释结果 | 部分相关；未必降低训练调参成本 |
| 建模／实验优化 | 诊断数据、选特征与模型、运行训练、比较验证结果、调整下一轮实验 | **核心范围** |
| 预测结果修正 | 按促销、节日、事件、专家意见修正数值预测 | 可作为后续模块，需单独验证 |
| 传统时序 AutoML | 在预设模型和参数空间中自动搜索 | 必须比较的非Agent基线，也可作为Agent执行工具 |
| 自主算法研发 | 改网络结构、损失、训练策略甚至发明新实现 | 相关，但工程与评估成本更高 |
| Agent／预测器的训练 | 对选择器或LLM做SFT、RL等训练 | 进阶路线，不是“用了Agent”的必要条件 |

本报告的“企业实际使用”以第一方明确描述业务用途为依据；“企业开源研发”不自动升级为“企业生产部署”；“作者报告有效”不自动升级为“已独立复现”。

## 3. 企业与产业项目：优先核实实际用途

### 3.1 Microsoft Finn / finnts：最接近训练辅助的企业开源项目

**已核实事实。** 官方文档把 Agent 定义为核心 finnts 预测管线之上的工具调用编排层。它能够分析数据、选择特征和模型、安排回测与集成；`iterate_forecast()` 在预算内迭代，`update_forecast()` 支持新数据更新及必要时重新调用 Agent；产物包括运行标识、配置、指标和预测表。[官方Agent文档](https://microsoft.github.io/finnts/articles/ai-agent.html)（S03）

**落地边界。** 微软另有内部财务预测案例，报告统一框架带来每年240万美元成本节约、预测时间减少50%；该资料并没有把这些收益归因于新增的LLM Agent，不能拼接成“Agent节约50%时间”的案例。[官方案例](https://cdn-dynmedia-1.microsoft.com/is/content/microsoftcorp/microsoft/mcaps/documents/fy26/Financial-forecasting.pdf)（S05）

**与你的关联。** 最值得借鉴的是“已有管线保持为执行核心，Agent调度实验”的结构。项目是R生态；不建议仅为Agent迁移掉Python baseline。可复用的是工作流设计，实际接口需重新适配。[仓库](https://github.com/microsoft/finnts)（S04）

### 3.2 Salesforce MoiraiAgent：有商业用途声明，但主要是选专家与上下文预测

**已核实事实。** Salesforce发布了MoiraiAgent：包含预测专家选择以及调用预处理、预测、后处理工具的流程。官方明确写明其专有版本用于业务，公开版本用于研究；未披露足以独立验证的客户部署规模和业务收益。[官方文章](https://www.salesforce.com/blog/moiraiagent/)（S12）

**与你的关联。** 可以学习“根据序列特征与回测证据选择预测器”，以及把业务上下文转化为工具操作。它不等于自主修改NeuralForecast训练脚本。已有选择器训练也不意味着采用RL；不能从“训练Agent”直接推断训练算法。

代码入口：[SalesforceAIResearch/uni2ts — moirai-agent](https://github.com/SalesforceAIResearch/uni2ts/tree/main/project/moirai-agent)（S13）。未在本地运行，商用复用应另行核查仓库具体许可。

### 3.3 AWS AutoGluon Assistant / MLZero：通用建模自动化的强参考

**已核实事实。** Amazon Science将其定位为自然语言驱动的端到端AutoML，涵盖时序等数据类型，模块包括输入感知、工具知识、历史实验记忆和迭代代码生成。官方报告的92%与86%分别是不同工程基准上的成功率，不能理解为销量预测准确率。[官方介绍](https://www.amazon.science/blog/autogluon-assistant-zero-code-automl-through-multiagent-collaboration)（S06）

**成熟度。** 有公开代码和MLZero论文；但仓库明确说明是研究代码、未达到生产就绪，并标注当前支持Linux。[仓库](https://github.com/autogluon/autogluon-assistant)（S07）、[论文](https://arxiv.org/abs/2505.13941)（S08）

**研究者判断。** 适合作为“数据不整齐时如何自动接入、出错后如何修复”的参考；当前Windows环境下不能假设开箱运行。若只包现有脚本，不一定需要它的全套代码生成能力。

### 3.4 Microsoft RD-Agent：实验研发闭环，不能直接当销售预测成品

项目覆盖研究假设、代码开发、执行与反馈，并提供量化和通用数据科学场景。其优势是把一次训练放入可重复的研究过程；本次没有找到它在具名企业内部用于多品牌销售预测的完整部署证明。[官方介绍](https://www.microsoft.com/en-us/research/articles/rd-agent/)（S09）、[仓库](https://github.com/microsoft/RD-Agent)（S10）

**研究者判断。** 当baseline需要的不只是调参数，而是设计新特征、替换模块、分析失败实验时，RD-Agent更有借鉴价值。量化交易收益不能作为零售销量效果的替代指标。

### 3.5 Google Cloud + App Orchid：企业预测编排，训练证据有限

官方描述数据Agent、预测Agent及编排Agent配合：前者理解企业数据与语义，后者调用TimesFM等模型。这是企业预测系统的具体架构示例，但文章未给出完整自主训练调参过程、客户对照试验或可核验的ROI。[官方方案](https://cloud.google.com/blog/products/ai-machine-learning/how-we-built-a-multi-agent-system-for-superior-business-forecasting)（S11）

**研究者判断。** 跨品牌最费力的部分如果是字段口径、渠道映射和取数，这种数据语义层值得参考；若瓶颈是深度模型训练，不能把它当成已解决该问题的证据。

### 3.6 其他商业平台：可接工具，但证据不足以认定自主训练

| 项目 | 可以确认 | 不能据此确认 | 来源 |
| --- | --- | --- | --- |
| DataRobot | Agent平台可接入时序预测等工具 | 客户使用Agent自主选模和重训的完整闭环及收益 | [产品页](https://www.datarobot.com/product/agentic-ai/) S48 |
| 腾讯云 DataBuddy | 官方文档列出时序AutoML、参数搜索和实验Notebook产物 | 产品名称含“智能体”就意味着LLM决定训练策略 | [创建实验](https://cloud.tencent.com/document/product/1835/137607) S49 |
| IBM AutoAI-TS | 自动生成与筛选时序模型管线，有正式产品说明 | 它属于LLM Agent系统 | [研究文章](https://research.ibm.com/blog/autoai-time-series) S44；产品文档S45 |

### 3.7 企业传统预测案例：用于判断“到底哪些成本需要Agent”

- **LinkedIn Greykite**：内部用于业务指标与容量规划，借助模板和搜索减少逐序列手调。说明重复配置问题可以先用标准化和AutoML解决；不是LLM Agent案例。[企业文章](https://www.linkedin.com/blog/engineering/open-source/greykite-a-flexible-intuitive-and-fast-forecasting-library)（S42）
- **Uber Orbit**：营销数据科学团队用于测量、规划和预测。可参考统一建模接口，但不能计为Agent训练案例。[企业文章](https://www.uber.com/us/en/blog/orbit/)（S43）
- **Salesforce Merlion**：提供时序框架和AutoML能力。可作为执行工具设计参考，企业开源身份本身不能证明Agent落地。[仓库](https://github.com/salesforce/Merlion)（S46）
- **Decathlon + Chronos-2**：AWS与迪卡侬联合文章描述多地区需求预测，以及旧体系周重训和扩展新区域的工程负担。它提供了另一种降低维护成本的路线：使用时序基础模型；并非Agent案例。[联合案例](https://aws.amazon.com.cdn.amazon.com/blogs/machine-learning/how-decathlon-runs-demand-forecasting-at-scale-with-chronos-2/)（S47）

**研究者判断。** 这些案例意味着应同时比较三种办法：配置标准化、传统AutoML、Agent。不能只拿“纯人工”当对照，否则容易把本来由自动化带来的收益都归给LLM。

### 3.8 PepsiCo线索：保留，暂不当成已证实训练案例

检索发现二手材料提到PEP Planner、实验日志和建模建议（S55）。已找到的企业公告只确认AI合作方向（S53）；相关Databricks演讲页的活动页头显示2027，与本次截止日不一致（S54）。因此不采信其中具体规模、成本下降或自主调参细节作为当前结论。相关出处全部保存在来源目录，便于后续追踪。

## 4. 最贴近现有baseline的开源与论文

### 4.1 TimeCopilot：与NeuralForecast距离最近

- **机制**：由LLM协调序列特征分析、候选模型比较、交叉验证、预测和解释。
- **直接连接**：论文明确包含NeuralForecast的AutoNHITS与AutoTFT；发行记录还列出新增自动神经模型。
- **关键证据边界**：论文的GIFT-Eval强结果使用基础模型集成；不能将集成效果当成“LLM选择比固定策略更好”的消融证明。
- **可借鉴部分（判断）**：预测器统一接口、模型比较产物、LLM与数值模型分工。自定义baseline是否能无改动接入，仍须查看具体训练接口。

来源：[仓库](https://github.com/TimeCopilot/timecopilot)（S14）、[论文](https://arxiv.org/html/2509.00616)（S15）、[发行记录](https://github.com/TimeCopilot/timecopilot/releases)（S16）。本次没有执行示例或确认所有模型在Windows上的兼容性。

### 4.2 金融训练TS-Agent：最像“围绕已有脚本做实验”

论文：*Structured Agentic Workflows for Financial Time-Series Modeling with LLMs and Reflective Feedback*，2025。

- **机制**：围绕模块化 `train.py` 选择模型、修改训练方法、调整超参数，执行并保存实验记录；利用既有资源和历史结果迭代。
- **范围**：金融时序预测和生成；是具体研究实验，不是具名企业生产案例。
- **可借鉴部分（判断）**：把你的baseline拆成可配置模块，保留每次变更与结果之间的对应关系。
- **限制**：本次没有确认作者公开的完整可运行仓库；不要与另一个同名TS-Agent混淆。形式化动作或策略符号并不证明用了RL。

来源：[论文正文§3–5](https://arxiv.org/html/2508.13915v1)（S19）。

### 4.3 TimeSeriesScientist：覆盖数据准备到报告

论文定义Curator、Planner、Forecaster、Reporter四类角色，覆盖诊断与预处理、缩小候选空间、拟合验证与集成、生成报告；作者报告八个基准上的改进。[论文](https://arxiv.org/abs/2510.01538)（S17）

代码README则按Preprocess、Analysis、Validation、Forecast、Report五阶段描述实现，需注意论文与当前工程组织的差别。[官方仓库](https://github.com/Y-Research-SBU/TimeSeriesScientist)（S18）

**研究者判断。** 适合参考新品牌接入的完整任务分解。多角色不必对应多个独立LLM进程；首版可先以单控制器调用同一组工具实现。作者结果没有证明对本项目数据有效，也没有证明收益超过同预算传统AutoML。

### 4.4 DCATS：让Agent挑训练数据，而不只挑模型

论文：*Empowering Time Series Forecasting with LLM-Agents*，2025。

系统根据时序元数据与历史实验选择有帮助的相邻序列，再用预测模块验证和改进提案。研究使用交通数据，评估60个查询和四种预测模型；约6%是作者在该设置下报告的汇总改善。[论文方法与实验](https://arxiv.org/html/2508.04231v1)（S20）

**研究者判断。** 这是跨品牌场景的有价值假设：与其把所有品牌混在一起训练，可以选择相似商品／渠道／季节性的数据。但交通邻居不是销售品牌，需验证负迁移、品牌权限及仅使用当时可见信息。

论文给出的项目入口已记录（S21），本次工具无法打开，代码状态未核实。

### 4.5 GenAutoML：让Agent生成网络结构

2026年预印本将自然语言需求转为PyTorch架构，并加入沙箱内代码反思与接口一致性检查，实验覆盖ETTh1、ETTm1和Weather；页面标记Under review。[论文](https://arxiv.org/abs/2606.05860)（S22）

**研究者判断。** 若任务以后升级为结构创新，可深入阅读；当前降低已有baseline调用成本不必先开放任意网络生成。需要额外证明结构搜索比配置搜索值得其算力和调试成本。

### 4.6 通用Agentic AutoML：能借方法，不宜直接搬结论

| 项目 | 主要方法 | 对本项目的借鉴 | 证据限制 |
| --- | --- | --- | --- |
| AutoML-Agent，ICML 2025 | 按角色组织全流程AutoML | 分离需求、数据、模型与执行职责 | 通用任务实验不能直接推出品牌预测效果；S23–S24 |
| AIDE，2025 | 在代码空间做树搜索，复用和改进候选方案 | 保存多个实验分支，避免只沿最后一次修改前进 | 通用ML工程基准；S25–S26 |
| RD-Agent | 假设—开发—反馈循环 | 保存实验依据、失败经验和下一步计划 | 量化与销售场景不同；S09–S10 |
| ERA，Nature 2026 | 通过代码搜索开发经验算法 | 研究级方法设计与公平实验 | 有COVID与GIFT-Eval实验，但不是企业销售上线证明；S27 |
| NeoResearch，2026项目页 | 文献、数据、实验联合形成研究闭环 | 值得跟踪中文自主时序研究方向 | 页面明确后续补代码、论文及benchmark；当前仅作方案线索；S50 |

ERA论文比较的是特定历史时间点榜单，不能表述为“截至今天仍然第一”。该研究在GIFT-Eval还区分逐数据集方案与统一预测方案；跨品牌推广时应特别区分“每个品牌单独优化”与“真正迁移”。[正式论文](https://www.nature.com/articles/s41586-026-10658-6)（S27）

## 5. 邻接研究：保留视野，但不作为核心训练Agent证据

| 研究 | Agent主要作用 | 为什么值得保存／为什么不能混用 | 来源 |
| --- | --- | --- | --- |
| FLAIRR-TS，EMNLP Findings 2025 | 检索历史片段、反复修正LLM预测 | 不更新权重；不是底层模型训练管理 | S34 |
| TimeXL，2025 | 多模态预测、文本反思与改进，触发编码器重训 | 接近闭环训练，但任务与架构专用；AUC改善不是销量误差改善 | S35 |
| 推理TS-Agent，2025/2026 | 对原始时序调用分析工具并核验答案 | 与金融训练TS-Agent同名不同工作 | S36 |
| TS-Reasoner，2024–2026 | 领域计算工具与多步推理 | 可借诊断工具设计；不代表训练自动化 | S37 |
| Last-mile forecasting，2026 | 结合节日、活动和专家反馈修正预测 | 适合促销信息进入预测；需与训练优化分开评估 | S38 |
| Nexus，2026 | 综合宏观、微观变化与上下文预测 | 侧重预测推理的组织 | S39 |
| CastFlow，2026 | 对专用预测LLM采用SFT与RLVR | 真正涉及训练的延伸；训练对象与NeuralForecast控制器不同 | S40 |
| LangTime，ICML 2025 | 语言引导预测与PPO微调 | 可说明RL与预测结合，但不是本项目必需组件 | S41 |
| 能源领域本地多Agent预测系统，2026 | 特征工具、神经模型选择 | 有出版商检索摘要；正文未取得，完整实验与可用代码未核实 | S52 |

综述入口：[LLM Agents for Time-Series: A Survey](https://arxiv.org/abs/2608.26226)（S32）。概念框架入口：[Agentic Time Series Forecasting立场论文](https://arxiv.org/abs/2602.01776)（S33）。综述和立场论文用于建立分类，不能代替落地证据。

## 6. 对照矩阵：应该优先看哪几个

“高／中”表示对当前任务的研究者适配判断，不是论文排名。代码“有”表示发现官方仓库，未表示本地已跑通。

| 项目 | 数据／特征决策 | 选模／调参 | 训练执行或代码迭代 | 结果解释 | 官方代码 | 当前适配判断 |
| --- | --- | --- | --- | --- | --- | --- |
| Finn | 有 | 有 | 管线调用 | 有 | 有，R | 高：工程闭环 |
| TimeCopilot | 特征分析 | 有 | 调用预测／Auto模型 | 有 | 有，Python | 高：NeuralForecast接口 |
| 金融训练TS-Agent | 利用案例和知识 | 有 | 明确编辑训练脚本 | 有日志 | 本次未确认 | 高：baseline改造思路 |
| TimeSeriesScientist | 有 | 有 | 模型拟合与验证 | 有 | 有，Python | 高：全流程分解 |
| DCATS | 重点是训练序列选择 | 重点不在调参 | 调用预测模块 | 有提案理由 | 入口未能打开 | 高：跨品牌数据选择假设 |
| AutoGluon Assistant | 有 | 经工具／代码实现 | 有 | 有工作流记录 | 有，Python | 中高：通用自动化 |
| RD-Agent | 可迭代开发 | 可迭代开发 | 有 | 有 | 有，Python | 中高：实验研发 |
| AIDE | 通过代码实现 | 通过代码实现 | 有 | 实验记录 | 有，Python | 中：实验树 |
| MoiraiAgent | 预处理与上下文 | 重点为专家选择 | 未证明通用训练脚本优化 | 有 | 研究版有 | 中：模型路由与业务上下文 |
| Google / App Orchid | 企业取数与语义 | 未证明调参 | 主要模型调用 | 有 | 本次未确认完整方案代码 | 中：数据接入 |
| ERA / GenAutoML | 研究范围内支持 | 有搜索 | 代码／架构生成 | 研究输出 | 本次未核实完整复现链 | 中：长期研究方向 |
| NeuralForecast Auto模型 | 依赖外部准备 | 有 | 有 | 数值结果为主 | 有 | 必须有的无Agent对照 |

## 7. 如何判断Agent真的有效

### 7.1 已有评估资源

**TimeSeriesGym**最贴近本任务：包含33个挑战，涵盖模型选择、参数优化、数据处理与研究代码迁移等工程能力；有公开仓库。README中仍存在占位克隆地址，说明复现前须检查工程说明，不宜直接复制所有命令。[论文](https://arxiv.org/abs/2505.13291)（S28）、[代码](https://github.com/moment-timeseries-foundation-model/timeseriesgym)（S29）

**TemporalBench**评估历史理解、无上下文预测、上下文推理和事件条件预测，并覆盖零售等领域。适合补充测试“Agent是否真的理解促销与事件”，而不仅是产出一个预测文件。[论文](https://arxiv.org/abs/2602.13272)（S30）

**Context is Key**可作为文本上下文预测评估参考（S31）。sktime也有Agent选模与传统AutoML比较的公开提案，但本次读取的是issue，不能称作已完成的基准（S51）。

### 7.2 本项目建议的实验设计（研究者建议，尚未实施）

固定数据快照、时间切分、候选预测器和总预算，对比：

| 对照组 | 用途 |
| --- | --- |
| 原始人工baseline | 测量当前效果与实际人工耗时 |
| 固定模板自动运行 | 判断简单工程自动化已能解决多少问题 |
| NeuralForecast Auto + 固定规则 | 衡量传统HPO，不使用LLM |
| Agent + 相同候选模型和HPO | 测量Agent的额外价值 |
| 视资源加入基础模型／固定集成 | 避免把模型本身更强错归给Agent |

应报告：品牌宏平均误差、全局销量加权误差、系统性高估／低估、训练失败率、人工干预分钟数、运行时长、GPU/CPU用量和LLM费用。WAPE在总销量为零时无定义，必须预先规定处理方式；不能只看大品牌主导的总平均。

评价方法应预先冻结；调参只看训练／验证数据，最终测试集用于冻结方案后的评价。按滚动时间窗口回测，并保留未参与策略设计的品牌做跨品牌检验。NeuralForecast已有时间交叉验证接口可作为底座，但具体参数需与安装版本核对。[官方文档](https://nixtlaverse.nixtla.io/neuralforecast/docs/capabilities/cross_validation.html)（S02）

另外记录“某次实验为什么失败”：资源不足、字段错误、特征不可用、算法不适配或实际性能退化。Agent是否减少这些工作，比自然语言报告是否流畅更重要。

## 8. 面向NeuralForecast的可行起点

以下是基于资料的工程推断，不是已经完成的系统设计。

```text
品牌数据与任务说明
        ↓
固定数据检查与时间切分
        ↓
Agent读取数据画像、可用模型和历史实验
        ↓
生成受约束的实验配置
        ↓
已有baseline / NeuralForecast Auto执行训练与回测
        ↓
固定评估器输出指标、资源消耗、失败原因
        ↓
Agent决定下一轮实验或停止
        ↓
保存最佳配置、模型、预测、数据版本与实验报告
```

建议先把已有脚本封装成以下能力，而不是立刻重写模型：

| 工具能力 | 输入 | 输出 |
| --- | --- | --- |
| 数据诊断 | 品牌数据、预测目标、频率 | 缺失／重复／历史长度／季节性等统计 |
| 检查配置 | 模型、窗口、特征、预算 | 合法配置或清晰错误 |
| 运行训练 | 数据快照、配置、随机种子 | 模型与训练日志 |
| 运行回测 | 冻结切分、训练配置 | 各窗口、品牌、SKU指标 |
| 比较实验 | 合法实验记录 | 改善幅度、成本、失败分类 |
| 保存结果 | 最优合法实验 | 模型、配置、版本、可复现报告 |

NeuralForecast本来就有Auto模型及Ray/Optuna调参支持；Agent较可能带来的额外价值，是配置搜索空间、选择数据与特征、识别失败、复用相似品牌经验，而不是逐个随意猜学习率。[官方HPO文档](https://nixtlaverse.nixtla.io/neuralforecast/docs/capabilities/hyperparameter_tuning.html)（S01）

跨品牌至少要区分三件事：同一训练流程复用、同一组模型权重迁移、同一套实验经验复用。三者成本和验证方式不同。新品牌如果完全没有销量历史，增加Agent不能自动解决冷启动；仍需商品元数据、相似品牌或其他可用信息。

还需特别记录促销和价格在预测时点是否已知、缺货是否压低观测销量、商品上下架以及退货口径。Agent应该拿到这些明确规则，而不是自行猜测业务含义。

## 9. 当前研究缺口与后续阅读顺序

### 已有证据支持

- 工具调用Agent可以组织预测训练和实验流程，且存在贴近NeuralForecast的开源项目。
- 可以从“模型选择”“训练数据选择”“脚本优化”“结果修正”等不同位置减少人工操作。
- 评估Agent需要同时考核工程成功率、预测效果与总成本，已有专门时序工程基准可参考。

### 尚未被本次调研证明

- 在你的品牌数据上，Agent是否优于固定模板加Optuna。
- 跨品牌记忆能否减少试验次数，同时避免负迁移。
- 多Agent分工是否比单Agent更划算。
- 在相同计算预算下，自动修改代码是否值得其调试成本。
- 长期自动重训是否稳定，以及验证集反复使用产生的选择偏差有多大。

### 后续精读与验证清单

1. 先读你的baseline，识别人工真正反复修改的部分。
2. 看TimeCopilot的模型封装方式，判断能否复用或只借接口设计。
3. 看Finn的迭代和更新流程，列出本项目需要保留的运行产物。
4. 看金融TS-Agent如何限制训练脚本改动范围；把评估器与可修改训练逻辑分开。
5. 用DCATS提出“选择相似品牌训练数据”的独立实验，而不是直接假设有效。
6. 使用相同预算比较无Agent和有Agent系统，再决定是否扩展到架构搜索或Agent训练。

## 10. 来源与方法说明

完整出处、查阅程度、版本及限制见[来源目录](02_sources.md)。检索范围和暂缓采信资料见[检索记录](03_search_log.md)。

本次交付是调研整理：没有安装这些项目、训练模型、调用付费LLM服务或测量实际收益；没有把所有论文通读或逐行审计代码。公开基准数字和企业收益均按原作者归属说明，不当成独立验证结论。

