---
name: knowledge_apb_3_transfers
description: APB 写/读传输、等待周期、写选通、错误响应和 PPROT 保护编码
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T03:08:49.490Z
---

# APB Chapter 3：Transfers

## 知识点摘要

APB transfer 至少由 Setup phase 和 Access phase 组成：

- Setup phase：地址、方向、选择、写数据等控制信息被建立。
- Access phase：`PENABLE` asserted；如果 slave 通过 `PREADY` 表示 ready，则 transfer 在下一个 `PCLK` 上升沿完成；如果 `PREADY` 为 LOW，则 Access phase 被扩展。

写传输、读传输、错误响应和保护属性都遵循这个基本握手模型。

## 写传输

### 无等待周期写传输

在无 wait state 的基本写传输中：

1. 在 T1，write transfer 开始，`PADDR`、`PWDATA`、`PWRITE` 和 `PSEL` 在 `PCLK` 上升沿被寄存。这一周期称为 Setup phase。
2. 在 T2，`PENABLE` 和 `PREADY` 在 `PCLK` 上升沿被寄存。
3. `PENABLE` asserted 表示 transfer 的 Access phase 开始。
4. `PREADY` asserted 表示 slave 可以在下一个 `PCLK` 上升沿完成 transfer。
5. `PADDR`、`PWDATA` 和控制信号保持有效，直到 transfer 在 Access phase 结束时完成。
6. transfer 结束时 `PENABLE` deasserted；`PSEL` 也 deasserted，除非下一笔 transfer 立即访问同一个 peripheral。

### 有等待周期写传输

在 Access phase 期间，当 `PENABLE` 为 HIGH 时，slave 可以通过驱动 `PREADY` 为 LOW 来扩展 transfer。`PREADY` 保持 LOW 时，下列信号必须保持不变：

- `PADDR`
- `PWRITE`
- `PSEL`
- `PENABLE`
- `PWDATA`
- `PSTRB`
- `PPROT`

`PREADY` 在 `PENABLE` 为 LOW 时可以取任意值。因此，固定两周期访问的 peripheral 可以把 `PREADY` tied HIGH。

文档还建议：transfer 结束后不要立即改变 address 和 write signals，而是保持稳定直到下一次 access，以降低功耗。

## 写选通 `PSTRB`

`PSTRB` 支持 write data bus 上的 sparse data transfer：

- 每个 write strobe 对应 write data bus 的一个 byte。
- 当某个 strobe 为 HIGH 时，表示对应 byte lane 上包含有效信息。
- 每 8 bit write data 有一个 strobe。
- 编码关系：`PSTRB[n]` 对应 `PWDATA[(8n+7):(8n)]`。
- 32-bit data bus 上通常对应 `PSTRB[3:0]` 四个 byte lane。
- 对于 read transfer，bus master 必须驱动 `PSTRB` 的所有 bit 为 LOW。

## 读传输

### 无等待周期读传输

读传输中，address、write、select、enable 信号的时序与写传输相同。差异在于：slave 必须在读传输结束前提供 `PRDATA`。

### 有等待周期读传输

当 `PREADY` 在 Access phase 中被驱动为 LOW 时，读传输被扩展。协议保证下列信号在额外周期内保持不变：

- `PADDR`
- `PWRITE`
- `PSEL`
- `PENABLE`
- `PPROT`

可以通过 `PREADY` 添加任意数量的额外周期，数量可以从 0 开始。

## 错误响应 `PSLVERR`

`PSLVERR` 用于指示 APB transfer 的 error condition。错误可以发生在 read transaction 和 write transaction 上。

关键规则：

- `PSLVERR` 只在 APB transfer 的最后一个周期有效。
- 有效采样条件是 `PSEL`、`PENABLE` 和 `PREADY` 全部为 HIGH。
- 推荐但非强制：当 `PSLVERR` 未被采样时驱动 LOW，即当 `PSEL`、`PENABLE` 或 `PREADY` 任一为 LOW 时驱动 LOW。
- 收到 error 的 transaction 可能已经改变 peripheral 状态，也可能没有改变；这是 peripheral-specific，二者都可接受。
- write transaction 返回 error 不代表 peripheral 内部寄存器没有被更新。
- read transaction 返回 error 时可以返回 invalid data；文档不要求 peripheral 在 read error 时把 data bus 驱动为全 0。
- APB peripheral 不要求必须支持 `PSLVERR`。不支持时，APB bridge 对应输入 tied LOW。

### `PSLVERR` 到其它总线响应的映射

从 AXI 到 APB bridge 时：

- APB error 映射回 `RRESP/BRESP = SLVERR`。
- 具体通过把 `PSLVERR` 映射到 AXI read 的 `RRESP[1]` 和 write 的 `BRESP[1]` 实现。

从 AHB 到 APB bridge 时：

- `PSLVERR` 映射回 `HRESP = ERROR`，适用于 reads 和 writes。
- 具体通过把 `PSLVERR` 映射到 AHB `HRESP[0]` 实现。

## 保护单元支持 `PPROT[2:0]`

APB interface 通过 `PPROT[2:0]` 提供 protection 信息，用于支持 interconnect 和其它系统设备阻止 illegal transactions。

### `PPROT[0]`：normal / privileged

- LOW：normal access。
- HIGH：privileged access。
- 用于某些 masters 表示 processing mode。privileged processing mode 通常在系统中有更高访问权限。

### `PPROT[1]`：secure / non-secure

- LOW：secure access。
- HIGH：non-secure access。
- 文档特别说明该 bit 的配置是 HIGH 表示 non-secure、LOW 表示 secure。

### `PPROT[2]`：data / instruction

- LOW：data access。
- HIGH：instruction access。
- 这是一个 hint，并非在所有情况下都准确。
- 若 transaction 包含 instruction 与 data 的混合项，推荐默认标记为 data access，除非明确知道是 instruction access。

### 编码汇总

| bit | 1 | 0 |
| --- | --- | --- |
| `PPROT[0]` | privileged access | normal access |
| `PPROT[1]` | non-secure access | secure access |
| `PPROT[2]` | instruction access | data access |

文档 notes：`PPROT` 的 primary use 是作为 Secure / Non-secure transaction 的 identifier；对于 `PPROT[0]` 和 `PPROT[2]`，可以采用不同解释。

## 与 AXI protection 属性的关系

APB4 `PPROT[2:0]` 和 AXI 的 `AxPROT[2:0]` 在 bit 含义上对应：privileged、non-secure、instruction 三类属性。可与 [[knowledge_a4_transaction_attributes]] 对照使用。

## 原文位置

- 文档：AMBA APB Protocol Specification，Version 2.0 / Issue C，ARM IHI 0024C。
- Chapter 3 Transfers，3.1 “Write transfers”，文档页 3-2 至 3-3，PDF 物理页约 17 至 18。
- Chapter 3 Transfers，3.2 “Write strobes”，文档页 3-4，PDF 物理页约 19。
- Chapter 3 Transfers，3.3 “Read transfers”，文档页 3-5，PDF 物理页约 20。
- Chapter 3 Transfers，3.4 “Error response”，文档页 3-6 至 3-7，PDF 物理页约 21 至 22。
- Chapter 3 Transfers，3.5 “Protection unit support”，文档页 3-8 至 3-9，PDF 物理页约 23 至 24。

## 关联知识

- [[knowledge_apb_index]]
- [[knowledge_apb_2_signal_descriptions]]
- [[knowledge_apb_1_introduction]]
- [[knowledge_a4_transaction_attributes]]
