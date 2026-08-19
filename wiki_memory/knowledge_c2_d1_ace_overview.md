---
name: knowledge_c2_d1_ace_overview
 description: AXI5-Lite 定义、互操作与转换，以及 ACE 一致性概览、缓存状态和通道结构（PDF物理页139–158）
metadata:
  type: reference
  source: AXIACE5.pdf
  modified: 2026-07-21
---

# AXI5-Lite 与 ACE 概览

## 1. Chapter C2：AMBA AXI5-Lite（C2-139–C2-146）

### 1.1 AXI5-Lite 定义

AXI5-Lite 是 AXI5 的子集，所有事务均为单拍，适合寄存器型组件和不适合 burst 传输的简单存储器。与 AXI4-Lite 相比，AXI5-Lite 在数据宽度和响应排序方面更灵活：

- 所有事务仍为 burst length=1；
- 支持不同数据宽度；
- 具有不同 ID 的请求，其响应可以重排序；
- 所有访问被视为 Device Non-bufferable；
- 不支持 exclusive access。

典型应用是处理器通过 ACE5 连接共享存储系统，同时通过 AXI5-Lite 连接私有存储、SRAM、外设或异步桥。

### 1.2 与 AXI5、AXI4-Lite 的比较

| 特性 | AXI5 | AXI5-Lite | AXI4-Lite |
|---|---:|---:|---:|
| 数据宽度 | 最大 1024 bit | 最大 1024 bit | 32 或 64 bit |
| 事务长度 | 最大 256 beat | 1 beat | 1 beat |
| 单事务大小 | 不超过总线宽度 | 不超过总线宽度 | 完整总线宽度 |
| 内存类型 | 任意 | Device Non-bufferable | Device Non-bufferable |
| 写 strobes | 必需 | 必需 | 可选 |
| 响应排序 | 可按序或乱序 | 可按序或乱序 | 按序 |
| ID | 必需 | 必需 | 可选 |
| Exclusive | 支持 | 不支持 | 不支持 |
| Check_Type、Poison、Trace、Wakeup、Unique ID | 可选 | 可选 | 不支持 |

AXI5-Lite 的意义：如果不需要 burst，通常比 AXI5 更简单；但相较 AXI4-Lite，它允许更灵活的数据宽度、ID 和响应排序。响应重排序在不同 Subordinate 响应延迟差异较大时可降低整体响应延迟。

### 1.3 AXI5-Lite 互操作

- AXI5 Manager → AXI5-Lite Subordinate：如果 Manager 只使用 AXI5-Lite 子集，可直接连接；否则需要 conversion、protection 或 detection。
- AXI5-Lite Manager → AXI5 Subordinate：完全可互操作。
- AXI4-Lite Manager → AXI5-Lite Subordinate：完全可互操作。
- AXI5-Lite Manager → AXI4-Lite Subordinate：需要 Subordinate 进行 AXI ID reflection；如果 Manager 只发总线宽度事务，则可直接连接，否则需要转换、保护或检测。

### 1.4 AXI5 到 AXI5-Lite 转换

转换规则和 AXI4 到 AXI4-Lite 类似，但有两个关键区别：

- 事务 ID 原样、未经修改地传递；
- 数据宽度可变，窄于目标接口的事务直接传递，宽于目标接口的事务拆成多个事务。

具体规则：

1. burst length>1 拆成多个单拍事务；INCR、WRAP 和 FIXED burst 的后续地址按原 burst 类型生成。
2. 宽事务在目标 AXI5-Lite 宽度边界处拆分。
3. 拆分写事务的多个响应必须合并；错误响应 sticky，`SLVERR` 和 `DECERR` 同时出现时采用先收到的错误。
4. ID 直接传递。
5. `WSTRB` 直接传递。
6. exclusive sequence 中，exclusive write 必须失败。
7. `AxCACHE` 丢弃，所有访问按 Non-modifiable、Non-bufferable 处理。
8. `AxPROT` 原样传递。
9. `WLAST` 丢弃，`RLAST` 不要求存在，读数据通道的每个传输均视作已置位。

### 1.5 AXI4-Lite 升级到 AXI5-Lite

升级 AXI4-Lite Manager：

- 增加 ID 信号。若只支持按序响应，可使用单 bit ID，并将 `ARID`、`AWID` 置为 0。
- 增加 `AWSIZE`、`ARSIZE` 输出。原 AXI4-Lite Manager 只产生完整总线宽度事务，因此 32-bit 总线将其置为 `0b010`，64-bit 总线置为 `0b011`。

升级 AXI4-Lite Subordinate：

- 增加 ID 信号，并把 `ARID` 镜像到 `RID`、把 `AWID` 镜像到 `BID`；也可以增加乱序响应能力。
- 增加 `AWSIZE` 输入；可以利用它，也可以继续使用 `WSTRB` 判断实际写入字节。
- 完整支持 `WSTRB`：只写相关 strobe 指示的字节；不置位任何 strobe 的写事务也必须支持。
- 增加 `ARSIZE` 输入；可以据此只驱动传输中的有效字节，也可以继续驱动整个数据总线。

### 1.6 AXI5-Lite 信号重点

基础通道保留 AXI 的五通道结构。AXI5-Lite 信号表新增/强调：

- 全局：`ACLK`、`ARESETn`、`AWAKEUP`；
- 地址：`AWADDR`、`ARADDR`、`AWPROT`/`ARPROT`（接口实际使用 `AxPROT[1]` Secure 位）、`AWID`/`ARID`、`AWIDUNQ`/`ARIDUNQ`、`AWSIZE`/`ARSIZE`、`AWUSER`/`ARUSER`、`AWTRACE`/`ARTRACE`、`AWPOISON`/`RPOISON` 等按 property 条件出现；
- 响应：`BID`、`RID`、`BUSER`/`RUSER`、`BTRACE`/`RTRACE`；
- 写数据：`WDATA`、`WSTRB`、`WUSER`；
- 读数据：`RDATA`、`RRESP`、`RUSER`。

## 2. Chapter D1：ACE 概览（D1-149–D1-158）

### 2.1 系统一致性

系统级一致性允许系统组件共享内存，而不要求软件执行 cache maintenance。若两个组件对同一内存位置的写入被所有组件以相同顺序观察，则该内存区域是 coherent 的。

ACE 的目标包括：

- 在多个 cache 之间共享数据时保持正确性；
- 支持具有不同 cache 特性的组件交互；
- 最大化 cached data 的复用；
- 在高性能与低功耗之间提供选择。

系统设计者可以决定：

- 哪些内存范围保持一致性；
- 哪些系统组件实现一致性扩展；
- 软件采用何种模型与系统组件通信。

ACE 可用于：非均匀内存子系统的一致连接、具有本地一致性协议的组件、过滤一致性通信、MESI/ESI/MEI/MOESI 等不同协议的组件、把不具备一致性的组件包装进一致性系统、多级 cache 和不同 cache line 粒度的系统，以及优化主互连或多个子系统的设计。

### 2.2 ACE 版本与变体

规范演进：Issue D 首次描述 AXI Coherency Extensions；Issue E 加入澄清、建议和能力声明机制；Issue F 扩展 ACE 协议。当前列出的四种变体：

- ACE5
- ACE5-Lite
- ACE5-LiteDVM
- ACE5-LiteACP

低功耗接口的相关内容已经移除，改由 AMBA Low Power Interface Specification（ARM IHI 0068）覆盖。

### 2.3 ACE 术语

ACE 引入或重点使用的术语包括：

- Cache/拓扑：Caching Manager、Initiating Manager、Snooped Manager；Downstream cache、Local cache、Peer cache、Snooped cache；Main memory、Snoop filter。
- Cache 状态：Valid/Invalid、Unique/Shared、Clean/Dirty。
- Manager 操作：Load、Speculative read、Store；Permission to store、Permission to update main memory。
- 时间关系：AXI 使用 timely manner，ACE 还要求理解 “at approximately the same time”。

### 2.4 ACE 协议结构

ACE 在 AXI4 基础上提供硬件一致性 cache 支持：

1. 以五状态 cache 模型描述任意 cache line 的状态；
2. 在原 AXI 通道上增加信号，支持新的事务及把信息传到需要一致性支持的位置；
3. 增加与 Caching Manager 通信的通道，使另一 Manager 访问同一地址时能够进行 snoop。

ACE 还提供 barrier transactions（用于保证系统内事务排序）和 DVM（Distributed Virtual Memory）功能。ACE5 和 ACE5-Lite 不支持 barrier transactions。

### 2.5 Cache 状态模型

ACE cache line 的五种状态：

- **Invalid（I）**：该 cache 中不存在该 cache line。
- **UniqueClean（UC）**：只有本 cache 持有，未相对主存修改；可以直接 store，不必通知其他 cache。
- **UniqueDirty（UD）**：只有本 cache 持有，已相对主存修改；后续变化必须通知主存；可以直接 store，不必通知其他 cache。
- **SharedClean（SC）**：可能与其他 cache 共享；相对主存是否修改未知，但本 cache 不负责更新主存；store 前必须通知其他 cache。
- **SharedDirty（SD）**：可能共享，且相对主存已修改；本 cache 必须确保后续变化通知主存；store 前必须通知其他 cache。

状态规则：

- Unique 状态的 cache line 只能存在于一个 cache 中。
- 同一 cache line 若存在于多个 cache，每个副本都必须处于 Shared 状态。
- 某 cache 获得已有其他副本的 cache line 时，必须通知已有副本，使其转为 Shared。
- 删除一个副本时，不要求通知其他仍持有副本的 cache；因此 Shared 状态可能只剩一个副本。
- 相对主存已更新的 cache line 必须在某个 cache 中处于 Dirty 状态。
- 已更新且存在于多个 cache 时，只能在一个 cache 中处于 Dirty 状态。

### 2.6 ACE 通道

ACE 在现有 AXI4 通道上增加一致性相关信号：

- 读地址：`ARDOMAIN[1:0]`、`ARSNOOP[3:0]`、`ARBAR[1:0]`；
- 写地址：`AWDOMAIN[1:0]`、`AWSNOOP[2:0]`、`AWBAR[1:0]`、`AWUNIQUE`；
- 读数据：`RRESP[3:2]`；写数据和写响应通道没有额外 ACE 信号。

新增三个 ACE 专用通道：

- **AC snoop address channel**：Interconnect 向 Caching Manager 提供 snoop 地址及控制信息；
- **CR snoop response channel**：Caching Manager 对每个 snoop 返回一个响应；
- **CD snoop data channel**：可选，Caching Manager 返回 snoop data，典型用于 read 或 clean snoop 且该 Manager 拥有数据副本的情况。

还有两个 acknowledge 信号：

- `RACK`：Manager 表示读事务已完成；
- `WACK`：Manager 表示写事务已完成。

### 2.7 典型 load/store 流程

**从 Shareable 地址执行 load：**

1. Initiating Manager 在读地址通道发起读事务。
2. Interconnect 将地址通过 AC 通道 snoop 到可能持有副本的 Caching Manager。
3. 被 snoop 的 Manager 在 CR 返回响应，并可通过 CD 返回数据。
4. 若某个被 snoop 的 Manager 提供数据，Interconnect 可用读数据通道返回给 Initiating Manager。
5. 若没有提供数据，Interconnect 访问主存，再通过读数据通道返回数据。
6. Initiating Manager 用 `RACK` 表示事务完成。

**向 Shareable 地址执行 store：**

store 会删除其他 cache 中的副本，使发起 Manager 成为该 cache line 的唯一持有者。若只写部分 cache line，必须先取得当前 cache line 的完整副本，再通过 MakeReadUnique 等事务请求删除其他副本；如果写整个 cache line，则可直接发起 MakeUnique。完成 snoop 后，Manager 执行 store 并用 `RACK` 确认完成。

---

## 3. 与前文衔接

- AXI5-Lite 将 AXI4-Lite 的固定宽度/固定顺序放宽为可变宽度、带 ID、允许按 ID 重排序，但仍保持单拍且不支持 exclusive。
- AXI5-Lite 的 ID reflection 与前文 AXI4-Lite 互操作规则直接对应。
- ACE 的 Unique/Shared、Clean/Dirty 状态建立在前面 A4 内存属性和 A6 排序模型之上。
- `ARSNOOP`、`AWSNOOP` 与后续 ACE 事务章节相连；AC/CR/CD 三个 snoop 通道是理解 ACE 一致性流程的基础。

## 4. 原文定位

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 版本：ARM IHI 0022H.c
- PDF 物理页：139–158；印刷页 C2-139–C2-146、D1-149–D1-158。

## 5. 后续学习

下一批 PDF 物理页 159–178，继续 D1 的事务概览、事务处理、ACE 概念和协议错误，重点关注 load/store 的完整事务时序、snoop 响应含义、cache 状态转换及 RACK/WACK 的精确定义。
