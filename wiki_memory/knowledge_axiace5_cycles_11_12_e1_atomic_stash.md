---
name: knowledge_axiace5_cycles_11_12_e1_atomic_stash
description: AXIACE5.pdf 自动学习周期 11-12，E1 AMBA 5 Atomic transactions、Cache stashing、Deallocating transactions
metadata:
  node_type: memory
  source: amba_prot/AXIACE5.pdf
  learned: 2026-08-21
  cycles: 11-12
  source_pages: PDF 339-358
  estimated_source_chars: 63593
---

# 周期 11-12：E1 Atomic、Stash、Deallocate

## 覆盖范围

- 周期 11：PDF p.339-348，E1 入口与 E1.1 Atomic transactions 主体。
- 周期 12：PDF p.349-358，E1.1.10 支持要求、E1.2 Cache stashing、E1.3 Deallocating transactions、E1.4 Trace signals。

## E1.1 Atomic transactions 总览

- AMBA 5 引入 Atomic transactions：事务不仅做单次访问，还携带要在数据位置附近执行的操作。
- 相比 exclusive access，Atomic transaction 把操作送到数据处，可减少数据对系统其他 agent 不可访问的时间。
- 支持能力由 `Atomic_Transactions` 属性声明；未声明时视为 False。
- Atomic transactions 支持于 AXI5、ACE5-Lite、ACE5-LiteDVM；不支持 ACE5 Managers 使用 Atomic transactions。
- 若 Subordinate 或 interconnect 声明支持 Atomic transaction，必须支持所有操作类型、大小和端序。

## 四种 Atomic transaction 形式

- `AtomicStore`：发送一个数据值和操作；目标用发送值与地址处原值作为操作数，结果写回地址；只返回单个 response，不返回数据；数据大小 1/2/4/8 bytes。
- `AtomicLoad`：发送一个数据值和操作；返回地址处原始值；目标执行操作并把结果写回；入站数据大小等于出站数据大小。
- `AtomicSwap`：发送一个数据值；目标将地址处原值与发送值交换；返回原始值；数据大小 1/2/4/8 bytes。
- `AtomicCompare`：发送 compare value 和 swap value；若地址处原值等于 compare value，则写入 swap value，否则不写；返回地址处原始值；出站数据 2/4/8/16/32 bytes，入站数据为其一半。

## AtomicLoad/AtomicStore 操作集合

- `ADD`：内存值加发送值，结果写回。
- `CLR`：发送值中为 1 的 bit 清除内存对应 bit。
- `EOR`：发送值与内存值按位异或。
- `SET`：发送值中为 1 的 bit 置位内存对应 bit。
- `SMAX` / `SMIN`：按有符号比较写回最大/最小值。
- `UMAX` / `UMIN`：按无符号比较写回最大/最小值。

## Atomic transaction 属性约束

- `AWLEN` 和 `AWSIZE` 指明写数据字节数；对 `AtomicCompare`，字节数包含 compare 与 swap 两个值。
- 如果 `AWLEN` 表示 burst length 大于 1，`AWSIZE` 必须是完整数据总线宽度。
- 数据窗口外的 write strobe 必须 deassert；数据窗口内的 write strobe 必须 assert。
- `AtomicStore`、`AtomicLoad`、`AtomicSwap` 的 `AWADDR` 必须按数据大小对齐，`AWBURST` 必须为 `INCR`。
- `AtomicCompare` 的 `AWADDR` 必须按单个写数据值对齐，即总写数据大小的一半；若指向 lower half，compare value 先发且 `AWBURST=INCR`；若指向 upper half，swap value 先发且 `AWBURST=WRAP`。
- `AtomicCompare` 放宽部分 WRAP 规则：允许 length 1 burst，且 `AWADDR` 不要求按 transfer size 对齐。

## ID 与请求属性

- 一个 Atomic transaction 使用单个 AXI ID；request、write response、read data 使用同一个 ID。
- Atomic transaction 不能与同时 outstanding 的 non-atomic transaction 使用相同 AXI ID；多个同时 outstanding 的 Atomic transactions 也不能使用相同 ID。
- Atomic transaction 的 `AWCACHE` 和 `AWDOMAIN` 可为接口类型允许的任意合法组合。
- `AWSNOOP` 必须全 0。
- `AWLOCK` 必须为 `0b0`，即 Normal access；AMBA 5 Atomic transaction 不是用 `AxLOCK` 标识。

## `AWATOP` 信令

- Atomic transaction 在 AW channel 增加 `AWATOP`，表示 atomic 操作类型和端序。
- `AWATOP[5:0]=000000`：Non-atomic operation。
- `AWATOP[5:0]=01exxx`：AtomicStore。
- `AWATOP[5:0]=10exxx`：AtomicLoad。
- `AWATOP[5:0]=110000`：AtomicSwap。
- `AWATOP[5:0]=110001`：AtomicCompare。
- 对 AtomicStore/AtomicLoad，`AWATOP[3]` 表示算术操作端序：0 为 little-endian，1 为 big-endian；bitwise logical 操作忽略该位。
- 对 AtomicStore/AtomicLoad，`AWATOP[2:0]` 编码 `ADD/CLR/EOR/SET/SMAX/SMIN/UMAX/UMIN`。

## 事务结构与响应

- `AtomicLoad`、`AtomicSwap`、`AtomicCompare`：请求走 AW，发送数据走 W，原始数据从 R 返回，B 返回单个写响应。
- `AtomicStore`：请求走 AW，发送数据走 W，只返回 B response，不返回 R data。
- Subordinate 可等收齐 W data 后再发 R data，也可先发 R data 再接收 W data；Manager 必须能处理两种相对时序。
- B response 只能在 Subordinate 收齐所有 W data 且 atomic 结果可观察后返回。
- 若不支持某地址/类型的 Atomic transaction，应给出合适错误响应；Device transaction 必须送到 endpoint Subordinate，不得交给不支持 atomic 的组件。
- 对 Cacheable transaction，interconnect 可自己执行 atomic 操作，但必须完成相应 read、write、snoop；也可在 endpoint Subordinate 支持时向下传递。

## E1.2/E1.3 边界

- Cache stashing 让一个组件指示某 cache line 应被放入另一个组件的 cache，以把数据放到使用点附近。
- Stash 支持能力由 `Cache_Stash_Transactions` 声明，适用于 ACE5-Lite、ACE5-LiteDVM、ACE5-LiteACP，不支持 ACE5 Managers 使用或被 stash。
- Deallocating transactions 用于提示某 cache line 可从 cache 中释放或不再保留，和后续系统级 cache 管理相关。

## 监督记录

- 单周期抽取字符估算：28,256；35,337，均低于 80,000 字符上限。
- 本阶段完成 E1.1 Atomic transactions 的细化记忆，修正了之前只知道 `AWATOP` 入口但未展开的问题。
