# 降低重训成本与有依据的参数搜索：NeuralForecast 和 Agentic AutoML 调研

日期：2026-09-18。根据本轮对话与已查阅的官方文档、源码整理；本次仅保存调研，没有运行训练实验。动态文档与 main 分支没有锁定提交，功能能否直接使用仍需与企业环境安装版本对照。

## 阅读导航

- 想知道改参数后能否少训：第 2–3 节。
- 想知道如何有依据地调参：第 5–6 节。
- 想优先看生产、多指标与预算问题：第 6 节 FLAML、Ax。
- 想知道当前项目先学什么：第 7 节。
- 所有原始出处：第 8 节。

## 1. 结论与场景

两个问题都有现成方案，而且不依赖先引入 Agent。较适合当前场景的分工是：Agent 提出搜索范围和实验方向，成熟优化器负责采样与预算，NeuralForecast 负责训练，现有评估逻辑判断候选表现。

用户现有流程包含多个模型、扩展窗口训练、下一 AP 的逐日预测、四模型的 15 种等权组合，以及周月评估。模型和品牌未来都会增加。本报告不重新研究已有财务日历，不假设七个数据集就是七个品牌。

以下“迁移建议”属于本次分析，不是已经验证的企业收益。来源中的业务案例、软件能力和学术实验分别标明。

## 2. “部分重训”其实包含不同问题

| 修改内容 | 可复用什么 | 还需要做什么 |
|---|---|---|
| 组合成员、均值或权重 | 已保存的逐日预测 | 重新计算组合和评估；通常不训练 |
| 只改四模型中的一个 | 其他模型在相同数据与窗口上的预测 | 只运行受影响的模型 |
| 同一模型增加训练预算或调整兼容的学习率设置 | 已有模型权重 | 继续优化并重新评估 |
| 改层数、隐藏维度、模型类型或输入特征维度 | 部分兼容权重可能有价值 | 通常重新训练，或设计专门权重迁移 |
| 新 AP 数据到来 | 既有模型配置、可能兼容的权重 | 选择完整重训、续训或降低重训频率 |
| 冻结部分网络，只训练其余层 | 冻结层权重 | 属于模型专属微调；本轮未确认 NeuralForecast 的统一高层接口 |

前两项是当前流程最直接的省成本方向。复用成立的前提是数据版本、切分、特征、归一化和未修改模型配置一致。共享预处理一旦变化，影响范围可能不止一个模型。

多层级序列共同训练时，品牌通常不能被当作独立的权重块；只用某品牌数据更新共享模型，也可能影响其他序列。这与“只重训一个独立模型”不是同一件事。

### 三种复用必须分开

1. **预测或评估缓存**：不重复计算已完成的相同任务。
2. **权重 warm start**：从已有参数继续学习，可能重新初始化优化器。
3. **搜索 warm start**：复用历史配置及其成绩来指导下一次搜索，不意味着复用神经网络权重。

完整 checkpoint 恢复还可能包括优化器、调度器和步数；不要将模型文件存在视为完整恢复已经成立。

## 3. NeuralForecast 能支持到什么程度

### 3.1 普通模型：可以保留权重再训练

官方 fit 的 use_init_models=True 会丢弃已训练权重并重新初始化，默认 False。[N1]

因此，在保留已训练普通模型的情况下，可以研究继续优化已有权重。若外层脚本每次重新创建模型实例，这个参数本身不会凭空找回过去训练结果。

本轮查看普通本地训练路径，_fit 创建新 Lightning Trainer，并调用 trainer.fit；标准 save 保存 hyper_parameters 与 state_dict，没有完整优化器状态。[N2]

**结论：可研究权重续训，不能默认是精确断点续训。** Adam 动量、学习率调度和训练进度是否恢复需要另外确认。修改参数也必须进入实际模型或训练器使用的位置，仅修改一个外部字典不保证已生效。

### 3.2 Auto 模型：不同配置通常新建模型

当前 BaseAuto._fit_model 使用 cls_model(**config) 创建模型再训练。因此 AutoNHITS 等自动搜索不代表各 trial 自动共享权重。[N3]

同一源码也有特定流程的搜索结果复用逻辑，但不能将内部特殊路径当作通用跨品牌、跨参数 checkpoint 复用接口。

### 3.3 滚动评估：可以降低重训频率

cross_validation 的 refit=False 表示开头训练后继续预测后续窗口；正整数 refit=k 表示间隔若干窗口重训。[N4]

这减少的是重训次数，改变了“每个 AP 都重训”的策略。它不是完全不训练，也不是自动只更新一部分网络。是否合适应与原策略对照。

### 3.4 自动搜索和提前终止

NeuralForecast Auto 支持默认与自定义搜索空间、Ray 和 Optuna 后端；官方教程展示了 HyperOptSearch。[N5]

当前 BaseAuto 源码提供 Ray scheduler 选项，并在验证时报告指标。[N3] 可以据版本研究提前停止低价值候选，但“能传 scheduler”不等于所有调度算法的恢复需求都已满足。

### 3.5 当前场景的兼容性边界

已有逐序列 norm、三类特征和 28–35 的 horizon。判断能否续用权重时需要核对尺度、输入输出形状与语义；不是模型名称相同就一定兼容。某些参数可以改变训练策略而不改变权重形状，另一些参数会改变网络结构。

续训后的候选依赖它此前的训练历史。若要比较参数本身的价值，应明确使用共同起点或将续训路径作为实验配置记录，不能与从头训练的候选混成相同条件。

## 4. 之前收集的五个项目有什么相关策略

| 项目 | 已确认的策略 | 不应混淆的能力 |
|---|---|---|
| TS-Agent | 案例筛选模型、知识指导改进、日志指导参数；少量迭代后集中预算 | 脚本回退不等于权重回退；少量迭代不等于少量 epoch。[P1] |
| Finn | 正常更新与重新优化分开，退化触发、轮数限制和回退 | 复用配置不保证完整优化器状态续训。[P2] |
| TimeCopilot | NeuralForecast Auto 预测器暴露搜索配置 | 实际搜索策略仍取决于后端，不能认定默认是先进优化算法。[P3] |
| TSci | 提出候选与参数空间，再执行数值验证 | 方法可学；此前发现的失败回退问题仍限制直接复用代码。[P4] |
| RD-Agent | 历史实验驱动假设、运行与反馈 | 编排框架不自动等于条件参数优化器或权重续训器。[P5] |

这些 Agent 项目更擅长解释“为什么试”，但省训练成本和约束优化可以交给下面的专门工具。它们并不都需要 LLM。

## 5. 真正针对训练预算的公开方案

### 5.1 Ray Tune PBT：复用较好候选的训练状态再改参数

Population Based Training 让表现较差的 trial 复制较好 trial 的模型与优化器状态，调整学习率等超参数，再继续训练。官方有 checkpoint 保存、加载和应用新超参数的代码示例。[A1]

它学习的可能是随训练变化的参数安排，而非一组从头到尾固定的参数。

**接入限制**：必须实现恢复机制，并确保加载后新配置生效。NeuralForecast 默认 Auto 训练函数没有完整的 PBT checkpoint 恢复路径，不能因为支持 Ray 就宣称 PBT 开箱即用。[N3、A1]

迁移判断：适合进一步研究兼容架构内的训练参数调整；多个模型架构之间不能假设可以互相复制权重。PBT 自身需要维护多个 trial，也有保存和加载开销，不保证总费用必然降低。

### 5.2 ASHA / Hyperband：尽早停止差候选

调度器按中间表现分配资源，把更多预算留给有希望的候选。[A2] 主要节省无效训练，不是将不同结构的权重拼接复用。不同算法和运行模式对暂停／恢复的需求也不同。

迁移判断：通常比直接接 PBT 更适合作为第一批预算控制参考。评估信号需要具有可比性；有些模型收敛慢，过早淘汰可能错过最终好模型。训练早期的 loss 也不一定代表最终周月业务指标，完整候选仍需原评估流程验证。

## 6. 有依据的搜索空间与生产约束

“有依据”包含合法空间、历史实验引导和业务约束。仍然需要试验，但可以更合理地选择试什么、试多久。

### 6.1 NeuralForecast Auto + Optuna：最贴近现有技术栈

Optuna 支持条件分支与不同参数分布：选择模型后，只搜索适用于该模型的参数。TPE 根据已有 trial 结果引导后续采样。[O1、O2]

迁移建议：Agent 可解释为何重点探索窗口、学习率或某类模型，优化器负责在给定空间中选数值。无需让 LLM 每次猜全部参数。

训练 loss 与选型目标需要分开。Optuna 支持多目标优化，但 NeuralForecast Auto 默认围绕单个验证 loss；不能把底层支持多目标直接等同于当前包装自动支持完整的日周月业务目标。[O3、N3]

若要把现有评估脚本直接作为目标，可以研究外层自定义优化函数：一次候选训练后返回业务指标和成本。这是接口建议，本轮没有实施。

### 6.2 FLAML：搜索成本不一样时，优先看它

微软开源 FLAML 的 CFO / BlendSearch 考虑配置成本，支持低成本起点、类别成本信息和历史配置结果。提供 evaluated_rewards 可避免重跑同一评估任务的已知结果。[F1]

它还提供训练时间、预测时间与指标约束；其调优接口支持按优先顺序处理多个目标。[F1、F2]

迁移例子：先满足主要误差要求，在业务允许的性能差异内选择更低成本方案。模型专属空间可以同时纳入配置合法性和成本信息。

边界：这里的 warm start 多指搜索经验。品牌、数据或评估协议变了，不能把旧分数当作新任务已经得到的成绩。FLAML 是通用优化参考，不是本轮已确认可直接替换企业 NeuralForecast 训练的现成系统。

### 6.3 Ax + BoTorch：多指标生产优化的强参考

Meta 工程文章说明 Ax 用于生产推荐系统、基础设施和模型优化，并给出准确率与资源权衡、改善主要指标同时避免其他指标退化的用途。这是企业第一方使用证据，不只是学术单指标排行榜。[X1]

Ax 支持目标和结果约束配置。[X2] 生产中不一定需要把所有指标随意加权成一个数，也可以返回不同权衡的候选集合。

迁移例子：尽量降低主预测误差，同时要求周／月误差退化不超业务容忍范围，限制训练成本。具体选择哪些指标、约束阈值是多少，应来自用户现有 eval 和业务规则。

边界：软件支持约束优化，不代表对未来真实效果作保证；约束本身也需要通过实验测量。Meta 案例不是零售预测效果证明。

### 6.4 Syne Tune：跨品牌搜索经验迁移

Syne Tune 提供从历史任务选择起点、缩小搜索范围、学习高潜力配置区域的方法。BoundingBox 用旧任务较好配置构造较小空间；Quantiles 通过任务内排名信息引导搜索，不强行划死边界。[Y1]

这与跨品牌扩展很贴近：旧品牌的实验可以帮助新品牌安排候选优先级。迁移的是搜索知识，不能直接当作神经网络权重迁移。

项目开源。AWS 公开过使用它优化 S3 下载配置的工程例子，展示通用参数搜索的业务用途，但不是销售预测验证。[Y2、Y3]

迁移边界：品牌差异大时，过早缩小空间可能排除新品牌的好方案，应保留探索。历史成绩还要带上数据、预算和协议，才有解释价值。

## 7. 当前项目建议的学习顺序

| 顺序 | 研究动作 | 原因 |
|---|---|---|
| 1 | 缓存逐日预测，只重训受影响模型 | 先消除确定性的重复计算 |
| 2 | 模型专属搜索空间 + Optuna / FLAML | 让试验有范围、历史依据和成本边界 |
| 3 | 提前停止差候选 | 减少无效训练预算 |
| 4 | 参考 Ax 明确业务目标与约束 | 不把训练 MAE 当作全部选型依据 |
| 5 | 验证权重续训、PBT 和跨品牌搜索迁移 | 能力更强，但兼容性与实验公平性需要单独验证 |

可以考虑的职责划分：

- Agent：理解评估报告、提出动作、说明搜索范围、读取历史经验。
- 优化器：依据试验结果生成参数、控制预算、处理约束。
- 训练工具：执行已有 NeuralForecast 流程，记录实际生效配置。
- 评估工具：沿用业务协议，计算各 AP、粒度与层级表现。
- 缓存与记录：识别结果是否可复用，区分预测缓存、权重和搜索历史。

这些角色是逻辑职责，不要求实现成五个 Agent。

后续最少需要确认的事项是安装版本、模型是否每轮重新实例化、保存内容、最终 eval 口径。它们用于判断采用哪条省成本路径，不影响本轮结论。

## 8. 出处与证据类型

本轮查阅日期：2026-09-18。以下均为官方文档、项目源码或作者论文；没有运行对应示例，不引用未经核实的收益百分比。

| ID | 原始出处 | 支持的结论 |
|---|---|---|
| N1 | [NeuralForecast Core](https://nixtlaverse.nixtla.io/neuralforecast/core.html) | use_init_models、fit 接口 |
| N2 | [BaseModel 源码](https://raw.githubusercontent.com/Nixtla/neuralforecast/main/neuralforecast/common/_base_model.py) | _fit、configure_optimizers、save/load；权重与完整断点的区别 |
| N3 | [BaseAuto 源码](https://raw.githubusercontent.com/Nixtla/neuralforecast/main/neuralforecast/common/_base_auto.py) | 候选新建模型、验证报告、scheduler 和默认优化目标 |
| N4 | [Core 源码](https://raw.githubusercontent.com/Nixtla/neuralforecast/main/neuralforecast/core.py) | cross_validation 的 refit 语义 |
| N5 | [NeuralForecast 调参教程](https://nixtlaverse.nixtla.io/neuralforecast/docs/capabilities/hyperparameter_tuning.html) | 默认／自定义搜索空间、Ray、Optuna、HyperOptSearch |
| P1 | [金融 TS-Agent 论文 v1](https://arxiv.org/html/2508.13915v1) | 第 4 节候选与迭代策略 |
| P2 | [Finn 更新 API](https://microsoft.github.io/finnts/reference/update_forecast.html) | 更新与重新搜索的区别 |
| P3 | [TimeCopilot 神经模型 API](https://timecopilot.dev/api/models/neural/) | Auto 模型的可配置接口 |
| P4 | [TSci 验证模块](https://raw.githubusercontent.com/Y-Research-SBU/TimeSeriesScientist/main/time_series_agent/agents/validation_agent.py) | 候选、参数与数值验证 |
| P5 | [RD-Agent 循环源码](https://raw.githubusercontent.com/microsoft/RD-Agent/main/rdagent/components/workflow/rd_loop.py) | 实验与反馈编排 |
| A1 | [Ray Tune PBT 教程](https://docs.ray.io/en/latest/tune/examples/pbt_guide.html) | 权重与优化器复制、恢复和参数变更 |
| A2 | [Ray Trial Schedulers](https://docs.ray.io/en/latest/tune/api/schedulers.html) | ASHA、Hyperband 等资源分配方式 |
| O1 | [Optuna 条件搜索空间](https://optuna.readthedocs.io/en/stable/tutorial/10_key_features/002_configurations.html) | 分支、参数类型和范围 |
| O2 | [Optuna TPE](https://optuna.readthedocs.io/en/stable/reference/samplers/generated/optuna.samplers.TPESampler.html) | 历史 trial 引导搜索 |
| O3 | [Optuna 多目标示例](https://optuna.readthedocs.io/en/stable/tutorial/20_recipes/002_multi_objective.html) | 多目标能力及示例 |
| F1 | [FLAML 自定义函数调优](https://microsoft.github.io/FLAML/docs/Use-Cases/Tune-User-Defined-Function/) | 成本引导、CFO、BlendSearch、历史复用、目标优先级 |
| F2 | [FLAML 任务约束](https://microsoft.github.io/FLAML/docs/Use-Cases/Task-Oriented-AutoML/) | 训练时间、预测时间、指标约束 |
| X1 | [Meta：Ax 的生产使用](https://engineering.fb.com/2025/11/18/open-source/efficient-optimization-ax-open-platform-adaptive-experimentation/) | 企业第一方应用案例与多目标需求 |
| X2 | [Ax 结果约束](https://ax.dev/docs/next/recipes/outcome-constraints/) | 目标与约束接口；next 文档需核对版本 |
| Y1 | [Syne Tune 迁移调参教程](https://syne-tune.readthedocs.io/en/latest/tutorials/transfer_learning/transfer_learning.html) | ZeroShot、BoundingBox、Quantiles |
| Y2 | [Syne Tune 仓库](https://github.com/syne-tune/syne-tune) | 开源实现入口 |
| Y3 | [AWS：优化 S3 下载配置](https://aws.amazon.com/blogs/opensource/learn-amazon-simple-storage-service-transfer-configuration-with-syne-tune/) | 公开工程应用，非销量预测实验 |

与此前报告的关系：[场景化 Top 5](06_top5_for_existing_training_workflow.md) 讨论整体方法，本篇补充训练复用、数值搜索及生产约束。TSci 的失败回退代码问题详见前篇，不因本篇讨论其参数空间而撤销。
