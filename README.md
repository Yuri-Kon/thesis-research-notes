# 毕设研究笔记

本仓库用于存放**面向毕业设计与论文写作的研究材料**，并与实现仓库保持分离。

这里的内容主要服务于以下目标：

- 记录论文选题收敛过程
- 整理相关研究与带注释参考文献
- 沉淀算法定义、问题形式化与实验设想
- 为后续开题、论文撰写和答辩准备可复用材料

当前关注重点包括：

- 多 Agent 蛋白质设计工作流的选题边界与定位
- 与一般“生命科学助手”叙事的区分
- 高代价工作流中的风险、成本、恢复与人工介入问题
- 新算法“面向高代价蛋白质设计工作流的自适应工具链规划算法”的定义与收敛

______________________________________________________________________

## 目录说明

### `references/`

参考文献与阅读笔记目录。

- `references/annotated_references.md`
  - 带注释的论文阅读记录
- `references/seed_references.bib`
  - 初始参考文献条目

### `notes/`

研究备忘、方向比较、算法草稿与压缩摘要。

- `notes/possible_topic.md`
  - 早期毕设选题候选方向整理
- `notes/controller_direction_memo.md`
  - 关于 controller 路线、belief-state controller 路线、创新边界与风险的长篇分析备忘
- `notes/new-algorithm.md`
  - 新算法主笔记
  - 详细定义了“面向高代价蛋白质设计工作流的自适应工具链规划算法”
  - 包含问题定义、形式化描述、belief-state 的角色、研究来源，以及六个算法前置定义
- `notes/new-algorithm-summary.md`
  - 新算法压缩摘要
  - 用于后续快速读取核心定义并继续修改

### `report/`

论文式研究报告草稿目录。

- `report/main.tex`
  - 报告主入口
- `report/sections/`
  - 分章节研究内容，包括背景、问题重构、候选路线、belief-state controller、创新点、时间评估、预期结果与结论
- `report/refs.bib`
  - 报告所用参考文献

______________________________________________________________________

## 当前建议入口

如果要快速理解当前研究脉络，建议按以下顺序阅读：

1. `notes/controller_direction_memo.md`
1. `notes/new-algorithm.md`
1. `notes/new-algorithm-summary.md`
1. `report/sections/09-conclusion.tex`

______________________________________________________________________

## 新算法相关文件

当前“新算法”相关内容主要集中在以下两个文件：

- `notes/new-algorithm.md`
  - 完整版本，适合继续扩展问题定义、六个 schema、实验指标和论文表述
- `notes/new-algorithm-summary.md`
  - 精简版本，适合快速阅读并进行进一步修改

两份文件的关系是：

- `new-algorithm.md` 负责完整论证
- `new-algorithm-summary.md` 负责快速摘要

______________________________________________________________________

## 备注

本仓库偏向研究整理与论文准备，不承担实现仓库中的代码职责。
若涉及系统设计、运行时架构、FSM、Planner/Executor/Safety 等实现约束，应回到实现仓库及其设计文档中核对。
