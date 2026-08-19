---
name: knowledge_apb_1_introduction
description: APB 协议定位、基本特征和 APB2/APB3/APB4 修订差异
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T02:50:31.554Z
---

# 知识点摘要

APB（Advanced Peripheral Bus）属于 AMBA 协议族，是面向低带宽外设和控制寄存器访问的低成本接口。APB 的设计目标是低功耗、接口复杂度低，而不是 AXI 那样的高性能。APB 不是流水线协议，每次传输至少需要两个时钟周期，并且协议转换只发生在时钟上升沿，以简化外设集成。

# 关键细节

## APB 的定位

- APB 用于连接不需要 AXI 高性能的低带宽外设。
- 典型用途是访问外设的 programmable control registers。
- APB 可以与以下 AMBA 总线/接口互连：
  - AHB
  - AHB-Lite
  - AXI
  - AXI4-Lite

## 协议特性

- Non-pipelined：APB 不是流水线接口。
- 每次 transfer 至少需要两个 clock cycles。
- 协议相关信号转换到 rising edge of the clock，降低设计复杂度。
- 目标是 minimal power consumption 与 reduced interface complexity。

## APB 修订关系

- 老的 `APB Specification Rev E`（1998）已 obsolete，被后续三个修订取代：
  - AMBA 2 APB Specification
  - AMBA 3 APB Protocol Specification v1.0
  - AMBA APB Protocol Specification v2.0

## APB2 / APB3 / APB4 差异

- APB2：定义接口信号、基本读写传输、APB bridge 与 APB slave 两个组件。
- APB3：新增 wait states 与 error reporting；对应接口信号：
  - `PREADY`：ready signal，表示 APB transfer 完成。
  - `PSLVERR`：error signal，表示 transfer failure。
- APB4（本文档 Version 2.0）：新增 transaction protection 与 sparse data transfer；对应接口信号：
  - `PPROT`：protection signal，支持 APB 上 non-secure 与 secure transactions。
  - `PSTRB`：write strobe signal，在写数据总线上支持 sparse data transfer。

# 原文引用

- 文档：AMBA APB Protocol Specification，Version 2.0，ARM IHI 0024C (ID042910)，Issue C，13 April 2010。
- 位置：Chapter 1 Introduction，1.1 “About the APB protocol”，文档页 1-2 / PDF 物理页 12。
  - 原文要点：APB 是 AMBA family 的一部分；定义低成本接口；优化 minimal power consumption 与 reduced interface complexity；不是 pipelined；用于低带宽外设；每次 transfer 至少两个 cycles；可连接 AHB/AHB-Lite/AXI/AXI4-Lite；可访问外设 programmable control registers。
- 位置：Chapter 1 Introduction，1.2 “APB revisions”，文档页 1-3 / PDF 物理页 13。
  - 原文要点：APB Specification Rev E 已 obsolete；APB3 增加 wait states/error reporting 和 `PREADY`/`PSLVERR`；APB4 增加 transaction protection/sparse data transfer 和 `PPROT`/`PSTRB`。

# 适用条件与例外

- APB 适合控制寄存器和低带宽外设访问，不适合作为高吞吐、高并发、流水线数据通路的主接口。
- 需要等待周期或错误报告能力时，应关注 APB3 及以后版本的 `PREADY`、`PSLVERR`。
- 需要安全/非安全等保护属性或 byte/sparse write 能力时，应关注 APB4 的 `PPROT`、`PSTRB`。

# 关联章节

- [[knowledge_apb_index]]
- Chapter 2 Signal Descriptions：后续需学习 `PREADY`、`PSLVERR`、`PPROT`、`PSTRB` 等信号定义。
- Chapter 3 Transfers：后续需学习 wait states、error response、write strobes、protection encoding。
- [[knowledge_a4_transaction_attributes]]：AXI `AxPROT` 可作为理解 APB4 `PPROT` 的对照。

# 待核验问题

- `PPROT` 的各 bit 编码与 AXI `AxPROT` 是否完全对应，需要在 Chapter 3.5 “Protection unit support” 和 Table 3-1 中核验。
- `PSTRB` 对不同 `PWDATA` 宽度的 byte lane 映射规则需要在 Chapter 3.2 和 Figure 3-3 中核验。
