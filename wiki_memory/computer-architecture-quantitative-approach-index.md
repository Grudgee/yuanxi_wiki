---
name: computer-architecture-quantitative-approach-index
description: 《计算机体系结构：量化研究方法（第5版）》六章本地知识库统一检索索引，按章节、主题、公式和常见问题路由到具体memory。
---

# 使用说明

- 本文件是后续问答的首选入口；详细学习进度、OCR批次和页码边界见 `knowledge-index-computer-architecture-quantitative-approach.md`。
- 引用原书时必须同时标注 `PDF p.N / 书内 p.M / 章节或标题`。
- 六个正文主题已学习完成；第1章习题未逐题处理，第2–6章案例与练习已通读但数值题未逐题求解；附录A及以后尚未学习。
- 扫描PDF的图表、公式下标、伪代码符号和历史产品参数在精确引用前仍需回看原页。

# 六章总览

| 章 | 核心问题 | 主要主题 | 完成摘要 |
|---|---|---|---|
| 第1章 量化设计基础 | 如何度量与比较体系结构 | 性能方程、Amdahl、功耗、成本、可靠性、benchmark | `knowledge-chapter-1-integration-fallacies-conclusion.md` |
| 第2章 存储层次 | 如何缩小处理器与存储差距 | cache、AMAT、miss优化、DRAM、VM、VMM | `knowledge-chapter-2-conclusion-memory-futures.md` |
| 第3章 ILP | 如何发现并利用指令级并行 | 调度、分支预测、Tomasulo、ROB、推测、多发射、SMT | `knowledge-chapter-3-completion-summary.md` |
| 第4章 DLP | 如何利用向量、SIMD和GPU | vector、convoy/chime、SIMD、GPU、分歧、访存、依赖分析 | `knowledge-chapter-4-completion-summary.md` |
| 第5章 TLP | 如何构建可扩展共享存储多处理器 | snooping、directory、同步、一致性模型、多核、SMT | `knowledge-chapter-5-completion-summary.md` |
| 第6章 WSC | 如何把整座数据中心作为计算机设计 | RLP、MapReduce、网络、PUE、TCO、云、能量比例性 | `knowledge-chapter-6-completion-summary.md` |

# 第1章：量化设计基础

- 计算机类别与应用约束：`knowledge-chapter-1-classes-of-computers.md`
- 体系结构定义与接口边界：`knowledge-chapter-1-defining-computer-architecture.md`
- 带宽、延迟及技术趋势：`knowledge-chapter-1-bandwidth-latency.md`
- 功率与能量：`knowledge-chapter-1-power-energy.md`
- 成本、晶圆与良率：`knowledge-chapter-1-cost-trends.md`、`knowledge-chapter-1-section-1-6-cost-foundations.md`
- 可靠性、可用性、FIT、MTTF/MTTR：`knowledge-chapter-1-dependability.md`
- 性能测量与benchmark：`knowledge-chapter-1-performance-measurement-benchmarks.md`
- Amdahl定律、并行加速和量化原则：`knowledge-chapter-1-quantitative-design-principles.md`
- 谬误、陷阱与章节结论：`knowledge-chapter-1-integration-fallacies-conclusion.md`

# 第2章：存储层次

- 基础与AMAT：`knowledge-chapter-2-introduction.md`
- 降低hit time：`knowledge-chapter-2-cache-hit-time-optimizations.md`
- 提高cache bandwidth：`knowledge-chapter-2-cache-bandwidth-optimizations.md`
- 降低miss penalty：`knowledge-chapter-2-cache-miss-penalty-optimizations.md`
- 编译器降低miss rate：`knowledge-chapter-2-compiler-cache-optimizations.md`
- 硬件/软件预取：`knowledge-chapter-2-prefetch-optimizations.md`
- DRAM、Flash、ECC及存储技术：`knowledge-chapter-2-memory-technology-optimizations.md`
- 虚拟存储、保护与VMM：`knowledge-chapter-2-protection-virtual-memory-vms.md`
- Cortex-A8与Core i7案例：`knowledge-chapter-2-cortex-a8-core-i7-memory-hierarchies.md`
- 谬误、结论与未来：`knowledge-chapter-2-fallacies-pitfalls.md`、`knowledge-chapter-2-conclusion-memory-futures.md`
- 案例与练习：`knowledge-chapter-2-case-study-1-blocking.md`、`knowledge-chapter-2-case-study-1-prefetching.md`、`knowledge-chapter-2-case-study-2-memory-probing.md`、`knowledge-chapter-2-exercises-cache-organization.md`、`knowledge-chapter-2-exercises-memory-virtualization.md`、`knowledge-chapter-2-exercises-system-evaluation.md`

# 第3章：指令级并行

- 依赖与hazard入口：`knowledge-chapter-3-ilp-cycle-1-dependences.md`、`knowledge-chapter-3-ilp-cycle-2-hazards.md`、`knowledge-chapter-3-ilp-cycle-3-speculation-entry.md`
- 编译器调度和循环展开：`knowledge-chapter-3-cycle-04-basic-pipeline-scheduling.md`、`knowledge-chapter-3-cycle-05-loop-unrolling-scheduling.md`
- 分支预测：`knowledge-chapter-3-cycle-06-correlating-branch-prediction.md`、`knowledge-chapter-3-cycle-07-tournament-branch-predictors.md`
- Scoreboard与Tomasulo：`knowledge-chapter-3-cycle-08-scoreboarding-dynamic-scheduling.md`、`knowledge-chapter-3-cycle-09-tomasulo-organization.md`、`knowledge-chapter-3-cycle-10-tomasulo-state-examples.md`、`knowledge-chapter-3-cycle-11-tomasulo-loop-algorithm.md`
- ROB、精确异常和恢复：`knowledge-chapter-3-cycle-12-rob-speculation-basics.md`、`knowledge-chapter-3-cycle-13-speculation-recovery-memory-order.md`
- VLIW与动态多发射：`knowledge-chapter-3-cycle-14-vliw-static-multiple-issue.md`、`knowledge-chapter-3-cycle-15-dynamic-multiple-issue.md`
- 取指、BTB、返回预测和寄存器重命名：`knowledge-chapter-3-cycle-16-branch-target-return-prediction.md`、`knowledge-chapter-3-cycle-17-integrated-fetch-register-renaming.md`
- ILP上限、软硬件推测和多线程：`knowledge-chapter-3-cycle-18-ilp-limitations.md`、`knowledge-chapter-3-cycle-19-hardware-software-speculation.md`、`knowledge-chapter-3-cycle-20-multithreading-models.md`、`knowledge-chapter-3-cycle-22-smt-effectiveness.md`
- Cortex-A8/Core i7、谬误、结论与练习：从 `knowledge-chapter-3-cycle-23-cortex-a8-pipeline.md` 至 `knowledge-chapter-3-cycle-27-exercises-smt-and-design.md`

# 第4章：数据级并行

- 向量体系结构、执行时间和chaining：`knowledge-chapter-4-cycle-02-vector-architecture-basics.md`、`knowledge-chapter-4-cycle-03-vector-execution-convoys.md`
- 向量长度、mask、memory bank、stride和gather/scatter：从 `knowledge-chapter-4-cycle-04-vector-length-and-mask.md` 至 `knowledge-chapter-4-cycle-07-stride-gather-scatter-performance.md`
- 多媒体SIMD：`knowledge-chapter-4-cycle-08-simd-programming-limits.md`
- GPU编程模型、线程层次和PTX：从 `knowledge-chapter-4-cycle-09-gpu-programming-model.md` 至 `knowledge-chapter-4-cycle-11-ptx-isa.md`
- 分支发散、存储结构和向量/GPU比较：从 `knowledge-chapter-4-cycle-12-branch-divergence.md` 至 `knowledge-chapter-4-cycle-15-gpu-vector-comparison.md`
- 循环依赖分析与编译器变换：从 `knowledge-chapter-4-cycle-16-loop-dependence-basics.md` 至 `knowledge-chapter-4-cycle-18-dependence-tests-and-reductions.md`
- 能耗、性能比较、谬误、结论、历史和练习：从 `knowledge-chapter-4-cycle-19-crosscutting-energy-memory.md` 至 `knowledge-chapter-4-cycle-25-vector-gpu-exercises.md`

# 第5章：线程级并行

- 共享存储多处理器基础：`knowledge-chapter-5-cycle-02-introduction-architectures.md`
- 集中式共享存储与snooping：从 `knowledge-chapter-5-cycle-03-centralized-shared-memory.md` 至 `knowledge-chapter-5-cycle-06-coherence-extensions-limitations.md`
- 对称多处理器性能：`knowledge-chapter-5-cycle-07-symmetric-memory-performance-basics.md`、`knowledge-chapter-5-cycle-08-commercial-workload-performance.md`
- 分布式共享存储与directory：从 `knowledge-chapter-5-cycle-09-distributed-shared-memory.md` 至 `knowledge-chapter-5-cycle-11-example-directory-protocol.md`
- 同步、锁与屏障：`knowledge-chapter-5-cycle-12-synchronization-basics.md`、`knowledge-chapter-5-cycle-13-locks-and-barriers.md`
- 顺序一致性和宽松模型：从 `knowledge-chapter-5-cycle-14-sequential-consistency.md` 至 `knowledge-chapter-5-cycle-16-consistency-model-final-remarks.md`
- 编译器、推测、包含性和多线程收益：从 `knowledge-chapter-5-cycle-17-compiler-consistency.md` 至 `knowledge-chapter-5-cycle-20-multiprocessing-multithreading-gains.md`
- 多核与SMT：`knowledge-chapter-5-cycle-21-multicore-processors-performance.md`、`knowledge-chapter-5-cycle-22-multicore-smt.md`
- 谬误、结论、案例和练习：从 `knowledge-chapter-5-cycle-23-fallacies-pitfalls.md` 至 `knowledge-chapter-5-cycle-27-exercises.md`

# 第6章：仓库级计算机

- 定义、RLP及与HPC/数据中心区别：`knowledge-chapter-6-cycle-01-roadmap-introduction.md`
- MapReduce与工作负载：`knowledge-chapter-6-cycle-02-programming-models-workloads.md`
- 网络层次、超额订阅和故障：`knowledge-chapter-6-cycle-03-network-architecture.md`
- 供电、制冷、PUE和TCO：`knowledge-chapter-6-cycle-04-infrastructure-costs.md`
- 云计算、弹性、SLA和按需计费：`knowledge-chapter-6-cycle-05-cloud-computing.md`
- 网络瓶颈和能量比例性：`knowledge-chapter-6-cycle-06-crosscutting-network-energy.md`
- Google WSC案例：`knowledge-chapter-6-cycle-07-google-wsc-case.md`
- 谬误、结论与历史：从 `knowledge-chapter-6-cycle-08-fallacies-pitfalls.md` 至 `knowledge-chapter-6-cycle-10-historical-perspectives.md`
- TCO与资源配置案例：`knowledge-chapter-6-cycle-11-tco-case-study.md`、`knowledge-chapter-6-cycle-12-resource-allocation-case-study.md`
- 练习主题：从 `knowledge-chapter-6-cycle-13-exercises-parallelism-reliability.md` 至 `knowledge-chapter-6-cycle-15-exercises-energy-manageability.md`

# 跨章节主题路由

| 查询主题 | 首选记忆 |
|---|---|
| 性能方程、加速比、Amdahl | `knowledge-chapter-1-quantitative-design-principles.md` |
| 功率、能量、每焦耳性能 | `knowledge-chapter-1-power-energy.md`、`knowledge-chapter-6-cycle-06-crosscutting-network-energy.md` |
| 可靠性、可用性、冗余 | `knowledge-chapter-1-dependability.md`、`knowledge-chapter-6-cycle-15-exercises-energy-manageability.md` |
| cache、AMAT、miss penalty | `knowledge-chapter-2-introduction.md`、`knowledge-chapter-2-cache-miss-penalty-optimizations.md` |
| 虚拟存储与虚拟化 | `knowledge-chapter-2-protection-virtual-memory-vms.md` |
| 动态调度、Tomasulo、ROB | `knowledge-chapter-3-cycle-09-tomasulo-organization.md`、`knowledge-chapter-3-cycle-12-rob-speculation-basics.md` |
| 分支预测与推测 | `knowledge-chapter-3-cycle-06-correlating-branch-prediction.md`、`knowledge-chapter-3-cycle-13-speculation-recovery-memory-order.md` |
| SIMD、vector、GPU | `knowledge-chapter-4-cycle-02-vector-architecture-basics.md`、`knowledge-chapter-4-cycle-09-gpu-programming-model.md` |
| cache coherence、snooping、directory | `knowledge-chapter-5-cycle-04-snooping-coherence-protocols.md`、`knowledge-chapter-5-cycle-10-directory-protocol-basics.md` |
| memory consistency、同步 | `knowledge-chapter-5-cycle-12-synchronization-basics.md`、`knowledge-chapter-5-cycle-14-sequential-consistency.md` |
| 数据中心网络、PUE、TCO | `knowledge-chapter-6-cycle-03-network-architecture.md`、`knowledge-chapter-6-cycle-04-infrastructure-costs.md` |
| 云计算与MapReduce | `knowledge-chapter-6-cycle-02-programming-models-workloads.md`、`knowledge-chapter-6-cycle-05-cloud-computing.md` |

# 回答与引用策略

1. 先按本索引定位章节和主题文件。
2. 检查主题文件的“待核验问题”和适用条件；存在OCR或历史参数风险时回查PDF。
3. 需要公式或图表精确值时，不只依赖摘要；读取对应页图像并标注双页码。
4. 区分书中结论、历史产品案例和当前推断；不得把第5版历史规格当作当前事实。
5. 数值习题默认视为“题目已通读、答案未求解”，除非后续单独建立求解记忆。
