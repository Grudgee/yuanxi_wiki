---
name: knowledge-chapter-3-cycle-09-tomasulo-organization
description: 第3章本轮周期6，记录Tomasulo算法的保留站、标签与公共数据总线。
---

# 知识点摘要

- Tomasulo 算法把等待操作数的指令放入功能部件前的 reservation station，并用生产者标签而非体系结构寄存器名表示未就绪操作数。
- Common Data Bus 广播完成结果；所有匹配标签的保留站和寄存器状态同时捕获值，从而形成数据驱动唤醒。
- 保留站实现动态 register renaming，消除 WAR/WAW name dependence；RAW 仍由标签等待保证。

# 关键细节

- 周期：本轮周期 6/16。
- 来源范围：PDF p.198 下半页–203 / 书内 p.170 下半页–175。
- OCR 源字符数：15,250（小于 80,000）。
- 下一断点：PDF p.204 / 书内 p.176，§3.5 的 Tomasulo 状态表示例。

# 原文引用

- 文档：《计算机体系结构：量化研究方法》扫描版 PDF。
- 版本/日期：未知。
- 位置：第3章 §3.4，PDF p.198–203 / 书内 p.170–175。
- 依据：原文给出基于 Tomasulo 的 MIPS FP 单元结构，解释 reservation station、load/store buffer、register status 与 CDB 的连接和控制。

# 适用条件与例外

- CDB 数量、保留站容量和功能部件数量会形成结构瓶颈。
- 基础算法仍需单独处理控制推测、精确异常和内存次序。

# 关联章节

- §3.5 Tomasulo examples；§3.6 ROB speculation。

# 待核验问题

- 图3.6连线标签有少量 OCR 噪声；组件关系由正文说明核验。
