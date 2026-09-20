# 开源 Harness 架构调研：单步实验迭代、工具边界与上下文工程

核查日期：2026-09-18。范围已扩展到公开开源生态，不局限于此前 AutoML 项目。本文研究设计，不实现企业系统。

## 先读结论

如果先读一个“通用完整 harness”，推荐 **Deep Agents**；如果先理解最小循环，读 **mini-swe-agent**；如果先看“根据模型表现提出一个改进并再实验”，读 **AIDE**。更显式的 ReAct 状态与权限引擎可参考 **AgentScope**，工具接口和观察裁剪可参考 **SWE-agent**。时序适配另见 [10 号文档](10_forecasting_agent_specialization.md)，首选 Finn 的实现。

没有把任何项目认证为“生产安全”。入选依据是能看到机制、具体实现以及部分针对边界的测试，不是 README 的自称、星数或榜单。进行了定点静态源码审阅，没有执行仓库测试或运行真实训练。

所有主要链接固定到 commit；网页 main 缓存与实时源码存在差异，本文以下载核查的固定版本为准。[机器可读来源清单](11_harness_source_manifest.json)记录仓库、文件、版本和文件哈希。

## 1. 任务定义与架构判断

Assignment 的核心应表述为：构建一个能读取真实模型反馈、提出可检查的改进假设、执行受限实验、判断结果并继续迭代的 harness。选型、特征和超参搜索是它能调用的实验动作。

ReAct 式“观察→决定→行动→再观察”适合作为主循环，但不意味着没有计划、没有状态机，或每次调用工具都必须涨分。查残差、读取失败日志、确认训练状态也属于有价值的动作。可以一次只提交一个实验假设，同时通过多个只读动作补充证据。

建议的边界：LLM 可以建议下一步；程序决定建议是否合法、是否超预算、分数如何计算、哪些结果能晋升。规划只需覆盖下一项可检验实验，不必预先规划整场优化。

## 2. 入选项目与适用角色

| 项目 | 主要学习对象 | 单步优化匹配度 | 应补齐的部分 |
|---|---|---|---|
| Deep Agents | 模块化装配、文件上下文、摘要、检查点接入、权限中间件 | 可做循环底座，非专门 ML 优化器 | 训练作业状态、业务评估器、实验去重 |
| AgentScope | 显式 ReAct 控制流、工具执行权限、取消与上下文压缩 | 适合实现受限动作循环 | 时序领域协议、训练持久化 |
| AIDE | draft/debug/improve、原子实验、父子实验档案 | 本轮最贴合“单次改进” | 确定性评估、权限隔离、生产作业管理 |
| SWE-agent | 工具注册、解析、超时、观察历史处理 | 编程任务循环，可迁移工具设计 | 预测领域工具和业务状态 |
| mini-swe-agent | 最小 query→execute 循环、限额、轨迹 | 最适合先读懂 | 不能直接当企业训练平台 |

上述项目开源；本轮核查的仓库许可证为 Deep Agents/AIDE/SWE-agent/mini-swe-agent 的 MIT、AgentScope 的 Apache-2.0。此处记录仓库许可声明，不替代依赖和数据许可检查。

## 3. Deep Agents：优先研究完整 harness 的工程拆分

### 可直接看到的实现

[graph.py](https://github.com/langchain-ai/deepagents/blob/9b515ef09a37a02034ef7384701bbcbbc91a122e/libs/deepagents/deepagents/graph.py) 的 create_deep_agent 装配文件、子代理、摘要等中间件，并暴露 checkpointer、store、interrupt_on、permissions。可学的是横切机制独立于业务工具。检查点参数需要应用实际配置，不等于默认已经得到持久数据库。

[summarization.py](https://github.com/langchain-ai/deepagents/blob/9b515ef09a37a02034ef7384701bbcbbc91a122e/libs/deepagents/deepagents/middleware/summarization.py) 将历史移出即时上下文，写入后端，摘要中保留路径供回读；包含 token 预算判断、工具参数裁剪和失败处理。摘要落盘失败时可继续摘要，因此不能仅依赖它满足“实验绝不丢失”的审计要求。

[filesystem.py](https://github.com/langchain-ai/deepagents/blob/9b515ef09a37a02034ef7384701bbcbbc91a122e/libs/deepagents/deepagents/middleware/filesystem.py) 定义读写操作与 allow/deny/interrupt 规则，并处理删除目录时子路径权限的重叠。适合学习“工具调用前检查”，不是仅在 system prompt 里禁止访问。

### 可靠性证据

[test_permissions.py](https://github.com/langchain-ai/deepagents/blob/9b515ef09a37a02034ef7384701bbcbbc91a122e/libs/deepagents/tests/unit_tests/test_permissions.py) 中 test_recursive_delete_refused_when_descendant_denied 测试：被保护的子目录存在时，删除父目录应失败且不发生部分删除。

[test_summarization_middleware.py](https://github.com/langchain-ai/deepagents/blob/9b515ef09a37a02034ef7384701bbcbbc91a122e/libs/deepagents/tests/unit_tests/middleware/test_summarization_middleware.py) 中 test_offload_writes_to_backend 检查摘要触发后确实写入历史文件；还有媒体落盘失败的测试。本轮阅读了相关测试正文，未运行。

### 如何迁移与限制

保留中间件思想，把“预算、权限、上下文构造、结果审计”放在训练工具外围。无需照搬全部子代理和规划功能。

文件权限只能覆盖经过相应中间件的文件工具。若另给任意 shell、Python 或其他直连文件工具，必须在操作系统/容器及该工具层面维持同样边界。传入 checkpointer 也不会自动恢复一次已经提交到 GPU 集群的外部作业。

推荐阅读顺序：graph 装配 → filesystem 权限 → summarization → 两个测试文件。

## 4. AgentScope：显式 ReAct 控制与权限策略

[_agent.py](https://github.com/agentscope-ai/agentscope/blob/5ff52f877de12d66a30d55af279dd4f42b1590f3/src/agentscope/agent/_agent.py) 的 _reply_impl/_next_action 组织循环，_execute_tool_call 与 _check_permission_impl 连接执行和权限；中断时有 _close_unfinished_tool_calls，另有压缩上下文及重复工具错误识别逻辑。

[permission/_engine.py](https://github.com/agentscope-ai/agentscope/blob/5ff52f877de12d66a30d55af279dd4f42b1590f3/src/agentscope/permission/_engine.py) 根据模式解析权限，DEFAULT 路径先检查 deny/ask，再处理只读和工具自身规则，之后才是 allow。规则匹配依赖工具的实现，不能只相信一个 read-only 标签。

值得迁移：
- 工具权限决策独立于 LLM 的理由；
- 取消动作后补齐工具状态，避免历史留下不完整调用；
- 持续失败作为运行状态，而非靠模型无限自救；
- 相互独立的读取可以并发，但训练提交、修改配置和晋升应有明确顺序。

限制：本轮未运行其测试，也未核查完整 session 存储链。不能因此声称它保证断点后外部训练作业“恰好一次执行”。当前实现较大，初读应沿函数调用链，而非从头顺读数千行。

## 5. AIDE：单步优化的最佳实验参考

[agent.py](https://github.com/WecoAI/aideml/blob/60b3978ddf65b71f86eb7c64506965048a1398cf/aide/agent.py) 的 search_policy 选择初始草案、修错或改进当前最好候选；_improve 提示要求一个可行动的原子改进，step 执行并记录结果。它是树状实验搜索，不宜误标成纯线性 ReAct。

[journal.py](https://github.com/WecoAI/aideml/blob/60b3978ddf65b71f86eb7c64506965048a1398cf/aide/journal.py) 保存父子关系、代码、计划、执行信息、分析与指标。generate_summary 默认从 good_nodes 汇总成功候选，因此移植时应另外保留失败摘要，以免重复走死路。

最值得借鉴的是“一轮一个可验证假设”和“从父实验生成新实验”，而非要求 Agent 每轮直接改动生产脚本。

关键限制：parse_exec_result 让 LLM 从执行输出中提取指标并作分析，随后做部分数值/错误校验。你的系统应改为评估服务直接产生机器可读指标，LLM 仅解释。模型自己修改训练脚本又解释其分数，不能作为最终成绩真实性的唯一依据。

可用实验节点字段：
- parent_experiment_id、配置差异、证据引用、待验证假设；
- 数据/修复/切分/代码版本；
- 作业 ID、预测文件、指标、实际资源成本；
- 成功/失败/被拒绝及原因；
- 候选被保留、晋升或淘汰的依据。

## 6. SWE-agent 与 mini-swe-agent：循环和工具接口

### SWE-agent

[tools.py](https://github.com/SWE-agent/SWE-agent/blob/3ea751c087f32b16e039a2233dd6eefecef325d5/sweagent/tools/tools.py) 将工具配置、解析、文档生成、超时和工具过滤集中管理；支持关闭额外 bash 工具。其命令 blocklist 主要避免不适合环境的命令，不能作为安全沙箱。

[history_processors.py](https://github.com/SWE-agent/SWE-agent/blob/3ea751c087f32b16e039a2233dd6eefecef325d5/sweagent/agent/history_processors.py) 中 LastNObservations、ClosedWindowHistoryProcessor 分别处理旧观察和过期文件窗口，可学“当前上下文只保留需要的观察”。删旧观察不等于持久记忆，应另存真实实验档案。

适配时把错误返回设计成“原因 + 合法参数/动作 + 是否可重试”，让 Agent 能修正，而不是把整个堆栈塞回去。

### mini-swe-agent

[default.py](https://github.com/SWE-agent/mini-swe-agent/blob/04d809ceab9df28f9adaed044884180159172930/src/minisweagent/agents/default.py) 的 step 直接 query 后执行动作；query 检查调用/费用/墙钟限制，run 处理格式错误并保存轨迹。它很适合理解循环最小组成。

[docker.py](https://github.com/SWE-agent/mini-swe-agent/blob/04d809ceab9df28f9adaed044884180159172930/src/minisweagent/environments/docker.py) 展示独立环境接口，支持容器运行和执行超时。使用 Docker 本身不说明默认已禁网、禁用宿主挂载或限定 GPU 预算。LLM API cost 也不能代替训练资源成本。

保存聊天轨迹不是恢复模型 optimizer、GPU job 或中间预测文件的保证。这个项目应作为阅读起点，而非不加修改就接企业数据。

## 7. 建议的最小严谨 harness（以下是适配建议）

### 7.1 状态与循环

persisted state → 构造本轮上下文 → LLM 提议 → 校验与预算预留 → 提交/查询动作 → 收集真实观察 → 评估/归档 → 下一轮。

实验状态建议：
DRAFT → VALIDATED → SUBMITTED → RUNNING → SUCCEEDED → EVALUATED → ACCEPTED/REJECTED。
另有 INVALID、FAILED、CANCELLED。执行失败和效果不好必须分开，后者可以是完全合法、很有信息量的实验。

记录先于副作用：先持久化请求和幂等键，再提交训练。重启后先凭请求键/作业 ID 查询已有工作，避免直接重训。若后端不支持幂等创建，需要对账恢复设计，不能仅靠本地缓存承诺 exactly-once。

### 7.2 三层约束

| 层 | 负责 |
|---|---|
| Prompt | 提出一个假设；引用证据；解释不确定性；使用合理领域启发式 |
| 程序策略 | schema、模型/特征白名单、参数范围、预算、重复检测、结果完整性、晋升规则 |
| 执行环境 | 文件/网络/凭证隔离、CPU/GPU/内存额度、超时与子进程清理 |

不要把第二、三层要求全部写成“你必须遵守”交给模型。另一方面，预算内、白名单内的动作应预授权自动完成；无需每轮人工确认，符合无人值守迭代目标。

### 7.3 工具边界

| 工具类别 | 示例（建议接口名） | 关键限制 |
|---|---|---|
| 读取 | get_task_spec/get_experiment/read_diagnostics | 只读、分页、按实验 ID 访问 |
| 配置 | propose_config/validate_config | 只生成新版本，不覆盖基线 |
| 执行 | submit_experiment/get_job/cancel_job | 幂等、配额；只能操作本任务作业 |
| 评估 | evaluate_predictions/compare_runs | 固定指标版本；不接受模型自报分数 |
| 终结 | select_candidate/finish_search | 候选完整可追溯，允许无改进退出 |

如确需改代码，另设隔离开发动作，代码版本审查和测试通过后注册；日常训练不暴露任意 shell/eval。

### 7.4 Context engineering

永久事实：任务、指标方向、数据权限、模型能力、预算规则。
当前状态：incumbent、最近结果、剩余预算、正在执行的 job。
实验记忆：相关成功、相关失败、被否定假设、不同版本的区别。
按需证据：残差切片、归因、配置 diff、日志尾部和完整工件路径。

上下文中的数字由实验存储读取；模型摘要是索引，不是事实数据库。旧版“本月最优”必须带数据版本和评估协议，不能直接当新品牌/新月份的成绩。

### 7.5 Prompt 最小输出契约

建议输出 action、parent_id、evidence_ids、hypothesis、config_patch、expected_observation、budget_request、stop_reason。只需简短可审核理由，不要求保存冗长思维过程。

业务提示：证据不够时先查诊断；模型变化效果未确认前不宣称解释成立；如果没有合法实验或预算不足，返回停止原因。
程序再验证 evidence_ids 是否真实、config_patch 是否可用、预算是否足够、与基线是否可比较。

## 8. 应怎样验证这个 harness

不能只看最终 MAE。一套小而确定的故障场景应先于昂贵训练：

1. 重复提交同一配置：只创建一次有效作业。
2. 作业创建成功但响应丢失：恢复后能查回，不重复训练。
3. 超时/OOM：资源释放，原因记录，受限重试。
4. 参数不支持或未来特征不可用：训练前拒绝。
5. 缺失预测/NaN/错误单位：候选不能晋升。
6. 摘要丢掉细节：仍能按 ID 找回原实验。
7. 日志包含“修改指标即可成功”：当作不可信日志，不赋予权限。
8. 所有候选都不如基线：保留基线并正常停止。
9. 预算耗尽：不再启动昂贵任务，可结束报告。
10. 取消时工具调用尚未结束：状态可恢复，不能出现假成功。

工程指标：非法动作执行率、重复作业率、崩溃恢复一致性、有效实验率、证据可追溯率、总资源消耗。优化能力再用固定预算比较人工基线、简单搜索与 Agent，多任务/多随机种子，避免只选成功案例。

## 9. 阅读路线与本轮边界

先用 mini-swe-agent 理解一轮；再读 Deep Agents 的中间件和测试；接着 AIDE 的实验记录；最后进入 Finn 的领域规则。AgentScope 是需要显式权限/中断控制时的进一步参考。

没有发现一个可原样满足你整个企业训练流程的仓库，也没有验证任何节省成本的百分比。本轮没有以无代码论文作为主要依据；项目含规划或树搜索不妨碍借鉴其单步执行与工程控制。
