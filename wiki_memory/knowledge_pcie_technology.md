---
name: knowledge_pcie_technology
description: PCI Express Technology 3.0 中文版已完成学习内容的统一知识库，覆盖第1章至第4章§4.7。
---

# PCIe Technology 3.0 学习知识库

> 本文件将已完成学习的分散笔记合并为按章节连续阅读的知识库。当前完成边界为第 7 章 §7.5；下一入口为第 7 章 §7.6 等时服务支持。

## 内容索引

- [第 1 章：背景](#第-1-章背景)
- [第 2 章：PCIe 体系结构概述](#第-2-章pcie-体系结构概述)
- [第 3 章：配置](#第-3-章配置)
- [第 4 章：地址空间与事务路由](#第-4-章地址空间与事务路由)
- [第 5 章：TLP Elements（已完成至§5.2.4.5）](#第-5-章tlp-elements已完成至5245)
- [第 5 章：TLP Elements（已完成至§5.2.5.5）](#第-5-章tlp-elements已完成至5255)
- [第 6 章：流量控制（已完成至§6.4）](#第-6-章流量控制已完成至64)
- [第 7 章：QoS（已完成至§7.5）](#第-7-章qos已完成至75)
- [知识关联与当前断点](#知识关联与当前断点)
- [原始分散笔记](#原始分散笔记)

---

# 第 1 章：背景

## 1.1 PCI/PCI-X 与 PCIe 的动机

PCIe 在软件层延续 PCI 的配置空间、资源分配和驱动交互模型，但在物理层从共享并行总线转为高速串行、点到点互连。传统 PCI 设备可包含最多 8 个 Function（编号 0–7），Function 可作为事务 Target，多数也可作为 Initiator/Bus Master。传统 PCI 使用 REQ#/GNT# 仲裁共享总线，地址和数据复用 AD 总线，并通过 FRAME#、IRDY#、TRDY#、DEVSEL#、STOP# 等信号完成事务握手。

传统 PCI 的 PIO 由 CPU 参与数据搬运，DMA 由设备或 DMA Engine 自主执行地址序列和块传输，Peer-to-peer 则允许一个 Bus Master 直接访问另一设备。Retry 用于 Target 尚未开始传输且需要较长延迟的情形；Disconnect 用于已传输至少一个双字但暂时无法继续的情形。两者都释放共享总线，Master 稍后重新仲裁并重试或续传。

共享并行总线的限制主要来自信号反射、公共时钟歪斜、多位信号歪斜、渡越时间、电气负载、引脚数量和布线成本。PCI-X 通过输入寄存、PLL 相移、属性阶段、split transaction、MSI、No Snoop 和 Relaxed Ordering 等方式提高效率，但更高速度仍受到共享总线和电气连接的限制。PCIe 通过串行差分、点到点链路突破这些瓶颈，同时保持软件兼容性。

## 1.2 地址、配置、错误和中断模型

PCI 地址空间包括 Memory、I/O 和 Configuration。传统 PCI Function 最多有 256 B 配置空间，前 64 B 为 Header，其余空间可放置能力结构。配置识别使用 Bus/Device/Function（BDF）以及配置空间内的 dword 偏移；传统 x86 通过 CF8h–CFBh 配置地址端口和 CFCh–CFFh 配置数据端口间接访问。

PCI 可对地址和数据执行偶校验。数据校验错误通过 PERR# 报告，地址校验错误通过 SERR# 报告。传统中断使用 INTA#–INTD#，后续平台使用 APIC 或 MSI；MSI 以向预定义地址执行写事务的方式传递中断向量，避免共享中断引脚和设备扫描。

## 1.3 本章结论

PCI/PCI-X 保留了有价值的软件事务和配置模型，但共享并行总线已受到带宽、时序、负载、引脚和成本的根本限制。PCIe 的核心转变是：以高速串行、差分、点到点链路取代共享并行总线，而将兼容性主要保留在软件和事务抽象层。

---

# 第 2 章：PCIe 体系结构概述

## 2.1 链路、Lane 与带宽

一个 PCIe Lane 包含一对发送差分对和一对接收差分对，因此两个方向可以同时独立传输。链路宽度可以是 x1、x2、x4、x8、x12、x16 或 x32；增加 Lane 会提高带宽，但也会增加成本、面积、功耗和布线复杂度。

PCIe 不使用贯穿整个系统的公共数据时钟，时钟嵌入数据流并在接收端恢复。差分信号提高共模噪声容限；CDR/PLL 从跳变中恢复时钟，接收端还需要自己的本地时钟覆盖低功耗或链路无数据状态。多 Lane 传输时，发送端进行字节条带化，接收端完成反条带化和 Lane 去偏斜。

Gen1/Gen2 使用 8b/10b 编码，Gen3 使用 8 GT/s 和 128b/130b 编码。按书中的近似峰值计算：

```text
Gen1 = (2.5 Gb/s × 2) / 10 = 0.5 GB/s/Lane
Gen2 = (5.0 Gb/s × 2) / 10 = 1.0 GB/s/Lane
Gen3 ≈ (8.0 Gb/s × 2) / 8 = 2.0 GB/s/Lane
```

总链路带宽再乘以 Lane 数。实际有效吞吐量还需扣除编码、TLP/DLLP、流量控制和协议效率开销。

## 2.2 拓扑和端口角色

PCIe 拓扑保持 PCI 软件可理解的树结构：

- **Root Complex（RC）**：CPU/DRAM 子系统与 PCIe 层次结构的接口集合。软件通常把 RC 内部结构视作根部的 PCI 总线。
- **Root Port**：RC 面向下游的端口，在配置空间中表现为根侧端口。
- **Switch**：提供扇出、聚合和 TLP 路由。每个端口在软件配置模型中具有类似 Bridge 的作用。
- **Bridge**：连接 PCIe 与 PCI、PCI-X 或其他总线，可作前向或反向连接。
- **Endpoint**：通常位于树的末端，只有一个上行端口。Native Endpoint 面向 PCIe 设计，Legacy Endpoint 可保留 I/O、锁定请求等兼容特性。

软件枚举发现拓扑，分配 Bus 号、Memory/I/O 资源和配置窗口；硬件内部实现不必与软件看到的逻辑 PCI 总线完全相同。

## 2.3 四层结构

PCIe 接口可按职责分为设备核心/软件层、Transaction Layer、Data Link Layer 和 Physical Layer。

### Transaction Layer

事务层根据设备核心或软件请求生成并解析 TLP，处理 Memory、I/O、Configuration、Message 等事务，执行事务排序、属性、Traffic Class、Virtual Channel 和流量控制。读请求通常是 Non-Posted，请求方使用 Requester ID 和 Tag 将返回的 Completion 与未完成请求匹配。

### Data Link Layer

数据链路层只保证相邻端口之间的可靠传输。它为 TLP 增加 Sequence Number 和 LCRC，并保存重传副本；接收端检查序列号和 LCRC 后返回 Ack DLLP，检测错误则返回 Nak DLLP，发送方重传未确认的 TLP。DLLP 还用于流量控制信用和链路电源管理。

### Physical Layer

物理层负责 TLP/DLLP 组帧、字节条带化、扰码、编码、串并转换、差分发送、接收、链路训练和电气状态。Ordered Set 由物理层处理，不能作为端到端 TLP 路由。

## 2.4 事务、排序、QoS 与流量控制

PCIe 事务按 Posted/Non-Posted 分类。Memory Write 和 Message 通常是 Posted，不返回 Completion；Memory Read、I/O、Configuration 和 Atomic 等请求通常是 Non-Posted，需要 Completion。Completion 可携带数据（CplD）或仅携带状态（Cpl）。

TLP 中的 Traffic Class 是 3-bit 优先级标识。Virtual Channel 为不同流量提供独立缓冲，VC 仲裁与端口仲裁共同决定发送机会。发送方只有在接收方公布足够信用时才能发送，接收方消费 TLP 后通过 Flow Control DLLP 更新信用。设置 TC 本身并不自动保证确定性带宽或时延。

## 2.5 MRd/CplD 协议回顾

一次 Memory Read 的完整跨层过程如下：

1. 设备核心提供地址、事务类型、长度、字节使能、TC 和属性。
2. Transaction Layer 生成 MRd TLP；32-bit 地址使用 3DW Header，64-bit 地址使用 4DW Header。
3. Data Link Layer 加入 12-bit Sequence Number 和 32-bit LCRC，并保存副本。
4. Physical Layer 组帧、条带化、扰码、编码并串行发送。
5. 接收端反向执行解码、去扰码、反条带化和去帧。
6. 接收端数据链路层检查序列号/LCRC，并返回 Ack；错误则返回 Nak 并触发重传。
7. Completer 根据 Requester ID、Tag 等信息生成 CplD，携带 Completer ID、完成状态和数据。
8. CplD 按 ID 路由返回 Requester，Requester 将完成状态和数据交付设备核心或软件层。

ECRC 是可选的端到端检查；LCRC 和 Sequence Number 是逐链路可靠性机制。

---

# 第 3 章：配置

## 3.1 配置空间和 B/D/F

Bus/Device/Function（BDF）是 PCIe 配置识别和路由的基础。每个 Device 最多有 8 个 Function；多功能设备由 Header Type 等字段识别。PCI 兼容配置空间提供传统 Header，PCIe 扩展配置空间扩展到每个 Function 4 KB。

PCIe 配置访问由 Root Complex 发起，普通 Endpoint 不能发起 Peer-to-peer 配置访问。软件可以通过传统 CF8h/CFCh 模型或 ECAM 访问配置空间。ECAM 将配置空间映射为内存区域，每个 Bus/Device/Function 获得固定的 4 KB 区域，整个映射空间为 256 MB。

## 3.2 Type 0/Type 1 配置请求与路由

Type 0 配置请求用于到达同一配置总线上的 Function；Type 1 配置请求用于通过 Bridge/Switch 向下游总线传播。Bridge 根据 Secondary Bus Number 和 Subordinate Bus Number 判断是否转发；配置请求只能沿 Root Complex 建立的配置路径到达目标。

Switch 的每个端口作为独立的 Bridge 配置实体参与范围检查。下游 Endpoint 通常使用 Device 0，ARI 等机制可改变传统 Device/Function 分配模型。配置 Completion 使用 ID 信息返回请求方；无法识别或不支持的配置访问可能返回 Unsupported Request（UR），设备尚未准备好时可使用 Configuration Request Retry Status（CRS）。

## 3.3 枚举流程

典型枚举流程如下：

1. 从 Root Complex 的根总线开始，探测每个可能的 Device/Function 的 Vendor ID。
2. 读取 Header Type，区分 Endpoint、Bridge 和多功能设备。
3. 为 Endpoint 评估 BAR 大小，记录资源需求。
4. 遇到 Bridge 时临时设置较宽的 Subordinate Bus 范围，递归扫描下游。
5. 为下游 Bridge 分配 Secondary/ Subordinate Bus Number，并在深度优先扫描结束后回填实际最大 Bus。
6. 汇总下游 Memory、Prefetchable Memory 和 I/O 需求，为 Bridge 设置 Base/Limit 窗口。
7. 为 BAR 分配地址，启用 Command 寄存器中的 Memory Space、I/O Space 和 Bus Master 等功能。

单 Root Complex 和多 Root Complex 平台需要协调总线号和资源范围，避免配置地址空间冲突。

## 3.4 MindShare Arbor

Arbor 是用于 PCI/PCI-X/PCIe 调试和学习的工具，可以扫描系统中的 Function，读取或写入 Memory、I/O 和配置空间，解码标准配置结构及部分设备特定寄存器，并可通过 XML 定义自定义寄存器译码。相关信息可保存为开放 XML 格式。

---

# 第 4 章：地址空间与事务路由

## 4.1 地址空间

PCIe 主要涉及 Configuration、Memory 和 I/O 三类地址空间。Memory 空间又可按访问属性区分 Prefetchable MMIO（P-MMIO）和 Non-Prefetchable MMIO（NP-MMIO）。只有读取没有副作用、重复读取不会改变语义的区域才适合预取；设备控制寄存器和具有读副作用的区域通常必须放入 NP-MMIO。

## 4.2 BAR

Type 0 Header 通常有 6 个 BAR，Type 1 Header 有 2 个 BAR。软件通过向 BAR 写入全 1、再读回掩码来评估资源大小，然后按资源要求和对齐约束分配基地址。BAR 类型包括 32-bit Memory BAR、连续 BAR pair 组成的 64-bit Memory BAR，以及 I/O BAR。64-bit BAR 必须占用连续的 BAR 对，并按规定顺序评估和编程。

Resizable BAR 允许设备和系统在支持的多个大小中选择更大的可寻址窗口。BAR 描述的是 Function 自身资源；Bridge 的 Base/Limit 描述的是下游窗口，两者不能混为一谈。

## 4.3 Bridge Base/Limit

Bridge 为下游资源维护 P-MMIO、NP-MMIO 和 I/O 窗口。Base/Limit 的粒度和对齐取决于空间类型；窗口应覆盖下游多个 Function 所需的汇总范围，而不包括 Bridge 自身 BAR。无效窗口应使用 Base 大于 Limit 的无效关系表达，不能简单地把两者都设置为 0。原书对无效窗口方向存在勘误，配置时应以规范和勘误为准。

## 4.4 地址路由寄存器检查

地址路由检查需要区分两类资源：Bridge 自身 BAR 由该 Bridge 作为 Function 的资源匹配；下游 Base/Limit 窗口用于判断一个地址是否应被转发到下游。枚举多个分支时，系统必须先汇总各分支的 P-MMIO、NP-MMIO 和 I/O 需求，再设置足够且对齐的窗口。

## 4.5 TLP 路由基础

TLP 到达入口端口后，端口先决定本地接收、向下转发、向上转发、Peer-to-peer 转发或报告错误。主要路由方法包括：

- **ID/BDF 路由**：用于 Configuration，也用于 Completion 和部分 Message。目标由 Bus、Device、Function 标识。
- **地址路由**：用于 Memory、I/O 和 Atomic 等事务。Endpoint 通过 BAR 匹配，Bridge/Switch 通过自身 BAR 或下游 Base/Limit 窗口匹配。
- **隐式路由**：主要用于 Message。路由行为由 Message Header 的路由代码和拓扑方向决定，不依赖普通地址或 BDF。

Memory Write 和 Message 通常是 Posted；Memory Read、I/O、Configuration、Atomic 等请求通常是 Non-Posted。Completion 使用 ID 路由返回 Requester。

## 4.6.1 ID 路由

ID 路由使用 BDF。Endpoint 通常根据目标 Bus/Device/Function 完成检查；Switch 的端口作为 Bridge 可能需要对入口和出口分别执行配置范围判断。3DW/4DW Header 的结构和 Requester/Completer ID 共同支撑配置、Completion 等事务的返回路径。

## 4.6.2 地址路由

地址路由使用 TLP 中的 32-bit 或 64-bit 地址。Endpoint 检查地址是否命中自身 BAR；Bridge/Switch 检查自身 BAR 与下游 Base/Limit 窗口。下行请求可穿过多个 Switch/Bridge，向上请求则沿层次结构返回 Root Complex；不命中任何合法范围时可报告 UR。部分实现还支持多播或特殊地址路由行为，但必须遵守端口方向和事务类型约束。

## 4.6.3 隐式路由

Message TLP 使用 Header 中的路由子字段选择发往 Root Complex、向下广播、终止于接收者，或由多个端口收集后送往 Root Complex。Switch 的上行和下行端口对广播、面向 Root Complex 的消息以及面向接收者的消息有不同处理规则。路由代码和发送方向不一致时，可能构成 Malformed TLP。

## 4.7 DLLP 与 Ordered Set 不会被路由

DLLP 只在相邻的两个端口之间传输，到达对端 Data Link Layer 后立即处理，不会由 Switch 继续路由到更远设备。Ordered Set 只在相邻链路的 Physical Layer 处理，不会上送 Transaction Layer，也不会跨越多条链路。只有 TLP 能够由 Root Complex、Switch 或 Bridge 等 Routing Element 跨多条链路转发。

因此，Ack/Nak、流量控制信用和链路电源管理是链路本地机制；它们不能被当作端到端事务或拓扑路由消息。TLP 才是需要结合端口规则、ID、地址或 Message 路由代码进行跨拓扑处理的协议单元。

---

# 知识关联与当前断点

## 已覆盖的主线

1. **背景 → 体系结构**：PCI/PCI-X 共享并行总线的带宽和电气限制推动 PCIe 采用高速串行点到点链路。
2. **体系结构 → 协议**：Transaction Layer 产生 TLP，Data Link Layer 负责相邻链路可靠性，Physical Layer 负责编码、组帧和电气发送。
3. **配置 → 地址分配**：枚举发现 BDF 拓扑并分配总线号与资源；BAR 和 Bridge Base/Limit 共同形成地址路由范围。
4. **地址 → 路由**：TLP 根据 ID、地址或 Message 路由；DLLP 和 Ordered Set 只在相邻链路/物理层处理。

## 第 5 章：TLP Elements（已完成至 §5.2.4.5）

第5章前五个周期覆盖包协议基础、TLP组包/拆包、通用Header、Fmt/Type编码、Digest/ECRC、字节使能、事务描述符和数据荷载规则。TLP由事务层生成，数据链路层添加序列号/LCRC并保存重传副本，物理层执行组帧、条带化、扰码和编码；接收端逆向处理并通过Ack/Nak完成链路可靠性。Header的Fmt决定3DW/4DW及是否带数据，Type决定事务类别；First/Last DW Byte Enable处理非对齐首尾字节，Requester ID+Tag构成事务ID，TC映射到VC。

## 当前学习断点

- 已完成：第 1–4 章、第 5 章 §5.1–§5.2.5.5、第 6 章 §6.1–§6.4，以及第 7 章 §7.1–§7.5。
- 下一入口：第 7 章 §7.6 **等时服务支持**。
- 下一步重点：TLP Header、Format/Type、Length、Requester ID、Completer ID、Tag、Byte Enable、Address、Completion Status、First/Last DW Byte Enable、数据载荷，以及各类 TLP 的具体格式。
- 本文件没有把第 5 章及以后尚未学习的正文当作已完成内容。

## 统一来源与适用说明

- 主要来源：`PCI Express Technology 3.0 中文版`。
- 各节的 PDF 页码、书内页码、核验说明和适用条件来自原始分散笔记；本文件是整理后的连续阅读版本。
- 带宽、负载、利用率等背景章节中的典型数值不能替代具体平台的 SI、时序和实现分析。
- TLP、DLLP、配置访问和路由的详细字段约束应以后续章节及对应 PCIe 版本规范为准。

# 原始分散笔记

以下文件保留为细粒度来源和后续增量修订入口：

- [第 1 章：背景](knowledge_pcie_chapter_1_background.md)
- [第 2 章：简介、链路与拓扑](knowledge_pcie_chapter_2_introduction_links_topology.md)
- [第 2 章：体系结构概述](knowledge_chapter_2_pcie_architecture_overview.md)
- [第 2 章：协议回顾](knowledge_pcie_chapter_2_protocol_review.md)
- [第 3 章：配置基础](knowledge_pcie_chapter_3_configuration_basics.md)
- [第 3 章：配置路由与枚举](knowledge_pcie_chapter_3_configuration_routing_enumeration.md)
- [第 3 章：MindShare Arbor](knowledge_pcie_chapter_3_section_3_14_arbor.md)
- [第 4 章：地址空间](knowledge_pcie_chapter_4_section_4_1_address_spaces.md)
- [第 4 章：BAR](knowledge_pcie_chapter_4_section_4_2_bars.md)
- [第 4 章：Base/Limit](knowledge_pcie_chapter_4_section_4_3_base_limit.md)
- [第 4 章：地址路由寄存器](knowledge_pcie_chapter_4_section_4_4_address_routing_registers.md)
- [第 4 章：TLP 路由基础](knowledge_pcie_chapter_4_section_4_5_tlp_routing_basics.md)
- [第 4 章：ID 路由](knowledge_pcie_chapter_4_section_4_6_1_id_routing.md)
- [第 4 章：地址路由](knowledge_pcie_chapter_4_section_4_6_2_address_routing.md)
- [第 4 章：隐式路由](knowledge_pcie_chapter_4_section_4_6_3_implicit_routing.md)
- [第 4 章：DLLP 和 Ordered Set 不会被路由](knowledge_pcie_chapter_4_section_4_7_dllp_ordered_set_not_routed.md)
