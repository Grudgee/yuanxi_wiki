---
name: knowledge_pcie_chapter_6_flow_control_concepts_initialization
description: PCIe 第6章§6.1–§6.4流量控制概念、Buffer/Credit、初始通告和初始化流程。
---

# 知识点摘要

Flow Control 通过接收端公布的 Credit 约束发送端，防止 TLP 超过接收 Buffer 容量。Credit 按 Virtual Channel、Posted/Non-Posted/Completion 和 Header/Data 分类；链路初始化期间用 FC_Init1/FC_Init2 建立初始值，之后用 UpdateFC DLLP 更新释放出的空间。

# §6.1 流量控制概念

- PCIe 没有 PCI 的共享总线等待态、Retry 和 Disconnect；发送端必须在对端有足够 Credit 时才发送 TLP。
- 流量控制是逐链路、逐 VC 的本地机制，不是端到端吞吐保证。上游发送端只依据直接相邻接收端通告的 Buffer 状态。
- Credit 分离 Header 和 Data，使无数据请求不会错误消耗 Data Buffer，而带大 Payload 的 TLP 会同时占用 Header/Data Credit。

# §6.2 Buffer 与 Credit

- 每个端口至少有 VC0，其他 VC 可选；每个 VC 维护 Posted Request、Non-Posted Request 和 Completion 三类接收 Buffer。
- Header Credit 计数包头可用槽位；Data Credit 计数数据 DW 可用空间。发送一个 TLP 时相应 Credit 减少，接收端消费并释放空间后更新。
- Credit 的单位和字段宽度由 DLLP/规范定义；发送端不能把一个 Credit 类别挪给另一类别，也不能使用尚未通告的 VC。
- Infinite Credit 表示接收端对该类别无需有限 Credit 计数，但只适用于协议允许的缓冲类别，不能任意宣称无限。

# §6.3 初始流量控制通告

- 链路两端在正常发送 TLP 前通告各 VC 的初始 Header/Data Credit。通告包含 Posted、Non-Posted、Completion 三类的可用额度。
- Initial Advertisement 有最小/最大合法范围；发送端据此初始化本地可发送计数器，接收端据此初始化已分配/已消耗状态。
- FC_Init 期间尚未完成的 VC 不能按正常事务发送；错误的 Credit 值、重复初始化或不支持的 Infinite Credit 会构成协议违例。

# §6.4 流量控制初始化

1. 链路达到可用状态后，端口交换 FC_Init1，公布各 VC 的接收能力和 Credit 类型。
2. 双方确认收到并解析 FC_Init1 后，交换 FC_Init2，完成初始 Credit 状态建立。
3. FC_Init1/FC_Init2 的发送速率受协议约束；初始化完成后才允许正常 TLP 流量使用这些 Credit。
4. 若一端未完成初始化、发送非法字段或在规定时序外使用 Credit，链路应报告协议违例并阻止不安全的 TLP 发送。

## 状态与实现注意

- 接收端必须预留足够 Buffer，避免因延迟通告而让发送端超发；发送端必须以保守的可用 Credit 为准。
- VC 映射和流控状态相互关联：同一 TC 在不同链路可能映射到不同 VC，因此每条链路都要独立维护 Credit。
- 流控不能修复物理层 bit error；物理错误由 LCRC、Ack/Nak 和重传机制处理，流控只保证 Buffer 容量安全。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 位置：第6章§6.1–§6.4，PDF p.222–237；本周期源文本约 15,202 字符。
- 依据：Flow Control Concept、Buffer/Credit、Initial Advertisement、Infinite Credit 和 FC_Init1/FC_Init2 流程。

# 适用条件与例外

- Credit 是链路本地资源，不等于应用层可获得的带宽承诺。
- Infinite Credit、Credit 最大/最小值和初始化 DLLP 的精确字段应结合第9章 DLLP 和后续流控格式继续核对。

# 关联章节

- §6.5 流控机制；§6.6 流控示例；§6.7 流控更新；第7章 QoS；第9章 DLLP。

# 待核验问题

- 无。
