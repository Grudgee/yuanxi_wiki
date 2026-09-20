---
name: knowledge_pcie_chapter_5_section_5_1_packet_protocol
description: PCIe 第5章§5.1包协议基础、TLP/DLLP/Ordered Set与包完整性机制。
---

# 知识点摘要
- PCIe 活跃链路的基本信息单元是 Packet，主要包括事务层 TLP、数据链路层 DLLP 和物理层 Ordered Set。
- TLP/DLLP 使用明确格式和边界；Gen1/Gen2 通过 STP/END 控制符号组帧，Gen3 改用 STP token 携带包长度。
- 数据链路层为 TLP 加 Sequence Number 与 LCRC，并在 Replay Buffer 保存副本；Ack/Nak 提供链路级检测与重传。

# 关键细节
- Ordered Set 不使用 TLP/DLLP 的组帧符号和字节条带化，而是在每条 Lane 复制传输。
- TLP Header 字段固定，地址可为32/64 bit；接收方不能暂停或提前终止已开始的包。
- Gen3 128b/130b 不再使用 Gen1/2 的传统控制字符，包尾由 STP token 的大小信息确定。

# 原文引用
- 文档：PCI Express Technology 3.0 中文版；位置：第5章§5.1，PDF p.172–173。
- 依据：包类型、组帧、CRC、序列号和 Ack/Nak 动机说明。

# 适用条件与例外
- 本节为高层协议概述；Gen3 物理编码细节见后续物理层章节。

# 关联章节
- §5.2 TLP 细节；第9章 DLLP；第10章 Ack/Nak；第12章 Gen3 物理层

# 待核验问题
- 无。
