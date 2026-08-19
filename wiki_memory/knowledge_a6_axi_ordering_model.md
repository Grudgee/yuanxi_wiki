---
name: knowledge_a6_axi_ordering_model
description: AXI 顺序模型、内存/外设位置、观察完成保证和提前响应
metadata: 
  node_type: memory
  originSessionId: 07d5e602-b8a4-4424-94dd-570763cdaafc
  modified: 2026-07-21T09:23:14.426Z
---

# 知识点摘要

- AXI 顺序模型以 `ARID`/`AWID` 为基础：同一通道、同一 ID、同一目标的请求和响应必须保持顺序；不同 Manager、读写之间、不同 ID、不同 Peripheral region 或不同 Memory location 之间没有默认排序保证。
- Memory location 具有读回最近写值、写更新该位置、访问无其他位置副作用、每个位置有观察保证和 single-copy atomicity size 等属性。
- Peripheral region 可能读不到最近写值、写可能不更新该地址、访问一个地址可能影响其他地址；观察保证按 region 给出，区域大小由实现定义且必须包含在单个 Subordinate 内。
- 完成响应与“对其他请求可观察”是不同概念；Bufferable 写可从中间点返回响应，但不代表已到最终目的地。

# 关键细节

## 事务和观察/完成

- 事务访问一个或多个地址，其位置由 `AxADDR` 和 `AxPROT[1]` 的 Non-secure 属性决定。
- Memory location 之间才提供逐位置观察保证；一个 Peripheral region 必须整体包含在单个 Subordinate；跨多个 Memory location 的事务具有多个排序保证。
- Device 事务由 `AxCACHE[1]=0` 表示，可访问 Peripheral 或 Memory；Normal 事务由 `AxCACHE[1]=1` 表示，通常访问 Memory，访问 Peripheral 时只需协议合规，结果由实现定义。
- Non-bufferable 写：`AWCACHE[0]=0`；Bufferable 写：`AWCACHE[0]=1`。
- 写完成响应发生在 `BVALID` 与 `BREADY` 同时置位的周期；读完成响应发生在最后一个 `RDATA` beat 的 `RVALID`、`RLAST`、`RREADY` 同时置位的周期。

## Manager 排序保证

- 在收到完成响应前，同一 Manager、同一 ID 的保证包括：后发 Device 写到同一 Peripheral region 必须在先发 Device 写之后到达；后发 Device 读必须在先发 Device 读之后到达；同一 Memory location 的后发写必须在先发写之后被观察；已被读观察的写，在后续读之后发出的写中仍必须按序被观察。
- 收到完成响应后：读完成响应保证该读对任何 Manager 的后续读/写可观察；写完成响应同样给出对后续读/写的观察保证。要求系统满足 multi-copy atomicity 时，这个保证必须成立。
- Bufferable 写的响应可能来自中间点，因此不保证写已经到达最终端点。
- 响应顺序保证：同一 Manager、同一 ID 的后发读/写，其响应不能先于前一同类事务。

## Subordinate 和 Interconnect 要求

- Peripheral region 的实际执行顺序由实现定义，通常期望等于到达顺序但不是硬性要求。
- Memory location 要求后发同 ID 写在前发写之后执行；写响应后的写、读响应后的写也要满足相应顺序。
- Interconnect 必须保持同 ID、相同或重叠位置的读/写请求顺序，以及同 ID Device 事务到同一 Peripheral region 的顺序；响应也需保持顺序。修改 ID 时必须保持原 ID 对应的排序要求。

## 提前响应

- Normal 读可由中间组件从本地、最新 cache 副本提前返回，但必须遵守 ID 排序：同一 ID 的所有更早读已完成响应后才能提前返回。
- Bufferable 写可由中间组件提前响应；若随后丢弃本地副本，必须先把事务继续向下游传播。提前响应后，中间组件仍负责排序和可观察性。
- Normal 事务提前写响应后，同一/重叠 Memory location 的后续事务要在该写之后排序；Device 事务提前响应后，同一 Peripheral region 的后续事务也要在该写之后排序。

## Ordered Write Observation

- `Ordered_Write_Observation=True` 表示接口对同一 Manager、同一 ID 的写提供与目标地址无关的观察保证：后发写必须在先发写之后被观察。
- 未声明时视为 False。具有该属性的 Subordinate 可让 Producer-Consumer ordering Manager 在不等待前一写完成响应时发出依赖写。

# 原文引用

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 版本：ARM IHI 0022H.c
- PDF 物理页：83–98；印刷页 A6-83–A7-98。
- 章节：Chapter A6 `AXI Ordering Model`（A6-83–A6-92）；Chapter A7 开始至 A7-98。
- 关键位置：A6 总览 A6-84；Memory/Peripheral A6-85；事务分类 A6-86；观察/完成 A6-87；排序保证 A6-88–A6-89；提前响应 A6-90；Ordered Write Observation A6-91；A7 原子访问概览 A7-93–A7-98。

# 适用条件与例外

- 同一 ID 的排序保证依赖目标相同或地址重叠；不同目标位置通常无保证。
- Peripheral 的实际执行顺序可能是实现定义，不能简单等同于到达顺序。
- 提前响应不等于最终目的地完成；尤其要区分 Bufferable 写响应与最终可见性。

# 关联章节

- Chapter A4 Transaction Attributes
- Chapter A5 Transaction Identifiers
- Chapter A7 Atomic Accesses
- 后续 ACE/ACE5 一致性和排序章节

# 待核验问题

- A7 原子访问的独占、锁定和原子访问信号仍需下一批继续读取。
