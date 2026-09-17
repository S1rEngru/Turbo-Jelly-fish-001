# 来源目录与证据边界

检索截止与统一访问日期：2026-09-16（Asia/Shanghai）。这里保存的是书目信息、出处及简短研究笔记，不是原文全文存档。动态网页、main分支与当前论文版本可能变化；尚未锁定全部代码commit，未执行任何项目。检索正文表示搜索工具返回了原网站正文片段，未等同于通读全文。

来源的可靠性与落地成熟度是两回事：官方仓库可证明项目存在，但不能证明生产效果；论文结果均是作者报告，尚未独立复现。S55仅为二手线索，不作为技术依据。

## S01

- 标题：[NeuralForecast Hyperparameter Optimization](https://nixtlaverse.nixtla.io/neuralforecast/docs/capabilities/hyperparameter_tuning.html)
- 作者／机构：Nixtla
- 发布／版本：持续更新
- 查阅程度：官方文档；检索正文
- 支持的结论与限制：Auto 模型、Ray/Optuna；传统 HPO 底座，不是 LLM Agent。

## S02

- 标题：[NeuralForecast Cross Validation](https://nixtlaverse.nixtla.io/neuralforecast/docs/capabilities/cross_validation.html)
- 作者／机构：Nixtla
- 发布／版本：持续更新
- 查阅程度：官方文档；已打开
- 支持的结论与限制：时间回测接口；落地时仍须核对安装版本和切分设置。

## S03

- 标题：[AI Agent Capabilities](https://microsoft.github.io/finnts/articles/ai-agent.html)
- 作者／机构：Microsoft Finn
- 发布／版本：持续更新
- 查阅程度：官方文档；已打开
- 支持的结论与限制：直接的训练编排 Agent；iterate_forecast/update_forecast/ask_agent。

## S04

- 标题：[Microsoft Finance Time Series Forecasting Framework](https://github.com/microsoft/finnts)
- 作者／机构：Microsoft
- 发布／版本：持续更新
- 查阅程度：官方仓库；README级核实
- 支持的结论与限制：开源 R 项目；尚未在本地执行。

## S05

- 标题：[Financial forecasting — Innovation Story](https://cdn-dynmedia-1.microsoft.com/is/content/microsoftcorp/microsoft/mcaps/documents/fy26/Financial-forecasting.pdf)
- 作者／机构：Microsoft Frontier Finance
- 发布／版本：页面未标明确发布日期
- 查阅程度：官方案例；PDF文本已读取
- 支持的结论与限制：传统统一预测框架内部收益；没有把收益归因于 LLM Agent。

## S06

- 标题：[AutoGluon assistant: Zero-code AutoML through multiagent collaboration](https://www.amazon.science/blog/autogluon-assistant-zero-code-automl-through-multiagent-collaboration)
- 作者／机构：Amazon Science
- 发布／版本：2025-12-05
- 查阅程度：官方技术文章；检索正文
- 支持的结论与限制：通用 Agentic AutoML，说明覆盖时序；基准成功率不是业务准确率。

## S07

- 标题：[AutoGluon Assistant / MLZero](https://github.com/autogluon/autogluon-assistant)
- 作者／机构：AutoGluon
- 发布／版本：持续更新
- 查阅程度：官方仓库；已打开
- 支持的结论与限制：README 明确 research code / not production-ready；当前说明 Linux-only。

## S08

- 标题：[MLZero: A Multi-Agent System for End-to-end Machine Learning Automation](https://arxiv.org/abs/2505.13941)
- 作者／机构：Haoyang Fang 等
- 发布／版本：2025-05-20
- 查阅程度：论文摘要；已打开
- 支持的结论与限制：感知、语义与实验记忆、代码迭代；NeurIPS 2025 录用信息另见官方仓库。

## S09

- 标题：[开源工具 RD-Agent：让研究与开发过程更智能](https://www.microsoft.com/en-us/research/articles/rd-agent/)
- 作者／机构：Microsoft Research
- 发布／版本：2024；具体日期以页面为准
- 查阅程度：官方技术文章；已打开
- 支持的结论与限制：研究—开发—反馈闭环。

## S10

- 标题：[RD-Agent](https://github.com/microsoft/RD-Agent)
- 作者／机构：Microsoft
- 发布／版本：持续更新
- 查阅程度：官方仓库；已打开
- 支持的结论与限制：量化与通用数据科学场景；不能推出已用于销售预测生产。

## S11

- 标题：[How we built a multi-agent system for superior business forecasting](https://cloud.google.com/blog/products/ai-machine-learning/how-we-built-a-multi-agent-system-for-superior-business-forecasting)
- 作者／机构：Google Cloud / App Orchid
- 发布／版本：2025-12-11
- 查阅程度：官方方案文章；检索正文
- 支持的结论与限制：取数与基础模型预测编排；未公开完整训练调参闭环。

## S12

- 标题：[MoiraiAgent: An Agentic Framework for Context-Aware Time-Series Forecasting](https://www.salesforce.com/blog/moiraiagent/)
- 作者／机构：Salesforce AI Research
- 发布／版本：2026-01-14
- 查阅程度：官方技术文章；已打开
- 支持的结论与限制：专家选择及上下文工具编排；官方声明 proprietary version 用于 business purposes，缺少部署规模和收益细节。

## S13

- 标题：[Moirai-Agent research implementation](https://github.com/SalesforceAIResearch/uni2ts/tree/main/project/moirai-agent)
- 作者／机构：SalesforceAIResearch
- 发布／版本：持续更新
- 查阅程度：官方代码目录；已打开
- 支持的结论与限制：由官方文章链接到达；研究版本，复用前核对许可和依赖。

## S14

- 标题：[TimeCopilot](https://github.com/TimeCopilot/timecopilot)
- 作者／机构：TimeCopilot
- 发布／版本：持续更新
- 查阅程度：官方仓库；已打开
- 支持的结论与限制：Python 开源预测 Agent；本次未运行。

## S15

- 标题：[TimeCopilot](https://arxiv.org/html/2509.00616)
- 作者／机构：TimeCopilot 作者团队
- 发布／版本：2025；读取当前HTML版本
- 查阅程度：论文正文；重点读取§2–3
- 支持的结论与限制：明确接入 AutoNHITS/AutoTFT；GIFT-Eval结果依赖基础模型集成，不能等同于 LLM 决策增益。

## S16

- 标题：[TimeCopilot releases](https://github.com/TimeCopilot/timecopilot/releases)
- 作者／机构：TimeCopilot
- 发布／版本：持续更新；检索到v0.0.26
- 查阅程度：官方发行记录；检索正文
- 支持的结论与限制：发行说明增加 AutoNBEATS/AutoDeepAR/AutoPatchTST；不宣称该版本为最新稳定版。

## S17

- 标题：[TimeSeriesScientist: A General-Purpose AI Agent for Time Series Analysis](https://arxiv.org/abs/2510.01538)
- 作者／机构：Haokun Zhao 等
- 发布／版本：2025-10-02；v2 2025-10-06
- 查阅程度：论文摘要；已打开
- 支持的结论与限制：完整预测工作流；八个基准上的作者自报结果。

## S18

- 标题：[TimeSeriesScientist official repository](https://github.com/Y-Research-SBU/TimeSeriesScientist)
- 作者／机构：Y-Research-SBU
- 发布／版本：2025起；持续更新
- 查阅程度：官方仓库；已打开
- 支持的结论与限制：README五阶段实现与论文四角色表述并不完全相同；未进行代码审计或运行。

## S19

- 标题：[Structured Agentic Workflows for Financial Time-Series Modeling with LLMs and Reflective Feedback](https://arxiv.org/html/2508.13915v1)
- 作者／机构：Yihao Ang 等
- 发布／版本：2025-08；读取v1
- 查阅程度：论文正文；重点读取§3–5
- 支持的结论与限制：金融建模 TS-Agent；模块化train.py、模型选择、代码改进、调参、实验日志。

## S20

- 标题：[Empowering Time Series Forecasting with LLM-Agents (DCATS)](https://arxiv.org/html/2508.04231v1)
- 作者／机构：Chin-Chia Michael Yeh 等
- 发布／版本：2025-08-06；v1
- 查阅程度：论文正文；重点读取方法和实验
- 支持的结论与限制：选择相关序列/训练数据；交通实验60 queries、四种模型；约6%为作者汇总。

## S21

- 标题：[DCATS project/code entry](https://sites.google.com/view/ts-agent)
- 作者／机构：DCATS 作者团队
- 发布／版本：未核实
- 查阅程度：论文给出的链接；工具无法打开
- 支持的结论与限制：只保存入口；不据此声称代码可用、完整或可运行。

## S22

- 标题：[GenAutoML: An Agentic Framework for Dynamic Architecture Generation and Optimization in Time-Series Analysis](https://arxiv.org/abs/2606.05860)
- 作者／机构：Oleeviya Babu Poikarayil 等
- 发布／版本：2026-06-04；v2 2026-06-11
- 查阅程度：论文摘要；已打开；标记Under review
- 支持的结论与限制：生成PyTorch结构及沙箱修正；ETTh1/ETTm1/Weather；非生产证明。

## S23

- 标题：[AutoML-Agent: A Multi-Agent LLM Framework for Full-Pipeline AutoML](https://proceedings.mlr.press/v267/trirat25a.html)
- 作者／机构：Patara Trirat 等
- 发布／版本：ICML 2025
- 查阅程度：正式会议论文页；已打开
- 支持的结论与限制：通用全流程AutoML；销售时序效果需要另测。

## S24

- 标题：[AutoML-Agent official implementation](https://github.com/DeepAuto-AI/automl-agent)
- 作者／机构：DeepAuto-AI
- 发布／版本：持续更新
- 查阅程度：官方仓库；检索正文
- 支持的结论与限制：专门角色分工及计划；没有本地复现。

## S25

- 标题：[AIDE: AI-Driven Exploration in the Space of Code](https://arxiv.org/abs/2502.13138)
- 作者／机构：Zhengyao Jiang 等
- 发布／版本：2025-02-18
- 查阅程度：论文摘要；检索正文
- 支持的结论与限制：将机器学习工程视为代码空间树搜索。

## S26

- 标题：[AIDE ML](https://github.com/WecoAI/aideml)
- 作者／机构：WecoAI
- 发布／版本：持续更新
- 查阅程度：官方仓库；已打开
- 支持的结论与限制：可借鉴实验树和代码迭代；通用工具，不是销售时序专用系统。

## S27

- 标题：[An AI system to help scientists write expert-level empirical software (ERA)](https://www.nature.com/articles/s41586-026-10658-6)
- 作者／机构：Google 等研究团队
- 发布／版本：2026
- 查阅程度：正式期刊论文；读取时序实验章节
- 支持的结论与限制：COVID预测与GIFT-Eval算法搜索；论文榜单比较有历史截止日。

## S28

- 标题：[TimeSeriesGym: A Scalable Benchmark for (Time Series) Machine Learning Engineering Agents](https://arxiv.org/abs/2505.13291)
- 作者／机构：Yifu Cai 等
- 发布／版本：2025
- 查阅程度：论文摘要；已打开
- 支持的结论与限制：衡量时序工程Agent，不只是预测分数。

## S29

- 标题：[TimeSeriesGym official code](https://github.com/moment-timeseries-foundation-model/timeseriesgym)
- 作者／机构：moment-timeseries-foundation-model
- 发布／版本：持续更新
- 查阅程度：官方仓库；检索正文
- 支持的结论与限制：33挑战、23来源、8类问题；含调参/代码迁移；README仍有占位克隆URL。

## S30

- 标题：[TemporalBench: A Benchmark for Evaluating LLM-Based Agents on Contextual and Event-Informed Time Series Tasks](https://arxiv.org/abs/2602.13272)
- 作者／机构：Muyan Weng 等
- 发布／版本：2026-02-05
- 查阅程度：论文摘要；已打开
- 支持的结论与限制：零售等四领域，考核上下文推理与事件条件预测。

## S31

- 标题：[Context is Key: A Benchmark for Forecasting with Essential Textual Information](https://arxiv.org/abs/2410.18959)
- 作者／机构：Alex R. Williams 等
- 发布／版本：2024起；读取当前页面
- 查阅程度：论文页；已打开
- 支持的结论与限制：用于检验文本上下文是否真正帮助预测。

## S32

- 标题：[LLM Agents for Time-Series: A Survey](https://arxiv.org/abs/2608.26226)
- 作者／机构：Yilong Chen 等
- 发布／版本：2026-08-26
- 查阅程度：摘要与HTML参考文献；已打开
- 支持的结论与限制：作者注明Findings of EMNLP 2026录用；用于扩展检索，不替代具体项目证据。

## S33

- 标题：[Position: Beyond Model-Centric Prediction — Agentic Time Series Forecasting](https://arxiv.org/abs/2602.01776)
- 作者／机构：Mingyue Cheng 等
- 发布／版本：2026-02-02；v4 2026-03-11
- 查阅程度：立场论文；已打开
- 支持的结论与限制：工作流、Agentic RL、混合路线的概念框架；不是落地实验证明。

## S34

- 标题：[FLAIRR-TS — Forecasting LLM-Agents with Iterative Refinement and Retrieval for Time Series](https://aclanthology.org/2025.findings-emnlp.834/)
- 作者／机构：论文作者团队
- 发布／版本：Findings of EMNLP 2025
- 查阅程度：正式论文页；已打开
- 支持的结论与限制：无权重更新的检索/预测修正；不是NeuralForecast训练控制器。

## S35

- 标题：[TimeXL: Explainable Multi-modal Time Series Prediction with LLM-in-the-Loop](https://arxiv.org/abs/2503.01013)
- 作者／机构：Yushan Jiang 等
- 发布／版本：2025-03-02
- 查阅程度：论文摘要；检索正文
- 支持的结论与限制：文本反思改进并触发编码器重训；AUC结果不能解释成销量误差下降。

## S36

- 标题：[TS-Agent: Understanding and Reasoning Over Raw Time Series via Iterative Insight Gathering](https://arxiv.org/abs/2510.07432)
- 作者／机构：Penghang Liu 等
- 发布／版本：2025-10-08；v2 2026-04-06
- 查阅程度：论文摘要；已打开
- 支持的结论与限制：工具驱动时序问答/推理；与S19金融训练TS-Agent不是同一论文。

## S37

- 标题：[TS-Reasoner: Domain-Oriented Time Series Inference Agents for Reasoning and Automated Analysis](https://arxiv.org/abs/2410.04047)
- 作者／机构：Wen Ye 等
- 发布／版本：2024-10-05；v6 2026-04-10
- 查阅程度：论文摘要；已打开
- 支持的结论与限制：领域工具和错误反馈；以分析推理为主。

## S38

- 标题：[Bridging the Last Mile of Time Series Forecasting with LLM Agents](https://arxiv.org/abs/2606.02497)
- 作者／机构：Yuhua Liao 等
- 发布／版本：2026-06-01
- 查阅程度：摘要及HTML；已打开
- 支持的结论与限制：业务上下文修正预测；不能直接当成训练自动化或具名生产部署证明。

## S39

- 标题：[Nexus: An Agentic Framework for Time Series Forecasting](https://arxiv.org/abs/2605.14389)
- 作者／机构：论文作者团队
- 发布／版本：2026-05
- 查阅程度：论文摘要；已打开
- 支持的结论与限制：宏观/微观变化与上下文综合预测；邻接方向。

## S40

- 标题：[CastFlow: Learning Role-Specialized Agentic Workflows for Time Series Forecasting](https://arxiv.org/abs/2604.27840)
- 作者／机构：Pan 等论文作者团队
- 发布／版本：2026-04
- 查阅程度：论文摘要；已打开
- 支持的结论与限制：SFT+RLVR优化专用预测LLM；进阶分支，非首版必要条件。

## S41

- 标题：[LangTime: A Language-Guided Unified Model for Time Series Forecasting with Proximal Policy Optimization](https://proceedings.mlr.press/v267/niu25e.html)
- 作者／机构：Wenzhe Niu 等
- 发布／版本：ICML 2025
- 查阅程度：正式会议论文页；已打开
- 支持的结论与限制：RL用于预测模型微调；与Agent编排训练的层级不同。

## S42

- 标题：[Greykite: A flexible, intuitive, and fast forecasting library](https://www.linkedin.com/blog/engineering/open-source/greykite-a-flexible-intuitive-and-fast-forecasting-library)
- 作者／机构：LinkedIn Engineering
- 发布／版本：2021-05-13
- 查阅程度：企业第一方技术文章；已打开
- 支持的结论与限制：内部业务/容量预测、模板和自动搜索；非LLM Agent。

## S43

- 标题：[Introducing Orbit, An Open Source Package for Time Series Inference and Forecasting](https://www.uber.com/us/en/blog/orbit/)
- 作者／机构：Uber Engineering
- 发布／版本：2021
- 查阅程度：企业第一方技术文章；已打开
- 支持的结论与限制：营销团队规划/预测实际应用；非Agent。

## S44

- 标题：[Simplifying data: IBM’s AutoAI automates time series forecasting](https://research.ibm.com/blog/autoai-time-series)
- 作者／机构：IBM Research
- 发布／版本：2021-03-17
- 查阅程度：企业第一方技术文章；检索正文
- 支持的结论与限制：特征/窗口/模型/管线搜索；传统AutoML。

## S45

- 标题：[Building a time series experiment](https://www.ibm.com/docs/en/watsonx/saas?topic=learning-automating-time-series-forecast-experiment)
- 作者／机构：IBM
- 发布／版本：持续更新
- 查阅程度：官方产品文档；检索正文
- 支持的结论与限制：时序AutoAI产品能力；不是具名客户成效证据。

## S46

- 标题：[Merlion](https://github.com/salesforce/Merlion)
- 作者／机构：Salesforce
- 发布／版本：持续更新
- 查阅程度：官方仓库；已打开
- 支持的结论与限制：时序框架及AutoML；企业开源身份不等于Agent生产部署。

## S47

- 标题：[How Decathlon runs demand forecasting at scale with Chronos-2](https://aws.amazon.com.cdn.amazon.com/blogs/machine-learning/how-decathlon-runs-demand-forecasting-at-scale-with-chronos-2/)
- 作者／机构：AWS / Decathlon 联合作者
- 发布／版本：2026-08-28
- 查阅程度：第一方案例；检索正文
- 支持的结论与限制：零售跨区域、减少周重训负担；基础模型替代路线，非Agent。

## S48

- 标题：[Agentic AI](https://www.datarobot.com/product/agentic-ai/)
- 作者／机构：DataRobot
- 发布／版本：持续更新
- 查阅程度：官方产品页；检索正文
- 支持的结论与限制：预测工具接入Agent平台；不证明自主训练闭环在客户处落地。

## S49

- 标题：[创建实验 — 大数据智能体工作台 DataBuddy](https://cloud.tencent.com/document/product/1835/137607)
- 作者／机构：腾讯云
- 发布／版本：持续更新
- 查阅程度：官方文档检索正文；直接打开失败
- 支持的结论与限制：时序AutoML和试验Notebook留痕；不能仅凭产品名判为LLM训练Agent。

## S50

- 标题：[NeoResearch（智多星）](https://ustc-time-series.github.io/NeoResearch/)
- 作者／机构：USTC 项目团队
- 发布／版本：2026-07项目页
- 查阅程度：项目介绍页；已打开
- 支持的结论与限制：页面末尾明确后续补充代码/论文/benchmark；当前是方案线索。

## S51

- 标题：[Agentic Model Selection Benchmark — issue #9846](https://github.com/sktime/sktime/issues/9846)
- 作者／机构：sktime 社区
- 发布／版本：2026-04-10
- 查阅程度：第一方项目issue；检索正文
- 支持的结论与限制：Agent与传统AutoML公平对比的提案；不能写成已发布基准。

## S52

- 标题：[An on-premises end-to-end automated forecasting multi-agent system for the energy domain](https://doi.org/10.1016/j.engappai.2026.115728)
- 作者／机构：Engineering Applications of Artificial Intelligence 论文作者
- 发布／版本：2026
- 查阅程度：出版商检索摘要；正文访问失败
- 支持的结论与限制：能源领域本地部署、多Agent选模线索；未核对完整实验和代码。

## S53

- 标题：[PepsiCo deepens AI capabilities with Google Cloud](https://www.pepsico.com/newsroom/press-releases/2026/pepsico-deepens-ai-capabilties-with-google-cloud)
- 作者／机构：PepsiCo
- 发布／版本：2026-04-22
- 查阅程度：企业官方公告；检索正文
- 支持的结论与限制：确认合作方向；不能证明Agent自动调参细节。

## S54

- 标题：[Game Day at Global Scale: How PepsiCo Foods Uses AI to Feed the World Cup](https://www.databricks.com/dataaisummit/session/game-day-global-scale-how-pepsico-foods-uses-ai-feed-world-cup)
- 作者／机构：Databricks / PepsiCo 演讲页
- 发布／版本：页面活动页头为2027，与本次截止日不一致
- 查阅程度：第一方活动页；检索正文；时间存疑
- 支持的结论与限制：不将其列为截止2026-09-16的已验证生产事实。

## S55

- 标题：[Scaling Demand Forecasting at Global Scale with Agentic AI](https://www.zenml.io/llmops-database/scaling-demand-forecasting-at-global-scale-with-agentic-ai)
- 作者／机构：ZenML LLMOps Database
- 发布／版本：未核实
- 查阅程度：二手检索线索；不支撑技术结论
- 支持的结论与限制：PEP Planner与MLflow详细叙述未取得足够一手佐证；暂缓采信。

## S56

- 标题：[AutoAI-TS: AutoAI for Time Series Forecasting](https://arxiv.org/abs/2102.12347)
- 作者／机构：Syed Yousaf Shah 等
- 发布／版本：2021
- 查阅程度：论文摘要；已打开
- 支持的结论与限制：传统时序AutoML技术背景。

