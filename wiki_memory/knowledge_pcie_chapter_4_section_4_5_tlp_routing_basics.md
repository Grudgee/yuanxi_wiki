---
name: knowledge_pcie_chapter_4_section_4_5_tlp_routing_basics
description: PCIe 第4章§4.5 的 TLP 路由基础、路由元件、路由方法及事务类别。
---

# 知识点摘要

- PCIe 是点到点链路拓扑；TLP 到达入口端口后，端口先校验，再选择本地接收、转发到出口端口或拒绝。
- Ordered Set 与 DLLP 是链路本地流量；只有 TLP 可能跨越多条链路并由 Switch/RC 路由。
- TLP 路由方法有地址路由、ID(Bus-Device-Function)路由和隐式路由；选择由 TLP 类型决定，Message 可支持多种方法。
- PCIe 使用拆分事务：请求与完成是独立 TLP；Memory Write 和 Message 是 Posted，其余常见读/配置/IO/Atomic 请求为 Non-Posted。

# 关键细节

- RC、Switch 等多端口设备是 Routing Element，可接收发往自身资源的 TLP，也可在入口/出口端口间转发；Switch 必须支持 Peer-to-Peer，RC 的 P2P 支持可选。EP 只有一条链路，只接受或拒绝，不转发。
- 隐式路由不依赖地址或 ID，而依赖 Header 中的路由代码，用于把 Message 送往 RC、所有下行设备等已知拓扑位置，替代 PCI 的部分边带中断/错误/电源管理信号。
- Posted 请求不要求完成包，且不应收到完成包；Non-Posted 请求要求 Completer 返回状态完成包。Memory Read 可能拆成多个 Completion TLP；成功读完成带数据，失败完成通常不带数据。
- 每个 TLP Header 为 3DW 或 4DW；Format/Type 字段定义 Header 内容并影响路由方式。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 版本/日期：PDF 元数据标注 2026年6月1日
- 位置：第4章§4.5–§4.5.6，PDF p.148–p.156；书内 p.28–p.36
- 依据：`pdftotext -layout` 提取上述页，核对 §4.5.1–§4.5.6、表4-7/4-8及图4-12/4-13/4-14。

# 适用条件与例外

- 本节描述 PCIe 3.0 拓扑与 TLP 基础；具体 Type 与路由映射以表4-7及后续 ID/地址/隐式路由小节为准。
- RC 的 Peer-to-Peer 支持是可选能力；不能把所有 RC 默认视为可进行 P2P 转发。

# 关联章节

- §4.2 BAR；§4.3 Base/Limit；§4.6.1 ID路由；§4.6.2 地址路由；§4.6.3 隐式路由；第5章 TLP 元素

# 待核验问题

- 无。表4-7（PDF p.151）逐项核验为：Memory Read [Lock]、Memory Write、AtomicOp 使用地址路由；IO Read/Write 使用地址路由；Configuration Read/Write 使用 ID 路由；Message/Message with Data 可使用地址、ID 或隐式路由；Completion/Completion with Data 使用 ID 路由。
