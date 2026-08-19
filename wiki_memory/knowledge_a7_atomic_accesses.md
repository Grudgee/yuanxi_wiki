---
name: knowledge_a7_atomic_accesses
description: AXI4 single-copy atomicity、multi-copy atomicity 和 exclusive access 基础
metadata: 
  node_type: memory
  originSessionId: 07d5e602-b8a4-4424-94dd-570763cdaafc
  modified: 2026-07-21T09:33:41.812Z
---

# 知识点摘要

- AXI4 定义 single-copy atomicity size：事务原子更新的最小字节数；大于该大小的事务必须以至少该原子粒度分块更新内存。
- 原子性关注其他 Manager 不能观察到部分更新。例如 32-bit 链表指针不能被观察为先更新低 16 位、再更新高 16 位。
- 系统可以为不同组件组提供不同 single-copy atomicity size；通过同一 interconnect 通信的组件必须支持该通信所需的原子大小。
- `Multi_Copy_Atomicity=True` 表示同一位置的写被所有 agent 以相同顺序观察，且一个位置的写一旦被某 agent 观察，就被所有 agent 观察；未声明时为 False。Issue G 及以后要求为 True。
- Exclusive access 通过读-改写序列实现 semaphore 类操作，不要求 Manager 独占总线；`AxLOCK` 选择独占访问，`RRESP/BRESP` 的 `EXOKAY`/`OKAY` 表示成功或失败。

# 关键细节

## Single-copy atomicity

- 原子更新不要求规定精确的数据更新时间，只要求任何 Manager 不能观察部分更新。
- 原子保证不大于事务起始地址的对齐范围；例如未按 8-byte 边界对齐的 64-bit burst 不具有 64-bit 原子保证。
- 事务中的 byte strobe 不影响 single-copy atomicity size。

## Multi-copy write atomicity

- 支持方式由 `Multi_Copy_Atomicity` 属性表示：True 为支持，False 或未声明为不支持。
- 可通过单一 Point of Serialization 保证同一地址访问有序，并在新值对任意 agent 可见前使其他 cache 副本失效；也可避免位于 agent 上游的 forwarding buffer 让部分 agent 过早看到写入。
- Armv8 架构处理器使用该属性支持 Load-Acquire/Store-Release；Store-Release 要求系统具备 multi-copy atomicity。

## Exclusive access sequence

1. Manager 对目标地址执行 exclusive read。
2. 稍后对同一地址执行 exclusive write，并使用与读相匹配的 `AWID`。
3. 若自 exclusive read 后没有其他 Manager 写入该位置，exclusive write 成功并更新内存，Subordinate 返回 `EXOKAY`。
4. 若期间该位置被更新，exclusive write 失败、不更新内存，返回 `OKAY`。

- 不支持独占访问的 Subordinate 忽略 `AxLOCK` 并对普通/独占访问都返回 `OKAY`；Manager 可将其视为独占失败，不再完成写阶段。
- Manager 不必完成每次独占操作的写阶段；若未完成，Subordinate 持续监视该地址，直到下一次独占读开启新序列。
- Manager 必须先完成读阶段，不能在读阶段完成前开始写阶段。
- 支持独占的 Subordinate 使用 exclusive access monitor，记录 exclusive read 的地址和 `ARID`；exclusive write 到达时检查地址和 `AWID` 是否匹配，匹配表示期间无人写入。

## 独占访问限制

- exclusive access 地址必须按事务总字节数对齐，即 burst size × burst length。
- exclusive burst 传输字节数必须为 2 的幂：1、2、4、8、16、32、64 或 128；最大 128 字节。
- burst length 不得超过 16 transfers。
- `AxCACHE` 必须保证事务到达负责监视独占访问的 Subordinate；若上游有可能响应的 buffer/cache，应使用 Non-bufferable 或 Non-cacheable 属性。
- domain 必须是 Non-shareable 或 System-shareable，事务类型必须是 ReadNoSnoop 或 WriteNoSnoop。
- 作为同一 exclusive sequence 的读和写必须具有相同的：`AxID`、`AxADDR`、`AxREGION`、`AxLEN`、`AxSIZE`、`AxBURST`、`AxLOCK`、`AxCACHE`、`AxPROT`、`AxDOMAIN`、`AxSNOOP`、`AxMMUSECSID`、`AxMMUSID`、`AxMMUSSID`、`AxMMUATST` 等属性。

## Locked accesses

- AXI4 不支持 locked transactions；AXI3 实现必须支持。原因是 locked transaction 会显著增加 interconnect 复杂度并影响 QoS 保证，通常只建议用于 legacy 设备。
- Manager 用 `AxLOCK` 标识 locked transaction 后，interconnect 必须锁住目标 Subordinate region，直到同一 Manager 发出不带 locked 标记的解锁事务完成。
- Manager 开始 locked sequence 前必须确保没有其他事务等待完成；任一带 locked 标记的事务会迫使 interconnect 锁住后续事务，因此 sequence 最后必须有一个未锁定事务解除锁定。
- 解锁事务发出前，Manager 必须确认之前所有 locked transaction 已完成；解锁事务自身完成前也不能开始其他事务。
- 一个 locked sequence 中所有事务必须使用相同 `AxID`。规范建议 sequence 保持在单一 4KB region，且限制为两个事务，但这两项只是建议而非强制。

## Atomic access signaling

- AXI3 `AxLOCK[1:0]` 编码：`00=Normal`、`01=Exclusive`、`10=Locked`、`11=Reserved`。
- AXI4 删除 locked transaction，只保留 1-bit `AxLOCK`：`0=Normal`、`1=Exclusive`。
- 在 AXI4 环境中，AXI3 locked write (`AWLOCK=0b10`) 转换为普通写 (`AWLOCK=0b0`)；AXI3 locked read (`ARLOCK=0b10`) 转换为普通读 (`ARLOCK=0b0`)。执行转换的组件应提供可选机制标记发生过转换；不能正确处理转换结果的组件不能用于 AXI4 环境。
- legacy 软件可能需要修改，例如避免生成会强制 locked transaction 的 `SWP` 指令。

# 原文引用

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 版本：ARM IHI 0022H.c
- PDF 物理页：93–100；印刷页 A7-93–A7-100。
- 章节：Chapter A7 `Atomic Accesses`；A7.1–A7.4。

# 适用条件与例外

- `EXOKAY` 表示独占访问成功；普通访问成功使用 `OKAY`。独占访问失败也使用 `OKAY`，不能把所有 OKAY 都解释为独占成功。
- Exclusive read 可以有多个响应 beat，响应可以混合 `EXOKAY`、`SLVERR` 或 `DECERR`；同一事务中混合 `EXOKAY` 和 `OKAY` 不允许。
- 未遵守独占访问限制会导致不可预测行为。
- 本文件只覆盖 A7-93–A7-98；锁定访问和完整 atomic access signaling 需后续页继续学习。

# 关联章节

- Chapter A4 Transaction Attributes
- Chapter A5 Transaction Identifiers
- Chapter A6 AXI Ordering Model
- 后续 Chapter A7 Locked accesses / Atomic access signaling

# 待核验问题

- 继续读取 A7-99 之后的 locked access 和 atomic access signaling。
