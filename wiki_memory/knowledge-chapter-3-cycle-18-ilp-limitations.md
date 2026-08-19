---
name: knowledge-chapter-3-cycle-18-ilp-limitations
description: 第3章本轮周期15，记录理想处理器与可实现处理器的ILP上限研究。
---

# 知识点摘要

- 理想 ILP 研究通过逐步放宽资源约束估计可用并行度；“perfect processor”假定无限重命名、完美分支预测、完美内存别名分析、理想存储层次和极宽执行资源。
- 可实现模型加入有限 instruction window、有限 issue width、有限物理寄存器、现实分支预测与 cache 后，并行度显著下降。
- 整数程序尤其受分支、有限窗口、寄存器数量和较少固有并行度限制；FP 程序还受内存行为和算法数据流约束。
- 继续提取 ILP 往往带来不成比例的复杂度、时钟周期和功耗代价，促使设计转向其他并行层次。

# 关键细节

- 周期：本轮周期 15/16。
- 来源范围：PDF p.241 下半页–249 上半页 / 书内 p.213 下半页–221 上半页。
- OCR 源字符数：20,354（小于 80,000；本轮最大周期）。
- 下一断点：PDF p.249 / 书内 p.221，§3.11 `Cross-Cutting Issues: ILP Approaches and the Memory System` 标题处。

# 原文引用

- 文档：《计算机体系结构：量化研究方法》扫描版 PDF。
- 版本/日期：未知。
- 位置：第3章 §3.10，PDF p.241–249 / 书内 p.213–221，至 §3.11 标题前。
- 依据：图3.26比较 perfect processor 的可用 ILP，图3.27及后续讨论逐项加入窗口、预测器、重命名、cache 与 alias 限制。

# 适用条件与例外

- 研究基于特定 benchmark、trace 和编译器假设；不是所有程序的物理上限。
- 理想模型用于定位瓶颈，不能视为可实现微架构。

# 关联章节

- §3.3 branch prediction；§3.6 speculation；§3.9 renaming；§3.12 multithreading。

# 待核验问题

- 图3.26–3.27的曲线精确读数受扫描质量影响，未记录逐点数值。
