# 构建生成式蛋白质结合物设计流水线蓝图-NVIDIA

具体参考：[NVIDIA](https://build.nvidia.com/nvidia/protein-binder-design-for-drug-discovery/blueprintcard)

此蓝图展示了如何使用生成式AI和加速的NIM微服务，更智能、更快速地设计蛋白结合物。

使用到的工具有：

- rfdiffusion
- proteinmpnn
- alphafold2
- alphafold2-multimer

## 概述

在药物发现中，即便知道要针对的蛋白以及疾病，设计一个能够特异性结合该蛋白的治疗分子仍然是一项巨大挑战。

可以把它想象成在一个几乎无限的钥匙仓库里寻找一个完美契合的钥匙——每把钥匙都有独特的三维形状。对于长度为 n 的蛋白来说，可能存在 20^n 种序列，每种序列又能形成无数构象。平均人类蛋白长度约 430 个氨基酸，这意味着可能的序列数量远超宇宙中原子的数量（10^80）。这种潜在的多样性对生命进化至关重要，但对研究者来说极具挑战。

传统方法中，这种复杂性意味着使用者必须通过反复实验来筛选成千上万候选分析，每轮合成和验证可能耗费数月甚至数年。整个过程昂贵、缓慢且不确定。研究者通常依赖经验猜测，希望能够筛选出一个有效结合体。

这里，我们引入生成式AI，通过NIM微服务对分子进行预优化，并模拟其与目标蛋白的项目作用。BioNeMo蓝图展示了如何通过蛋白折叠、结构生成和序列生成微服务，加速蛋白结合体的设计周期，并生成更优的结合体。

## 流程体验

Protein Binder Design NVIDIA NIM Agent 蓝图利用封装在 NIM 微服务中的 AI 模型，设计优化的蛋白序列和结构。流程如下：

1. 用户提供氨基酸序列给AlphaFold2，预测目标蛋白的初步三维结构。AlphaFold2还需要对序列比对（MSA），可通过加速的MSA NIM生成。
1. 使用蛋白目标结构，RFdiffusion生成蛋白结合体的骨架。用户可引导模型探索特定结合界面或热点区域，以满足设计约束。
1. ProteinMPNN根据RFdiffusion生成的骨架生成并优化氨基酸序列，确保蛋白具备有效结合所需的生化性质。
1. AlphaFold2-Multimer验证蛋白复合体的相互作用和稳定性。

## 示例流程

完整示例请参照[NVIDIA BioNeMO Blueprints GitHub 仓库](https://github.com/NVIDIA-BioNeMo-blueprints/generative-protein-binder-design)

## 技术注意事项

- RFdiffusion能够生成蛋白骨架，ProtienMPNN能标注氨基酸序列，组合可生成应折叠成目标蛋白结合体的序列
- 蛋白具有柔性，可呈现多种构象，尤其是抗体的结合界面无序化。模型虽进步明显，但仍非完全准确。
