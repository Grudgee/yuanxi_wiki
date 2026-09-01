---
name: knowledge_axiace5_cycles_01_05_d5_d8
description: AXIACE5.pdf 自动学习周期 01-05，D5 Snoop 后半、D6 Interconnect、D7 Cache Maintenance、D8 Barrier 开始
metadata:
  node_type: memory
  source: amba_prot/AXIACE5.pdf
  learned: 2026-08-21
  cycles: 01-05
  source_pages: PDF 239-288
  estimated_source_chars: 151030
---

# 周期 01-05：D5 Snoop、D6 Interconnect、D7 CMO、D8 Barrier

## 覆盖范围

- 周期 01：PDF p.239-248，D5.2.4-D5.3.6，进入 D6.1。
- 周期 02：PDF p.249-258，D6.2-D6.5.2。
- 周期 03：PDF p.259-268，D6.5.3-D6.7.3，进入 D7.1-D7.3。
- 周期 04：PDF p.269-278，D7.4-D7.8.4。
- 周期 05：PDF p.279-288，D7.8.5-D7.9.3，进入 D8.1-D8.3。

## D5 Snoop Transactions 后半

- `WasUnique` 响应表示被 snoop 的 cache line 在 snoop 前处于 Unique 状态；若可确认 WasUnique，interconnect 可避免继续 snoop 不必要的 caching Managers。
- snooped Manager 的非阻塞规则核心是避免死锁：如果 snoop 响应可能触发主存写回或把写权限交给其他 Manager，则必须先完成同一 cache line 上已在进行的 `WriteBack`、`WriteClean` 或 `WriteEvict`。
- snoop transaction 包括 `ReadOnce`、`ReadClean`、`ReadShared`、`ReadNotSharedDirty`、`ReadUnique`、`CleanInvalid`、`MakeInvalid`、`CleanShared` 等；每类事务都约束初始 cache line state、结束 state 以及 `PassDirty`/`IsShared` 等响应组合。
- `ReadOnce` 表示发起方不会保留 cache line；如果被 snoop 方持有 dirty copy，通常必须提供数据或确保数据回写路径正确。

## D6 Interconnect Requirements

- D6 规定 coherent interconnect 如何排序、发起 snoop、合成响应以及与主存交互。
- `Read` / `Write Acknowledge` 与连续读数据返回约束用于保证 Manager 看到的事务完成点符合一致性协议。
- interconnect 发起 snoop transaction 时，需要根据请求类型、cache state 和 domain 判断是否必须 snoop、snoop 哪些 Managers，以及是否能从主存/其他 cache 取数。
- interconnect 对主存的读、由 interconnect 生成的主存更新、以及授予写权限的时机都必须维护 cache line 的单一有效权限模型。
- 允许的事务修改和 speculative reads 不能破坏一致性、权限传递或观察顺序。
- D6.7 讨论 cache line size 转换和地址空间大小；跨组件 cache line 大小不一致时，interconnect 必须保证 snoop、回写、权限授予仍按正确粒度工作。

## D7 Cache Maintenance

- D7 的 CMO（Cache Maintenance Operation）用于清理、失效或传播 cache line 状态，保证 cache 与主存/其他观察点的一致性。
- CMO 属性、传播规则和通道信令决定操作在 read channel 或 write channel 上表达，并决定是否需要继续传播到其他点。
- Persistent CMO 引入 Point of Persistence 和 Deep Persistence，用于非易失/持久化语义；PCMO 不只是普通 cache clean，而是要求数据达到特定持久化点。
- Write with CMO 把写事务和 cache maintenance 意图结合；需要独立理解配置、操作、属性、传播和响应。
- ACE Managers 执行 CMO 时，snooped Manager 有专门要求；软件 cache maintenance 若与硬件一致性状态不匹配，可能导致 unpredictable 行为。

## D8 Barrier 开始

- Barrier transaction 用于约束事务跨域可见性和完成顺序。
- D8.2 定义 `AxBAR`、`AxDOMAIN` 和响应信令。
- D8.3 讨论 barrier response 与 domain boundary；barrier 的完成不能只看本地接口，还要看指定 shareability domain 内的传播要求。

## 监督记录

- 单周期抽取字符估算：28,038；34,837；20,519；34,124；33,512，均低于 80,000 字符上限。
- 本阶段跨越 D5、D6、D7 并进入 D8，已在章节边界处保存阶段记忆。
