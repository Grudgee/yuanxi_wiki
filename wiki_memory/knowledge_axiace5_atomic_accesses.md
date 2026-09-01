---
name: knowledge_axiace5_atomic_accesses
description: 重新从 AXIACE5.pdf 学习的 A7 Atomic Accesses
metadata:
  node_type: memory
  source: amba_prot/AXIACE5.pdf
  relearned: 2026-08-21
  source_pages: PDF 93-100
---

# A7 Atomic Accesses

## Single-copy atomicity size

- AXI 定义 single-copy atomicity size：事务原子更新内存的粒度。
- 大于该 size 的事务必须以至少该原子粒度分块更新内存。
- 原子性关注的是其他 Manager 不能观察到部分更新，而不是规定精确更新时间。
- 原子保证受起始地址对齐限制；未按相应边界对齐的访问不能获得更大粒度的原子保证。
- Byte strobe 不改变 single-copy atomicity size。

## Multi-copy write atomicity

- `Multi_Copy_Atomicity=True` 表示同一位置的写被所有 agent 以相同顺序观察。
- 一个位置的写如果已经被某个 agent 观察到，则也应被所有 agent 观察到。
- 未声明或为 False 时，不能假定具备该性质。
- 该属性和架构层面的 Acquire/Release 语义有关；回答时应与 single-copy atomicity 区分。

## Exclusive access sequence

- Manager 先对目标地址发起 exclusive read。
- Manager 稍后对同一地址发起 exclusive write，并使用匹配的 `AWID`。
- 如果自 exclusive read 之后没有其他 Manager 更新该位置，exclusive write 成功并更新内存，Subordinate 返回 `EXOKAY`。
- 如果期间该位置被更新，exclusive write 失败，不更新内存，Subordinate 返回 `OKAY`。
- Manager 必须先完成 exclusive read，不能在读阶段完成前开始对应 exclusive write。
- 支持 exclusive 的 Subordinate 使用 exclusive access monitor，记录 exclusive read 的地址和 `ARID`，在 exclusive write 到达时检查地址和 `AWID` 是否匹配。
- 不支持 exclusive 的 Subordinate 可以忽略 `AxLOCK` 并返回 `OKAY`；对 Manager 来说这表示 exclusive 失败。

## Exclusive access 限制

- 地址必须按事务总字节数对齐，即 burst size × burst length。
- exclusive burst 的总传输字节数必须是 2 的幂：1、2、4、8、16、32、64 或 128 字节。
- exclusive burst 最大 128 字节。
- exclusive burst length 不得超过 16 transfers。
- `AxCACHE` 必须保证事务到达负责监视 exclusive access 的 Subordinate。
- 如果上游 buffer/cache 可能在事务到达 monitor 前响应，那么该 exclusive access 必须是 Non-bufferable 或 Non-cacheable。
- Domain 必须是 Non-shareable 或 System-shareable。
- 事务类型必须是 ReadNoSnoop 或 WriteNoSnoop。
- 同一 exclusive sequence 的读和写必须保持这些信号一致：`AxID`、`AxADDR`、`AxREGION`、`AxLEN`、`AxSIZE`、`AxBURST`、`AxLOCK`、`AxCACHE`、`AxPROT`、`AxDOMAIN`、`AxSNOOP`、`AxMMUSECSID`、`AxMMUSID`、`AxMMUSSIDV`、`AxMMUSSID`、`AxMMUATST`。

## Exclusive response 语义

- `EXOKAY` 表示 exclusive read 或 exclusive write 成功。
- `OKAY` 对普通访问可表示成功，但对 exclusive write 可表示 exclusive 失败。
- 不支持 exclusive 的 Subordinate 对 exclusive write 返回 `OKAY` 时，可能仍按普通写更新内存；支持 exclusive 的 Subordinate 只有在 exclusive write 成功时才更新内存。
- 一个 exclusive read 的多个 response beat 可以混合 `EXOKAY` 与错误响应，或混合 `OKAY` 与错误响应；同一事务内不得混合 `EXOKAY` 和 `OKAY`。

## Locked accesses

- AXI4 不支持 locked transactions；AXI3 实现必须支持。
- Locked transaction 会让 interconnect 锁住目标 Subordinate region，直到同一 Manager 发出不带 locked 标记的解锁事务完成。
- AXI4 删除 locked 的原因是它显著增加 interconnect 复杂度、影响 QoS，并且多数新组件不需要。
- locked access 主要用于 legacy 设备，不应与 AXI4 exclusive access 混为一谈。

## Atomic access signaling

- AXI3 `AxLOCK[1:0]` 编码：`00` Normal，`01` Exclusive，`10` Locked，`11` Reserved。
- AXI4 使用 1-bit `AxLOCK`：`0` Normal，`1` Exclusive。
- 在 AXI4 环境中，AXI3 locked transaction 会被转换为普通事务；不能正确接受这种转换的组件不能用于 AXI4 环境。
- 本轮 A7 记忆只确认 `AxLOCK` signaling；不要把未重新抽取的 AXI5 atomic transaction optional 字段细节混入 A7 结论。

## 与 `AxCACHE` 的核心关系

- `AxLOCK` 标识 normal/exclusive/locked；`AxCACHE` 不标识 atomic 类型。
- `AxCACHE` 决定事务是否可能被缓存、缓冲、改形或提前响应，因此会影响 exclusive monitor 能否观察到访问。
- 正确的 exclusive sequence 要求读写两侧 `AxCACHE` 一致，并保证访问到达监视点。
- Modifiable 事务仍不得破坏 single-copy atomicity。
