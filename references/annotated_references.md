# Annotated References

Last updated: 2026-03-21

This file collects a first-pass reading list for the thesis pivot:

- difficulty estimation and workload modeling
- multi-agent control under uncertainty
- scientific-task evaluation
- protein design and autonomous bio workflows

## A. Agent Benchmarks And Task Difficulty

### 1. MLE-bench

- Citation: OpenAI. "MLE-bench: Evaluating Machine Learning Agents on Machine Learning Engineering." October 10, 2024.
- URL: https://openai.com/index/mle-bench/
- Why it matters: shows that long-horizon ML engineering tasks are hard even for strong agent scaffolds; useful as evidence that workflow difficulty and resource allocation are central research problems.
- Key signal: benchmark contains 75 Kaggle-style ML engineering tasks; the best setup reached bronze level in only a minority of competitions.

### 2. PaperBench

- Citation: OpenAI. "PaperBench: Evaluating AI's Ability to Replicate AI Research." April 2, 2025.
- URL: https://openai.com/index/paperbench/
- Why it matters: directly supports the claim that multi-step research tasks need explicit decomposition, grading, and process-level evaluation rather than simple final-answer evaluation.
- Key signal: 20 ICML 2024 papers, 8,316 gradable subtasks, and strong models still remain well below expert humans.

### 3. BrowseComp

- Citation: OpenAI. "BrowseComp: a benchmark for browsing agents." April 10, 2025.
- URL: https://openai.com/index/browsecomp/
- Why it matters: useful for the "persistent search under uncertainty" angle; supports the view that hard tasks need persistence, strategic search, and nontrivial control policies.
- Key signal: benchmark has 1,266 hard information-seeking tasks and was designed around asymmetry of verification: hard to find, easy to verify.

### 4. SWE-Lancer

- Citation: Samuel Miserendino, Michele Wang, Tejal Patwardhan, Johannes Heidecke. "SWE-Lancer: Can Frontier LLMs Earn $1 Million from Real-World Freelance Software Engineering?" ICML 2025 Spotlight Poster.
- URL: https://icml.cc/virtual/2025/poster/43573
- Why it matters: real-world task value is tied to payout and effort; this is a good precedent for mapping agent performance to economic or workload-related quantities.
- Key signal: more than 1,400 real freelance software tasks with end-to-end grading; frontier models still fail on most tasks.

### 5. CoLLAB

- Citation: Saaduddin Mahmud, Eugene Bagdasarian, Shlomo Zilberstein. "CoLLAB: A Framework for Designing Scalable Benchmarks for Agentic LLMs." OpenReview, 2025.
- URL: https://openreview.net/forum?id=372FjQy1cF
- Why it matters: useful for thesis methodology because it treats coordination complexity as something that can be scaled and measured systematically.
- Key signal: benchmark design is built around controlled scaling of coordination difficulty and direct comparison with symbolic solvers.

### 6. Know the Ropes

- Citation: Zhenkun Li, Lingyao Li, Shuhang Lin, Yongfeng Zhang. "Know the Ropes: A Heuristic Strategy for LLM-based Multi-Agent System Design." arXiv:2505.16979, 2025.
- URL: https://arxiv.org/abs/2505.16979
- Why it matters: highly relevant to thesis positioning. The paper argues that naive multi-agent scaling often gives weak or even negative returns, and that gains come from disciplined decomposition with typed task interfaces.
- Key signal: explicitly frames multi-agent design as controller-mediated decomposition rather than ad hoc role stacking.

## B. Scientific Research And Expert-Level Evaluation

### 7. FrontierScience

- Citation: OpenAI. "Evaluating AI's ability to perform scientific research tasks." December 16, 2025.
- URL: https://openai.com/index/frontierscience/
- Why it matters: relevant for the "research-task benchmark" layer of the thesis. It supports rubric-based, expert-grounded evaluation for biology/chemistry/physics tasks.
- Key signal: expert-style grading with rubric decomposition; includes biology and open-ended research-style tasks.

## C. Protein Design, Protein Engineering, And Autonomous Bio

### 8. FSFP

- Citation: Ziyi Zhou et al. "Enhancing efficiency of protein language models with minimal wet-lab data through few-shot learning." Nature Communications 15, 5566 (2024).
- URL: https://www.nature.com/articles/s41467-024-49798-6
- Why it matters: supports the domain claim that protein engineering is data-scarce and uncertainty-heavy, which makes difficulty-aware control and sample-efficient decision policies meaningful.
- Key signal: improves protein fitness prediction under extreme data scarcity and reports wet-lab gains.

### 9. Context-aware geometric deep learning for protein sequence design

- Citation: "Context-aware geometric deep learning for protein sequence design." Nature Communications (2024).
- URL: https://www.nature.com/articles/s41467-024-50571-y
- Why it matters: supports the point that protein design quality depends on structural and environmental context, not only sequence generation.
- Key signal: useful when arguing that a control system should condition planning and evaluation on richer context features.

### 10. ProtDAT

- Citation: Xiao-Yu Guo et al. "Ab-initio amino acid sequence design from protein text description with ProtDAT." Nature Communications 16, Article 10544 (2025).
- URL: https://www.nature.com/articles/s41467-025-65562-w
- Why it matters: strong evidence that "text-to-protein design" is already being actively pursued. This supports differentiating the thesis away from generic assistant behavior and toward control, estimation, and orchestration.
- Key signal: directly maps textual descriptions to protein sequence design.

### 11. Autonomous enzyme engineering platform

- Citation: Nilmani Singh et al. "A generalized platform for artificial intelligence-powered autonomous enzyme engineering." Nature Communications 16, 5648 (2025).
- URL: https://www.nature.com/articles/s41467-025-61209-y
- Why it matters: probably the most important domain-side reference for the practicality argument. It shows that the field values closed-loop, autonomous, platform-style systems rather than plain conversational assistants.
- Key signal: positions AI-driven enzyme engineering as an integrated autonomous platform problem.

## D. Immediate Thesis Implications

These references currently point to one coherent thesis direction:

1. Do not position the work as a generic life-science assistant.
2. Position the work as a control layer for high-cost protein design workflows.
3. Make the core algorithm about difficulty estimation, workload prediction, and policy adaptation.
4. Use the current implementation repository as the execution platform, not as the thesis contribution by itself.

## E. Candidate Core Thesis Title Variants

- Difficulty-Aware Multi-Agent Control for Protein Design Workflows
- Workload Estimation and Adaptive Orchestration in Multi-Agent Protein Design Systems
- A Difficulty-Guided Human-in-the-Loop Multi-Agent Runtime for Protein Design
