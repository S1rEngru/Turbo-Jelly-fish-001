# 检索记录、覆盖范围与后续核实

日期：2026-09-16。任务是公开资料调研，不是系统性综述注册研究。以下记录代表性查询、追溯路径及取舍，不能声称穷尽所有企业内部项目。

## 1. 检索问题

1. 是否存在Agent真正参与时序模型训练、特征选择、调参或实验迭代？
2. 哪些资料能证明企业实际使用，哪些只说明企业研发、开源或产品能力？
3. 哪些实现可以接近现有NeuralForecast baseline？
4. 是否有足够强的非Agent基线和专门的时序工程Agent评估资源？
5. 哪些“forecasting agent”实际上处理事件预测、预测解释或结果修正？

## 2. 代表性实际查询

检索通过网页搜索与一手网页打开完成。主要英文查询：

- `time series forecasting agent automated training enterprise Salesforce Merlion Uber Greykite`
- `LLM agent time series forecasting training agent reinforcement learning paper`（早期概念澄清前的探索，后续主线已转为Agentic AutoML）
- `Nixtla NeuralForecast automatic hyperparameter optimization Ray Optuna`
- `TimeSeriesScientist forecasting agent`
- `agentic forecasting AutoML enterprise`
- `time series agent model selection training forecasting github`
- `site.microsoft.com Finn agent forecasting`
- `TimeCopilot official github timecopilot forecasting`
- `site.amazon.science AutoGluon Assistant`
- `site.microsoft.com research RD-Agent data science forecasting`
- `agentic forecasting site.aws.amazon.com/blogs`
- `forecasting agent site.cloud.google.com/blog model`
- `demand forecasting agent training enterprise case study`
- `site.databricks.com PepsiCo PEP agent`
- `PepsiCo MLflow agents`
- `site.aws.amazon.com Decathlon Chronos-2`
- `TS-Agent 2508.13915 github`
- `DCATS forecasting github`
- `AIDE AI-Driven Exploration github paper`
- `AutoML-Agent github DeepAuto`
- `TemporalBench 2602.13272`
- `TimeXL time series agent github paper`
- `site.h2o.ai Driverless AI agent time series`
- `site.dataiku.com agent forecasting training`
- `site.ibm.com AutoAI time series agent`
- `site.github.com/TimeCopilot/timecopilot AutoNHITS`
- `TimeSeriesGym github`
- `time series agent training Salesforce research`

中文补充查询：

- `时序 智能体 预测 训练 企业`
- `时序预测 建模智能体`
- `时序 AutoML 智能体`

查询中的引号和组合在搜索过程中有调整，这里保留搜索主题与可复用关键词。

## 3. 文献追溯路径

- TimeCopilot → 官方论文 → NeuralForecast的AutoNHITS/AutoTFT → 发行记录确认新增模型说明。
- TimeCopilot参考文献 → TimeSeriesGym → 官方仓库，补充工程能力评估。
- 微软Finn → 官方Agent说明 → 微软内部财务预测案例，分开标注软件能力与历史收益。
- 时序Agent综述 → TS-Reasoner、推理TS-Agent、TimeXL、CastFlow → 返回各论文原始页面核对任务。
- Salesforce官方MoiraiAgent介绍 → 文章的GitHub链接 → uni2ts中的研究实现目录。
- 中文搜索 → 腾讯云官方文档和NeoResearch项目页；后者明确是项目介绍，未升级为可复现系统。
- 企业需求预测搜索 → PepsiCo二手材料 → 官方公告／演讲页；因技术细节和时间证据不足暂缓采信。

## 4. 查阅深度

| 层次 | 本次完成的工作 | 未完成的工作 |
| --- | --- | --- |
| 直接相关项目 | 阅读官方说明、README、论文关键方法和结果段落 | 安装、运行、逐行代码审计 |
| 企业资料 | 检查第一方发布、业务用途和可归因的收益 | 获取企业内部数据、独立ROI审计 |
| 论文 | 核对标题、版本／年份、任务类型、重点方法与证据范围 | 每篇逐页精读、逐项重现实验 |
| 链接 | 打开主要核心来源；对部分使用原网站检索正文 | 不保证每个动态链接未来持续有效 |
| 归档 | 保存来源链接、书目信息和研究笔记 | 不保存全部网页全文或论文PDF副本 |

“Sxx来源记录数量”不是独立案例数量：同一项目可能有论文、代码和企业文章多条来源。

## 5. 被排除或降级的材料

| 材料 | 处理 | 原因 |
| --- | --- | --- |
| 只列Agent提示词的角色库 | 不纳入核心案例 | 没有训练执行、实验或部署证据 |
| TimeCopilot的非官方复制仓库 | 使用官方仓库替代 | 防止重复计数或引用过时副本 |
| 二手博客／聚合网站中的准确率与ROI | 不作技术依据 | 优先回到论文、企业或项目原始来源 |
| PEP Planner详细实现叙述 | 仅保留追踪来源S55 | 尚未取得充分一手证明 |
| Databricks PepsiCo演讲页 | S54标记时间存疑 | 页头活动时间为2027，不能当成截至2026-09-16已发生证据 |
| NeoResearch项目页 | 方案／路线图线索 | 页末说明以后补代码、论文和benchmark |
| sktime Agent选模benchmark issue | 社区提案 | issue关闭不证明已完成或合并发布 |
| 交通参与者预测、金融交易Agent、地缘事件概率预测 | 不作为主线 | “agent／forecasting”同词，但任务对象不同 |
| 普通LLM预测模型 | 不直接算建模Agent | 模型本身使用LLM并不意味着能控制训练流程 |
| H2O/Dataiku等广泛平台能力 | 未扩写为实际训练Agent案例 | 搜到的资料不足以证明本任务要求的闭环 |

## 6. 访问限制

- DCATS论文给出的项目页未能打开：保存入口，不确认代码完备性。
- 能源领域本地多Agent系统DOI全文访问失败：只用出版商检索摘要，标为延伸线索。
- 腾讯云DataBuddy文档直接打开失败，但搜索返回了官方正文：已按该程度标注。
- AutoGluon时序教程页面打开失败：主报告关于Assistant的结论来自可访问的Amazon Science文章、官方仓库与MLZero论文。
- TimeCopilot某些深层路径／单个tag页面返回cache miss：改以成功读取的官方总仓库、论文和发行列表支持结论，不声称审计了对应源码。
- 本次未固定动态仓库的commit，因此不承诺README与论文版本完全一致。

## 7. 数字与宣传声明的处理规则

- 企业收益保留其业务和系统范围，不归因于文档没有说明的LLM组件。
- 作者报告的预测误差、工程成功率、AUC、业务ROI分别解释，不能互换。
- “benchmark第一”需要榜单时间、模型组合与评估协议；不沿用动态宣传作永久结论。
- 对TimeCopilot明确区分：已有NeuralForecast模型接入 vs. 基础模型集成的榜单结果 vs. 尚待测量的Agent增益。
- 不因为作者来自大公司，就把学术实验列为企业内部生产案例。

## 8. 可直接接续的核实任务

| 优先级 | 下一步 | 需要得到的证据 |
| --- | --- | --- |
| 1 | 阅读用户baseline | 实际模型、数据schema、手动修改点、验证方式 |
| 1 | 精读TimeCopilot接口与源码 | 是否能包装现有训练脚本；可传递哪些NeuralForecast配置 |
| 1 | 设计固定模板/HPO/Agent三组对照 | 同数据、同时间切分、同预算的误差和人工成本 |
| 2 | 核查Finn迭代及更新源码 | 哪些决策来自LLM，哪些来自确定性规则 |
| 2 | 获取金融训练TS-Agent官方实现 | 作者仓库、许可、评估器边界和依赖 |
| 2 | 验证DCATS训练数据选择在品牌间的迁移 | 对负迁移和新品牌表现的实测 |
| 3 | 核查MoiraiAgent商用与研究版差异 | 商用边界、版本、实际适用任务 |
| 3 | 复核企业具名案例与收益 | 公开演讲原始视频／讲稿、部署时间、效果口径 |

以上均为下一阶段建议，不是本次已经执行的实验。

