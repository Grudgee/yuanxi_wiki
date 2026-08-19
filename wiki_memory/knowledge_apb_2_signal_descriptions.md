---
name: knowledge_apb_2_signal_descriptions
description: APB 信号定义、方向、数据总线和 APB3/APB4 新增信号
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T03:08:49.187Z
---

# APB Chapter 2：Signal Descriptions

## 知识点摘要

Chapter 2 用表格定义 AMBA APB 接口信号，核心分工是：APB bridge 驱动地址、控制、写数据和保护属性；slave interface 驱动读数据、ready 和错误响应。所有 APB 传输都由 `PCLK` 上升沿定时。

APB 有独立的读数据总线和写数据总线，但它们没有各自独立的握手机制，因此不能在同一时刻同时发生读传输和写传输。

## 信号分组

### 时钟与复位

- `PCLK`：时钟源。`PCLK` 的上升沿对 APB 上所有传输进行定时。
- `PRESETn`：系统总线等价复位信号。APB reset 为 active LOW，通常直接连接到系统总线复位信号。

### APB bridge 驱动的信号

- `PADDR`：地址信号，最多 32 bit，由 peripheral bus bridge unit 驱动。
- `PPROT`：保护类型，表示 transaction 的 normal/privileged、secure/non-secure，以及 data/instruction 属性。详见 [[knowledge_apb_3_transfers]]。
- `PSELx`：select 信号。APB bridge 为每个 peripheral bus slave 生成一个 `PSELx`，表示该 slave 被选中且需要一次 data transfer。
- `PENABLE`：enable 信号，表示 APB transfer 的第二个周期及后续周期。
- `PWRITE`：方向信号。HIGH 表示 APB write access，LOW 表示 APB read access。
- `PWDATA`：写数据总线。`PWRITE` 为 HIGH 的写周期中由 peripheral bus bridge unit 驱动，最多 32 bit。
- `PSTRB`：写选通信号，表示写传输中要更新哪些 byte lane。每 8 bit 写数据对应 1 个 write strobe，`PSTRB[n]` 对应 `PWDATA[(8n+7):(8n)]`。读传输期间 write strobe 不得有效。

### Slave interface 驱动的信号

- `PREADY`：ready 信号。slave 使用该信号扩展 APB transfer。
- `PRDATA`：读数据总线。被选中的 slave 在 `PWRITE` 为 LOW 的读周期中驱动该总线，最多 32 bit。
- `PSLVERR`：传输失败指示。APB peripheral 不要求必须支持 `PSLVERR`；如果某个 peripheral 设计不包含此引脚，则 APB bridge 对应输入应 tied LOW。

## 数据总线

APB protocol 有两个独立的数据总线：

- read data bus：`PRDATA`
- write data bus：`PWDATA`

两者都最多 32 bit 宽。由于这两个总线没有各自独立的 handshake signal，所以不能在两个总线上同时发生数据传输。

## 适用条件与注意事项

- `PSELx` 是按 slave 分发的 select 信号；每个 slave 有一个对应的 `PSELx`。
- `PSTRB` 只用于写传输；读传输时必须不激活。
- `PSLVERR` 是可选错误响应支持；不支持时由桥侧输入固定为 LOW。
- APB4 引入的 `PPROT` 与 `PSTRB` 分别服务于 transaction protection 和 sparse data transfer，和 Chapter 1 的 APB4 修订说明一致。见 [[knowledge_apb_1_introduction]]。

## 原文位置

- 文档：AMBA APB Protocol Specification，Version 2.0 / Issue C，ARM IHI 0024C。
- Chapter 2 Signal Descriptions，2.1 “AMBA APB signals”，Table 2-1 “APB signal descriptions”，文档页 2-2，PDF 物理页约 15。
- Chapter 2 Signal Descriptions，2.1.1 “Data buses”，文档页 2-2，PDF 物理页约 15。

## 关联知识

- [[knowledge_apb_index]]
- [[knowledge_apb_1_introduction]]
- [[knowledge_apb_3_transfers]]
