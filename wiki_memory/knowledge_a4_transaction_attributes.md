---
name: knowledge_a4_transaction_attributes
description: AXI3/AXI4 事务属性、AxCACHE 内存类型、缓冲行为和访问权限
metadata: 
  node_type: memory
  originSessionId: 07d5e602-b8a4-4424-94dd-570763cdaafc
  modified: 2026-07-21T09:10:52.427Z
---

# 知识点摘要

- `ARCACHE`/`AWCACHE`（统称 `AxCACHE`）控制事务如何经过系统，以及系统级 cache 如何处理事务。
- AXI4 将 AXI3 的 `AxCACHE[1] Cacheable` 重命名为 `Modifiable`，实际功能保持一致，但进一步规定了 Non-modifiable 和 Modifiable 事务可否拆分、合并或改变属性。
- `AxCACHE[3:2]` 是分配/缓存查找提示：只要其中任一位为 1，事务必须执行 cache lookup；两位都为 0 时无需 cache lookup。
- 内存类型分为 Device、Normal Non-cacheable、Write-Through 和 Write-Back，并结合 Bufferable、Read-Allocate、Write-Allocate 等属性。
- 多个 Manager 访问同一内存区域时，必须对该区域是否 cacheable 保持一致认知；分配提示可以不同。
- `ARPROT[2:0]`/`AWPROT[2:0]`（统称 `AxPROT`）分别编码特权级、安全状态和指令/数据访问类型。

# 关键细节

## Subordinate 分类

- Memory Subordinate 必须正确处理所有事务类型。
- Peripheral Subordinate 的正确访问方式是 IMPLEMENTATION DEFINED；不符合该方式的访问仍必须按协议完成，以避免系统死锁，但之后不要求外设继续正确工作。
- Peripheral Subordinate 如果只需支持有限访问方式，可减少接口信号集合。

## AXI3 AxCACHE 定义

- `AxCACHE[0]`：Bufferable。置位后，interconnect 或其他组件可让事务延迟任意周期再到最终目的地；通常只与写有关。
- `AxCACHE[1]`：Cacheable。为 0 时禁止 allocation；为 1 时允许 allocation，且原始事务到最终目的地时的特性不必完全保持，可合并写或预取读。
- `AxCACHE[2]`：Read-Allocate 建议位；只有 `AxCACHE[1]=1` 时才允许置位。
- `AxCACHE[3]`：Write-Allocate 建议位；只有 `AxCACHE[1]=1` 时才允许置位。

## AXI4 Non-modifiable

- `AxCACHE[1]=0` 表示 Non-modifiable；事务不能拆成多个事务，也不能与其他事务合并。
- 必须保持不变的参数：`AxADDR`（因此也包括 `AxREGION`）、`AxSIZE`、`AxLEN`、`AxBURST`、`AxLOCK`、`AxPROT`。
- `AxCACHE` 只允许从 Bufferable 改为 Non-bufferable，不能做其他变化；事务 ID 和 QoS 可以改变。
- burst 长度大于 16 的 Non-modifiable 事务仍可拆分，生成事务只允许缩短 burst 并适当调整地址。
- Non-modifiable 的独占访问可调整 `AxSIZE`/`AxLEN`，但总访问字节数必须保持一致。
- 某些总线宽度变换无法满足 Non-modifiable 要求，此时实现可提供 IMPLEMENTATION DEFINED 机制标记发生了修改。

## AXI4 Modifiable

- `AxCACHE[1]=1` 表示 Modifiable；允许事务拆分、多个事务合并、读事务多取数据、写事务扩大地址范围（必须用 `WSTRB` 保证只更新目标位置）。
- 生成事务可修改 `AxADDR`、`AxSIZE`、`AxLEN`、`AxBURST`；不得修改 `AxLOCK` 和 `AxPROT`。
- `AxCACHE` 可修改，但必须确保事务按要求向其他组件可见，并且同一地址范围的所有生成事务使用一致的内存属性。
- ID 和 QoS 可修改。
- 禁止把访问改到原事务之外的不同 4KB 地址空间；也禁止把对 single-copy atomicity sized region 的单次访问变成多次访问。

## Allocate 位的 AXI4 含义

- AXI4 中一位表示“该事务发生过 allocation”，另一位表示“可能因另一事务发生 allocation”；读写通道对这两位的具体分配不同。
- 写通道：`AWCACHE[3]=Allocate`，`AWCACHE[2]=Other Allocate`。
- 读通道：`ARCACHE[2]=Allocate`，`ARCACHE[3]=Other Allocate`。
- 只要 `AxCACHE[3:2] != 0b00`，事务必须在 cache 中查找；若为 `0b00`，无需查找。
- 因定义改变，同一内存位置的读事务和写事务 `AxCACHE` 值可以不同。

## AXI4 内存类型编码

- `0000`：Device Non-bufferable。
- `0001`：Device Bufferable。
- `0010`：Normal Non-cacheable Non-bufferable。
- `0011`：Normal Non-cacheable Bufferable。
- Write-Through：基础行为为可从中间点获得写响应、写必须及时对最终目的地可见、读可来自中间 cache、事务可修改、读可预取、写可合并，并要求 cache lookup。
- Write-Back：基础行为为可从中间点获得写响应、写不要求在最终目的地可见、读可来自中间 cache、事务可修改、读可预取、写可合并，并要求 cache lookup。
- Write-Through/Write-Back 的 No-Allocate、Read-Allocate、Write-Allocate、Read-and-Write-Allocate 变体只改变 allocation 推荐提示，不禁止实际 allocation。
- 表 A4-5 未列出的编码为 Reserved；同一内存类型可在读/写通道使用不同合法编码，以保持 AXI3 兼容。

## 各内存类型的关键行为

- Device Non-bufferable：读写均来自/到达最终目的地；Non-modifiable；不可读预取、不可写合并。
- Device Bufferable：写响应可来自中间点，但写必须及时到达最终目的地；读必须来自最终目的地；Non-modifiable；不可预取/合并。两种 Device 类型都等同于 Non-modifiable。
- Normal Non-cacheable Non-bufferable：响应和读数据来自最终目的地；Modifiable；写可合并。
- Normal Non-cacheable Bufferable：写响应可来自中间点，写必须及时到最终目的地；读可来自最终目的地或正在前往最终目的地的写，但必须取最新版本且该数据不能为后续读缓存；Modifiable；写可合并。

## 属性不匹配和变更

- 同一地址区域在某层级被定义为 Not Cacheable 时，所有 Manager 都必须使用 `AxCACHE[3:2]=00`；定义为 Cacheable 时，所有 Manager 都必须使其中至少一位为 1。
- 不同 Manager 可以使用不同 allocation hints。
- Normal Non-cacheable 区域可以用 Device 事务访问；Bufferable 区域也可使用不允许 bufferable 行为的事务。
- 从一种内存类型切换到不兼容类型时，典型流程是：所有 Manager 停止访问；一个 Manager 执行必要的 cache maintenance；所有 Manager 再使用新属性访问。

## Transaction buffering

- Device Bufferable、Normal Non-cacheable Bufferable、Write-Through 写事务可以从非最终目的地获得响应，但必须及时向最终目的地推进。
- 中间 buffer 转发 Normal Non-cacheable Bufferable 读时，读数据不可无限保留；重复读取不得重置用于确保其最终推进的超时机制。
- 能保存和合并写的中间 buffer 也不得让写无限保留；持续写同一位置不得重置最终推进机制。
- 规范没有定义 `in a timely manner` 的具体周期数或通用超时探测机制。

## AxPROT 访问权限

- `AxPROT[0]`：`0=Unprivileged`，`1=Privileged`。
- `AxPROT[1]`：`0=Secure`，`1=Non-secure`；置位代表 Non-secure，与 Arm Security Extensions 的常见定义一致。
- `AxPROT[2]`：`0=Data`，`1=Instruction`。该位只是提示，并非所有情况都准确；混合或未知时建议 Manager 置 0 表示 data，除非明确是 instruction。
- AXI 只区分特权/非特权两个级别，若处理器有更多级别，映射方式由处理器文档决定。

## Legacy 和使用示例

- AXI4 要求同一 ID、发往同一 Subordinate 的所有 Device 事务相互有序；AXI3 未显式要求。依赖该行为的 AXI4 组件不能连接到不支持该行为的 AXI3 interconnect。
- 新 AXI3 设计强烈建议实现 AXI4 的该排序要求。
- Device Bufferable 写响应只能说明中间 buffer 已接受，不能直接说明所有其他 Manager 已可见。若同一 Manager、同一 ID、同一 Subordinate 先发 Device Bufferable 写，再发 Device Non-bufferable 写，则后者响应可作为前面写已到最终目的地的屏障；只保证满足这些同一条件的事务。

# 原文引用

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 版本/日期：ARM IHI 0022H.c，Copyright 2003–2021。
- PDF 物理页：61–78；文档印刷页：A4-61–A4-78。
- 章节：Chapter A4 `Transaction Attributes`；A4.1–A4.9。
- 关键位置：Subordinate/属性概览 A4-62；AXI3 AxCACHE A4-63；AXI4 Modifiable 变化 A4-64–A4-68；内存类型 A4-69–A4-72；不匹配属性 A4-73；事务缓冲 A4-74；AxPROT A4-75；legacy/使用示例 A4-76–A4-77。

# 适用条件与例外

- AXI3 和 AXI4 对 `AxCACHE[1]` 的命名及 Read/Write Allocate 含义不同，回答时必须标明协议版本。
- `in a timely manner` 没有统一周期数，不能自行推导具体 timeout。
- 本批次只覆盖 A4-78；后续 ordering、exclusive access、cache 一致性章节可能增加约束。

# 关联章节

- Chapter A3 Single Interface Requirements
- Chapter A5 Transaction ordering
- Chapter A7 Exclusive accesses
- ACE/ACE5 cache coherency chapters

# 待核验问题

- 后续需结合 A5 排序模型确认 Device 事务和不同 ID 的完整顺序规则。
- 后续需结合 A7 核对 Non-modifiable 独占访问调整 `AxSIZE/AxLEN` 的限制。
