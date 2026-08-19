---
name: knowledge_d4_d5_ace_write_snoop_requirements
description: ACE D4 Clean/Make/Write/Evict事务、重叠写，以及D5 snoop映射和通用要求（PDF物理页219–238）
metadata: 
  node_type: memory
  type: reference
  source: AXIACE5.pdf
  originSessionId: 6edbb5d2-e302-4c52-9f34-5cef89b0dde4
  modified: 2026-07-21T11:09:09.505Z
---

# ACE D4 写事务与 D5 Snoop 通用要求

## 1. Clean 与 Make 事务（D4-219–D4-221）

### CleanInvalid

`CleanInvalid` 是广播式 cache clean + invalidate，可用于 Shareable 和 Non-shareable 区域，目标是：

- 主存已得到最新数据；
- 系统中不再存在该地址的 cache 副本；
- `IsShared=0`、`PassDirty=0`。

如果发起 Manager 自己持有 Dirty line，必须先执行 `WriteBack` 或 `WriteClean`，再把本地 line 置为 Invalid，最后才能发出 CleanInvalid。

### MakeUnique

`MakeUnique` 用于 Shareable 区域，使发起 Manager 获得 Unique 权限，但不返回数据，并移除其他所有副本。它只能与完整 cache line store 配合使用：

- `IsShared=0`、`PassDirty=0`；
- 完整行写入后通常进入 UniqueDirty；
- 允许最终为 SharedDirty；
- 即使起始状态是 Invalid、SharedClean、SharedDirty、UniqueClean 或 UniqueDirty，也必须将事务与完整行 store 作为耦合操作理解。

### MakeInvalid

`MakeInvalid` 是广播式 invalidate，可用于 Shareable 和 Non-shareable 区域，确保不存在任何 cached copy，但不要求先把 Dirty 数据写回主存：

- 发起 Manager 若持有 Valid line，必须先在本地使其 Invalid；
- `IsShared=0`、`PassDirty=0`。

## 2. Write 与 Evict 事务（D4-222–D4-227）

### WriteNoSnoop

用于不与其他 Manager Shareable 的区域，可来自普通 store 或 Non-shareable cache line 的主存更新。只有完整行 store 才允许从 Invalid 变为 Valid。

### WriteUnique 与 WriteLineUnique

二者用于 Shareable 区域，写入必须传播到主存或下层 cache：

- `WriteUnique` 可写部分数据；
- `WriteLineUnique` 必须是完整 cache line store，且更新所有字节；
- 若发起 Manager 在写期间仍持有 Clean 副本，收到写响应时必须把该副本更新为新值；
- Caching Manager 使用这两类事务时受额外死锁规避约束，见下文。

### WriteBack 与 WriteClean

两者都把 Dirty cache line 写到主存或下层 cache，可用于 Shareable 和 Non-shareable 区域：

- `WriteBack`：Shareable 区域写完后不再分配该 line，通常 Dirty → Invalid；
- `WriteClean`：Shareable 区域写完后继续分配，UniqueDirty → UniqueClean，SharedDirty → SharedClean；
- 对 Non-shareable 区域，规范允许更宽松的 Clean/Invalid 合法结束状态；
- 若 store 与写回事务原子组合，应先应用 coherent store 状态变化，再应用 WriteBack/WriteClean 状态变化。

### WriteEvict

用于把 `UniqueClean` line 下推到 L3/系统级 cache，不要求更新主存：

- 只能从 UniqueClean 发起；
- line 不能是从其他 shareability domain 推测读取而来，否则可能已陈旧；
- 事务可被丢弃；
- 支持能力由 `WriteEvict_Transaction` 属性声明，Manager 必须允许禁用以兼容旧版 ACE；
- 完成后 line 进入 Invalid。

### Evict

`Evict` 仅用于 Shareable 区域，不传数据，只通知外部 snoop filter：本地 cache 已驱逐 UniqueClean 或 SharedClean line。它只供支持 snoop filter 的 Manager 使用；协议允许偶尔不发通知，但这不是推荐行为。

## 3. Caching Manager 使用 WriteUnique/WriteLineUnique 的限制（D4-226）

为了避免进度依赖或死锁，Caching Manager 必须：

1. 发出 WriteUnique/WriteLineUnique 前，完成所有未结束的 WriteBack、WriteClean、WriteEvict、Evict；
2. 在所有 WriteUnique/WriteLineUnique 完成前，不再发出这些 memory-update/evict 事务；
3. 期间仍能完成所有传入 snoop，且不能依靠新的 WriteBack/WriteClean/WriteEvict/Evict；
4. 因此它必须能在不返回数据的情况下处理 snoop（如不保留 Shareable Dirty line 的 write-through cache），或通过 CD snoop data channel 返回数据。

## 4. 重叠写事务（D4-228–D4-229）

两个 Manager 几乎同时写同一 Shareable cache line 时，由 Interconnect 排序。后排序的 Manager2 会在自己的 snoop 端看到 Manager1 对应的 `ReadUnique`、`CleanInvalid` 或 `MakeInvalid`。

- **Overlapping ReadUnique**：Manager2 使本地副本失效；自己的 ReadUnique 完成时会取得已包含 Manager1 store 的最新 line，然后可执行本地 store。
- **Overlapping MakeUnique**：Manager2 使本地副本失效；MakeUnique 完成后直接执行完整行 store。
- **Overlapping CleanUnique**：Manager2 响应 snoop 并丢失本地副本；CleanUnique 完成后不能直接做部分行 store，必须重新发 ReadUnique。可用以下方式避免重读：一开始就用 ReadUnique；完成后做部分行 WriteBack；若原本就是完整行 store，则可直接写完整行。

## 5. 发起事务到 Snoop 事务的映射（D5-231–D5-234）

Interconnect 负责把发起 Manager 的一致性操作映射成 snooped Manager 在 AC 通道看到的事务。推荐映射：

- ReadOnce/Clean/NotSharedDirty/Shared/Unique → 同名 snoop；
- CleanUnique → CleanInvalid；
- MakeUnique → MakeInvalid；
- CleanShared/CleanInvalid/MakeInvalid → 同名 snoop；
- WriteUnique → CleanInvalid；
- WriteLineUnique → MakeInvalid；
- ReadNoSnoop、WriteNoSnoop、WriteBack、WriteClean、WriteEvict、Evict → 不 snoop。

AC 通道只允许以下 8 类：`ReadOnce`、`ReadClean`、`ReadNotSharedDirty`、`ReadShared`、`ReadUnique`、`CleanInvalid`、`MakeInvalid`、`CleanShared`。协议允许用产生相同或更强状态变化的替代 snoop；例如实现可把所有 snoop 都按 ReadUnique 处理，因为 ReadUnique 可作为其他所有 snoop 的更强替代。

## 6. Snoop 的强制行为与状态规则（D5-235–D5-237）

强制要求：

- 除 `MakeInvalid` 外，若命中 Dirty line，必须让最新数据可用；
- ReadClean、ReadNotSharedDirty、ReadShared 结束时必须 Shared 或 Invalid；
- ReadUnique、CleanInvalid、MakeInvalid 结束时必须 Invalid；
- CleanShared 结束时必须 Clean 或 Invalid。

Snoop 不得导致以下状态变化：

- Invalid → Valid；
- Clean → Dirty；
- Shared → Unique；
- UniqueDirty → UniqueClean。

最后一条尤其重要：UD→UC 意味着 Interconnect 接管写回责任，但 Manager 随后若未重新获取 store 权限，就不能合法发出 WriteBack/WriteClean。

响应位规则：

- 结束状态为任意 Valid 时，`IsShared=1`；
- Dirty → Clean 时，`PassDirty=1`；
- 除 MakeInvalid 外，Dirty → Invalid 时，`PassDirty=1`；
- MakeInvalid 导致 Dirty → Invalid 时，`PassDirty` 可为 0 或 1。

所有 snoop 的通道活动相同：AC 收地址，CR 返回响应，必要时 CD 返回数据；CR/CD 只能在 ACVALID/ACREADY 握手后出现。

## 7. Snoop 数据与正在进行的内存更新（D5-237–D5-238）

若缓存以 Dirty 状态命中非 MakeInvalid snoop，必须通过以下任一方式保证最新数据可用：

- 在 CD 通道直接返回数据；
- 响应 snoop 前先执行 WriteBack 或 WriteClean。

协议推荐所有 read snoop 在 Clean 或 Dirty 命中时都返回数据；CleanInvalid/CleanShared 只在 Dirty 命中时推荐返回；MakeInvalid 不返回数据。这是性能/功耗建议，不是强制映射。

若收到 snoop 时本地正用 WriteBack/WriteClean 更新同一主存区域，必须避免两个组件并发更新：

- 返回 `PassDirty=0, IsShared=1`，既不传 store 权限，也不转移写回责任；或
- 延迟 snoop response，直到本地 memory update 完成。

## 8. 原文定位与可靠性

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 版本：Arm IHI 0022H.c，Non-Confidential，Copyright 2003–2021
- PDF 物理页：219–238；印刷页 D4-219–D4-230、D5-231–D5-238
- 证据形式：逐页提取规范正文和表 D4-15～D4-29、D5-1～D5-6
- 可靠性：高；内容为一手协议规范摘要。状态表在本文中按规则归纳，精确逐状态组合应回查对应原表。
- 下一批：PDF 物理页 239–258，继续 D5 memory-update ordering、各类 snoop transaction 详细行为，并进入后续章节。

关联：[[knowledge_d3_d4_ace_snoop_data_states]]、[[knowledge_d3_ace_channel_signaling]]、[[knowledge_index_axiace5]]。
