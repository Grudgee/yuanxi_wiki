---
name: knowledge-computer-architecture-question-map
description: 《计算机体系结构：量化研究方法（第5版）》离线问答路由层，按问题类型把查询导向具体章节记忆。
---

# 离线问答定位层

这个文件不是章节摘要，而是给 `wiki_assistant` 用的**检索路由器**：当用户离线提问时，先按问题类型定位到具体章节，再读对应记忆文件回答。

## 使用原则

- 先确认问题属于性能、存储、ILP、向量/GPU、并行一致性，还是仓库级系统设计。
- 先读“首选记忆”，再按问题继续下钻到更细的周期文件。
- 需要公式、图表或数值时，优先回看原 PDF 页码；不要把 OCR 摘要当成最终真值。
- 章节正文虽然已经完整，但第1章习题仍未逐题求解，第2–6章案例/练习也只通读了高价值部分。

## 问题路由

| 问题类型 | 首选记忆 | 次级记忆 | 典型可回答内容 |
|---|---|---|---|
| 性能方程、Amdahl、benchmark、比较方法 | `knowledge-chapter-1-quantitative-design-principles.md` | `knowledge-chapter-1-performance-measurement-benchmarks.md`、`knowledge-chapter-1-integration-fallacies-conclusion.md` | CPU time、CPI、加速比、局部优化边界、benchmark 可信度 |
| 功耗、能量、成本、可靠性 | `knowledge-chapter-1-power-energy.md`、`knowledge-chapter-1-cost-trends.md`、`knowledge-chapter-1-dependability.md` | `knowledge-chapter-1-integration-fallacies-conclusion.md` | 动态功耗、PUE/成本趋势、MTTF/MTTR、冗余和可用性 |
| Cache、AMAT、miss penalty、预取、虚拟内存 | `knowledge-chapter-2-introduction.md` | `knowledge-chapter-2-cache-hit-time-optimizations.md`、`knowledge-chapter-2-cache-miss-penalty-optimizations.md`、`knowledge-chapter-2-prefetch-optimizations.md`、`knowledge-chapter-2-protection-virtual-memory-vms.md` | 命中时间、miss rate、miss penalty、VIPT、TLB、prefetch 约束 |
| ILP、依赖、动态调度、Tomasulo、ROB | `knowledge-chapter-3-ilp-cycle-1-dependences.md` | `knowledge-chapter-3-cycle-08-scoreboarding-dynamic-scheduling.md`、`knowledge-chapter-3-cycle-09-tomasulo-organization.md`、`knowledge-chapter-3-cycle-12-rob-speculation-basics.md`、`knowledge-chapter-3-cycle-13-speculation-recovery-memory-order.md` | RAW/WAR/WAW、scoreboard、重命名、精确异常、恢复路径 |
| 分支预测、推测、恢复、BTB、return prediction | `knowledge-chapter-3-cycle-06-correlating-branch-prediction.md` | `knowledge-chapter-3-cycle-07-tournament-branch-predictors.md`、`knowledge-chapter-3-cycle-16-branch-target-return-prediction.md`、`knowledge-chapter-3-cycle-17-integrated-fetch-register-renaming.md` | taken/not taken、历史表、目标预测、RAT 与取指耦合 |
| 向量、SIMD、GPU、分歧、gather/scatter | `knowledge-chapter-4-completion-summary.md` | `knowledge-chapter-4-cycle-02-vector-architecture-basics.md`、`knowledge-chapter-4-cycle-09-gpu-programming-model.md`、`knowledge-chapter-4-cycle-18-dependence-tests-and-reductions.md`、`knowledge-chapter-4-cycle-25-vector-gpu-exercises.md` | lane、mask、convoy/chime、warp divergence、访存布局 |
| snooping、directory、一致性、同步、内存模型 | `knowledge-chapter-5-completion-summary.md` | `knowledge-chapter-5-cycle-04-snooping-coherence-protocols.md`、`knowledge-chapter-5-cycle-10-directory-protocol-basics.md`、`knowledge-chapter-5-cycle-12-synchronization-basics.md`、`knowledge-chapter-5-cycle-14-sequential-consistency.md`、`knowledge-chapter-5-cycle-22-multicore-smt.md` | cache coherence、lock/barrier、SC、relaxed consistency、SMT |
| 数据中心、云、网络、TCO、PUE、能量比例性 | `knowledge-chapter-6-completion-summary.md` | `knowledge-chapter-6-cycle-02-programming-models-workloads.md`、`knowledge-chapter-6-cycle-04-infrastructure-costs.md`、`knowledge-chapter-6-cycle-05-cloud-computing.md`、`knowledge-chapter-6-cycle-06-crosscutting-network-energy.md`、`knowledge-chapter-6-cycle-11-tco-case-study.md` | WSC、MapReduce、SLA、总拥有成本、数据中心网络和供电 |

## 可直接回答的稳定结论

- 第1章的核心是：用定量方法比较设计方案，Amdahl 定律解释局部优化为何会有上限。
- 第2章的核心是：存储层次要同时优化 hit time、bandwidth、miss rate 和 miss penalty。
- 第3章的核心是：ILP 的难点不只是执行单元，而是依赖、分支、重命名和精确状态恢复。
- 第4章的核心是：向量/SIMD/GPU 都在扩大数据级并行，但必须处理分歧和访存模式。
- 第5章的核心是：可扩展多处理器离不开一致性、同步和可解释的内存模型。
- 第6章的核心是：把整座数据中心当作计算机设计对象，成本、能耗、网络和运维都要一起看。

## 答案结构

离线回答时优先采用以下结构：

1. 先给结论。
2. 再给适用条件、公式或边界。
3. 然后列出对应记忆文件。
4. 如果问题需要数值核算或图表细节，明确说明要回看原 PDF 页码。

## 适用边界

- 这是知识库路由层，不替代章节本体。
- 如果新问题超出已学习的第1–6章主题，先说明知识库未覆盖，再建议去 PDF 原文补记忆。
- 如果用户问的是“怎么从记忆回答”，优先读 `wiki_memory/README.md`、`agents/wiki_assistant.md` 和 `wiki_memory/MEMORY.md`。

