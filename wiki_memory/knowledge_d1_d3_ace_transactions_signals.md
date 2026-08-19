---
name: knowledge_d1_d3_ace_transactions_signals
 description: ACE事务类型、处理流程、域与屏障、DVM、协议错误及ACE信号定义（PDF物理页159–178）
metadata:
  type: reference
  source: AXIACE5.pdf
  modified: 2026-07-21
---

# ACE 事务、域及信号定义

## 1. 已缓存 cache line 的 Store 与重叠 Store（D1-159）

### 1.1 已持有 Shared 副本时的 Store

Manager 已持有 Shared 副本时，可发起 `CleanUnique`：

1. 发起 Manager 请求获得该 cache line 的唯一副本；此事务本身不把 cache line 返回给发起者。
2. Interconnect 在 AC 通道向其他 cache 发 snoop；被 snoop 的 Manager 在 CR 通道报告副本是否已删除，以及是否有 Dirty 数据需要写回主存。
3. 如需写回，持有 Dirty 副本的 Manager 经 CD 通道把数据交给 Interconnect，由后者构造写回事务。
4. Interconnect 经 R 通道只返回响应字段，不发生数据传输。
5. Manager 执行 store，并用 `RACK` 表示事务完成。

### 1.2 两个 Manager 同时 Store 同一 Shareable cache line

Interconnect 决定事务顺序。先获准的 Manager 正常完成；后获准者会在 snoop 端观察到前者的 store：

- 如果后者需要数据，则在自己的事务完成时取得数据，再执行 store。
- 如果后者执行整行写，在观察到前一事务 snoop 时删除原副本，但仍可继续自己的整行 store。
- 如果后者执行部分行写，且因原先持有副本而未请求数据，则观察到前一事务后必须删除旧副本。随后可重新请求数据再完成 store，或直接对主存执行 partial-line write 且不保留 cache 副本。

## 2. ACE 事务分类（D1-160–D1-163）

### 2.1 Non-snooping

用于其他 Manager cache 中不存在该地址的访问，不产生 snoop：

- `ReadNoSnoop`
- `WriteNoSnoop`

典型用途是 Non-shareable 或 Device 类型访问。

### 2.2 Shareable Load 的一致性读事务

- `ReadClean`：请求者只接受 Clean line，不能承担 Dirty line 最终写回主存的责任；常用于不具备接收 Dirty line 能力或使用 write-through cache 的 Manager。
- `ReadNotSharedDirty`：可接受除 SharedDirty 外的任意状态；返回线可为 Clean（Unique/Shared）或 UniqueDirty。
- `ReadShared`：请求者可接受任意状态的 cache line。

被 snoop 的 cache 可以返回 Dirty line，即使发起者不能接受该 Dirty 状态；此时 Interconnect 负责把 Dirty line 写回主存。如果提供数据的 cache 原为 Unique 且保留副本，事务后必须转为 Shared。规范建议持有数据的被 snoop cache 直接提供数据，以完成 snoop。

### 2.3 Shareable Store 事务

三类事务都确保 store 发生时没有其他副本：

- `ReadUnique`：部分 cache line store，且发起者尚无副本；取得 cache line 并删除其他副本。
- `CleanUnique`：部分 cache line store，且发起者已有副本；删除其他副本，如发现 Dirty 副本则确保其写回主存。
- `MakeUnique`：整条 cache line store；使其他副本失效，不需要先取得原数据。

### 2.4 不保留 cache 副本的 Shareable 访问

- `ReadOnce`：获得数据快照，不要求缓存；提供数据的 Unique cache 不必因该事务转为 Shared。
- `WriteUnique`：写入前删除所有 cache 副本，可用于整行或部分行；先确保 Dirty 数据写回。
- `WriteLineUnique`：只允许整条 cache line 写，且所有字节都必须由该事务写入；写入前删除其他副本。

`ReadOnce` 和 `WriteUnique` 不要求事务为整条 cache line；`WriteLineUnique` 必须是整行。

### 2.5 主存更新事务

这些事务不 snoop 其他 cache：

- `WriteBack`：把 Dirty line 写回主存以释放 cache line，不保留副本。
- `WriteClean`：把 Dirty line 写回主存，但允许继续保留副本。
- `WriteEvict`：驱逐 Clean line，将其写到较低级 cache（例如 L3 或系统级 cache），不要求更新主存。
- `Evict`：只通知某 cache line 已从本地 cache 驱逐，不需要更新主存，不携带数据，便于 snoop filter 跟踪。

这些事务不会像普通 snoop transaction 那样序列化；其他 cache 不必知道数据是否已写到主存。

### 2.6 Cache maintenance

广播 cache maintenance 用于软件 cache maintenance，也可传播到 downstream cache：

- `CleanShared`：要求 Dirty line 提供数据，使其写回主存；被 snoop cache 可保留副本。
- `CleanInvalid`：clean 后 invalidate。Clean line 删除本地副本；Dirty line 必须先提供数据供写回，再删除副本。
- `MakeInvalid`：仅 invalidate；删除副本，即使 Dirty 也无需提供数据。

发起 cache maintenance 的 Manager 也必须对自己的 local cache 执行相应操作。

### 2.7 Snoop、Barrier、DVM

- Snoop transaction 使用 AC、CR、CD 通道，是 coherent transaction 和 cache maintenance transaction 的子集。
- Barrier 提供事务顺序和可观察性保证，包括 Memory barrier 和 Synchronization barrier；ACE5/ACE5-Lite 不支持 barrier transaction。
- DVM transaction 用于虚拟内存系统维护，通常在分布式虚拟内存组件间传递消息。

## 3. 一致性事务处理流程（D1-164）

典型流程：

1. Initiating Manager 发起事务。
2. 根据是否需要一致性支持，事务直接送到 Subordinate，或送入 Interconnect 的 coherency support logic。
3. 一致性事务与其他 Manager 的后续事务比较，以保证正确处理顺序。
4. Interconnect 决定所需 snoop。
5. 每个收到 snoop 的 Caching Manager 必须返回 CR 响应，必要时经 CD 返回数据。
6. Interconnect 判断是否还需访问主存。
7. Interconnect 汇总 snoop responses 和所需数据。
8. Initiating Manager 完成事务。

## 4. Shareability Domain（D1-165–D1-166、D3-176–D3-177）

Shareability domain 决定 coherent/barrier transaction 需要涉及哪些 Manager：

- `Non-shareable`：仅一个 Manager。
- `Inner Shareable`：可包含额外 Manager；ACE5 和 ACE5-Lite 已不推荐使用。
- `Outer Shareable`：至少包含对应 Inner domain 的全部 Manager，还可包含其他 Manager。
- `System`：系统全部 Manager。

同一 domain 内的关系必须对称一致；domain 互不重叠。Outer domain 中各 Manager 的 Inner domain 成员必须共同属于同一 Outer domain。System domain 用于必须对所有 Manager 可见的事务；它包含不具备硬件一致性 cache 的 Manager，因此该地址不能缓存在任何层级。

`AxDOMAIN[1:0]` 编码：

| 编码 | Domain |
|---|---|
| `00` | Non-shareable |
| `01` | Inner Shareable |
| `10` | Outer Shareable |
| `11` | System |

内存类型限制：

- Device（`AxCACHE[1]=0`）只能使用 System domain。
- Cacheable（`AxCACHE[3:2] != 0`）不得使用 System domain。
- Non-cacheable 可在部分情况下使用 Non-shareable、Inner、Outer，也可合法使用 System。
- System domain 的地址不能存在于任何 cache。
- Outer Shareable cache 范围包含相应的 Inner Shareable peer caches。

## 5. Barrier（D1-166）

- Memory barrier：若 domain 内另一 Manager 能观察到 barrier 后事务，则也必须能够观察到 barrier 前全部事务。
- Synchronization barrier：用于确定 domain 内所有 Manager 何时均可观察到 barrier 前事务；System domain 下，barrier 前事务还必须在 barrier 完成前到达目标 Subordinate。

Barrier 有地址阶段和响应，但无数据传输。Manager 必须同时在 AR、AW 通道发出 barrier，并在读数据和写响应都返回 barrier 响应前，不得发出必须排序在 barrier 后的事务。

`AxBAR[1:0]`：

| 编码 | 含义 |
|---|---|
| `00` | 普通访问，遵守 barrier |
| `01` | Memory barrier |
| `10` | 普通访问，忽略 barrier |
| `11` | Synchronization barrier |

## 6. DVM（D1-167）

DVM 用于分布式虚拟内存维护。典型地址翻译过程：

1. Manager 以虚拟地址 VA 发事务。
2. SMMU 将 VA 翻译为物理地址 PA；若 TLB 命中直接取翻译，否则执行 translation table walk。
3. SMMU 使用 PA 代表请求者发出事务。

页表更新后，旧翻译可能仍缓存在各组件 TLB 中，因此使用 DVM message 发起 TLB invalidate；所有相关操作完成后再执行 DVM Sync。若 table walk 产生 fault，必须通知维护页表的 agent，由其补充 VA→PA 映射并通常更新页表。

## 7. 协议错误（D1-168）

### 7.1 Software protocol error

同一地址的多次访问使用不匹配的 shareability 或 cacheability 属性时发生。可能破坏一致性和数据，但系统不得因此死锁，事务必须继续向前推进。一个 4KB 区域中的软件协议错误不得破坏另一个 4KB 区域的数据。

Normal memory 可通过适当 barrier 和软件 cache maintenance 恢复到定义状态。对 peripheral 使用 `AxCACHE[1]=1` 的 Modifiable 事务时，外设行为无法保证；但外设仍必须协议合规响应。恢复到已知工作状态的方法由实现定义。

### 7.2 Hardware protocol error

除 software protocol error 外的协议错误都属于 hardware protocol error。协议不要求提供恢复支持；系统可能崩溃、锁死或出现其他不可恢复故障。

## 8. ACE 信号定义（D2-169–D2-174）

### 8.1 原 AXI 通道新增信号

- AR：`ARSNOOP[3:0]`（Shareable read 类型）、`ARDOMAIN[1:0]`、`ARBAR[1:0]`。
- AW：`AWSNOOP[2:0]`（Shareable write 类型）、`AWDOMAIN[1:0]`、`AWBAR[1:0]`、`AWUNIQUE`（允许本事务数据以 Unique 状态缓存；仅支持 WriteEvict 时需要）。
- R：扩展 `RRESP[3:2]`，报告一致性读状态。
- W/B：无额外 ACE 信号。

### 8.2 ACE 专用通道

AC snoop address：

- `ACVALID`：Interconnect 表示地址/控制有效。
- `ACREADY`：Manager 可接受 AC 传输。
- `ACADDR`：snoop 首个传输地址。
- `ACSNOOP[3:0]`：snoop transaction 类型。
- `ACPROT[2:0]`：保护属性；ACE 仅赋予 `ACPROT[1]` 含义。

CR snoop response：

- `CRVALID`：Manager 表示响应有效。
- `CRREADY`：Interconnect 可接受响应。
- `CRRESP[4:0]`：snoop transfer 状态。

CD snoop data：

- `CDVALID`：Manager 表示 snoop data 有效。
- `CDREADY`：Interconnect 可接受数据。
- `CDDATA`：snoop data。
- `CDLAST`：snoop transaction 最后一个数据传输。

### 8.3 RACK/WACK 与复位

- `RACK`：Manager 表示读事务完成。
- `WACK`：Manager 表示写事务完成。

复位期间：Manager 必须把 `RACK`、`WACK`、`CRVALID`、`CDVALID` 拉低；Interconnect 必须把 `ACVALID` 拉低。`ARESETn` 拉高后，Interconnect 最早可在下一个 `ACLK` 上升沿开始拉高 `ACVALID`。

## 9. D3 信令开端（D3-175–D3-178）

D3 将 ACE 事务分为六组：

- Non-snooping：不得 snoop 其他 Manager。
- Coherent：目标可存在于其他 Manager cache，必须 snoop。
- Memory update：更新主存，不得 snoop。
- Cache maintenance：目标可存在于其他 cache，必须 snoop，并可能传播到 downstream cache。
- Barrier：建立其他事务间的顺序。
- DVM：在分布式虚拟内存参与者之间传递操作。

## 10. 原文定位与后续

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 版本：ARM IHI 0022H.c
- PDF 物理页：159–178；印刷页 D1-159–D1-168、D2-169–D2-174、D3-175–D3-178。
- 下一批：PDF 物理页 179–198，继续 D3 Channel Signaling，重点学习 ARSNOOP/AWSNOOP 编码、RRESP 一致性位、RACK/WACK 时序、AC/CR 通道握手和 snoop 响应字段。
