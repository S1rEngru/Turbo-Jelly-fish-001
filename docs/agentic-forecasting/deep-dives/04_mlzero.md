# AutoGluon Assistant / MLZero：优先参考数据适配和错误记忆

查阅日期：2026-09-17。官方开源实现采用 Apache-2.0；项目 README 明示是研究代码，当前列为 Linux-only。[官方仓库](https://github.com/autogluon/autogluon-assistant)

## 1、背景

AutoGluon Assistant 将原始数据与自然语言需求转成机器学习工作流，覆盖多种数据类型；MLZero 是其研究方法名称。它面向端到端 ML 自动化，时序只是应用范围的一部分。[Amazon Science 介绍](https://www.amazon.science/blog/autogluon-assistant-zero-code-automl-through-multiagent-collaboration)

**入选理由（本项目分析）**：不同品牌的数据结构、文件命名和字段口径可能不一致。在这种情况下，自动化瓶颈可能首先是数据接入和代码适配，而不在预测算法本身。

## 2、问题

MLZero 论文将数据理解、库知识与执行经验结合：原始目录需要被理解；代码需要正确调用库；执行报错后需要避免重复犯错。[论文 v1](https://arxiv.org/html/2505.13941v1)

**映射到本项目**：如果 baseline 已经可以稳定处理标准表，没必要重新生成整个训练工程。更合理的参考点是让 Agent 将品牌原始数据转成标准接口，并帮助定位已有脚本的配置或调用错误。

## 3、难点

这类自动编程系统需要面对“代码能运行但任务理解错了”的问题。不同品牌中相同的 `sales` 列，可能表示件数、含税收入或净收入；模型无法仅凭字段名可靠判定业务口径。此处是本项目风险分析，不是论文中的特定实验结果。

另一个实际限制是版本变化。仓库记录了 2025-11 的节点管理与 MCTS 扩展；2026-09 的 ExTS 公告则说明代码待发布。不能把这些后续内容当作原始 MLZero 论文已实现和评估的部分。[官方 README](https://github.com/autogluon/autogluon-assistant)

## 4、实现方法

### 原始论文的四部分

| 部分 | 功能 |
|---|---|
| Perception | 理解文件、任务与适用库 |
| Semantic memory | 检索相关库文档，提供编程知识 |
| Episodic memory | 保存尝试与错误，提炼错误摘要和修复建议 |
| Iterative coding | 生成、执行和修订代码 |

这套设计把“查文档”和“记住本次失败”分开。二者一个提供通用知识，一个提供当前任务上下文。[论文 v1，方法部分](https://arxiv.org/html/2505.13941v1)

### 如何借鉴（建议）

对 NeuralForecast 项目，语义知识可以从已确认版本的 API、现有 baseline 参数说明和业务数据字典开始；实验记忆则记录运行编号、错误类型、修复动作及验证结果。

例如，训练因为未来协变量缺失失败，下一轮应该首先检查预测区间的数据，而不是盲目更换模型。错误记忆最好表达为可验证条件：“指定变量在全部预测日期必须存在”，而不是一句“下次注意数据”。

建议将修复分级：字段映射、参数值和路径可以受控自动修改；目标定义和时间切分由固定配置约束。评估程序与最终留出集不交给代码生成器修改，否则成功运行的含义容易被改变。

本地为 Windows；如以后试运行，应按当前官方平台要求准备环境。本轮只是研究参考，没有安装依赖。

## 5、效果

Amazon Science 报告：MAAB 的 25 个任务上成功率为 92%；MLE-bench Lite 的 21 个任务中完成并提交有效方案 18 个，约 86%，并报告六枚金牌。[官方结果说明](https://www.amazon.science/blog/autogluon-assistant-zero-code-automl-through-multiagent-collaboration)

这些是作者在综合 ML 任务上的结果。成功率衡量流程是否产出有效方案；竞赛成绩另行衡量方案质量。**不能解释成时序预测准确率为 92%，也不能据此推算人工时间节省比例。**

对本项目，值得测量的是：遇到新品牌文件后多久完成有效接入，多少次需要人工修复，重复错误是否减少。预测效果应在标准化数据之后用独立回测再评估。

## 出处与阅读顺序

1. [Amazon Science 项目介绍](https://www.amazon.science/blog/autogluon-assistant-zero-code-automl-through-multiagent-collaboration)：背景与实验口径。
2. [MLZero 论文 v1](https://arxiv.org/html/2505.13941v1)：四部分架构与记忆机制。
3. [官方仓库](https://github.com/autogluon/autogluon-assistant)：平台要求、研究代码声明和后续版本变化。

核验范围：官方说明与论文，未在本地试运行，也没有企业跨品牌生产收益的直接验证。
