# 回答问题

## 研究层面

### 主问题到底是什么

如何在高代价蛋白质设计工作流中，基于运行时动态规划工具链，以降低无效高代价调用并提高整体任务成功率。

- 主目标：降低无效高代价调用，同时保持或提升成功率
- 次目标：减少恢复链长度，减少不必要人工介入

### 核心研究假设是什么

1. 蛋白质设计工作流中存在明显的高代价步骤，错误决策会导致显著额外开销
1. 运行时中间观测对后续成功率、风险和恢复空间具有预测价值
1. 与静态工具链相比，动态规划和运行时修正能带来更优的成本-成功率权衡
1. 轻量 belief-state 能比固定阈值更好地表达当前链路状态，但不需要完整POMDP

### 课题边界

- 研究对象：`Planner` + `runtinme adaptation`
- 纳入范围：
  - 候选工具链生成
  - 候选评分与裁剪
  - 运行时状态更新
  - `patch/suffix_replan/stop` 决策
- 可利用但不作为主要研究对象：
  - `Safety` 输出
  - HITL记录
  - `EventLog`
- 不纳入主线：
  - 新工具开发
  - 新FSM设计
  - 完整HITL交互机制设计
  - 端到端`RL/Full POMDP`

### 与 belief-state 的关系

本文提出的是 “自适应工具链规划算法”，其中引入轻量的 `belief-state` 作为运行时状态估计模块。

### 创新点

1. 问题形式化创新：将蛋白质设计任务从“小工具集检索问题”提升为“高代价、可恢复、多阶段工作流中动态工具链规划问题”
1. 方法创新：提出带轻量 `belief-state` 的运行时工具链规划方法，用于联合估计成功概率、结构性风险和恢复空间
1. 决策机制创新：提出恢复感知的动作选择机制，使系统能在`continue/patch/suffix_replan/stop`之间做出更合理的工具流级决策。

### 主要的研究风险

1. 算法容易退化成规则系统
   回应：显式区分状态、观测、动作和效用，不把方法写成单阈值门控
1. 观测不足，`beleif-state`没有明显收益
   回应：采用Lite版本，只保留少量高价值状态变量，并与“不带 `belief-state` 的动态版本”做对照
1. 工具空间太小，算法价值不明显
   回应：强调研究对象不是工具检索，而是高代价工作流中的动态规划、恢复和止损决策

______________________________________________________________________

## 结论层面

### 最终主结论

相较于静态工具链执行策略，带轻量 `belief-state` 的自适应工具链规划方法，能够在高代价蛋白质设计工作流中实现更优的“成功率-成本-恢复复杂度”权衡

### 证明的核心命题

1. 动态规划优于静态规划
   即：运行时修正工具链，比“一次规划后执行到底”更合理
1. 轻量 `belief-state` 优于简单阈值门控
   即：显式状态估计比“看指标超阈值就切换动作”更稳定

### 哪些结果可接受

A：

- 成功率相近
- 但平均成本显著下降、无效高代价调用减少

B：

- 成功率略提升
- 成本基本持平
- 恢复链更短

C：

- 整体平均不大
- 但在高难度任务上明显更优

### 结果分析

1. 总体指标

   - 成功率
   - 平均总成本
   - 平均时间
   - 高代价步骤调用次数
   - 平均 patch/replan 次数

1. 分层分析

   - 简单任务
   - 中等任务
   - 高难任务

1. 案例分析

   - 一个成功案例说明算法如何止损或修正
   - 一个失败案例说明算法哪里判断错了
   - 一个对照案例说明静态方法错，自适应方法对

### 基线对应结论

- 静态 Top-1 工具链：用来支持“动态规划有必要”
- 固定阈值门控：用来支持“简单规则不够”
- 不带 `belief-state`的动态版本：用来支持“状态估计确实有价值”

______________________________________________________________________

## 实现层面

### 实现总原则

不应重写现有 `Planner/FSM` 架构，而应在当前静态候选排序与恢复闭环之上，增加一层轻量的运行时状态估计与动作选择。

实现上的基本判断是：

- 现有系统已经有 `plan_top_k / patch_top_k / replan_top_k`
- 已经有 `score_breakdown / risk_level / cost_estimate`
- 已经有 `patch/replan` 执行闭环
- 已经有 `PendingAction`、`StepResult`、`SafetyResult` 和实验统计框架

因此缺的不是新的执行平台，而是：

**一个把运行时观测转化为状态估计，并进一步修正候选排序和恢复动作选择的轻量算法层。**

### 现有代码和设计中可直接复用的部分

结合当前代码结构，可直接复用的模块包括：

- `src/agents/planner.py`
  - 已有候选生成
  - 已有静态打分 `_score_payload`
  - 已有 `risk_level / cost_estimate`
  - 已有 `evaluate_top_k_gate`
- `src/workflow/plan_runner.py`
  - 已有失败摘要
  - 已有 `patch/replan` 闭环
  - 已有 `WAITING_PATCH / WAITING_REPLAN` 流程
- `src/workflow/context.py`
  - 已有运行时上下文对象 `WorkflowContext`
- `src/models/contracts.py`
  - 已有 `PendingActionCandidate.metadata`
  - 已有 `TaskSnapshot.artifacts`
- `src/infra/w12_vertical_experiment.py`
  - 已有实验指标提取逻辑
  - 已有恢复链相关统计

这意味着算法实现应该采取“增量插入”的方式，而不是推翻现有代码。

### 最小可实现版本

Lite 版本建议只实现以下四部分：

1. 候选链生成  
   直接复用现有 `PlannerAgent.plan_top_k / patch_top_k / replan_top_k`

1. 多目标静态评分  
   直接复用现有 `_score_payload` 作为**静态先验分**

1. 轻量 belief-state 更新  
   新增一个运行时状态估计模块，只维护少量关键状态

1. 关键动作选择  
   第一版只支持：
   - `continue`
   - `patch_local`
   - `suffix_replan`
   - `stop`

其余动作如 `expand_candidates / shrink_candidates / request_hitl / full_replan` 暂不作为第一版核心。

### belief-state 最稳妥的代码落点

最稳妥的做法是双落点：

- 运行中放在 `WorkflowContext`
- 持久化时写入 `TaskSnapshot.artifacts`

也就是说：

- 在 `src/workflow/context.py` 中新增一个可选字段，例如 `belief_state` 或 `runtime_state`
- 在快照构建和恢复过程中，将该状态写入和读出 `TaskSnapshot.artifacts["belief_state"]`

这样做的原因是：

- 它属于运行时状态，不适合写入 `Plan.metadata`
- 它又需要在等待和恢复之后保留

### 最小状态集

第一版不应纳入过多状态变量。

建议只保留：

- `p_success`
- `p_structural_failure`
- `recovery_margin`
- `expected_remaining_cost`

这一版先不把 `intervention_value` 作为强主状态，除非后续明确把 `HITL` 也纳入主要动作空间。

### 建议新增的实现模块

建议新增独立模块，而不是把所有逻辑直接写进 `planner.py`：

- `src/workflow/belief_state.py`
  或
- `src/workflow/adaptive_planning.py`

该模块负责：

- 定义轻量 belief-state 数据结构
- 从 `StepResult / SafetyResult / budget usage` 提取观测
- 根据观测更新 belief-state
- 根据 belief-state 产出动作建议

建议包含的接口可以是：

- `build_initial_belief(...)`
- `update_belief_from_step_result(...)`
- `update_belief_from_safety_event(...)`
- `choose_runtime_action(...)`

### 与 Planner 的结合方式

`src/agents/planner.py` 中现有 `_score_payload` 应保留，不应重写。

原因是它已经提供了一套稳定的静态先验评分，包括：

- feasibility
- objective
- risk
- cost
- confidence
- fallback_depth

更合理的实现方式是：

- `_score_payload` 负责回答“这条链先验上看起来怎么样”
- `belief_state` 负责回答“在当前运行到这里时，这条链是否仍值得继续”

因此更推荐的组合方式是：

`final_score = static_score + runtime_adjustment`

而不是彻底替换原有评分器。

### 与 PlanRunner 的结合方式

`src/workflow/plan_runner.py` 已经有适合插入决策逻辑的位置：

- `_summarize_failure_result`
- `_request_replan`
- `_perform_replan`
- 失败后的 patch/replan 升级路径

第一版最适合在这里增加动作选择层：

- 若 `p_structural_failure` 高且 `recovery_margin` 低：倾向 `suffix_replan`
- 若 `p_success` 尚可且失败表现为局部问题：倾向 `patch_local`
- 若 `expected_remaining_cost` 高且 `p_success` 低：倾向 `stop`

这样改动最小，也最贴近当前研究主问题。

### 成本 schema 的第一版落地方式

成本不应一开始做成复杂模型。

第一版可以采用：

- 先验成本：直接使用 `ToolSpec.cost`
- 在线成本：真实步骤耗时、重试次数、是否进入 patch/replan
- 剩余成本：当前后缀步骤成本的近似求和

因此第一版可写成近似：

`expected_remaining_cost = suffix_tool_cost_sum + recovery_penalty`

这已经足以支撑 Lite 版本。

### 观测来源的第一版范围

第一版只使用最可靠的三类来源：

- `StepResult`
  - `metrics`
  - `outputs`
  - `error_details`
- `SafetyResult`
- 当前上下文中的恢复历史
  - `step_results`
  - `safety_events`
  - 是否已发生 patch

不建议在第一版就依赖复杂日志挖掘或额外外部信号。

### 状态更新策略

当前时间条件下，更现实的不是完整概率模型，而是：

- 显式规则更新
或
- 分层加权更新

不建议第一版就做 full POMDP 或学习式策略更新。

关键是：

- 状态更新规则可解释
- 状态与动作的关系可验证

### 测试实现建议

最低限度应补以下几类测试：

1. `tests/unit/test_planner_agent.py`
   - 验证静态分 + 运行时修正后的候选排序

1. `tests/unit/test_plan_runner.py`
   - 验证 belief-state 驱动的 `patch / suffix_replan / stop`

1. `tests/integration/test_candidate_score_gate.py`
   - 验证 gate 不再只依赖固定 `confidence/cost`

1. `tests/integration/test_s6_control_layer_e2e.py`
   - 验证接入新状态后控制层闭环仍正常

1. `tests/unit/test_w12_vertical_experiment.py`
   - 验证新增实验指标统计

### 实验与基线实现建议

实验层面应尽量复用现有框架，尤其是：

- `src/infra/w12_vertical_experiment.py`

建议新增统计：

- 高代价步骤调用次数
- 平均恢复链长度
- `patch / suffix_replan / stop` 的触发分布
- belief-state 版本与无状态动态版本的差异

对应基线实现建议：

- 静态 Top-1 工具链
- 固定阈值门控
- 不带 belief-state 的动态版本

### 实现顺序建议

更合理的实现顺序是：

1. 定义 Lite belief-state
1. 在 `WorkflowContext` 和 `TaskSnapshot.artifacts` 中打通该状态
1. 在 `PlanRunner` 中接入基于状态的动作选择
1. 在 `Planner` 中加入“静态先验 + 运行时修正”的组合评分
1. 最后补基线与实验统计

这样做的原因是：

- 先让状态和动作闭环跑起来
- 再去优化候选排序
- 最后再做实验验证

### 实现层面的最终指导

实现阶段最重要的不是“再设计一个新系统”，而是：

**把它做成现有 Planner/Recovery 架构之上的轻量决策层。**

也就是说，后续实现应始终坚持：

- 不推翻原有架构
- 不扩张到新的系统设计
- 不追求 full controller
- 只补足运行时状态估计与动作选择这一层

这是当前时间条件下最稳、也最符合毕设主线的实现方案。
