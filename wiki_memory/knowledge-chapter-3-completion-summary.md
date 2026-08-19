---
name: knowledge-chapter-3-completion-summary
description: 第3章 Instruction-Level Parallelism and Its Exploitation 的四要素完成摘要。
---

# 核心概念

- ILP依赖分析、编译器调度、分支预测、动态调度、寄存器重命名、推测执行、多发射、ILP上限与多线程。

# 关键结论

- 第3章把性能提升建立在“发现独立指令并安全恢复精确状态”之上；真实依赖、控制流、存储别名、缓存未命中、复杂度和功耗共同限制可利用ILP。
- 多线程用不同线程的工作填补单线程停顿，补充而非替代ILP；本章正文、案例入口和习题已连续读至 PDF p.287 / 书内 p.259。

# 未解决问题

- 习题未逐题求数值答案；小字号图表、公式下标和代码标点存在已记录的OCR不确定点。
- §3.16详细历史资料位于原文指向的在线附录，本地扫描正文未展开。

# 记忆路径

- wiki_memory/knowledge-chapter-3-ilp-cycle-1-dependences.md 至 wiki_memory/knowledge-chapter-3-cycle-27-exercises-smt-and-design.md；总索引见 wiki_memory/knowledge-index-computer-architecture-quantitative-approach.md。

# 原文位置与下一断点

- 文档：books/计算机体系结构量化研究方法.pdf，第3章 PDF p.173–287 / 书内 p.145–259（本批次接续范围 PDF p.252–287 / 书内 p.224–259）。
- 第3章完成边界：PDF p.287 / 书内 p.259。
- 下一精确断点：PDF p.288 / 书内 p.260，第4章标题页；下一完整正文从 PDF p.290 / 书内 p.262 开始。
