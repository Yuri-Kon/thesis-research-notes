# 摘要：面向高代价蛋白质设计工作流的自适应工具链规划算法

## 1. 核心定义

该算法不是简单的 ToolKG 检索算法，也不是“从几个工具里选一个工具”的问题。

它的定义是：

**在高代价、长链路、可失败、可恢复的蛋白质设计工作流中，动态生成、评估、裁剪并修正工具链，以更低成本获得更高任务成功率。**

它本质上是一个**工作流级动态规划器**。

---

## 2. 它解决的问题

蛋白质设计领域的工具数量较少，因此“工具检索”本身很难构成核心算法。

该算法真正要解决的是：

- 如何把少量工具组合成可执行的多阶段工具链
- 如何在成本、风险和目标约束下选择更值得尝试的链路
- 如何在中间结果变差时决定继续、patch、replan 或止损
- 如何减少一次错误高代价调用引起的后续浪费

因此，该算法研究的是：

**高代价蛋白质设计工作流中的工具链规划与运行时修正问题。**

---

## 3. 输入、输出与优化目标

### 输入

- 蛋白质设计目标 `g`
- 结构化约束 `c`
- 工具能力图 `K`
- 当前执行上下文 `x_t`
- 当前运行时观测 `o_t`

### 输出

- 当前候选工具链集合 `Pi_t`
- 默认推荐链 `pi*`
- 或局部修正 `patch`
- 或后缀/整体重规划 `replan`

### 优化目标

- 提高最终任务成功率
- 降低总执行成本
- 降低无效高代价调用次数
- 降低恢复复杂度
- 减少不必要人工介入

可用一个多目标评分形式概括：

`Score(pi) = alpha * Feasibility + beta * GoalFit - gamma * Cost - delta * Risk - eta * RecoveryComplexity`

其中：

- `Feasibility` 是硬约束
- `Cost`、`Risk`、`RecoveryComplexity` 是联合决策变量

---

## 4. 算法的核心组成

### 4.1 候选工具链生成

基于任务约束、阶段结构和工具能力图生成若干可执行候选链，而不是直接确定一条链。

### 4.2 多目标评分

对候选链进行联合排序，至少考虑：

- 可执行性
- 目标匹配度
- 成本
- 风险
- 恢复复杂度

### 4.3 预算感知裁剪

根据预算、风险和当前状态决定：

- 保留多少候选链
- 是否展开更深后续步骤
- 是否先做廉价验证再继续高代价步骤

### 4.4 运行时自适应修正

根据新的中间观测，动态选择：

- `continue`
- `patch`
- `replan`
- `stop`

### 4.5 恢复链集成

失败是算法的一部分，而不是异常情况。恢复闭环为：

`retry -> patch -> replan`

---

## 5. “自适应”具体指什么

该算法的自适应主要体现在三个方面：

- 候选数量自适应：简单任务少展开，复杂任务多保留候选
- 搜索深度自适应：稳定时继续推进，信号变差时提前验证或止损
- 恢复范围自适应：局部问题优先 `patch`，结构性问题优先 `replan`

因此它是：

**budget-aware + risk-aware + recovery-aware planning**

---

## 6. belief-state 的角色

该算法最好内置一个**轻量 belief-state 模块**，但不应直接等同于完整的 belief-state controller。

在本算法中，belief-state 的角色是：

**内部的状态估计模块**

可维护的关键估计量包括：

- `p_success`
- `p_structural_failure`
- `recovery_margin`
- `expected_remaining_cost`
- `intervention_value`

它们用于影响：

- 候选排序
- 候选裁剪
- patch/replan 决策
- 是否提前止损

因此关系应理解为：

- 主算法：自适应工具链规划算法
- 子模块：轻量 belief-state 估计

---

## 7. 六个算法前置定义

若要让该算法成立为核心算法，必须明确以下六个 schema。

### 7.1 Cost Schema

回答：

- 代价由哪些部分构成
- 代价如何确认
- 代价如何归一化

至少包括：

- `compute_cost`
- `time_cost`
- `opportunity_cost`
- `recovery_cost`
- `human_cost`

建议来源：

- `prior_cost`
- `historical_cost`
- `online_cost`

### 7.2 Risk Schema

回答：

- 风险是什么
- 风险与成本如何区分
- 风险如何从观测中估计

风险至少反映：

- 当前链路失败概率
- 当前链路低质量结果概率
- 当前动作引发结构性失败的概率
- 当前动作扩大风险暴露的可能性

### 7.3 Recovery Schema

回答：

- 当前失败后还能否恢复
- 恢复代价多高
- 原有有效前缀还能保留多少

至少包括：

- `retry_budget`
- `patch_feasibility`
- `suffix_replan_preservability`
- `recovery_complexity`
- `human_rescue_value`

### 7.4 State Schema

回答：

- 哪些变量属于隐状态，而不是直接观测值

典型隐状态：

- `task_difficulty`
- `evidence_sufficiency`
- `toolchain_stability`
- `structural_failure_tendency`
- `recovery_margin`
- `intervention_value`

### 7.5 Observation Schema

回答：

- 系统能看到哪些观测
- 这些观测如何更新隐状态

可观测信号包括：

- 结构预测质量指标
- QC/gate 失败码
- 候选分歧
- patch/replan 历史
- 时间和预算消耗
- Safety 触发信息
- 人工确认记录

### 7.6 Action-Utility Schema

回答：

- 动作空间有哪些
- 如何比较动作价值

动作空间可包括：

- `continue`
- `expand_candidates`
- `shrink_candidates`
- `patch_local`
- `suffix_replan`
- `full_replan`
- `request_hitl`
- `stop`

动作效用至少考虑：

- 成功率提升
- 即时成本
- 未来恢复代价
- 剩余预算消耗
- 人工介入机会成本

---

## 8. 该算法真正的核心

该算法真正的核心不是“生成工具链”这一动作本身，而是：

**在不完整观测下，对成本、风险和恢复空间进行联合建模，并据此做工作流级动作选择。**

这也是它区别于：

- ToolKG 检索
- 静态模板链
- 简单阈值门控

的根本原因。

---

## 9. 可用于验证该算法的指标

不应只看“工具选对率”，应采用工作流级指标：

- Top-1 计划最终成功率
- 平均执行成本
- 平均执行时间
- 平均高代价步骤调用次数
- 平均 patch 次数
- 平均 replan 次数
- 人工介入次数
- 失败后的恢复效率

---

## 10. 研究来源与启发

### 工具使用与工具链生成

- ToolLLM
- Gorilla

启发：大规模工具检索是另一类问题；本课题应上升为多阶段工具链规划。

### 测试时预算分配与路由

- BEST-Route
- EcoAct

启发：预算和动作暴露应动态控制。

### 步骤级搜索与价值引导

- QLASS
- LATS

启发：中间步骤价值评估会影响整体决策质量。

### 不确定性感知决策

- UoT

启发：不确定性应进入决策闭环。

### 多 Agent 失败分析与恢复治理

- Why Do Multi-Agent LLM Systems Fail?
- Which Agent Causes Task Failures and When?

启发：`patch/replan` 应被视为算法的一部分，而非事后补丁。
