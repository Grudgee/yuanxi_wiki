---
name: knowledge_pcie_chapter_5_section_5_2_5_io_memory
description: PCIe 第5章§5.2.5.1–§5.2.5.2 IO与Memory请求TLP格式、字段和传输约束。
---

# 知识点摘要

本周期覆盖 IO Request 与 Memory Request。IO 事务保留给传统设备；Memory 事务是 PCIe 设备主要的数据通路，分读、读锁定和写。请求 TLP 的 Header 携带路由地址或 ID、长度、字节使能、Requester ID、Tag、TC 和属性；读请求由 Completion 返回，写请求通常为 Posted。

# 关键细节

## IO 请求

- IO Read 与 IO Write 均使用 3DW Header，地址是 32 bit；IO 空间的目标通常是遗留设备的寄存器。
- IO Read/Write 在本书表格中属于需要确认的 Non-Posted 类型；这与 Memory Write 的 Posted 语义不同。
- IO 请求由 Type 字段区分读写，Byte Enable 选择首/尾 DW 中真正有效的字节。IO 路由仍使用地址路由，端口通过 IO Base/Limit 窗口判断下游范围。

## Memory 请求

- Memory Read（MRd）和 Memory Read Locked（MRdLk）不带数据，使用 3DW 或 4DW Header，分别对应低于 4GB 或 64-bit 地址；Completer 以 Cpl/CplD 返回。
- Memory Write（MWr）带数据，使用 3DW 或 4DW Header；Memory Write 是 Posted，不要求 Completion，但仍需要数据链路层 Ack/Nak。
- AtomicOp（Fetch-and-Add、Unconditional Swap、Compare-and-Swap）属于带数据的 Memory 类请求，按 Type 编码区分，目标必须支持相应原子操作能力。

## Header 与地址

- 32-bit 地址使用 3DW Header；64-bit Memory 地址使用 4DW Header。低于 4GB 的地址不应使用高位全零的 4DW Header，协议将该组合的接收行为视为未定义。
- Header 的 Length 表示数据荷载 DW 数；首/尾 DW Byte Enable 描述非 DW 对齐传输的有效字节。Requester ID 是发起 Function 的 BDF，Tag 用于同时存在多个未完成读请求时匹配 Completion。
- TC 在请求生成时写入，并沿路径保持不变；Attr 可表达 No Snoop、Relaxed Ordering 和 ID-based Ordering 等属性。

## 传输过程与限制

1. Device Core 提供地址/ID、事务类型、长度、字节使能、TC、Attr 和写数据。
2. Transaction Layer 选择合法的 Fmt/Type，构建 Header；读请求不附带数据，写/AtomicOp 附带数据。
3. 流量控制确认对端 VC 有足够 Header/Data Credit 后，TLP 才下传 Data Link Layer。
4. Data Link Layer 加 Sequence Number 与 LCRC，保存 Replay 副本；Physical Layer 条带化、扰码和编码后发送。

## 与 PCIe 路由的关系

- Memory/IO 请求使用地址路由；Switch/Bridge 先比较自身 BAR，再比较 IO、NP-MMIO 或 P-MMIO Base/Limit。
- Non-Posted 读的 Completion 通过 Requester ID 和 Tag 返回；Posted Memory Write 不返回 Completion，因此软件不能用 Completion 作为写入完成确认。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 位置：§5.2.5.1–§5.2.5.2.2，PDF p.188–196；本周期源文本约 15,411 字符。
- 依据：IO Request、Memory Request、Memory Header 字段和 Memory Request 注意事项。

# 适用条件与例外

- IO 事务主要服务 Legacy 兼容，不代表所有 Native Endpoint 都必须实现 IO。
- Locked、AtomicOp、No Snoop 和 Relaxed Ordering 都受设备能力、配置和具体事务类型约束。
- 本文件不展开 Completion Header 的状态码和多包返回，那些内容在下一周期记忆中处理。

# 关联章节

- §5.2.4 Header；§5.2.5.3 Configuration；§5.2.5.4 Completion；第4章地址路由；第8章排序。

# 待核验问题

- 无。
