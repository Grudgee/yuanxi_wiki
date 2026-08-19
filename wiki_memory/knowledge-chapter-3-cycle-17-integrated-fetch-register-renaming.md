---
name: knowledge-chapter-3-cycle-17-integrated-fetch-register-renaming
description: 第3章本轮周期14，记录集成取指、物理寄存器重命名与别名预测。
---

# 知识点摘要

- integrated instruction fetch unit 把分支预测、目标预测、指令预取与 instruction memory 访问协同起来，持续向宽发射后端供给指令。
- 显式 physical register renaming 用映射表把体系结构寄存器绑定到更大的物理寄存器集合；新目的寄存器分配新物理项，旧项在不再可能被读取后回收。
- ROB 方案与物理寄存器方案都能支持 speculation；前者在提交时复制结果，后者以提交映射确定体系结构可见版本，但回收逻辑更复杂。
- 通用 value prediction 成本与收益不足；较窄的 address alias prediction 只预测内存访问是否冲突，更稳定且已被实际处理器采用。

# 关键细节

- 周期：本轮周期 14/16。
- 来源范围：PDF p.235–241 上半页 / 书内 p.207–213 上半页。
- OCR 源字符数：17,494（小于 80,000）。
- 下一断点：PDF p.241 / 书内 p.213，§3.10 `Studies of the Limitations of ILP` 标题处。

# 原文引用

- 文档：《计算机体系结构：量化研究方法》扫描版 PDF。
- 版本/日期：未知。
- 位置：第3章 §3.9，PDF p.235–241 / 书内 p.207–213，至 §3.10 标题前。
- 依据：原文列出 integrated fetch 的功能，比较 ROB 与显式 register renaming，并讨论 issue bottleneck、value prediction 和 address aliasing prediction。

# 适用条件与例外

- 物理寄存器回收必须等到旧映射不再被任何在途指令使用。
- 别名预测错误必须可检测并通过 replay/恢复保证正确性。

# 关联章节

- §3.6 ROB；§3.8 multiple issue；§3.10 ILP limits。

# 待核验问题

- 图3.24曲线点未转录；正文结论清晰。
