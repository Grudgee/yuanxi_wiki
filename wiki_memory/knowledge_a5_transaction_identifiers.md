---
name: knowledge_a5_transaction_identifiers
description: AXI 事务 ID、同 ID 有序性、读写数据排序和 interconnect ID 路由
metadata: 
  node_type: memory
  originSessionId: 07d5e602-b8a4-4424-94dd-570763cdaafc
  modified: 2026-07-21T09:22:32.549Z
---

# 知识点摘要

- AXI transaction ID 用于区分必须按序完成的事务，并支持多个 outstanding 事务和乱序完成。
- 同一 AXI ID 的事务必须保持顺序；不同 ID 之间没有排序限制。组件不强制必须使用 ID，但 Subordinate 必须在 `BID`/`RID` 中返回对应 ID。
- 读地址通道使用 `ARID`，读数据通道使用 `RID`；写地址使用 `AWID`，写响应使用 `BID`；AXI3 写数据还有 `WID`，后续协议不使用。
- 同一 `ARID` 的读数据必须按地址发行顺序返回；Manager 发出的写数据必须与写地址顺序一致。AXI4 及以后不允许 AXI3 那样按不同 ID 交错写数据。
- Interconnect 可在 ID 中附加 Manager 端口标识，使不同 Manager 的 ID 不冲突，并在返回响应时去掉附加位、路由到正确 Manager。

# 关键细节

- 读数据重排深度是 Subordinate 中可重排的 pending 地址数量；完全按序处理的 Subordinate 深度为 1。Manager 无法通过协议查询该深度。
- Interconnect 转发到 Subordinate 时，Subordinate 侧 ID 可能比 Manager 侧更宽；读数据用附加 `RID` 位选择返回的 Manager 端口，随后 interconnect 移除附加位。
- 写响应使用附加 `BID` 位选择目标 Manager，返回前移除附加位。

# 原文引用

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 版本：ARM IHI 0022H.c
- PDF 物理页：79–82；印刷页 A5-79–A5-82。
- 章节：Chapter A5 `Transaction Identifiers`，A5.1–A5.2.3。

# 适用条件与例外

- `WID` 仅 AXI3 实现；AXI4/AXI5 使用写地址 ID 关联写数据。
- “同 ID 有序”不等于不同 ID 或不同 Manager 之间有序；需要全局排序时应依据 A6 规则。

# 关联章节

- Chapter A3 Single Interface Requirements
- Chapter A6 AXI Ordering Model

# 待核验问题

- 后续需结合 ACE5 的 snoop/一致性通道确认 ID 扩展对一致性事务的影响。
