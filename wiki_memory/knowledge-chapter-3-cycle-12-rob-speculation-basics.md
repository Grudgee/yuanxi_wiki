---
name: knowledge-chapter-3-cycle-12-rob-speculation-basics
description: 第3章本轮周期9，记录基于ROB的硬件推测与精确提交基础。
---

# 知识点摘要

- hardware speculation 把动态分支预测、Tomasulo 式乱序执行与按程序序提交结合起来。
- Reorder Buffer 保存在途指令的目的位置、结果、状态与异常信息；结果先进入 ROB，只有到达队首且无异常时才更新寄存器或内存。
- 将执行完成与 commit 分离，可在分支预测错误或异常时丢弃未提交状态，从而恢复精确体系结构状态。

# 关键细节

- 周期：本轮周期 9/16。
- 来源范围：PDF p.211 下半页–216 / 书内 p.183 下半页–188。
- OCR 源字符数：15,329（小于 80,000）。
- 下一断点：PDF p.217 / 书内 p.189，§3.6 的预测错误恢复与 store/exception 示例。

# 原文引用

- 文档：《计算机体系结构：量化研究方法》扫描版 PDF。
- 版本/日期：未知。
- 位置：第3章 §3.6，PDF p.211–216 / 书内 p.183–188。
- 依据：图3.11–3.12给出加入 ROB 后的结构和状态表；正文把流程分为 issue、execute、write result、commit。

# 适用条件与例外

- ROB 容量限制可同时在途和可推测的指令数。
- 对 store、内存别名和异常的处理需要严格的提交顺序。

# 关联章节

- §3.5 Tomasulo；§3.6 branch recovery。

# 待核验问题

- 图3.12个别表格值 OCR 错位，未保存逐项状态快照。
