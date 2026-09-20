---
name: knowledge_pcie_chapter_5_section_5_2_5_config_completion
description: PCIe 第5章§5.2.5.3–§5.2.5.4配置请求与Completion TLP的字段、状态和分段返回。
---

# 知识点摘要

Configuration TLP 复用 PCI 的 Type 0/Type 1 模型：Type 0 面向当前总线设备，Type 1 穿过 Bridge 向下游总线传播。Completion 是对 Non-Posted 请求的响应，使用 Completer ID、Requester ID、Tag、状态、Byte Count 和 Lower Address 把响应与原请求关联起来。

# 配置请求

- Configuration Read/Write 的 Type 字段分别为 CfgRd0/CfgWr0 或 CfgRd1/CfgWr1；Header 固定为 3DW，不携带 64-bit Memory 地址。
- Type 0 请求到达目标 Secondary Bus 后，由设备解码 Device Number、Function Number、Register Number 和 First DW Byte Enable；外部链路上的 Endpoint 通常为 Device 0。
- Type 1 请求只由 Bridge 处理。Bridge 比较目标 Bus 与 Secondary/Subordinate 范围：目标等于 Secondary 时转换为 Type 0；目标仍在下游范围内时继续转发 Type 1；不在范围内则不接收。
- Configuration Write 可写配置寄存器，Configuration Read 返回寄存器内容；配置访问中的 Byte Enable 选择目标 DW 的有效字节。

# Completion Header

- Completion/Cpl 是无数据完成包，CplD 是带数据完成包；两者均使用 3DW Header。
- Completer ID 标识生成响应的 Function；Requester ID 和 Tag 从请求中复制，使 Requester 能在多个未完成事务中匹配对应响应。
- Completion Status 表示 Successful Completion、Unsupported Request、Completer Abort 或 CRS 等结果。状态与是否有数据必须符合请求类型。
- Byte Count 表示当前 Completion 以及后续 Completion 尚需返回的总字节数；Lower Address 表示本包第一个有效字节在原始请求中的低地址位置。

# 多包读返回

- 一个 Memory Read 可以拆成多个 CplD。Requester 根据 Requester ID/Tag 识别同一事务，再依据 Byte Count、Lower Address 和完成顺序拼接数据。
- 首个、末个和中间 Completion 的数据量受最大 Payload、Read Completion Boundary（RCB）以及自然对齐限制；中间包应在 RCB 边界结束。
- Completion 不应超出原始请求要求的字节数；状态错误时通常返回不带数据的 Completion。
- Completer 必须检查请求是否存在、是否支持、是否有权限以及是否能在规定时间内完成；无法支持时返回 UR，无法完成时可报告 Completer Abort，暂未准备好时使用 CRS。

# 接收者处理

- Requester 收到 Completion 后先由 Physical/Data Link Layer 完成解码、LCRC/序列号检查，再由 Transaction Layer 检查 ECRC（若启用）、Requester ID、Tag、状态、Byte Count、Lower Address 和数据边界。
- 不匹配未完成请求的 Completion、长度超出请求范围、Byte Count 不一致或非法状态可能导致 Unexpected Completion/Malformed TLP 等错误。
- CRS 只允许作为 Configuration 请求的响应；把 CRS 用于普通 Memory/IO 请求会形成协议错误。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 位置：§5.2.5.3–§5.2.5.4.6，PDF p.197–205；本周期源文本约 17,817 字符。
- 依据：Configuration Request Header、Completion Header、状态码、Lower Address、Byte Count Modified、读数据返回和接收者处理规则。

# 适用条件与例外

- Byte Count Modified 主要用于 PCI-X 兼容语义；Native PCIe 设备应按 PCIe Completion 规则处理。
- Completion 的具体合法分段仍受 RCB、最大 Payload 和链路流量控制影响。

# 关联章节

- 第3章配置访问与枚举；§5.2.4字节使能；第8章事务排序；第15章错误报告。

# 待核验问题

- 无。
