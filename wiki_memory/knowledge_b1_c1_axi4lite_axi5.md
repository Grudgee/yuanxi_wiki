---
name: knowledge_b1_c1_axi4lite_axi5
 description: AXI4-Lite 定义、互操作与转换机制，以及 AXI5 属性和新增信号（PDF物理页119–138）
metadata:
  type: reference
  source: AXIACE5.pdf
  modified: 2026-07-21
---

# AXI4-Lite 与 AXI5（PDF 物理页 119–138）

## 1. Chapter B1：AMBA AXI4-Lite（印刷页 B1-119–B1-126）

### 1.1 AXI4-Lite 的定义（B1-120–B1-121）

AXI4-Lite 面向不需要完整 AXI4 功能的简单控制/寄存器接口。核心约束：

- 所有事务都是 burst length=1。
- 数据访问总线宽度固定为 32-bit 或 64-bit，且每个事务使用完整数据总线宽度。
- 所有访问均为 Non-modifiable、Non-bufferable。
- 不支持 exclusive access。

接口仍有五个通道：写地址 AW、写数据 W、写响应 B、读地址 AR、读数据 R。AXI4 信号在 AXI4-Lite 中的变化：

- `RRESP`、`BRESP` 不支持 `EXOKAY`；AXI4-Lite 不能报告 exclusive 成功。
- `AWLEN`、`ARLEN` 被固定为 1（等价于 AXI4 中 AxLEN=0）。
- `AWSIZE`、`ARSIZE` 等于数据总线宽度；仅允许 32/64-bit 固定宽度。
- `AWBURST`、`ARBURST` 没有意义，因为 burst 长度恒为 1。
- `AWLOCK`、`ARLOCK` 等价为 0，所有访问均为 Normal。
- `AWCACHE`、`ARCACHE` 等价于 `0b0000`，即 Non-modifiable、Non-bufferable。
- `WLAST`、`RLAST` 没有实际 burst 边界意义；单 beat 事务等价于已置位。

### 1.2 总线宽度、写选通和可选信号（B1-121）

- AXI4-Lite 只有固定 32-bit 或 64-bit 总线宽度；多数组件采用 32-bit，确需 64-bit 原子访问的组件才采用 64-bit。
- 64-bit 组件可以服务 32-bit Manager，但实现必须把看到的所有事务当作 64-bit 事务；通常通过寄存器映射提供适合 32-bit 访问的低 32-bit 位置来实现互操作。
- 支持 `WSTRB`，因此可以实现多字节寄存器和 8/16-bit 访问。Subordinate 可：完整使用 strobe、忽略 strobe 并视作全宽访问，或检测不支持的 strobe 组合并返回错误。
- AXI4-Lite 不支持 AXI ID；事务必须按序，且所有访问使用单一固定 ID。AXI4-Lite Subordinate 可以选择支持 ID 信号，以便不经修改地连接到 full AXI，但这不是基础 AXI4-Lite 要求。
- 不支持 data interleaving，因为 burst length=1。

### 1.3 AXI 与 AXI4-Lite 互操作（B1-122）

互操作表的结论：AXI Manager↔AXI Subordinate、AXI4-Lite Manager↔AXI Subordinate、AXI4-Lite Manager↔AXI4-Lite Subordinate 均可正常工作；唯一需要特殊考虑的是 **full AXI Manager 连接 AXI4-Lite Subordinate**。

原因是 AXI4-Lite 没有 ID，而 full AXI Manager 需要在响应中获得与地址关联的 AXI ID。因此需要 ID reflection：AXI4-Lite Subordinate 将事务地址关联的 AXI ID 原样返回到读/写响应。如果不能保证 AXI Manager 只发出 AXI4-Lite 子集事务，则必须增加适配逻辑。

直接连接的 AXI4-Lite Subordinate 也可以内部实现 ID reflection，从而在系统能保证访问符合 AXI4-Lite 子集时直接挂到 full AXI 连接。规范建议 ID reflection 使用 `AWID` 而不是 `WID`，以兼容 AXI3 和 AXI4。

### 1.4 Defined conversion mechanism（B1-123–B1-124）

转换要求源 AXI 数据宽度大于等于目标 AXI4-Lite 宽度；否则必须先做数据宽度转换。合法 full AXI 事务转换规则：

1. burst length>1：拆成多个 length=1 事务；事务数量等于原 burst beat 数。
2. 后续 beat 地址必须按原 burst 类型生成：未对齐起始地址对 INCR/WRAP 的处理不同；FIXED burst 对所有 beat 使用同一地址。
3. 长度>1 的写 burst 拆分后，转换组件必须合并所有写响应为原始事务的单一响应。错误响应具有 sticky 语义；如果同时收到 `SLVERR` 和 `DECERR`，以先收到的响应作为合并结果。
4. 单个事务宽度大于 AXI4-Lite 接口宽度：拆成多个与接口宽度相同的事务，且在按目标宽度对齐的边界处拆分。
5. 写事务拆成多个较窄事务时，也必须合并响应；错误同样 sticky，`SLVERR`/`DECERR` 冲突时取第一份收到的响应。
6. 比 AXI4-Lite 更窄的事务直接传递，不转换。
7. `WSTRB` 直接传递并保持不变；没有 strobe 的写事务也直接传递，AXI4-Lite 不要求抑制它们。
8. 所有 `AxLOCK` 被丢弃。锁定事务序列的 lock guarantee 只在下游仲裁时丢失；由于 AXI 信令要求，exclusive write 必须失败。
9. 所有 `AxCACHE` 被丢弃；事务按 Non-modifiable、Non-bufferable 处理。这是允许的，因为 AXI 允许把 Modifiable 当作 Non-modifiable、把 Bufferable 当作 Non-bufferable。
10. `AxPROT` 原样传递。
11. `WLAST` 丢弃；`RLAST` 不要求存在，读数据通道上的每个传输都视作已置位。

### 1.5 Conversion、protection、detection（B1-125–B1-126）

- **Conversion**：把所有事务转换成符合 AXI4-Lite 要求的格式。
- **Protection**：检测不合规事务，丢弃它并向产生事务的 Manager 返回错误。
- **Detection**：观察超出 AXI4-Lite 要求的事务，通知控制软件，同时允许事务在硬件接口层继续进行。

三种实现层级：

- Full conversion：按定义的转换机制转换所有 AXI 事务。
- Simple conversion with protection：可简单转换的事务继续传播；需要更复杂转换的事务被抑制并报错。典型地丢弃 `AxLOCK`/`AxCACHE`，而对 burst length 或数据宽度转换报错。
- Full protection：所有不符合 AXI4-Lite 要求的事务都抑制并报错。

保护机制丢弃事务时必须给出协议合规的错误响应，避免死锁；例如 full AXI 读 burst 必须对每个 beat 返回读数据并正确置 `RLAST`。Conversion+Detection 的组合适合硬件调试和未来软件扩展：既不阻止意外访问，又能通知软件。

### 1.6 设计权衡

- Protection 门数较少，但不能处理预见之外的访问。
- Conversion 更能兼容未预见访问，提高软件可移植性。
- Conversion 可让 AXI 基础设施更高效，例如把写 FIFO 的多个写操作作为一个 burst 发出。
- 对窄链路，转换可更高效地复用地址与数据 payload 信号；也能让更多组件适配 AXI4-Lite，通过把 burst 转成单事务来支持 sparse strobe。

## 2. Chapter C1：AMBA AXI5（印刷页 C1-129–C1-138）

### 2.1 AXI5 的定位和接口属性（C1-130–C1-131）

AXI5 在 AXI4 基础上扩展能力。接口通过可设置的 property 宣告是否支持某项能力；Manager/Subordinate 根据这些属性决定新增信号和事务行为。

表 C1-1 的属性及含义：

- `Atomic_Transactions`：支持不只是单次访问、还具有与访问关联操作的原子事务。
- `Check_Type`：数据检查信令，可检测并潜在纠正已损坏的数据字节。
- `Poison`：标记一组数据字节之前已经损坏。
- `QoS_Accept`：Subordinate 可声明将接受的事务 QoS 值。
- `Trace_Signals`：每个通道增加 Trace 信号，用于调试、追踪和性能测量。
- `Loopback_Signals`：允许发起事务的 agent 在索引表中存储与事务有关的信息，以支持 loopback。
- `Wakeup_Signals`：表示接口上存在活动，用于低功耗唤醒。
- `Untranslated_Transactions`：支持 untranslated transaction，使同一接口上的不同事务使用不同地址转换方案。
- `NSAccess_Identifiers`：支持 Non-secure access identifier，用于受保护数据的存储和处理。
- `MPAM_Support`：支持 Memory Partitioning and Monitoring（MPAM）。
- `Unique_ID_Support`：支持 Unique ID Indicator。
- `Read_Interleaving_Disabled`：支持来自不同事务的读数据 beat 交错控制/禁用能力。
- `Read_Data_Chunking`：支持以可重排序 chunks 返回读数据。
- `MTE_Support`：支持 Memory Tagging Extension（MTE）。
- `Regular_Transactions_Only`：定义 Manager 是否只发 Regular transactions，以及 Subordinate 是否只支持 Regular transactions。
- `Exclusive_Accesses`：定义 Manager 是否发出 exclusive access 或 Subordinate 是否支持它。
- `Max_Transaction_Bytes`：定义事务的最大字节数。
- `Consistent_DECERR`：定义 Subordinate 是否在读/写响应的所有 beat 上一致地产生 DECERR。

### 2.2 AXI5 新增通道信号（C1-132–C1-137）

信号只在相应 property 启用时出现；A8 中的 AXI4 附加信号仍适用。

**写地址通道（Manager 发出）**：

- `AWATOP`：原子事务的类型和端序。
- `AWTRACE`：写事务追踪。
- `AWLOOP`：写事务 loopback 值。
- `AWMMUSECSID`、`AWMMUSID`、`AWMMUSS​​IDV`、`AWMMUSSID`、`AWMMUATST`、`AWMMUFLOW`：untranslated transaction 相关的 secure stream ID、stream ID、valid、substream ID、是否经过 PCIe ATS 转换、SMMU flow 信息。
- `AWNSAID`：写事务 Non-secure Access Identifier。
- `AWMPAM`：写地址通道的 MPAM 信息。
- `AWIDUNQ`：写地址通道 Unique ID Indicator，高有效。
- `AWTAGOP`：写请求的 MTE tag operation。

**写数据通道（Manager 发出）**：

- `WPOISON`：标记本次传输中的写数据已损坏。
- `WTRACE`：写数据追踪。
- `WTAG`：与写数据关联的 tag。
- `WTAGUPDATE`：指示 Update 操作中必须写入内存的 tags。

**写响应通道**：

- `BTRACE`：Interconnect 对特定写事务的追踪。
- `BLOOP`：写响应的 loopback 值。
- `BIDUNQ`：Subordinate 返回的 Unique ID Indicator，高有效。
- `BTAGMATCH`：写事务 tag 比较结果。
- `BCOMP`：写操作完成性指示，和 CMO on Write、Persist CMO、MTE 等能力有关。

**读地址通道（Manager 发出）**：

- `ARTRACE`、`ARLOOP`：读事务追踪和 loopback 值（`ARLOOP` 在响应中通过 `RLOOP` 反射）。
- `ARMMUSECSID`、`ARMMUSID`、`ARMMUSS​​IDV`、`ARMMUSSID`、`ARMMUATST`、`ARMMUFLOW`：untranslated read transaction 信息。
- `ARNSAID`：读事务 Non-secure Access Identifier。
- `ARMPAM`：读地址通道 MPAM 信息。
- `ARIDUNQ`：读地址通道 Unique ID Indicator。
- `ARCHUNKEN`：读数据 chunking enable。
- `ARTAGOP`：读请求的 MTE tag operation。

**读数据通道**：

- `RPOISON`：标记读数据已损坏。
- `RTRACE`、`RLOOP`：读响应追踪和 loopback 值。
- `RIDUNQ`：读数据通道 Unique ID Indicator。
- `RCHUNKV`：`RCHUNKNUM` 和 `RCHUNKSTRB` 有效指示。
- `RCHUNKNUM`：读数据 chunk 编号。
- `RCHUNKSTRB`：读数据 chunk strobe。
- `RTAG`：与读数据关联的 tag。

**额外信令（C1-137）**：

- `VAWQOSACCEPT`：Subordinate 对写事务 QoS 值的接受级别。
- `VARQOSACCEPT`：Subordinate 对读事务 QoS 值的接受级别。
- `AWAKEUP`：Manager 指示写或读地址通道已发起活动，用于 wakeup/低功耗。

## 3. 与前文的衔接

- A4 的 `AxCACHE` 属性在 AXI4-Lite 中被固定为 Non-modifiable、Non-bufferable；B1 的转换规则明确允许丢弃原始 `AxCACHE`。
- A5/A6 的 ID 与排序模型解释了 AXI4-Lite 为何必须按序且无 ID；full AXI Manager 接 AXI4-Lite Subordinate 时需 ID reflection。
- A7 的 exclusive access 在 AXI4-Lite 中完全不支持；B1 转换中丢弃 `AxLOCK`，exclusive write 必须失败。
- A8 中的 QoS、region、user 等附加信令是 AXI4 的扩展；C1 则以 property 体系继续扩展 AXI5 的原子、保护、追踪、转换、MPAM 和 MTE 能力。

## 4. 本批次原文定位

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 版本：ARM IHI 0022H.c，Copyright 2003–2021
- PDF 物理页：119–138；印刷页 B1-119–B1-126、Part C 起始页、C1-129–C1-138。
- 章节：Chapter B1 `AMBA AXI4-Lite`；Chapter C1 `AMBA AXI5`。

## 5. 后续学习

下一个批次为 PDF 物理页 139–158，继续 Chapter C1（若有后续内容）及之后章节。重点关注 AXI5 property 的约束和新增信号的精确定义，尤其 `AWATOP` 原子事务、数据检查/Poison、读数据 chunking、untranslated transactions、MTE 与 MPAM。
