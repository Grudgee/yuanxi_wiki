---
name: knowledge_d3_ace_channel_signaling
 description: ACE地址事务编码、cache line约束、RRESP/RACK/WACK及AC/CR通道信令（PDF物理页179–198）
metadata:
  type: reference
  source: AXIACE5.pdf
  modified: 2026-07-21
---

# ACE Channel Signaling（D3，PDF 179–198）

## 1. ARSNOOP/AWSNOOP 地址控制编码

### 1.1 读地址通道

`ARBAR[0]`、`ARDOMAIN[1:0]`、`ARSNOOP[3:0]` 组合决定读事务：

- Non-snoop：`ARSNOOP=0000`，`ARDOMAIN=Non-shareable 或 System`，事务为 `ReadNoSnoop`。
- Coherent：`0000=ReadOnce`、`0001=ReadShared`、`0010=ReadClean`、`0011=ReadNotSharedDirty`、`0111=ReadUnique`。
- Cache maintenance：`1000=CleanShared`、`1001=CleanInvalid`、`1101=MakeInvalid`。
- Barrier：`ARBAR[0]=1`、`ARSNOOP=0000`，domain 可为任意合法 domain。
- DVM：`ARDOMAIN=Inner/Outer Shareable`，`1110=DVM Complete`、`1111=DVM Message`。

未使用的 `ARSNOOP` 编码保留。无 cache 的组件只需用 `ARDOMAIN` 表示读事务 shareability，并将 `ARSNOOP` 置 0：Non-shareable 按 ReadNoSnoop 处理，Shareable 按 ReadOnce 处理。

### 1.2 写地址通道

`AWBAR[0]`、`AWDOMAIN[1:0]`、`AWSNOOP[2:0]` 组合决定写事务：

- Non-snoop：`AWSNOOP=000=WriteNoSnoop`，domain 为 Non-shareable 或 System。
- Coherent：`000=WriteUnique`，`001=WriteLineUnique`。
- Memory update：`010=WriteClean`、`011=WriteBack`、`100=Evict`、`101=WriteEvict`。
- Barrier：`AWBAR[0]=1`、`AWSNOOP=000`，domain 可为任意合法 domain。

无 cache 的组件只需用 `AWDOMAIN` 表示 shareability，并将 `AWSNOOP` 置 0：Non-shareable 按 WriteNoSnoop，Shareable 按 WriteUnique 处理。

## 2. AWUNIQUE

`AWUNIQUE` 用于表示写事务的数据是否允许以 Unique 状态保留在低层 cache：

- `WriteNoSnoop`、`Evict`、`Barrier`：无意义，可任意值。
- `WriteUnique`、`WriteLineUnique`：如果 Manager 不保留 cache line 副本，可以置位；若保留副本，必须清零。
- `WriteClean`：必须清零，因为发起者保留副本，不能成为 Unique。
- `WriteBack`：cache line 为 Unique 时可置位；若为 Shared，必须清零。
- `WriteEvict`：必须置位；该事务只允许用于 UniqueClean cache line。

支持 `WriteEvict` 的 Manager 必须支持 `AWUNIQUE`。如果 Interconnect 不支持该信号，实现 AWUNIQUE 的 Manager 仍可连接；不支持 AWUNIQUE 的 Manager 也可连接支持它的 Interconnect，但除 WriteEvict 外必须驱动 LOW。WriteBack/WriteEvict 进行期间，Manager 不得返回允许其他 cache 创建副本的 snoop response。

## 3. Cache line 和事务约束

### 3.1 Cache line 大小

- 最小 cache line：16 bytes。
- 最大 cache line：`min(2048 bytes, 16 × 数据总线字节数)`。

必须是 cache line size 的事务包括：

`ReadClean`、`ReadNotSharedDirty`、`ReadShared`、`ReadUnique`、`CleanUnique`、`MakeUnique`、`CleanShared`、`CleanInvalid`、`MakeInvalid`、`WriteLineUnique`、`WriteEvict`、`Evict`。

这些 cache-line-size 事务还需满足：

- burst 长度为 1、2、4、8 或 16；
- burst 长度大于 1 时，每个 transfer 大小必须等于数据总线宽度；
- INCR 地址必须按 cache line 对齐，完整 burst 长度等于 `AxLEN × AxSIZE`；
- WRAP 地址按 `AxSIZE`（等于数据总线宽度）对齐；
- FIXED 不支持；
- `AxBAR` 必须为普通访问；
- `AxCACHE` 必须为 Modifiable；
- `AxPROT`、`AxQOS` 无额外限制；
- `AxLOCK`：`ReadNotSharedDirty`、`ReadUnique`、`MakeUnique`、`CleanShared`、`CleanInvalid`、`MakeInvalid`、`WriteLineUnique`、`WriteEvict`、`Evict` 必须为普通访问；`ReadClean`、`ReadShared`、`CleanUnique` 可为普通或独占编码。

`WriteLineUnique` 和 `WriteEvict` 必须所有写 strobe 都置位，不允许 sparse strobe。即使不传数据，`CleanUnique`、`MakeUnique`、`CleanShared`、`CleanInvalid`、`MakeInvalid`、`Evict` 仍必须用 `AxLEN` 表示正确 cache line 大小。

### 3.2 ReadOnce/WriteUnique

二者不受 cache line 大小限制，便于 legacy 组件使用：

- domain 必须 Inner 或 Outer Shareable；
- burst 类型只能 INCR 或 WRAP；
- `AxCACHE` 必须 Modifiable；
- `AxLOCK` 必须普通访问；
- `AxPROT`、`AxQOS` 无额外限制；
- WriteUnique 允许 sparse write strobe；
- FIXED 不支持，AXI 到 ACE-Lite 转换时必须转换 FIXED burst。

### 3.3 WriteBack/WriteClean

不受 cache line 大小限制，但必须限制在单个 cache line 范围内，允许部分更新：

- WRAP：长度只能 2、4、8、16；地址按数据总线宽度对齐；`AxSIZE × AxLEN` 不得超过 cache line 大小；
- INCR：长度不超过 16，不能跨 cache line 边界；
- FIXED 不支持；
- domain 不能为 System；
- barrier 必须为普通访问；
- `AxCACHE` 必须 Modifiable；
- `AxLOCK` 必须普通访问；
- 允许 sparse write strobes；
- 支持 snoop filter 的组件必须正确给出 domain，使 filter 跟踪 Inner/Outer Shareable 分配。

### 3.4 Barrier 事务

Barrier 地址控制约束：

- `AxADDR` 全 0；
- `AxBURST=INCR`；
- `AxLEN` 全 0；
- `AxSIZE` 等于数据总线宽度；
- `AxCACHE` 为 Normal、Non-cacheable；
- `AxLOCK` 为普通访问；
- `AxSNOOP` 全 0；
- `AxPROT` 无限制；
- domain 可任意选择，以决定 barrier 对哪些 Manager 生效。

## 4. RRESP 一致性响应位

ACE 在 `RRESP[3:2]` 增加：

- `RRESP[2] = PassDirty`：HIGH 表示 cache line 相对主存为 Dirty，且写回责任已交给发起 Manager 或 Interconnect；LOW 表示发起 Manager 不承担写回责任。
- `RRESP[3] = IsShared`：HIGH 表示其他 cache 可能仍有副本，或响应后该 cache line 应为 Shared；LOW 表示这是唯一缓存副本，可保持 Unique。

限制：

- `IsShared` 和 `PassDirty` 必须在一个 burst 的所有数据传输中保持不变；
- `IsShared` 对要求删除所有其他副本的 `ReadUnique`、`CleanUnique`、`MakeUnique`、`CleanInvalid`、`MakeInvalid` 必须为 LOW；
- `PassDirty` 对不允许传递 Dirty 数据的事务必须为 LOW，包括 `ReadOnce`、`ReadClean`、`CleanUnique`、`MakeUnique`、`CleanShared`、`CleanInvalid`、`MakeInvalid`；
- ReadNoSnoop、Barrier、DVM 中两个 bit 都为 LOW；
- CleanUnique、MakeUnique、CleanShared、CleanInvalid、MakeInvalid、Barrier、DVM 只有一次 R 通道传输，必须 `RLAST=1`，`RDATA` 可任意且被忽略；
- `EXOKAY` 只允许用于 ReadNoSnoop、ReadClean、ReadShared、CleanUnique。

## 5. RACK 与 WACK

### RACK

- Manager 发出，单周期脉冲；Interconnect 必须单周期接受；
- 不能早于最后一个读数据传输的 `RVALID/RREADY` 握手完成后的下一个周期；
- 对所有事务都必须发出，包括 coherent、barrier、DVM；
- 与读响应关联，顺序等同于最后一个读数据项，不携带额外排序信息；
- Interconnect 使用 RACK 确保不会在 Manager 仍未完成同地址前序事务时向其发 snoop。

### WACK

- Manager 发出，单周期脉冲；Interconnect 必须单周期接受；
- 不能早于关联事务 `BVALID/BREADY` 握手完成后的下一个周期；
- 对所有写事务都必须发出，包括 barrier；
- 顺序等同于对应写响应；
- Interconnect 使用 WACK 防止在 Manager 同地址前序事务未完成时再次向其发 snoop。

## 6. AC Snoop Address Channel

AC 通道是发给 Caching Manager 的输入通道，适用于：

- 持有 Shareable 数据副本的 Manager；
- 支持 DVM transaction 的 Manager。

AC 控制信息比普通地址通道少，不传递 burst type、burst length、transaction size、Modifiable/shareable 属性或 transaction ID。

不在 AC 通道出现的事务包括：`ReadNoSnoop`、`WriteNoSnoop`、`WriteBack`、`WriteClean`、`WriteEvict`、`Evict`。`WriteUnique` 通常以 `CleanInvalid` snoop 表示，`WriteLineUnique` 以 `MakeInvalid` 表示，`MakeUnique` 通常转换为 `MakeInvalid`，`CleanUnique` 通常转换为 `CleanInvalid`。

AC 信号：

- `ACVALID`：Interconnect 表示有效；
- `ACREADY`：Manager 可接受；
- `ACADDR`：snoop 首个传输地址；
- `ACSNOOP[3:0]`：snoop 类型；
- `ACPROT[2:0]`：保护属性，仅 `ACPROT[1]` 有 ACE 定义。

遵守普通 VALID/READY 规则：VALID 置位后，地址和控制必须保持不变直到 READY；READY 可以提前或同周期置位。`ACADDR` 按 snoop data transfer 大小对齐。

AC snoop 编码：

- `0000` ReadOnce
- `0001` ReadShared
- `0010` ReadClean
- `0011` ReadNotSharedDirty
- `0111` ReadUnique
- `1000` CleanShared
- `1001` CleanInvalid
- `1101` MakeInvalid
- `1110` DVM Complete
- `1111` DVM Message

Snoop transaction 必须为完整 cache line 长度；长度>1 时必须 WRAP，长度=1 时必须 INCR；宽度必须等于 CD snoop data channel 宽度。原始事务若不是完整 cache line，Interconnect 必须扩展转换。

## 7. CR Snoop Response Channel

CR 通道每个 AC 地址都必须有一个响应，响应顺序与 AC 地址顺序相同：

- `CRVALID`：Manager 表示响应有效；
- `CRREADY`：Interconnect 可接受；
- `CRRESP[4:0]`：snoop 响应。

VALID 置位后 CRRESP 必须保持不变直到 READY。

### CRRESP 位

- bit0 `DataTransfer`：HIGH 表示将在 CD 通道提供完整 cache line 数据；
- bit1 `Error`：HIGH 表示 snooped cache line 检测到错误，例如 ECC 发现损坏；
- bit2 `PassDirty`：HIGH 表示 snoop 前为 Dirty，写回主存责任转交给发起 Manager 或 Interconnect；
- bit3 `IsShared`：HIGH 表示 snoop 后仍保留副本；
- bit4 `WasUnique`：HIGH 表示 snoop 前 cache line 为 Unique，只有确认没有其他 cache 能持有副本时才允许置 HIGH。

关键约束：

- `IsShared=1` 对 `ReadUnique`、`CleanInvalid`、`MakeInvalid` 非法；
- `PassDirty=1` 且 `DataTransfer=0` 对任何事务非法；
- `DataTransfer` 的必要性由事务类型和 cache 状态决定：ReadOnce/ReadClean/ReadNotSharedDirty/ReadShared/ReadUnique 命中时可传数据；CleanInvalid/CleanShared 命中 Dirty 时必须传；MakeInvalid 从不要求传数据；
- Dirty 数据责任可对 `ReadNotSharedDirty`、`ReadShared`、`ReadUnique` 交给请求者；其他情况下由 Interconnect 负责写回；
- `WasUnique` 响应可让 snoop 过程结束，因为没有其他 cache 能保留副本；不支持生成该信息的 cache 可以永久置 LOW，但可能导致原本可返回 Unique 的数据被按 Shared 处理。

## 8. 与前文衔接

- `ARSNOOP/AWSNOOP` 把前一批 D1 的事务分类落到具体 AXI 地址信号编码。
- `RRESP[3:2]` 的 IsShared/PassDirty 将 cache 状态模型与读响应连接起来。
- AC/CR/CD 的 VALID/READY 握手延续 A3 的通道规则，但 AC 不携带普通 AXI 的完整事务属性。
- RACK/WACK 是 A6 排序保证在 ACE snoop 端的完成标志。

## 9. 原文定位与后续

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 版本：ARM IHI 0022H.c
- PDF 物理页：179–198；印刷页 D3-179–D3-198。
- 下一批：PDF 物理页 199–218，继续 CD snoop data、snoop channel dependencies，以及后续 ACE transaction signaling。
