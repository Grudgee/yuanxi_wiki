---
name: knowledge-chapter-3-cycle-19-hardware-software-speculation
description: 第3章本轮周期16，记录软硬件推测权衡及其与存储系统的耦合。
---

# 知识点摘要

- hardware speculation 能在运行时做 memory disambiguation、适应动态分支行为并维持 precise exception，但需要 ROB、重命名、恢复和复杂调度硬件。
- software speculation 可降低硬件调度负担，却必须保守处理指针别名，并依赖 nonfaulting/检查机制、predication 或补偿代码避免错误路径副作用。
- 推测执行的内存访问不能让错误路径异常或 cache stall 主导执行；因此需要 nonblocking cache，并让后端内存并行度匹配多个 outstanding miss。
- 条件移动与 register renaming 结合时仍会产生数据流和资源副作用，不能简单等同于“无分支成本”。

# 关键细节

- 周期：本轮周期 16/16。
- 来源范围：PDF p.249 下半页–251 顶部 / 书内 p.221 下半页–223 顶部，止于 §3.12 标题前。
- OCR 源字符数：5,389（小于 80,000）。
- 下一断点：PDF p.251 / 书内 p.223 的 §3.12 `Multithreading: Exploiting Thread-Level Parallelism to Improve Uniprocessor Throughput` 标题；下一完整正文页 PDF p.252 / 书内 p.224。

# 原文引用

- 文档：《计算机体系结构：量化研究方法》扫描版 PDF。
- 版本/日期：未知。
- 位置：第3章 §3.11，PDF p.249–251 / 书内 p.221–223，至 §3.12 标题前。
- 依据：原文对比 hardware/software speculation，并说明推测内存访问需要 cache 与内存系统支持，尤其是 nonblocking 与并发 miss 能力。

# 适用条件与例外

- 软件推测可行性高度依赖 ISA 提供的异常抑制、检查或 predication 支持。
- 原文指出编译器通常只推测 L1 miss；对某些规则科学程序，多个 L2 miss 可借助足够内存并行度重叠。

# 关联章节

- §3.6 hardware speculation；Appendix H software speculation；§2 memory hierarchy；§3.12 multithreading。

# 待核验问题

- PDF p.251页眉被 OCR 识别为“4,12”，正文标题图像核验为 §3.12；不影响断点。
