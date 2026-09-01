---
name: knowledge-superscalar-processor-design-cycle-12-multiport-cache
description: 超标量处理器设计第12个学习周期，记录多端口Cache的三种实现路线。
---

# 知识点摘要

- 超标量处理器可能同周期产生多个 Cache 访问，因此单端口 SRAM 容易成为带宽瓶颈。
- True multi-port 直接在 SRAM cell 上提供多个读写端口，功能直接但面积、功耗和时序代价高。
- Multiple Cache Copies 通过复制 Cache 数据支持多个读端口，但写入和一致更新复杂；Multi-banking 通过分 bank 支持并行访问，但同 bank 冲突仍会阻塞。

# 关键细节

- 周期：第 12 个学习周期。
- 源文件：`books/超标量处理器设计.pdf`。
- 已读范围：第 2 章 §2.3.1–§2.3.3，书内 p.37–39 左右。
- 批次规模：估计 6,000–12,000 源字符，低于 80,000 字符监督阈值。

# 原文依据

- 位置：第 2章“多端口 Cache”。
- 依据：正文依次介绍 True Multi-port、Multiple Cache Copies 和 Multi-banking，并比较 SRAM 端口和 bank 冲突问题。

# 适用条件与限制

- Multi-banking 只有在访问均匀落到不同 bank 时才能提供理想并行性。

# 下一断点

- 学习 AMD Opteron 多端口 D Cache 的真实例子和超标量取指。

