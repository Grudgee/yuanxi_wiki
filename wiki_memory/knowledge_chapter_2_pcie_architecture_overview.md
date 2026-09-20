---
name: knowledge_chapter_2_pcie_architecture_overview
description: PCIe Technology 3.0 中文版第2章概述，涵盖串行链路、拓扑、分层架构、TLP/DLLP、事务、流量控制与物理层基础。
---

# 知识点摘要

- PCIe 在软件层保持 PCI 兼容，但物理层改为双单工（同时双向传输）的串行点到点链路；每条链路由 1、2、4、8、12、16 或 32 条 lane 组成，宽度写作 x1–x32。增加 lane 可提高带宽，但也增加成本、面积和功耗。
- Gen1/Gen2 使用 8b/10b，单 lane 双向聚合带宽分别约为 0.5 GB/s、1.0 GB/s；Gen3 使用 8 GT/s 与 128b/130b，单 lane 双向约 2.0 GB/s（精确值还应计入 128b/130b 开销）。总带宽再乘链路宽度。
- PCIe 以嵌入数据流的时钟恢复、差分信号和多 lane 去歪斜，缓解并行总线的渡越时间、公共时钟歪斜和多 bit 信号歪斜问题；接收端仍需本地内部时钟，不能把恢复时钟当作所有逻辑的工作时钟。
- PCIe 拓扑保持 PCI 软件可理解的树结构：Root Complex 位于根部，Switch 提供扇出/聚合和路由，Bridge 连接 PCI/PCI-X 或其他总线，Endpoint 位于分支末端。RC 内部总线和交换机内部结构可由软件抽象为 PCI 总线/桥；枚举负责发现拓扑并分配总线号与资源。
- PCIe 接口按职责分为设备核心/软件层、事务层、数据链路层和物理层。事务层产生/解析 TLP、执行事务排序、QoS 和流量控制；数据链路层产生/解析 DLLP，并用 Ack/Nak、序列号和 LCRC 做链路级可靠传输；物理层负责组帧、条带化、扰码、编码、串行差分发送及链路训练。
- 事务分为 Memory、IO、Configuration、Message 四类；读、IO 写和配置写属于非报告式事务，需要完成包；内存写和消息属于报告式事务，不返回完成包但仍参加数据链路层 Ack/Nak。非报告式读用 Requester ID/BDF 和 Tag 将多个完成包关联回原请求。
- TLP 的核心由事务层提供，可选 ECRC 用于端到端检查；数据链路层加入序列号和 LCRC，物理层再加入相应帧界定/编码信息。LCRC 覆盖每条链路，ECRC 可覆盖交换设备内部转发可能遗漏的错误。
- QoS 通过 3-bit Traffic Class、Virtual Channel 缓冲区以及 VC/端口仲裁实现优先级；同一 VC 通常保持顺序，不同 TC（通常进入不同 VC）之间不形成软件可见的排序关系。流量控制由 DLLP 报告接收缓冲区可用空间，发送方不得发送超过对端信用额度的 TLP。

# 关键细节

## 链路、速率与信号

- 单 lane 包含一对发送差分对和一对接收差分对；每个方向独立，因此一条链路可同时收发。差分接收以正负信号之差判定位值，对共模噪声更有容限。
- Gen1：`(2.5 Gb/s × 2) / 10 = 0.5 GB/s/lane`；Gen2：`(5.0 Gb/s × 2) / 10 = 1.0 GB/s/lane`；Gen3：`(8.0 Gb/s × 2) / 8 ≈ 2.0 GB/s/lane`，后者的 128b/130b 约 1.54% 编码开销在书中该处计算中暂略。
- 发送端把时钟嵌入数据流，接收端用 PLL/CDR 恢复；8b/10b 限制连续相同位的长度，帮助 PLL 保持锁定。链路进入低功耗且无数据跳变时，设备必须依靠自己的本地时钟。
- 物理层先对多 lane 做 byte striping，接收端再聚合并校正 lane-to-lane skew；每个字节还会扰码以减少连续 0/1 和 EMI。Gen1/2 使用 8b/10b，Gen3 使用 128b/130b。

## 拓扑与兼容性

- RC 是 CPU/DRAM 一侧与 PCIe 层次结构的接口；Root Port 在配置空间中表现为根侧端口。Switch 根据地址或其他路由信息选择输出端口；Bridge 可以做 PCIe↔PCI/PCI-X 或其他总线的前向/反向连接。
- Native Endpoint 是为 PCIe 设计的内存映射设备；Legacy Endpoint 可保留 IO、IO 事务和锁定请求等遗留能力。Endpoint 通常只有一个向上的端口，不再生成下级分支。
- 软件仍看到兼容的配置首部、桥和总线；枚举后的总线号分配使 RC 内部结构和 Switch 内部结构呈现为 PCI 风格的逻辑拓扑。

## 分层与包处理

- 设备核心层提供事务类型、地址、数据量等请求信息，接收端则消费事务层上送的结果；它不是规范定义的 PCIe 协议层。
- 事务层把请求放入相应 VC 缓冲区，支持 Memory/IO/Configuration/Message TLP；读请求可产生多个完成包，单个 TLP 数据荷载上限为 4 KB（实际设备往往更小）。
- 非报告式读的完成包携带 Requester 的 BDF 作为返回路由，并复制 Tag；发起方用 Tag 将完成包与未完成请求匹配。完成状态字段可报告事务错误。
- DLLP 只在相邻两个端口的数据链路层之间传递，不经过交换机路由到更远设备；通常为 8 bytes。其用途包括 Ack/Nak、流量控制和链路电源管理。
- 发送方为每个 TLP 保存重传副本；接收方校验序列号/LCRC 后发 Ack DLLP，发送方按 Ack 序列号清除已确认及更早副本；检测错误则发 Nak，发送方重传未确认 TLP。
- 物理层逻辑部分完成包边界、条带化、扰码和编码；电气部分通过 AC 耦合的差分发送器/接收器连接。Ordered Set 不是可路由 TLP/DLLP，而是只到达相邻端口物理层，用于训练、时钟容忍度补偿和低功耗状态指示。

## QoS、排序与流量控制

- TC 是包内 3-bit 优先级标识；端口为不同 VC 保留独立缓冲区，VC 仲裁和交换机端口仲裁共同决定发送机会。能否提供确定性时延/带宽取决于拓扑、缓冲和仲裁配置，而不是仅设置 TC 就自动保证。
- 发送方只在接收方公布足够缓冲空间时发送。接收方消费 TLP、释放空间后，以 Flow Control DLLP 更新信用值。使用 DLLP 而不是 TLP 传递信用报告，可避免双方因接收缓冲满而形成死锁。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 版本/日期：PDF 元数据标注 2026 年 6 月 1 日；内容标注 PCI Express Technology 3.0 中文版
- 位置：PDF p.41（第2章标题页）、p.42–p.82（第2章正文 §2.1–§2.2.4.5）；书内约 p.1–p.42
- 依据：本周期用 `pdftotext -layout` 提取 PDF p.42–p.82，覆盖第2章“PCIe 体系结构概述”的 §2.1“PCI Express 简介”和 §2.2“设备层次介绍”至 §2.2.4.5“字符集”；在进入 §2.3“协议回顾”前停止。

# 适用条件与例外

- 本记忆是第2章高层概述，不替代后续章节对 TLP 字段、配置空间、路由、流量控制、Ack/Nak、物理层 Gen3 编码和链路训练状态机的规范级描述。
- 书中 Gen3 带宽示例为便于计算暂忽略 128b/130b 的 2/130 开销；工程吞吐量还会受到 TLP/DLLP 开销、协议效率、链路宽度协商和实现缓冲的影响。
- “数据链路层通常 8 bytes”“TLP 最大 4 KB”等是本书本节的概述性表述，具体实现和规范限制应以后续专章及 PCIe 版本规范为准。

# 关联章节

- 第3章 配置综述：配置空间、枚举、配置事务和端点/桥识别
- 第4章 地址空间和事务路由：地址空间、路由方法和 TLP 路径
- 第5章 TLP 元素：TLP 首部、字段、Requester ID、Tag 和完成状态
- 第6章 流量控制；第7章 QoS；第10章 Ack/Nak 协议
- 第14章 链路初始化和训练；第16章 电源管理

# 待核验问题

- 无。§2.3 的 MRd/CplD 跨层时序已在 `knowledge_pcie_chapter_2_protocol_review.md` 中完成核验。
- 表2-1已于 PDF p.46 逐项核验；表2-2已于 PDF p.62 核验为：Memory Read/Memory Read Lock/IO Read/IO Write/Configuration Read/Configuration Write 都是 Non-Posted，Memory Write 和 Message 是 Posted。
- 表2-3已于 PDF p.63 核验缩写：MRd、MRdLk、MWr、IORd、IOWr、CfgRd0/CfgRd1、CfgWr0/CfgWr1、Msg、MsgD、Cpl、CplD、CplLk、CplDLk。
