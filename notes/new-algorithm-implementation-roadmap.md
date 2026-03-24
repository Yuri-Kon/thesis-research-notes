# 面向高代价蛋白质设计工作流的自适应工具链规划算法：实现路线图

## 0. 文件目的

本文件用于回答一个更实际的问题：

**如何在现有系统基础上，分阶段实现“面向高代价蛋白质设计工作流的自适应工具链规划算法”。**

这里的目标不是立即给出全部编码细节，而是明确：

- 最小实现应做什么
- 中期增强应做什么
- 未来扩展应做什么
- 每一阶段涉及哪些模块
- 每一阶段能够做什么实验
- 每一阶段应产出什么结果

本路线图允许对 `Planner` 或系统架构做必要调整，但前提是：

- 调整必须服务于该算法
- 调整应尽量兼容现有 FSM、HITL 和恢复链
- 不为架构而架构

______________________________________________________________________

## 1. 当前系统已经具备的基础

现有系统并不是从零开始，已经具备实现该算法所需的若干关键基础。

### 1.1 已有规划基础

当前 `PlannerAgent` 已有：

- 候选工具链生成
- `plan_top_k / patch_top_k / replan_top_k`
- `score_breakdown`
- `risk_level / cost_estimate`
- `evaluate_top_k_gate`

对应代码主要在：

- `src/agents/planner.py`

### 1.2 已有运行时恢复基础

当前运行时已具备：

- `retry -> patch -> replan` 恢复闭环
- `WAITING_PATCH / WAITING_REPLAN`
- `PendingAction`
- `Decision Apply`

对应代码主要在：

- `src/workflow/plan_runner.py`
- `src/workflow/patch_runner.py`
- `src/workflow/decision_apply.py`
- `src/workflow/pending_action.py`

### 1.3 已有运行时观测基础

当前系统已能提供：

- `StepResult`
- `SafetyResult`
- `EventLog`
- `Snapshot`
- `WorkflowContext`

对应代码主要在：

- `src/models/contracts.py`
- `src/workflow/context.py`
- `src/workflow/snapshots.py`
- `src/storage/log_store.py`

### 1.4 已有实验统计基础

当前系统已具备：

- 垂直实验指标提取
- patch/replan 事件统计
- suffix replan 指标统计
- requirement2 相关聚合

对应代码主要在：

- `src/infra/w12_vertical_experiment.py`
- `tests/unit/test_w12_vertical_experiment.py`

因此，本算法的实现不应从“重做平台”开始，而应从：

**在现有规划与恢复架构上增加运行时状态估计与动作选择层**

开始。

______________________________________________________________________

## 2. 实现总原则

### 2.1 总体原则

实现阶段应坚持以下原则：

- 不推翻现有 FSM
- 不改变 Agent 边界
- 不重写整个 Planner
- 不做 full POMDP / full RL
- 先做 Lite 版本，再扩展

### 2.2 设计原则

该算法实现时应保持如下结构：

- 现有 `Planner` 继续负责候选生成和静态先验评分
- 新增的轻量状态估计模块负责运行时状态建模
- 新增的动作选择模块负责在 `continue / patch / replan / stop` 间决策
- `PlanRunner / PatchRunner` 负责执行这些动作对应的流程

换句话说：

- `Planner` 不应直接变成 controller
- controller-like 行为应以单独模块注入到当前工作流中

### 2.3 当前最重要的工程判断

对于当前毕设阶段，最稳妥的实现方式是：

**把该算法做成现有 Planner/Recovery 架构之上的轻量决策层。**

而不是：

- 新起一套工作流引擎
- 新定义一套状态机
- 大规模重构多 Agent 架构

______________________________________________________________________

## 3. 目标架构：建议的算法实现分层

建议将算法实现为四层。

### 3.1 Static Planning Layer

职责：

- 生成 `plan_top_k / patch_top_k / replan_top_k`
- 产出静态先验评分

当前基础：

- 已基本存在于 `src/agents/planner.py`

主要输出：

- 候选集合
- 静态 `score_breakdown`
- `risk_level / cost_estimate`

### 3.2 Runtime State Layer

职责：

- 定义轻量 belief-state
- 根据运行时观测更新状态

建议新增模块：

- `src/workflow/belief_state.py`
  或
- `src/workflow/adaptive_planning.py`

主要状态：

- `p_success`
- `p_structural_failure`
- `recovery_margin`
- `expected_remaining_cost`

### 3.3 Action Selection Layer

职责：

- 根据 belief-state 和当前上下文决定下一步动作

第一版动作空间：

- `continue`
- `patch_local`
- `suffix_replan`
- `stop`

后续可扩展：

- `expand_candidates`
- `shrink_candidates`
- `request_hitl`
- `full_replan`

### 3.4 Execution Integration Layer

职责：

- 把动作决策接入当前 `PlanRunner / PatchRunner`
- 保证状态转换、快照和日志仍然合法

当前接入点主要在：

- `src/workflow/plan_runner.py`
- `src/workflow/patch_runner.py`

______________________________________________________________________

## 4. Phase 0：实现前的系统基线整理

### 4.1 目标

在正式实现算法前，先把系统中与算法直接相关的基础接口和基线固定下来。

### 4.2 需要做的事情

- 审核现有 `Planner` 的静态评分组成
- 审核现有 `patch/replan` 触发路径
- 固定第一版实验基线
- 固定高代价步骤定义
- 固定第一版任务集

### 4.3 可能涉及的模块

- `src/agents/planner.py`
- `src/workflow/plan_runner.py`
- `src/workflow/patch_runner.py`
- `src/infra/w12_vertical_experiment.py`
- `tests/unit/test_planner_agent.py`

### 4.4 建议实验

只做基线实验，不加入新算法：

- 静态 Top-1 链路执行
- 固定阈值 gate
- 当前 patch/replan 恢复效果统计

### 4.5 产出

- 一份清晰的 baseline 指标
- 一份高代价步骤定义表
- 一份第一版任务集清单
- 一份待接入状态变量清单

### 4.6 这一阶段的意义

这一步不是浪费时间，而是为后续算法收益提供“干净的对照组”。

______________________________________________________________________

## 5. Phase 1：最小可运行算法版本（Lite）

### 5.1 目标

实现一个最小可运行版本，使系统第一次具备：

- 运行时状态估计
- 基于状态的 patch/replan/stop 选择

这一阶段是毕设主线的最小闭环。

### 5.2 核心实现内容

#### (1) 定义轻量 belief-state

建议新增：

- `BeliefState` 结构

建议字段：

- `p_success`
- `p_structural_failure`
- `recovery_margin`
- `expected_remaining_cost`

#### (2) 为 WorkflowContext 增加运行时状态

建议修改：

- `src/workflow/context.py`

新增可选字段，例如：

- `belief_state`

#### (3) 为快照恢复增加 belief-state 持久化

建议修改：

- `src/workflow/snapshots.py`
- `src/workflow/recovery.py`

持久化位置建议：

- `TaskSnapshot.artifacts["belief_state"]`

#### (4) 实现观测提取与状态更新

建议新增：

- 从 `StepResult` 提取状态更新信号
- 从 `SafetyResult` 提取风险更新信号
- 从恢复历史提取 `recovery_margin` 更新信号

#### (5) 在 PlanRunner 中接入动作选择

建议修改：

- `src/workflow/plan_runner.py`

将现有失败处理从“固定流程”升级为“状态驱动的动作选择”。

#### (6) 保留 Planner 静态评分，不重写

这一阶段不重写 `Planner` 的 `_score_payload`。
只将它作为静态先验，不做大的评分架构改造。

### 5.3 可能的代码改动范围

核心新增：

- `src/workflow/belief_state.py`

核心修改：

- `src/workflow/context.py`
- `src/workflow/snapshots.py`
- `src/workflow/recovery.py`
- `src/workflow/plan_runner.py`

较少修改：

- `src/models/contracts.py`
  只在必要时补充更明确的状态结构

### 5.4 这一阶段能做的实验

对比三类策略：

- 静态 Top-1
- 固定阈值动态 gate
- Lite belief-state 动态版本

指标：

- 成功率
- 平均执行成本
- 平均高代价步骤调用次数
- patch 次数
- replan 次数
- 恢复链长度

### 5.5 这一阶段的产出

- 一个可运行的 Lite 算法原型
- 一组最小对照实验结果
- 一批关键案例分析样本

### 5.6 对毕设的意义

如果这一阶段完成，已经足以支撑：

- 方法章节
- 核心实验章节
- 初步答辩主线

______________________________________________________________________

## 6. Phase 2：强化 Planner 与运行时的联合规划

### 6.1 目标

在 Lite 版本跑通后，把算法从“只在失败后决定 patch/replan”提升为：

**在候选排序阶段就显式考虑运行时状态与剩余代价。**

### 6.2 核心实现内容

#### (1) 将静态评分升级为“静态先验 + 运行时修正”

当前 `Planner` 的 `_score_payload` 基本是静态评分。

这一阶段可引入：

`final_score = static_score + runtime_adjustment`

运行时修正项由当前 belief-state 给出。

#### (2) 引入剩余成本和恢复复杂度修正

在候选排序时，不只看：

- 当前工具风险
- 当前工具成本

还看：

- 当前链的后缀剩余成本
- 当前链失败后的恢复复杂度

#### (3) 在 patch_top_k / replan_top_k 中引入 belief-state

不只是 plan 阶段使用状态估计，patch 和 replan 候选也应受到当前 belief-state 影响。

### 6.3 可能涉及的模块

- `src/agents/planner.py`
- `src/workflow/belief_state.py`
- `src/workflow/plan_runner.py`

### 6.4 这一阶段可做的实验

重点回答：

- belief-state 是否仅在恢复动作上有效，还是能改善初始候选排序
- “静态先验 + 运行时修正”是否优于纯静态评分

实验对照：

- 纯静态评分
- Lite belief-state 仅控制动作
- belief-state 修正评分 + 动作

### 6.5 这一阶段的产出

- 更完整的方法版本
- 更强的 ablation
- 更清晰的“算法作用机制”结论

### 6.6 对毕设的意义

这一阶段会显著增强论文中的算法含量。

如果 Phase 1 对应“可成立”，Phase 2 对应“更像核心算法”。

______________________________________________________________________

## 7. Phase 3：系统完备性增强

### 7.1 目标

在算法主线跑通之后，增强系统完备性，使算法依赖的状态、观测和执行闭环更稳。

这一阶段不是为了做新课题，而是为了保证算法有可靠载体。

### 7.2 建议增强方向

#### (1) 状态与观测的显式建模

当前状态和观测可能仍散落在：

- `StepResult.metrics`
- `SafetyResult`
- `EventLog`

这一阶段可考虑将其规范化，例如：

- 新增 `ObservationBundle`
- 新增 `BeliefUpdateInput`

让状态更新逻辑不再依赖临时字段拼装。

#### (2) 更强的快照恢复一致性

如果 `belief-state `成为运行时关键状态，那么等待和恢复后它必须保持一致。

这一阶段应重点验证：

- `WAITING`前 `belief-state`是否被持久化
- 恢复后 `belief-state` 是否能正确还原
- patch/replan 后 belief-state 是否与新计划兼容

#### (3) 更好的实验追踪与审计

建议增强：

- belief-state 变化日志
- 动作选择理由日志
- 候选排序修正日志

这有助于案例分析和答辩解释。

### 7.3 可能涉及的模块

- `src/workflow/context.py`
- `src/workflow/snapshots.py`
- `src/workflow/recovery.py`
- `src/infra/event_log_factory.py`
- `src/storage/log_store.py`

### 7.4 这一阶段可做的实验

重点做系统性实验：

- 恢复前后状态一致性测试
- WAITING/Decision/Resume 场景测试
- 长链路任务上的稳定性对比

### 7.5 这一阶段的产出

- 更完整的工程支撑
- 更可靠的运行日志与案例材料
- 更强的系统章节内容

### 7.6 对毕设的意义

该阶段的价值在于：

- 证明算法不是纸面方法
- 证明算法在真实工作流里可持续运行

______________________________________________________________________

## 8. Phase 4：未来扩展方向

这一阶段不一定要在当前毕设周期内全部完成，但应在路线图中给出清晰方向。

### 8.1 扩展动作空间

未来可考虑引入：

- `expand_candidates`
- `shrink_candidates`
- `request_hitl`
- `full_replan`

这会让算法从“恢复决策器”进一步走向“更完整的工作流规划器”。

### 8.2 扩展状态变量

未来可考虑加入：

- `intervention_value`
- `evidence_sufficiency`
- `toolchain_stability`
- `agent_coordination_risk`

这会使状态设计更接近完整 belief-state controller，但不应在第一版就纳入。

### 8.3 引入学习式更新

当前更适合显式规则或加权更新。

未来可考虑：

- 由历史运行日志学习状态更新权重
- 学习动作效用近似函数
- 做小规模 imitation / ranking 学习

这部分适合作为未来工作，而不是当前毕设的必要内容。

### 8.4 扩展实验范围

未来可扩展到：

- 更多任务难度分层
- 更多工具故障场景
- 更复杂的恢复链对比
- 跨任务类型迁移验证

### 8.5 对应产出

这一阶段的产出不必要求当前落地，但可在论文中写成：

- 后续工作方向
- 方法扩展空间
- 更强系统化研究路线

______________________________________________________________________

## 9. 建议的阶段性实验设计

为了让实现与论文收敛，建议每一阶段都绑定实验目标。

### Phase 0 实验目标

证明：

- 当前基线在高代价步骤上存在明显浪费
- 当前恢复链存在优化空间

实验输出：

- baseline 统计表
- 高代价步骤定义与占比

### Phase 1 实验目标

证明：

- Lite belief-state 动态策略优于静态或简单阈值策略

实验输出：

- 核心对照表
- 初步案例分析

### Phase 2 实验目标

证明：

- belief-state 不只是影响恢复动作，也能改善候选排序

实验输出：

- ablation 结果
- “静态先验 vs 运行时修正”对比

### Phase 3 实验目标

证明：

- 算法在等待、恢复、长链路场景下仍然稳定

实验输出：

- 稳定性与一致性实验
- 更完整的日志与案例

______________________________________________________________________

## 10. 每一阶段建议形成的文档与代码产出

### Phase 0

文档产出：

- baseline 分析文档
- 高代价步骤清单

代码产出：

- 少量实验统计增强

### Phase 1

文档产出：

- Lite 方法定义
- 初步实验结论

代码产出：

- `belief_state` 模块
- `WorkflowContext`/`snapshot`接入
- `PlanRunner` 动作选择接入

### Phase 2

文档产出：

- 强化版方法描述
- `ablation`结论

代码产出：

- `Planner` 的 `runtime-adjusted scoring`
- patch/replan 候选的状态感知排序

### Phase 3

文档产出：

- 稳定性分析
- 系统章节增强材料

代码产出：

- belief-state 审计日志
- 更强恢复一致性支持

______________________________________________________________________

## 11. 推荐的实施顺序

当前时间条件下，推荐采用以下顺序：

1. 先完成 Phase 0 的 baseline 固定
1. 再完成 Phase 1 的 Lite 闭环
1. 若时间允许，再做 Phase 2 的联合规划增强
1. 最后再做 Phase 3 的系统完备性增强

也就是说：

- **Phase 1 是必须完成的**
- **Phase 2 是增强论文算法性的关键**
- **Phase 3 是增强系统支撑性的关键**
- **Phase 4 是未来工作**

______________________________________________________________________

## 12. 最终实现判断

该算法的实现目标不应被误解为：

- 再造一个新的多 Agent 系统
- 再设计一套新的 FSM
- 完整实现 belief-state controller

更准确的目标应是：

**在现有蛋白质设计多 Agent 系统上，补足一个面向高代价工作流的运行时工具链规划层，使系统能够基于运行时观测更合理地进行继续执行、patch、suffix replan 或止损。**

因此，系统完备性确实是该算法的基础，但系统改造应始终服务于这一算法目标，而不应反客为主。
