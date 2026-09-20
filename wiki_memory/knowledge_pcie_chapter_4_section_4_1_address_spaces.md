---
name: knowledge_pcie_chapter_4_section_4_1_address_spaces
description: PCIe 第4章§4.1 配置、memory、IO 三类地址空间及可预取属性。
---

# 知识点摘要

- PCIe 支持配置空间、memory 地址空间和 IO 地址空间三类地址空间。
- 配置空间用于标准化控制/检查设备状态，也可放置设备特定控制、状态和指针寄存器。
- 新设备通常使用 MMIO；IO 空间因传统兼容性保留，PCIe 不鼓励使用且可能在未来版本弃用。
- PCIe memory 地址可达 64 bit，IO 映射限制为 32 bit；许多 x86 系统实际只使用低 16 bit IO。

# 关键细节

- P-MMIO 的两个属性是读取无副作用、允许写合并，因此可以安全预取；NP-MMIO 可能包含“读取即清除”的状态寄存器，不能推测性读取或随意合并。
- 预取在 PCI 中尤其有助于 Bridge 猜测跨总线读传输量；PCIe 请求自身携带传输量信息，所以该区分的重要性较低，但为兼容性仍保留。
- MMIO/IO 映射不只适用于 Endpoint，Switch 和 Root Complex 内部也可有通过这些空间访问的寄存器。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 版本/日期：PDF 元数据标注 2026年6月1日
- 位置：第4章 §4.1–§4.1.2.2，PDF p.122–p.124（书内 p.2–p.4）
- 依据：`pdftotext -layout` 提取 PDF p.122–p.125，核对三类空间、MMIO/IO、P-MMIO/NP-MMIO 定义和示例。

# 适用条件与例外

- 地址宽度与实际可用范围仍由平台 CPU、RC 和操作系统资源决定；书中 64 bit/32 bit 是 PCIe 地址空间能力概述。
- 是否可预取必须依据设备寄存器读取副作用定义，不能仅按“memory”名称判断。

# 关联章节

- §4.2 BAR；§4.3 Bridge Base/Limit；第3章配置空间

# 待核验问题

- 无（本周期范围内的定义和限制已核对）。
