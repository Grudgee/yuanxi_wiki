---
name: knowledge_axiace5_front_matter_a1_a2
description: AXIACE5.pdf 前言、目录、A1 Introduction 与 A2 Signal Descriptions 的补充记忆
metadata:
  node_type: memory
  source: amba_prot/AXIACE5.pdf
  learned: 2026-08-21
  source_pages: PDF 1-43
---

# 前言、目录、A1 与 A2

## 前言与版本信息

- 文档标题为 `AMBA AXI and ACE Protocol Specification`，版本为 `ARM IHI 0022H.c`。
- 版本历史显示：从 A 到 H.c 覆盖了 AXI、ACE 以及 AMBA 5 相关扩展的逐步加入。
- F 版开始引入 AXI5、AXI5-Lite、ACE5、ACE5-Lite、ACE5-LiteDVM、ACE5-LiteACP。
- H.b 更正术语为 `Manager` 与 `Subordinate`。

## 文档结构

- Part A：AXI 协议。
- Part B：AXI4-Lite。
- Part C：AXI5 与 AXI5-Lite。
- Part D：ACE 与 ACE-Lite。
- Part E：AMBA 5 protocol features。
- Part F：ACE5 家族。
- Part G：appendices。

## A1 Introduction

- AXI 面向高性能、高频率系统设计，适合高带宽、低延迟和高初始访问延迟的内存系统。
- 核心特征包括：地址/控制与数据分离、支持非对齐传输、burst 事务、多 outstanding、乱序完成、便于插入 register slice。
- AXI 通过独立的读地址、读数据、写地址、写数据、写响应五个通道组织事务。
- AXI architecture 允许多种 interconnect 形态：共享地址和数据总线、共享地址总线+多个数据总线、multilayer。
- 由于通道彼此独立，几乎任意通道位置都能插入 register slice，以延迟换频率。

## A1 术语

- `Manager` 是发起事务的 agent，`Subordinate` 是响应事务的 agent。
- `Transaction` 是 AXI 总线上完成一次操作所需的完整信号集合。
- `Burst` 是 payload 数据在总线上的传输集合，`Beat` 是 burst 中的单次数据传输。
- `Upstream` / `Downstream` 描述 topology 中相对位置。

## A2 Signal Descriptions

- 全局信号只有 `ACLK` 与 `ARESETn`，同步信号在 `ACLK` 上升沿采样，`ARESETn` 低有效。
- 写地址通道信号包括 `AWID/AWADDR/AWLEN/AWSIZE/AWBURST/AWLOCK/AWCACHE/AWPROT/AWQOS/AWREGION/AWUSER/AWVALID/AWREADY`。
- 写数据通道包括 `WDATA/WSTRB/WLAST/WUSER/WVALID/WREADY`。
- 写响应通道包括 `BID/BRESP/BUSER/BVALID/BREADY`。
- 读地址通道包括 `ARID/ARADDR/ARLEN/ARSIZE/ARBURST/ARLOCK/ARCACHE/ARPROT/ARQOS/ARREGION/ARUSER/ARVALID/ARREADY`。
- 读数据通道包括 `RID/RDATA/RRESP/RLAST/RUSER/RVALID/RREADY`。
- `AWLOCK/ARLOCK` 在 A7 定义 atomic characteristics；`AWCACHE/ARCACHE` 在 A4 定义 memory progression；`AWPROT/ARPROT` 在 A4 定义 protection。

## 学习要点

- 前言页主要用于版本、范围和结构定位；A1/A2 才开始给出协议的功能目标和信号总览。
- 后续回答 AXI 问题时，A1/A2 提供的是“这个协议整体长什么样”和“每条通道有哪些信号”的基础。
