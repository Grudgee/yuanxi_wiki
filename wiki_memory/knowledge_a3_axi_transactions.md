---
name: knowledge_a3_axi_transactions
description: AXI 单接口时钟复位、VALID/READY 握手、通道依赖和事务结构
metadata: 
  node_type: memory
  originSessionId: 07d5e602-b8a4-4424-94dd-570763cdaafc
  modified: 2026-07-21T09:09:48.399Z
---

# 知识点摘要

- AXI 接口使用单一全局时钟 `ACLK`；输入信号在 `ACLK` 上升沿采样，输出信号只能在上升沿之后改变。Manager 和 Subordinate 接口的输入到输出之间不得存在组合路径。
- 复位信号 `ARESETn` 为 active-Low，可异步置位，但只能与 `ACLK` 上升沿同步解除。复位期间 Manager 必须将 `ARVALID`、`AWVALID`、`WVALID` 拉低，Subordinate 必须将 `RVALID`、`BVALID` 拉低；其他信号可为任意值。
- 五个 AXI 事务通道都采用独立的 `VALID/READY` 两相握手；只有在同一个 `ACLK` 上升沿 `VALID=1` 且 `READY=1` 时才发生传输。
- 发送端不能等待接收端先拉高 `READY` 才拉高 `VALID`；发送端一旦拉高 `VALID`，必须保持到握手完成。接收端可以先拉高 `READY`，也可以等待 `VALID`。
- 五个通道握手对分别是：写地址 `AWVALID/AWREADY`、写数据 `WVALID/WREADY`、写响应 `BVALID/BREADY`、读地址 `ARVALID/ARREADY`、读数据 `RVALID/RREADY`。
- AXI 通道之间基本独立，但读数据必须跟随读地址，写响应必须跟随完整写事务；依赖关系必须避免死锁。
- AXI 为 burst-based 协议：Manager 只发送 burst 首个字节的地址和控制信息，Subordinate 根据 burst 参数计算后续地址；burst 不得跨越 4KB 地址边界。

# 关键细节

## 握手规则

- 写地址：`AWVALID` 只能在 Manager 驱动有效地址/控制信息时置位，并保持到 Subordinate 在上升沿前置 `AWREADY` 后的握手完成。规范建议 `AWREADY` 默认 HIGH，但只有 Subordinate 能单周期接受任意地址时才适用。
- 写数据：`WVALID` 只能在有效写数据存在时置位，并保持到 `WREADY` 握手；最后一个 beat 必须伴随 `WLAST`。`WDATA` 的非活动字节 lane 建议驱动为 0。
- 写响应：Subordinate 仅在有效写响应存在时置位 `BVALID`，保持至 Manager 接受；AXI4/ACE 等场景下，Subordinate 必须等待规定的地址和数据握手依赖后才能产生响应。
- 读地址：`ARVALID` 保持至 `ARREADY` 握手；规范建议 `ARREADY` 默认 HIGH，但前提是 Subordinate 能立即接受地址。
- 读数据：Subordinate 用 `RVALID` 表示有效读数据，必须保持至 `RREADY`；最后一个 beat 必须伴随 `RLAST`。Manager 用 `RREADY` 表示可接受数据，默认 HIGH 仅在能立即接受时适用。

## 跨通道依赖

- 读事务中，Manager 不得等待 `ARREADY` 后才发 `ARVALID`；Subordinate 可以先发 `ARREADY`，但读数据有效必须依赖 `ARVALID` 与 `ARREADY` 的地址握手。
- AXI3 写事务中，Manager 不得等待 `AWREADY` 或 `WREADY` 后才发 `AWVALID`/`WVALID`。Subordinate 可以等待 `AWVALID`、`WVALID` 或两者后再发对应 `READY`；`BVALID` 必须等待 `AWVALID`、`AWREADY`、`WVALID`、`WREADY`，且还必须等待 `WLAST`。
- AXI4/AXI5 增加更严格的写响应依赖：Subordinate 在置位 `BVALID` 前必须等待 `AWVALID`、`AWREADY`、`WVALID`、`WREADY`，并等待 `WLAST`。这体现了写响应必须在最后一个写数据 beat 完成之后产生。
- 依赖图中的单箭头表示目标信号可在源信号之前或之后置位；双箭头表示目标只能在源已经置位后置位。
- 不遵守依赖可能导致死锁。例如 Manager 等待 `AWREADY` 后才发 `AWVALID`，而 Subordinate 又等待 `AWVALID` 才发 `AWREADY`。

## Burst 地址和长度

- 读 burst 长度由 `ARLEN[7:0]`，写 burst 长度由 `AWLEN[7:0]` 指定；`Burst_Length = AxLEN + 1`。
- AXI3 所有 burst 类型支持 1–16 个 transfers；AXI4 对 `INCR` 支持 1–256 个 transfers，其他 burst 类型仍为 1–16 个。
- `AxSIZE[2:0]` 指定每个 beat 的字节数：`000=1`、`001=2`、`010=4`、`011=8`、`100=16`、`101=32`、`110=64`、`111=128` 字节。
- `AxBURST[1:0]` 编码：`00=FIXED`、`01=INCR`、`10=WRAP`、`11=Reserved`。
- FIXED：每个 beat 使用相同地址，适合 FIFO 反复访问；有效 byte lane 集合保持不变，但 `WSTRB` 可因 beat 而不同。
- INCR：每个 beat 地址按 transfer 大小递增，适合顺序内存访问。
- WRAP：地址递增到 wrap boundary 后回绕；起始地址必须按 transfer 大小对齐，长度只能为 2、4、8 或 16，常用于 cache line 访问。
- AXI4 中长度超过 16 的 INCR burst 可转换为多个更短 burst；生成的 burst 必须保留原事务特性，仅缩短长度并适当调整地址。
- burst 不得跨越 4KB 边界；规范说明这也限制了一个 burst 中可发出的地址数量，并防止跨越两个 Subordinate 的地址空间。

## 地址计算

- `Start_Address = AxADDR`
- `Number_Bytes = 2 ^ AxSIZE`
- `Burst_Length = AxLEN + 1`
- `Aligned_Address = INT(Start_Address / Number_Bytes) * Number_Bytes`
- INCR 或未发生回绕的 WRAP：`Address_N = Aligned_Address + (N - 1) * Number_Bytes`
- `Wrap_Boundary = INT(Start_Address / (Number_Bytes * Burst_Length)) * (Number_Bytes * Burst_Length)`
- 若 WRAP 当前地址到达 `Wrap_Boundary + Number_Bytes * Burst_Length`，则回到 `Wrap_Boundary`。
- 事务容器大小：`Container_Size = Number_Bytes * Burst_Length`；INCR 容器从 `Aligned_Address` 开始，WRAP 容器从 `Wrap_Boundary` 开始。

## Regular transactions

- Regular 事务要求：`AxLEN` 为 1、2、4、8 或 16；当 `AxLEN > 1` 时 `AxSIZE` 等于数据总线宽度；`AxBURST` 为 INCR 或 WRAP；INCR 时 `AxADDR` 对齐事务容器，WRAP 时 `AxADDR` 按 `AxSIZE` 对齐。
- `Regular_Transactions_Only=True` 表示仅支持 Regular 事务，False 表示支持 `AxBURST`、`AxSIZE`、`AxLEN` 的所有组合；未声明时视为 False。该属性可用于 AXI5、ACE、ACE5、ACE5-Lite 和 ACE5-LiteDVM 接口。
- Manager 支持所有组合而 Subordinate 只支持 Regular 事务时不兼容，可能产生数据损坏或死锁；两边均支持或 Subordinate 支持所有组合时才兼容。

## 数据、字节 lane 和端序

- 每 8 个写数据位对应一个 `WSTRB` 位；`WSTRB[n]` 为 HIGH 表示 `WDATA[(8n)+7:(8n)]` 包含有效数据。Manager 只能对有效字节 lane 置 HIGH；`WVALID=LOW` 时 strobe 可任意值。
- Narrow transfer 中，递增或回绕 burst 的每个 beat 可使用不同 byte lane；FIXED burst 使用相同 lane。传输大小不能超过事务任一 agent 的数据总线宽度。
- AXI 使用 byte-invariant endian scheme：同一个多字节元素使用连续内存字节；地址对应的数据总线 byte lane 与元素端序无关；同一地址传送该地址对应的 8 位数据。
- big-endian byte-invariant 示例中，最高有效字节位于寄存器 MSB，最低地址存放 MSB；little-endian 示例中，最低有效字节位于寄存器 LSB，最低地址存放 LSB。支持单一端序的组件可直接连接到 byte-invariant 接口；混合端序数据结构可以在同一内存空间共存。
- AXI 支持非对齐传输。Manager 可用低地址线表示未对齐起始地址，或提供对齐地址并用 byte strobe 表示未对齐位置；低地址线信息必须与 byte strobe 一致，Subordinate 不必自行推断。

## 读写响应结构

- 读响应通过读数据通道的 `RRESP[1:0]` 返回，因此 burst 中每个读 beat 可以有不同响应；写响应通过 `BRESP[1:0]` 返回，整个写 burst 只有一个响应。
- `RRESP/BRESP` 编码：`0b00=OKAY`、`0b01=EXOKAY`、`0b10=SLVERR`、`0b11=DECERR`。
- `OKAY` 表示普通访问成功，也可表示独占访问失败或目标不支持独占访问；`EXOKAY` 只用于表示独占读或独占写成功。
- `SLVERR` 表示事务已正确到达 Subordinate，但 Subordinate 向原始 Manager 返回错误。典型原因包括 FIFO/缓冲区溢出或下溢、不支持的传输大小、写只读位置、Subordinate 超时、访问已禁用或掉电的功能。
- `DECERR` 表示 interconnect 无法解码到目标 Subordinate；规范建议把此访问路由到默认 Subordinate 并由其返回 `DECERR`。
- 即使报告错误，协议要求仍执行事务规定的全部 data transfers。读 burst 可逐 beat 报告不同错误；写 burst 完成后统一返回一个响应。

# 原文引用

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 版本/日期：ARM IHI 0022H.c，Copyright 2003–2021。
- PDF 物理页：39–60；文档印刷页：A3-39–A3-60。
- 章节：Chapter A3 `Single Interface Requirements`；A3.1–A3.4.5。
- 关键位置：时钟/复位 A3-40；握手 A3-41–A3-43；通道依赖 A3-44–A3-47；burst 和地址 A3-48–A3-53；数据结构和端序 A3-54–A3-58；读写响应 A3-59–A3-60。

# 适用条件与例外

- AXI3、AXI4、AXI5 的 burst 长度和写响应依赖有差异；引用规则时必须明确协议版本。
- `AWLEN`/`ARLEN` 字段的 AXI3/AXI4 长度扩展和 AXI4/ACE5 的 regular 事务属性不能混为一谈。
- 本文件覆盖本批次 PDF 物理页 39–58；没有覆盖后续章节对信号参数、缓存属性、独占访问或一致性扩展的进一步约束。

# 关联章节

- Chapter A2 Signal Descriptions
- Chapter A4 Additional ACE5 requirements
- Chapter A5 AXI ordering model
- Chapter A6 AXI attributes
- Chapter A7 Exclusive accesses

# 待核验问题

- 需要在后续章节核对 ACE5 对 AXI 基础通道依赖、缓存属性和一致性事务的额外约束。
