---
name: knowledge_d3_d4_ace_snoop_data_states
 description: ACE CD snoop data、通道依赖及D4一致性事务和cache状态变化（PDF物理页199–218）
metadata:
  type: reference
  source: AXIACE5.pdf
  modified: 2026-07-21
---

# ACE CD 通道与 D4 一致性事务

## 1. CD Snoop Data Channel（D3-199–D3-200）

信号：`CDVALID`（Manager）、`CDREADY`（Interconnect）、`CDDATA`、`CDLAST`。遵守标准 AXI VALID/READY：CDVALID 置位后，CDDATA/CDLAST 必须保持到 CDREADY；CDREADY 可提前或同周期置位。

- CD 数据宽度可不同于普通读写数据通道，允许 32/64/128/256/512/1024 bit。
- 完整 cache line 事务必须使用完整数据宽度；burst 长度必须为 1、2、4、8 或 16。
- CD 通道不支持 byte strobe；CDVALID 置位时所有字节都有效。
- 只有 CRRESP[0]（DataTransfer）要求时才必须返回 snoop data，且顺序必须与 AC 地址顺序一致。
- 长度大于 1 的 snoop burst 都是 WRAP，CDLAST 必须在最后一个传输置位。
- CD 通道是可选的；不支持 CD 的 Caching Manager 仍必须支持 AC 上所有 snoop 类型，但不能返回数据。
- Manager 不应被迫返回 Dirty data，也绝不能在 CR 中声称 DataTransfer=1 却不提供数据。可通过不持有 Dirty 数据，或在 snoop 前执行 WriteBack/WriteClean，避免此要求；但后一策略不兼容 WriteUnique/WriteLineUnique。

## 2. Snoop channel dependencies（D3-201）

关键规则：

- Interconnect 不必等待 Manager 先置 `ACREADY` 才置 `ACVALID`。
- Manager 可以等待 `ACVALID` 后再置 `ACREADY`。
- Manager 置 `CRVALID` 前必须等待 `ACVALID` 和 `ACREADY`。
- Manager 置 `CDVALID` 前必须等待 `ACVALID` 和 `ACREADY`。
- Manager 不必等待 Interconnect 置 `CRREADY` 或 `CDREADY` 才置 CRVALID。
- 若需要数据，Manager 不必等待 CRREADY/CDREADY 才置 CDVALID。
- Interconnect 可等待 CRVALID 才置 CRREADY/CDREADY。
- 若需要数据，Interconnect 可等待 CDVALID 才置 CRREADY。

## 3. D4 发起 Manager 与事务流程

发起 Manager 的内部操作需要：

- Load：得到合适 cache line 的有效副本，或得到返回有效数据的事务结果。
- Store：获得 Unique 状态副本，或获得允许 store 的权限。

事务流程：

- 读类：AR → R 数据/响应 → RACK；
- Clean/Make/读 Barrier/DVM：AR → 单次 R 响应（无数据）→ RACK；
- 写类：AW → W → B → WACK；
- Evict：AW → B（无数据）→ WACK；
- 写 Barrier：AW → B → WACK；
- DVM：AR → 单次 R → RACK。

## 4. Snoop filter 与状态变化

外部 snoop filter 跟踪 Manager cache 中已分配的 cache line；支持外部 filter 的 Manager 必须能够广播 cache line 的分配和驱逐。状态变化由事务类型、AR 响应、是否支持外部 snoop filter、是否执行 speculative read 决定。

一般规则：

- `PassDirty=1` 时，cache line 必须转为 Dirty；适用于 `ReadNotSharedDirty`、`ReadShared`、`ReadUnique`。
- `IsShared=1` 时，cache line 必须转为 Shared 或 Invalid；适用于 `ReadOnce`、`ReadClean`、`ReadNotSharedDirty`、`ReadShared`、`CleanShared`。
- Unique 可转为对应 Shared，但不是规范推荐的结果。
- 不支持外部 snoop filter 时，Clean cache line 允许转为 Invalid。
- Coherent store 前，必须有 UniqueClean 或 UniqueDirty 状态，或通过事务获得 store 权限：`ReadUnique`/`CleanUnique`/`MakeUnique`，或 `WriteUnique`/`WriteLineUnique`。
- 主存更新只能从 Dirty 状态进行；使用 WriteBack/WriteClean 后必须为 Clean 或 Invalid。支持外部 filter 时，WriteBack 后必须 Invalid，WriteClean 后必须 Clean。
- Cache maintenance 前：CleanShared 要求 Clean/Invalid；CleanInvalid 和 MakeInvalid 要求 Invalid；maintenance 本身不改变发起 Manager 的 cache state。

## 5. Read 事务的状态语义（D4-211–D4-216）

- `ReadNoSnoop`：用于 Non-shareable；IsShared、PassDirty 都必须为 0。Invalid 起始时通常结束为 Invalid 或 UniqueClean；已有 Dirty 时保留 Dirty。
- `ReadOnce`：Shareable 区域读取但不保留本地副本；IsShared 表示共享/唯一，PassDirty 必须为 0。Invalid 起始仍为 Invalid；已缓存副本可保持原状态。
- `ReadClean`：请求 Clean 副本，不接收 Dirty 责任；PassDirty 必须为 0。Invalid 通常得到 UC（唯一）或 SC（共享）。
- `ReadNotSharedDirty`：可接受除 SharedDirty 外状态；IsShared 表示 Unique/Shared，PassDirty 表示 Clean/Dirty。若响应 IsShared=1，则 PassDirty 必须表示 Clean。
- `ReadShared`：可接受任意状态；IsShared 和 PassDirty 分别表示 Shared/Unique 与 Clean/Dirty。SharedClean 起始且 PassDirty=1 时必须转为 Dirty。
- `ReadUnique`：取得数据并确保本地 cache line 可处于 Unique，IsShared 必须为 0；PassDirty 表示数据是 Clean 还是 Dirty。通常用于部分 cache line store 前获取权限。

## 6. Clean 事务开端（D4-217–D4-218）

`CleanUnique` 用于 Shareable 区域已有本地副本的部分行 store：

- 允许 cache line 进入 Unique 状态，但不向 Manager 返回数据；
- 其他 cache 的 Dirty 数据必须写回主存；其他副本全部删除；
- IsShared 和 PassDirty 都必须为 0；
- 完成后 Manager 获得对该 cache line 的 store 权限；若原来 Invalid，随后 store 必须是完整 cache line，并且 store 与 CleanUnique 的完成原子关联。
- 支持 snoop filter 时，若事务后仍分配该 line，filter 得到正确分配信息；若不再分配，需执行合适 WriteBack、重新发起事务或 Evict 通知 filter。

`CleanShared` 是广播 cache clean，确保主存位置的所有 cache 副本为 Clean，可用于 Shareable 和 Non-shareable 区域。发起 Manager 若持有 Dirty line，必须先 WriteBack/WriteClean；执行期间允许再次变 Dirty，但 CleanShared 完成时要达到要求。

## 7. 原文定位与后续

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 版本：ARM IHI 0022H.c
- PDF 物理页：199–218；印刷页 D3-199–D3-202、D4-203–D4-218。
- 下一批：PDF 物理页 219–238，继续 CleanInvalid、Make、Write、Evict 事务及重叠写处理。
