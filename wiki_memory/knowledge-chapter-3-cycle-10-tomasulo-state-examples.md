---
name: knowledge-chapter-3-cycle-10-tomasulo-state-examples
description: 第3章本轮周期7，记录Tomasulo状态字段、示例和两项主要优势。
---

# 知识点摘要

- reservation station 的 `Vj/Vk` 保存已就绪值，`Qj/Qk` 保存生产者标签，`A` 保存 load/store 地址信息，`Busy/Op` 表示占用与操作。
- register status 的 `Qi` 指向将产生该寄存器新值的保留站；结果广播后，匹配消费者获取值并清除等待标签。
- 原文归纳两项主要优势：hazard 检测分布化，以及通过动态重命名消除 WAR/WAW 停顿。

# 关键细节

- 周期：本轮周期 7/16。
- 来源范围：PDF p.204–207 上半页 / 书内 p.176–179 上半页。
- OCR 源字符数：7,387（小于 80,000）。
- 下一断点：PDF p.207 / 书内 p.179，`Tomasulo's Algorithm: A Loop-Based Example` 标题处。

# 原文引用

- 文档：《计算机体系结构：量化研究方法》扫描版 PDF。
- 版本/日期：未知。
- 位置：第3章 §3.5，PDF p.204–207 / 书内 p.176–179。
- 依据：图3.7与图3.8展示指令状态、保留站和寄存器标签；正文解释标签如何解除 WAW/WAR 并分布 hazard logic。

# 适用条件与例外

- 表中标签名称是示例保留站标识，不是 ISA 可见寄存器。
- 内存访问还需要地址顺序与别名检查。

# 关联章节

- §3.4 Tomasulo organization；§3.5 loop example。

# 待核验问题

- 图表中的部分数值单元 OCR 错位，未保存为精确时序表。
