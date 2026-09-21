---
name: knowledge_pcie_chapter_9_dllp_elements
description: PCIe第9章DLLP元素：本地流量、固定8字节格式、Ack/Nak、Power Management、Flow Control和Vendor-Specific DLLP。
---

# 学习范围

- 位置：第9章，PDF p.314–324
- 本周期源文本约4,653字符；与下一章节前段合计满足本周期15,000字符要求，章节分界单独建立本文件。

# DLLP的本地性质

DLLP 只在一条链路两端的相邻 Data Link Layer 之间传输，不被 Switch/RC 作为 TLP 路由到更远设备。DLLP 用于维护当前链路：确认/否定确认 TLP、更新流量控制 Credit、协商链路电源管理以及厂商特定的本地信息。

- TLP 可跨多条链路，DLLP 不能；DLLP 的接收者永远是相邻端口。
- Physical Layer 和 Data Link Layer 都可能检查 DLLP；CRC 错误的 DLLP 被丢弃，不上送 Transaction Layer。
- DLLP 错误不能像 TLP 那样通过 Ack/Nak 重传 DLLP 本身，链路逻辑必须依据超时、Credit 或协议状态恢复。

# 固定包结构

- 原生 DLLP 核心长度固定为8 byte：4 byte DLLP 核心字段、2 byte CRC、物理层组帧/编码开销。
- DLLP 不携带数据 Payload；全部语义位于核心字段。Gen1/Gen2 使用 STP/END，Gen3 使用相应物理层 token/格式。
- DLLP 的 CRC 是16 bit，与 TLP 的32 bit LCRC不同；接收端重新计算并比较。

# Ack/Nak DLLP

Ack DLLP 携带已正确接收的 TLP 序列号；发送端用该序列号释放 Replay Buffer 中已确认的 TLP。Nak DLLP 指示序列号或 LCRC 错误，发送端从对应序列号开始重放。Ack/Nak 只作用于相邻链路，不能当作端到端 Completion。

# Power Management DLLP

Power Management DLLP 用于相邻链路的电源状态协调，例如进入/退出低功耗状态。它不会被路由到 Root Complex 或 Endpoint 的远端端口；系统级电源语义通过 Message、配置寄存器和链路状态机共同完成。

# Flow Control DLLP

Flow Control DLLP 携带 VC 的 Posted、Non-Posted、Completion Header/Data Credit 更新。接收端消耗 Buffer 后发送 UpdateFC，发送端据此增加可用计数；Credit DLLP 的延迟和丢失会影响发送端的可用窗口。

# Vendor-Specific DLLP

Vendor-Specific DLLP 用于双方明确约定的实现扩展。它仍必须符合固定长度、CRC 和链路本地边界；不能借此创建未经协议定义的拓扑路由。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版；位置：第9章§9.1–§9.5.4，PDF p.314–324。

# 适用条件与例外

- DLLP 的具体字段编码和时序由 Ack/Nak、流控、电源管理章节细化；本文件记录第9章的元素和作用边界。

# 待核验问题

- 无。
