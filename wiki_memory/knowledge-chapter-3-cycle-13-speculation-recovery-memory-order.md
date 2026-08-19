---
name: knowledge-chapter-3-cycle-13-speculation-recovery-memory-order
description: 第3章本轮周期10，记录推测恢复、精确异常与内存次序约束。
---

# 知识点摘要

- 分支预测错误时，已提交的较早指令保留，分支及其后的未提交 ROB 项被清除，再从正确目标取指。
- 异常延迟到异常指令到达 ROB 队首时识别，可保证所有更早指令已提交、更晚指令尚未改变体系结构状态。
- store 的地址和值可先计算，但对内存的写入必须等到 commit；load 必须检查更早 store 的地址，命中时可从最近匹配 store 转发。

# 关键细节

- 周期：本轮周期 10/16。
- 来源范围：PDF p.217–220 上半页 / 书内 p.189–192 上半页。
- OCR 源字符数：10,507（小于 80,000）。
- 下一断点：PDF p.220 / 书内 p.192，§3.7 `Exploiting ILP Using Multiple Issue and Static Scheduling` 标题处。

# 原文引用

- 文档：《计算机体系结构：量化研究方法》扫描版 PDF。
- 版本/日期：未知。
- 位置：第3章 §3.6，PDF p.217–220 / 书内 p.189–192，至 §3.7 标题前。
- 依据：原文通过 ROB 示例解释 misprediction 清空、exception 延迟处理、store commit 与 load/store hazard 检查。

# 适用条件与例外

- speculative load 是否可越过 store 取决于地址是否已知以及 memory disambiguation 规则。
- precise exception 语义要求任何预测路径副作用在提交前可撤销或保持不可见。

# 关联章节

- §3.6 ROB；§3.8 dynamic multiple issue。

# 待核验问题

- 图3.14伪代码字体较小，未转录每条条件语句。
