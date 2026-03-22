# 面向毕设的 Controller 方向研究备忘

更新时间：2026-03-21

## 1. 文档目的

本文档用于收束此前多轮讨论，回答以下核心问题：

1. 当前毕设如果继续沿“生命科学多 Agent 助手”方向推进，是否会沦为另一生命科学助手毕设的补充或子集。
2. 如果转向 `controller`，它是否足以体现计算机专业毕设应有的算法性、创新性、工作量与可用性。
3. 如果考虑 `Belief-State Controller`，其算法家族、研究现状、优势、风险和创新点分别是什么。
4. 在本毕设语境下，应该如何正式定义该算法，而不是仅仅把它写成“一个更复杂的门控器”。

本文档不讨论具体实现细节，不给出编码方案，重点讨论研究问题、算法定义、创新边界、答辩风险和落地可行性。

---

## 2. 当前问题的本质

### 2.1 当前系统的问题并不是“功能太少”

当前系统实际上已经具备较强的工作流骨架能力，包括：

- 显式 FSM
- HITL 等待与恢复
- Planner / Executor / Safety / Summarizer 的职责分离
- patch / replan / suffix replan 恢复链
- 蛋白设计 S1-S4 分阶段流程
- 可复现实验管线

因此，当前短板不在于“没有系统”，而在于：

- 尚未形成一个足够独立、足够明确、足够可证明的核心算法问题；
- 现有 `risk_level / cost_estimate / score_breakdown` 更接近启发式评分与排序，不足以单独构成毕设核心算法；
- 只要系统的主叙事仍是“生命科学任务 -> 多 Agent -> 工具调用 -> 结果输出”，就很容易被另一“生命科学助手”毕设包住。

也就是说，真正需要变化的不是“多做几个功能”，而是**研究对象**。

### 2.2 为什么“加门控、加 HITL、加 patch/replan”仍然不够

原因在于这些机制虽然合理，但本质上仍属于：

- 执行安全增强
- 人工介入增强
- 工程鲁棒性增强

它们可以显著提升系统质量，但未必自动形成一个独立的算法贡献。  
如果另一个毕设是“生命科学助手”，那么它完全可以声称：

> 我也可以在我的助手内部加门控、加 HITL、加恢复逻辑。

这时你的工作很容易被视为：

- 父系统的 runtime 组件
- 某一类工作流执行器
- 或者父系统的底层增强模块

因此，若要从根本上规避“被包含”的风险，必须把课题主语从“生命科学助手”改成更底层、更通用、更算法化的问题。

---

## 3. 候选方向总览

从此前讨论来看，主要存在三个候选突破方向：

1. `Controller`
2. `Automatic Benchmark`
3. `Workflow Compiler / Verifier / Repair`

### 3.1 Controller

定义：

- 面向高代价科学工作流的风险、成本、不确定性和人工预算控制算法。

优势：

- 最容易与现有系统结合；
- 最容易体现“系统可用性”；
- 能与蛋白设计场景天然结合；
- 若问题定义正确，算法性可以很强。

风险：

- 如果只做成启发式门控，容易被质疑为规则系统；
- 如果仍然写成“蛋白设计多 Agent 助手中的一个模块”，则仍有可能被视为父集子模块。

### 3.2 Automatic Benchmark

定义：

- 自动生成、分层标定并自动评测高代价科学工作流任务的 benchmark。

优势：

- 最不容易成为“生命科学助手”的子集；
- 难度本身就是研究对象；
- 更容易形成独立研究资产。

风险：

- 自动生成和自动评分如果不够强，容易退化成“整理测试集”；
- 工程工作量较大；
- 直接的落地感与控制算法相比略弱。

### 3.3 Workflow Compiler / Verifier / Repair

定义：

- 从自然语言科研目标到可执行工作流图/DSL 的编译、验证、最小修复与可恢复执行。

优势：

- 形式化味道更强；
- 有 compiler / verification / synthesis 的 CS 风格。

风险：

- 容易和当前系统已有的 planner / patch / replan 机制重叠；
- 差异化不如 benchmark 明显；
- 仍有一定概率被视为“助手底层实现层”。

### 3.4 为什么最终更看好 Controller

综合来看，`controller` 的优势在于它同时满足：

- 可用性强
- 与现有系统天然兼容
- 具有算法升级空间
- 可以借助蛋白设计场景体现高代价与高风险

但要成立，controller 必须被上升为：

> 面向高代价科学工作流的控制算法

而不是：

> 蛋白设计助手内部的路由器

这一点是课题能否独立成立的关键。

---

## 4. 为什么 Controller 方向能够体现计算机专业毕设

### 4.1 因为它的对象不是“回答问题”，而是“控制执行”

生命科学助手的主问题通常是：

- 如何理解任务
- 如何分解任务
- 如何选工具
- 如何生成回答

而 controller 的主问题是：

- 当前工作流是否值得继续推进
- 继续推进的风险有多大
- 是否应该切换策略
- 是否需要人工介入
- 在预算、时间、风险约束下下一步该做什么

这已经从“对话/助手”问题，转成了：

- 调度问题
- 控制问题
- 约束决策问题
- 在线优化问题

这在计算机专业语境下是天然成立的。

### 4.2 因为它的动作空间更像系统治理动作

普通 agent 系统常见动作是：

- 生成下一条消息
- 调用一个工具
- 选择一个 agent

而 controller 的动作可以是：

- 继续执行
- 扩大候选探索
- 切换到保守工具链
- 进入 patch
- 直接 suffix replan
- 请求人工确认
- 提前终止任务

这意味着 controller 决定的不是“下一句说什么”，而是“系统接下来如何治理执行”。

### 4.3 因为它天然是多目标约束问题

controller 不追求单一指标，而通常同时优化：

- 成功率
- 任务质量
- 计算成本
- 时间消耗
- 人工介入次数
- 风险暴露
- 可追踪性

这非常符合计算机系统、控制与决策类毕设的典型风格。

---

## 5. Controller 的算法本质

controller 的本质，不是“给流程打个分”，而是：

> 在部分可观测、代价不对称、恢复受限、允许人工介入的工作流环境中，进行预算约束与风险约束下的序贯决策。

换句话说，它可以被理解成：

- `Constrained Sequential Decision Making`
- `Risk-aware Scheduling`
- `Budgeted Control`
- `Partially Observable Workflow Control`

这里有四个关键词非常重要：

### 5.1 部分可观测

系统并不能直接知道：

- 当前任务到底有多难
- 当前失败是不是偶发故障
- 当前候选质量是否真的足够
- 此时是否值得继续高成本结构预测
- 当前是否已经接近 recovery 空间耗尽

它只能通过运行时信号推断这些状态。

### 5.2 代价不对称

蛋白设计工作流中的不同阶段代价差异很大。  
某些步骤失败一次的代价很低，某些步骤进入错误路径后会导致高额无效计算或大量后续恢复。

### 5.3 恢复受限

系统虽然有 retry、patch、replan、HITL，但这些恢复手段不是无限的：

- 每次恢复都要付出代价；
- 恢复越多，往往意味着失败风险越高；
- 人工介入预算有限；
- 高频 replan 会破坏执行效率与稳定性。

### 5.4 序贯决策

controller 不是一次性决定全部流程，而是在执行过程中不断根据新证据更新判断。

因此，一个真正的 controller 必须是在线的、逐步更新的、面向未来代价与风险的。

---

## 6. 算法家族：Controller 可以属于哪些方法

若从方法论上看，controller 可以落在以下算法家族中。

### 6.1 启发式门控 / 规则控制

特点：

- 使用手工规则
- 使用阈值
- 使用打分后排序

优点：

- 简单、稳定、容易落地

缺点：

- 算法性弱
- 创新空间小
- 容易被认定为工程规则系统

结论：

- 可以作为 baseline
- 不适合作为毕设核心算法

### 6.2 Contextual Bandit / Online Routing

特点：

- 根据当前上下文在几个动作中选一个
- 更关注“此刻选什么”
- 不强调长期状态

优点：

- 比规则更强
- 比完整 RL 简单

缺点：

- 难以显式表达长期恢复成本
- 对多步工作流不一定足够

结论：

- 可作参考
- 适合局部 routing，但未必最适合完整工作流控制

### 6.3 MPC / Receding-Horizon Control

特点：

- 每一步只看未来若干步
- 用局部预测做滚动优化
- 执行后再重算

优点：

- 很适合 patch / replan / HITL 这类关键控制点
- 不需要完整环境建模
- 在线可更新

缺点：

- 需要定义未来若干步的近似代价与风险模型
- 难度更多来自预测模块和目标函数设计

结论：

- 是非常强的 controller 家族候选

### 6.4 POMDP / Belief-State Control

特点：

- 假设真实系统状态不可完全观测
- 通过观测维护对隐状态的 belief
- 根据 belief 做动作选择

优点：

- 非常适合“任务难度是隐变量”的叙事
- 能统一表示不确定性、失败风险、工具可靠性、恢复空间
- 算法上更有理论味道

缺点：

- 若做完整 POMDP 求解，难度非常高
- 若直接照搬理论，容易脱离实际

结论：

- `Belief-State Controller` 非常适合做毕设核心方向
- 但必须做“降阶版本”，不能追求完整最优控制求解

### 6.5 强化学习 / CMDP

特点：

- 学习策略
- 可显式处理约束与长期收益

优点：

- 理论上强

缺点：

- 数据需求高
- 训练不稳定
- 奖励设计困难
- 答辩时很容易被问到策略可信性和泛化问题

结论：

- 不建议作为本科毕设主线

---

## 7. 为什么重点讨论 Belief-State Controller

在所有 controller 家族中，`Belief-State Controller` 是最值得重点讨论的，原因如下：

### 7.1 它和“难度”天然匹配

此前讨论中最关心的问题之一是：

- 如何体现难度
- 如何体现工作量
- 如何避免只做出一个助手系统

`Belief-State` 最大的优势是：  
它可以把“难度”从一个静态标签，提升为一个运行时隐变量。

也就是说：

- 难度不是预先给定的
- 难度不是用户输入字段
- 难度也不是单次评分
- 难度是系统在执行过程中不断推断和更新的 latent state

这会让“难度建模”本身成为算法对象。

### 7.2 它比规则系统明显更强

如果最终系统只是：

- 打一个 difficulty score
- 超过阈值则进入 HITL
- 否则继续执行

那么本质还是规则系统。

但若 controller 维护 belief，并根据：

- 失败链
- 候选分歧
- 结构预测结果
- 质量门控拒绝原因
- 恢复历史
- 人工介入历史

不断更新系统对“未来风险/成本/收益”的判断，那么它已经远远超过普通规则系统。

### 7.3 它比 full RL / full POMDP 更可做

完整的 POMDP 或 RL 对本科毕设风险太高：

- 状态空间难以建模
- 数据与模拟环境不足
- 求解与训练复杂

而 `Belief-State Controller Lite` 可以做成：

- 显式状态变量
- 显式观测更新
- 显式动作选择

这样既保留算法深度，又避免过高风险。

---

## 8. 现状：Belief-State 不是创新点本身

这一点必须非常明确。

### 8.1 什么不是创新点

以下说法都不够：

- “提出了一个 belief-state controller”
- “引入了动态难度评估”
- “把 HITL 纳入控制流程”
- “在多 Agent 系统中考虑不确定性”

原因是：

- belief state 是经典控制与 POMDP 中已有概念；
- uncertainty-aware planning 在 agent 和 LLM 文献中已有工作；
- HITL 与门控也并非新概念。

因此，若直接把“用了 belief state”写成创新点，会很危险。

### 8.2 那么创新点应该落在哪里

真正可能成立的创新点主要来自以下四类：

#### (1) 新的问题定义

将高代价科学工作流中的多 Agent 执行形式化为：

> 一个带预算约束、风险约束、恢复动作和人工介入动作的部分可观测控制问题

这和普通 agent routing、普通对话控制、普通工具调用并不相同。

#### (2) 新的结构化隐状态设计

不是一个单分数，而是把隐状态拆成多个成分，例如：

- 任务内在难度
- 当前证据质量
- 工具可靠性状态
- 恢复空间剩余量
- agent 协作失配风险
- 人工介入的边际价值

这类结构化状态定义，才是真正的“科学工作流控制建模”。

#### (3) 新的观测更新机制

controller 接收的观测不是普通 token，而是异构运行时事件：

- S1 候选分布与分歧
- S2 结构投影结果
- S3 质量门控失败类型
- patch / replan 成败
- EventLog 中的恢复和等待链
- 决策与人类确认行为

若提出一种合理、可解释、可在线更新的 belief update 机制，这会很有算法价值。

#### (4) 新的治理动作空间

动作不是简单“选哪个工具”，而是 workflow governance action：

- 继续执行
- 增加探索深度
- 切换保守执行策略
- 局部 patch
- suffix replan
- 请求 HITL
- 提前终止

这会明显区别于传统 routing 或普通 tool-selection。

---

## 9. 对本毕设最合适的算法定义

下面给出一个适合本毕设使用的、足够正式但不过度实现化的算法定义。

### 9.1 问题对象

输入不是单轮问答，而是一个**高代价科学工作流**，可以形式化为：

- 一个有向工作流图 `G = (V, E)`
- 节点 `V` 表示阶段、工具调用、质量门控、恢复动作或人工确认点
- 边 `E` 表示数据依赖与控制依赖

对于蛋白设计场景，可以把工作流看作由：

- 候选生成
- 结构预测
- 质量控制
- 结构条件优化
- 恢复与人工确认

构成的多阶段执行链。

### 9.2 隐状态

对任意执行时刻 `t`，系统存在一个不可完全观测的隐状态 `z_t`，它刻画：

- 当前任务的真实执行难度
- 当前工具链的可靠性
- 当前观测证据的充分性
- 后续恢复空间是否仍然充足
- 是否值得继续投入更多计算或人工审查

`z_t` 不是单值，而是一个结构化状态向量。

### 9.3 可观测信息

系统在时刻 `t` 可获得观测 `o_t`，例如：

- 当前阶段输出质量
- 候选集合的一致性或分歧度
- 结构置信指标
- 质量门控拒绝原因
- 当前失败码
- 过往 patch / replan 轨迹
- 执行耗时和资源消耗
- 已发生的人工介入记录

### 9.4 Belief

系统维护一个 belief `b_t = P(z_t | o_{1:t}, a_{1:t-1})`，即：

> 在已有观测和历史动作条件下，对当前隐状态的估计

在实际毕设表述中，不必把它写成完整概率分布求解；更合理的写法是：

> 维护对“难度、风险、不确定性、恢复余量、人工价值”的联合信念表示

### 9.5 动作空间

controller 的动作空间 `A` 可以定义为工作流治理动作集合，例如：

- `continue`
- `expand_search`
- `switch_to_conservative_route`
- `patch_local`
- `suffix_replan`
- `request_hitl`
- `terminate`

注意：  
这些动作是系统治理动作，而不是普通工具调用动作。

### 9.6 目标

controller 的目标不是单一成功率最大化，而是：

> 在风险、成本、人工预算和时间约束下，最大化任务期望效用

效用可以综合：

- 任务成功概率
- 结果质量
- 计算成本
- 时间成本
- 人工介入次数
- 恢复链长度
- 治理与可追踪性指标

### 9.7 决策机制

controller 在每个关键控制点执行：

1. 根据新观测更新 belief；
2. 估计不同动作在未来若干步的收益、风险、成本；
3. 在约束条件下选择当前最优治理动作；
4. 推动系统进入相应分支；
5. 在新的执行结果出现后继续更新。

这个定义已经足以把问题从“增强版门控器”提升为“工作流控制算法”。

---

## 10. 该定义如何体现“难度、创新性、工作量”

### 10.1 难度如何体现

这里的“难度”不是人工打标，而是系统对未来执行不确定性的综合估计。  
它至少可以体现在三个层面：

#### (1) 内在任务难度

- 目标约束是否复杂
- 工作流链路是否长
- 对高成本步骤依赖是否强

#### (2) 运行时执行难度

- 当前证据是否不足
- 当前候选是否分歧过大
- 当前质量门控是否频繁拒绝
- 当前结构预测是否表现不稳定

#### (3) 恢复难度

- patch 是否仍有空间
- suffix replan 是否仍能保留有效前缀
- 人工介入是否能显著降低风险

这使“难度”从静态属性变成动态状态。

### 10.2 创新性如何体现

可成立的创新点可以压缩为以下几条：

1. 提出高代价科学工作流的部分可观测治理控制问题定义；
2. 提出面向科学工作流的结构化隐状态表示，而非单一难度评分；
3. 提出基于异构运行时事件的 belief update 机制；
4. 将控制动作定义为工作流治理动作，而非普通工具选择。

### 10.3 工作量如何体现

该方向的工作量并不轻，但它是“算法 + 系统验证”型工作量，而不是纯堆功能：

- 需要做问题形式化；
- 需要定义隐状态和观测；
- 需要定义 belief update；
- 需要定义控制目标与动作；
- 需要设计与基线的比较；
- 需要在高代价场景中证明收益。

这足以构成本科毕设级别的工作量。

---

## 11. 选择 Belief-State Controller 的权衡

### 11.1 选择它的理由

#### 理由 1：最容易把“难度”做成算法对象

如果做 benchmark，难度是评测对象；  
如果做 belief-state controller，难度是控制变量。  
就“难度如何体现在算法里”这一点来说，belief-state controller 是最自然的。

#### 理由 2：最容易与现有系统兼容

当前系统已经存在：

- 明确的状态机
- 明确的控制点
- 明确的恢复链
- 明确的人工确认点

因此 controller 可以自然映射到现有系统的运行时控制逻辑之上。

#### 理由 3：最容易兼顾可用性和算法性

benchmark 更独立，但 controller 更能直接回答实际问题：

> 什么情况下值得继续自动执行，什么情况下应该让人介入或改变策略？

这是高代价科学工作流中非常实用的问题。

### 11.2 不选择它的理由

#### 风险 1：很容易退化为规则系统

如果最后只做成：

- 一堆手工特征
- 一堆人工权重
- 一堆阈值

那么控制器会被认为是“复杂规则系统”，创新性会显著下降。

#### 风险 2：如果问题定义不够通用，仍可能被视为子模块

如果你把它表述为：

> 蛋白设计助手中的智能门控器

那么它还是可能被归入生命科学助手父集。

因此表述必须上升到：

> 高代价科学工作流控制算法，蛋白设计只是验证场景

#### 风险 3：如果试图做 full RL / full POMDP，工作量过大

这条路真正适合的是“降阶的、结构化的、可解释的 belief-state controller”，而不是端到端策略学习。

---

## 12. 与自动 Benchmark 的关系

是否需要把 controller 和 benchmark 结合起来，是此前一个重要问题。

结论是：

- 可以结合
- 但不应双核心并列

更合理的关系是：

> 以 controller 为主贡献，以 benchmark/实验框架为验证基座

原因：

- controller 提供主要算法创新；
- benchmark 提供难度分层和评测环境；
- 两者结合可以增强说服力；
- 但如果把两者都当主贡献，会导致范围过大、主线发散。

因此，从毕设管理角度，最合理的是：

- 主体：controller
- 支撑：benchmark 化评测

---

## 13. 它是否能落到当前系统中

结论：**可以，而且兼容性较高。**

原因在于当前系统已经具备 controller 所需的多个前提：

### 13.1 已有显式状态机

controller 需要知道系统处于哪个治理阶段，当前系统恰好已有显式状态机与 `WAITING_*` 状态。

### 13.2 已有候选、恢复、人工确认机制

controller 最关键的动作空间恰好与现有机制对应：

- 候选选择
- patch
- replan
- HITL
- 状态暂停与恢复

### 13.3 已有实验框架

controller 若要证明自己有效，需要实验对照；当前已有纵向实验框架和指标口径，这是非常重要的现成基础。

### 13.4 已有领域场景

蛋白设计工作流本身已经足够体现：

- 高代价
- 长链路
- 风险不对称
- 多阶段质量控制

这是 controller 非常合适的验证域。

因此，从“是否可落地”看，controller 不是空中楼阁，而是有现实落点的。

---

## 14. 该方向最适合写进开题/答辩的正式表述

下面给出一组适合后续整理为正式材料的表述模板。

### 14.1 研究问题表述

传统生命科学多 Agent 助手主要关注任务理解、工具调用与结果生成，但对于高代价科学工作流而言，核心难点还包括执行风险、恢复代价、人工介入时机与预算约束。现有系统虽具备门控、HITL 与恢复机制，但缺少一个将“任务难度与执行不确定性”纳入统一决策闭环的控制算法。因此，本研究关注的问题不是如何构建一个新的生命科学助手，而是如何面向高代价科学工作流设计一种难度感知、风险约束、可在线更新的控制方法。

### 14.2 核心算法表述

将高代价科学工作流形式化为部分可观测的治理控制问题，维护对任务难度、工具可靠性、证据充分性、恢复余量和人工介入价值的联合 belief，并基于运行时观测对 belief 进行在线更新，再在预算、风险和时间约束下选择治理动作，以实现对执行路径的动态控制。

### 14.3 创新点表述

1. 将多 Agent 科学工作流执行形式化为带恢复与 HITL 动作的部分可观测治理控制问题；
2. 提出面向高代价科学工作流的结构化隐状态表示，用于联合刻画难度、风险、恢复余量与人工介入价值；
3. 提出基于异构运行时事件的 belief update 机制，并将其用于工作流级治理动作选择。

### 14.4 差异化表述

本研究并不试图构建另一个生命科学助手，而是为高代价科学工作流提供一个底层控制层。生命科学助手可以作为该控制层的上层任务入口，而该控制层本身也可扩展到其他科学工作流场景，因此其研究对象和贡献边界不同于一般生命科学多 Agent 助手系统。

---

## 15. 最终判断

综合此前全部讨论，可以给出如下判断：

### 15.1 这条方向是否能体现计算机专业毕设

能。  
前提是其核心被定义为：

- 工作流控制
- 部分可观测决策
- 预算/风险约束
- 在线更新

而不是简单门控或启发式打分。

### 15.2 这条方向是否有创新空间

有。  
但创新不在“用了 belief state”，而在：

- 新问题定义
- 新隐状态设计
- 新观测更新机制
- 新治理动作空间

### 15.3 这条方向是否会成为生命科学助手的子集

如果表述不当，会。  
如果把研究对象上升为“高代价科学工作流控制算法”，则可以显著降低这一风险。

### 15.4 这条方向是否能落到当前系统

能，而且兼容性较高。  
当前系统已经具备控制器运行所需的 runtime 基础。

### 15.5 是否值得继续深入

值得。  
在当前几个候选突破方向中，controller 尤其是 `Belief-State Controller Lite`，是兼顾：

- 算法性
- 创新性
- 工作量
- 可用性
- 与现有系统兼容性

最有潜力的一条线。

---

## 16. 参考文献

以下文献和资料支撑了本文档中的判断，建议继续阅读。

### 16.1 Agent、评测与复杂任务

1. OpenAI. PaperBench: Evaluating AI's Ability to Replicate AI Research. 2025.  
   链接：https://openai.com/index/paperbench/

2. OpenAI. BrowseComp: a benchmark for browsing agents. 2025.  
   链接：https://openai.com/index/browsecomp/

3. OpenAI. Evaluating AI's ability to perform scientific research tasks. 2025.  
   链接：https://openai.com/index/frontierscience/

4. OpenAI. MLE-bench: Evaluating Machine Learning Agents on Machine Learning Engineering. 2024.  
   链接：https://openai.com/index/mle-bench/

5. Samuel Miserendino, Michele Wang, Tejal Patwardhan, Johannes Heidecke. SWE-Lancer: Can Frontier LLMs Earn $1 Million from Real-World Freelance Software Engineering? ICML 2025 Spotlight Poster.  
   链接：https://icml.cc/virtual/2025/poster/43573

6. Saaduddin Mahmud, Eugene Bagdasarian, Shlomo Zilberstein. CoLLAB: A Framework for Designing Scalable Benchmarks for Agentic LLMs. 2025.  
   链接：https://openreview.net/forum?id=372FjQy1cF

7. Zhenkun Li, Lingyao Li, Shuhang Lin, Yongfeng Zhang. Know the Ropes: A Heuristic Strategy for LLM-based Multi-Agent System Design. arXiv:2505.16979, 2025.  
   链接：https://arxiv.org/abs/2505.16979

### 16.2 多 Agent 失败、不确定性与控制

8. Zitian Chen, Yikang Shen, Yawen Kou, Jiahao Chen, Dongyan Zhao, Rui Yan. Why Do Multi-Agent LLM Systems Fail? arXiv:2503.13657, 2025.  
   链接：https://arxiv.org/abs/2503.13657

9. Lu Zhang, Corina S. Pasareanu, Kexin Pei, Yixuan Li, Na Zou, Mengdi Wang. Which Agent Causes Task Failures and When? PMLR 267, 2025.  
   链接：https://proceedings.mlr.press/v267/zhang25cq.html

10. Ming Lin, Yue Wang, Jiawei Liao, et al. QLASS: A Q-Guided Stepwise Search for Effective and Efficient LLM Agent. ICML 2025.  
    链接：https://web.cs.ucla.edu/~kwchang/bibliography/lin2025qlass/

11. Yuhang Yao, Huayu Chen, Shenyi Zhang, et al. Uncertainty of Thoughts: Uncertainty-Aware Planning Enhances Information Seeking of Language Agents. arXiv:2402.03271, 2024.  
    链接：https://arxiv.org/abs/2402.03271

12. Microsoft Research. EcoAct: Economic Agent Determines When to Register What Action. 2025.  
    链接：https://www.microsoft.com/en-us/research/publication/ecoact-economic-agent-determines-when-to-register-what-action/

13. Qian Jin, Justin K. Huang, et al. Echoing in Agent-to-Agent Communication: Identity Threats in LLM Multi-Agent Systems. 2025.  
    链接：https://openreview.net/forum?id=XNO4eQcAOc

### 16.3 蛋白设计与自主生物系统

14. Ziyi Zhou et al. Enhancing efficiency of protein language models with minimal wet-lab data through few-shot learning. Nature Communications, 2024.  
    链接：https://www.nature.com/articles/s41467-024-49798-6

15. Context-aware geometric deep learning for protein sequence design. Nature Communications, 2024.  
    链接：https://www.nature.com/articles/s41467-024-50571-y

16. Xiao-Yu Guo et al. Ab-initio amino acid sequence design from protein text description with ProtDAT. Nature Communications, 2025.  
    链接：https://www.nature.com/articles/s41467-025-65562-w

17. Nilmani Singh et al. A generalized platform for artificial intelligence-powered autonomous enzyme engineering. Nature Communications, 2025.  
    链接：https://www.nature.com/articles/s41467-025-61209-y

---

## 17. 后续可补充内容

本文档已适合作为后续开题、选题论证和答辩准备的研究备忘。  
后续若需要，可继续补充：

- 面向答辩的“选题风险对比表”
- “为什么不是另一个生命科学助手”的专门论证稿
- belief-state controller 的更正式数学化定义
- 适合作为 baseline 的方法族对比
- 对现有系统能力与 controller 接口的映射说明

---

## 18. 再次澄清：Controller 到底是什么、最终要做什么、前人做过什么

本节用于进一步回答一个最核心、也最容易混淆的问题：

> controller 到底是什么？我最后究竟要做一个什么东西出来？它的创新和难点到底在哪里？前人是否已经做过？

### 18.1 Controller 不是另一个 Agent，也不是普通门控器

本文语境中的 controller 不是：

- 一个额外的对话 Agent
- 一个普通的工具路由器
- 一个只负责“打分后决定是否继续”的阈值模块
- 一个生命科学助手里的通用工程增强插件

它更准确的定义应当是：

> 一个面向高代价科学工作流的运行时治理控制器

这里的“治理控制”强调的是：

- 它控制的是整条 workflow 的推进策略，而不是一条回复文本
- 它决定的是系统级动作，而不是某个单步 prompt
- 它服务于高成本、长链路、可失败、可恢复、可人工介入的执行过程

因此，它真正回答的问题不是“下一句说什么”，而是：

- 当前是否值得继续执行
- 是否需要扩大搜索
- 是否应该转向保守路径
- 是 patch 还是 suffix replan
- 是否应让人介入
- 是否应在代价进一步扩大前提前终止

从这个角度看，controller 更像：

- 工作流调度器
- 风险感知控制器
- 预算约束决策器
- 运行时治理层

而不是一个新的助手系统。

### 18.2 最终要做出来的“东西”是什么

如果这一方向作为毕设成立，那么最终产物不应是“一个新助手”，而应当是：

> 一套可嵌入多 Agent runtime 的工作流控制方法与控制器原型

这个“东西”至少应包含三部分：

#### (1) 正式的问题定义

需要明确把科学工作流建模成一个怎样的控制问题，例如：

- 工作流图
- 隐状态
- 可观测事件
- 动作空间
- 目标函数
- 约束条件

没有这一层，系统很容易退化成工程规则。

#### (2) 控制算法

控制算法的职责不是自己完成任务，而是：

- 读取运行时观测
- 更新对当前执行状态和未来风险的 belief
- 在若干治理动作中做决策
- 推动 runtime 进入更合理的执行分支

因此，它本质上是**运行时决策层**。

#### (3) 实验与验证结果

必须证明：

- 相比静态流程、阈值规则或固定策略
- controller 在成本、成功率、恢复链长度、人工介入次数等维度具有更优权衡

所以最终交付物的本质是：

> 一种控制方法 + 一个控制器原型 + 一组相对于基线更优的验证结果

而不是单纯的 demo 页面或功能堆叠。

### 18.3 前人有没有做过

答案是：**做过相关思想，但没有直接等于本课题的完整问题。**

这点必须客观面对。  
如果最终选 controller，不能声称“从零提出了控制思想”，而应说明：

- 相关方法家族已经存在
- 但面向高代价科学工作流、带恢复链与 HITL 动作的控制问题仍有明显空白

下面按最相关的工作脉络说明。

#### 18.3.1 不确定性感知规划

`Uncertainty of Thoughts: Uncertainty-Aware Planning Enhances Information Seeking of Language Agents`（UoT, 2024）研究的是：

- 当任务信息不完整时
- agent 如何根据不确定性主动提问，以降低未来决策风险

它的重要启发在于：

- 不确定性不是只能被动承受
- 不确定性可以进入决策闭环
- 决策动作可以被“减少未来不确定性”这一目标驱动

这说明：

> uncertainty-aware planning 已经是成立方向

但 UoT 的主要场景是信息寻求与提问，不是高代价科学工作流中的恢复控制、人工介入与工作流治理。

参考：

- Yuhang Yao, Huayu Chen, Shenyi Zhang, et al. *Uncertainty of Thoughts: Uncertainty-Aware Planning Enhances Information Seeking of Language Agents*. arXiv:2402.03271, 2024.  
  链接：https://arxiv.org/abs/2402.03271

#### 18.3.2 成本感知路由与 test-time compute 控制

`BEST-Route: Adaptive LLM Routing with Test-Time Optimal Compute`（ICML 2025）研究的是：

- 根据 query 的难度与质量阈值
- 动态决定调用哪个模型、采样多少次
- 在成本和质量之间做更优权衡

它证明了：

- “根据难度分配计算资源”是一个有意义且可量化的问题
- 动态 compute allocation 可以显著降低成本

但它仍然主要是：

- 模型路由问题
- 单 query 层面的问题
- 不涉及工作流级恢复、patch/replan 和 HITL

参考：

- Dujian Ding, Ankur Mallick, Shaokun Zhang, et al. *BEST-Route: Adaptive LLM Routing with Test-Time Optimal Compute*. ICML 2025.  
  链接：https://proceedings.mlr.press/v267/ding25d.html

另一个相关方向是 `EcoAct`，其关注点是：

- 在 agentic 系统中
- 何时注册哪些工具或动作
- 通过经济性决策减少冗余开销

它同样说明：

> 动作选择应当受成本约束，已经是前沿 agent 研究中的明确方向

但它距离工作流级治理控制仍有明显差距。

参考：

- Microsoft Research. *EcoAct: Economic Agent Determines When to Register What Action*. 2025.  
  链接：https://www.microsoft.com/en-us/research/publication/ecoact-economic-agent-determines-when-to-register-what-action/

#### 18.3.3 长程价值引导的 Agent 搜索

`QLASS: Boosting Language Agent Inference via Q-Guided Stepwise Search`（ICML 2025）研究的是：

- 为 agent 在复杂交互任务中的每一步提供价值引导
- 使 agent 在 test-time search 中做更优中间选择

这说明：

- 不只是最终答案，而是中间决策质量也很重要
- stepwise value / Q-value 可以进入 agent 推理过程

但 QLASS 主要关注：

- 交互任务中的搜索质量
- 并非工作流级治理动作
- 也没有直接面对 patch / replan / HITL 这类恢复与治理动作

参考：

- Zongyu Lin, Yao Tang, Xingcheng Yao, Da Yin, Ziniu Hu, Yizhou Sun, Kai-Wei Chang. *QLASS: Boosting Language Agent Inference via Q-Guided Stepwise Search*. ICML 2025.  
  链接：https://proceedings.mlr.press/v267/lin25l.html

#### 18.3.4 多 Agent 失败模式与失败归因

`Why Do Multi-Agent LLM Systems Fail?`（2025）系统性总结了多 Agent 系统的失败原因，包括：

- 规范和接口不一致
- agent 之间职责错位
- 缺乏有效验证
- 终止条件设计不良
- 错误在多 agent 链路中被放大

这篇工作的重要意义在于：

- 多 Agent 系统的难点不只是模型能力
- 还包括系统设计、通信、验证和治理

参考：

- Zitian Chen, Yikang Shen, Yawen Kou, Jiahao Chen, Dongyan Zhao, Rui Yan. *Why Do Multi-Agent LLM Systems Fail?* arXiv:2503.13657, 2025.  
  链接：https://arxiv.org/abs/2503.13657

`Which Agent Causes Task Failures and When?`（PMLR 267, 2025）进一步表明：

- 在多 Agent 执行链中
- 自动判断究竟是谁导致失败，以及在何时引入了关键错误，并不容易

这意味着 controller 面临的不是普通单模型错误，而是：

- 难以定位来源的协作失败
- 带历史依赖和责任不清的链式失败

参考：

- Lu Zhang, Corina S. Pasareanu, Kexin Pei, Yixuan Li, Na Zou, Mengdi Wang. *Which Agent Causes Task Failures and When?* Proceedings of Machine Learning Research 267, 2025.  
  链接：https://proceedings.mlr.press/v267/zhang25cq.html

### 18.4 所以本毕设还能有什么创新

由于前人已经做过 uncertainty-aware planning、cost-aware routing、stepwise value guidance 和 MAS failure analysis，因此：

> 本毕设的创新点绝不能写成“使用了 belief state / uncertainty / routing / 控制”

这类说法都过于泛。

真正可能成立的创新，应落在以下层面。

#### 18.4.1 新的问题定义

把对象定义为：

> 高代价、长链路、带恢复动作和 HITL 动作的科学工作流治理控制问题

这里的特殊性在于它同时包含：

- 显式 workflow graph
- 代价不对称
- 失败后的恢复链
- 人工介入动作
- 长期成本与长期收益的权衡

这不同于普通 query routing，也不同于普通 agent 搜索。

#### 18.4.2 新的状态定义

真正强的 controller 不能只有一个 difficulty score。  
它需要有结构化 latent state，例如：

- 当前任务真实难度
- 当前证据充分性
- 当前工具链稳定性
- 当前恢复空间余量
- 当前人工介入边际价值
- 当前协作失配风险

状态设计是算法深度最集中的地方之一。

#### 18.4.3 新的观测更新机制

controller 看到的观测不是普通自然语言输入，而是异构运行时事件：

- 候选分歧程度
- 结构预测质量
- 质量门控失败码
- patch / replan 结果
- 已消耗预算
- 已发生 HITL 决策

若提出一种合理的 belief update 机制，将这些异构事件转化为系统对未来风险和收益的估计，那么这将是非常核心的算法贡献。

#### 18.4.4 新的动作定义

工作流 controller 的动作不应只是“换个模型”或“选个工具”，而应是治理动作，例如：

- `continue`
- `expand_search`
- `switch_to_conservative_route`
- `patch_local`
- `suffix_replan`
- `request_hitl`
- `terminate`

这一动作空间与现有很多 routing 类工作并不相同。

### 18.5 难点体现在哪里

如果选 controller，这个方向的真正难点不在工程接线，而在以下算法问题：

#### 难点 1：难度是隐变量，无法直接观测

系统不能直接知道：

- 当前任务是不是已经进入高风险状态
- 当前失败究竟是偶发故障还是结构性困难
- 继续投入更多计算是否值得

因此必须做隐状态估计。

#### 难点 2：观测是异构且带噪声的

不同阶段提供的观测信息完全不同，且不一定稳定：

- 候选分歧和结构预测质量本身带噪声
- 错误码不一定能完全反映真实根因
- 人工决策记录也不一定完全一致

#### 难点 3：动作代价强烈不对称

一次无意义的高成本结构预测、一次错误的 replan、一次过早或过晚的 HITL，都会造成明显不同的代价后果。

#### 难点 4：局部最优可能伤害全局

当前看似省成本的动作，可能导致后续恢复链变长、整体风险提高，甚至最终造成更大浪费。

#### 难点 5：多 Agent 特有失败会污染控制信号

多 Agent 系统会引入：

- 角色漂移
- 接口不匹配
- 错误归因困难
- 验证不足

这些都意味着 controller 面对的不是干净环境，而是带协作噪声的环境。

### 18.6 最终一句话定义

如果要把这个方向压缩成一句最准确的话，可以写成：

> 本研究不试图构建另一个生命科学助手，而是为高代价科学工作流提供一个运行时治理控制器；该控制器根据执行过程中的不确定性、成本、失败信号和人工介入价值，动态决定工作流下一步如何推进、恢复或终止。

这句话比“做一个更聪明的多 Agent 生物助手”要强得多，也更能避免沦为父集子集。
