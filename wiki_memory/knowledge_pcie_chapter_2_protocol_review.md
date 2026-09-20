---
name: knowledge_pcie_chapter_2_protocol_review
description: PCIe 第2章§2.3协议回顾，通过内存读请求和带数据完成包串联事务层、数据链路层与物理层。
---

# 知识点摘要

- MRd 请求由设备核心/软件层提供地址、事务类型、长度、TC、字节使能和属性；事务层构建 3DW/4DW Header 的 TLP，并放入相应 VC 发送缓存。
- 数据链路层为 TLP 加 12 bit 序列号和 32 bit LCRC，并保存重传副本；物理层完成组帧、字节条带化、扰码、8b/10b 编码和串行差分发送。
- 接收端按相反方向解码、去扰码、反条带化和去帧；数据链路层检查 LCRC/序列号并返回 Ack DLLP，错误时通过 Nak 触发重传。
- 完成方根据 Requester ID、Tag 等信息生成 CplD TLP；CplD 使用固定 3DW Header，并携带请求数据和完成状态，最终由请求方事务层交付软件层。

# 关键细节

- 32 位地址使用 3DW Header，64 位地址使用 4DW Header；MRd 的 Requester ID 使完成方能够返回完成包。
- 流量控制先确认对端 VC 接收缓存有足够空间。Ack 只在数据链路层处理，不上送事务层；发送方收到 Ack 后清除对应重传副本。
- 物理层接收包括串并转换、弹性缓存、符号解码、解扰和多通道反条带化；完成方可选执行 ECRC 检查。
- CplD 返回 Requester ID、Tag、完成类型、状态和数据；请求方事务层校验后将 Header、数据荷载和完成状态交给软件层。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 位置：第2章 §2.3–§2.3.2；PDF p.83–p.90 左右（书内 p.43–p.47）
- 依据：MRd 与 CplD 协议回顾示例，在术语翻译表后停止。

# 适用条件与例外

- 这是协议路径概述，不替代后续章节对 TLP 字段、DLLP 格式、链路训练和错误恢复的详细规范。
- ECRC 在示例中为可选端到端校验；LCRC/序列号是链路级可靠性机制。

# 关联章节

- 第5章 TLP 元素；第10章 Ack/Nak 协议；第14章链路初始化和训练

# 待核验问题

- 无。图2-32（PDF p.84）与图2-33（PDF p.86）已结合 §2.3.1/§2.3.2 正文核对：MRd 使用 3DW/4DW Header、Requester ID 和 VC/流量控制；数据链路层加入 12-bit Sequence Number 与 32-bit LCRC并保存重传副本；物理层组帧、条带化、扰码和编码。CplD 固定为 3DW Header，回传 Requester ID、Tag、Completer ID、状态和数据，并经历同样的 Ack/Nak 链路可靠性流程。
