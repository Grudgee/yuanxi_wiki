---
name: knowledge_pcie_chapter_5_section_5_2_4_3_5_byte_enable_descriptor
description: PCIe 第5章§5.2.4.3–§5.2.4.5字节使能、事务描述符和数据荷载规则。
---

# 知识点摘要
- First DW BE 与 Last DW BE 用于描述非 DW 对齐首尾的有效字节；单 DW 时 Last DW BE 必须为0。
- 事务描述符由 Requester ID+BDF、Tag、TC 和 Attr 组成，用于识别拆分事务及其排序/服务属性。
- 带数据 TLP 的 Length 只表示数据荷载长度；多 Completion 的中间包受 RCB 64/128B自然对齐边界约束。

# 关键细节
- Length>1DW 时 First DW BE 至少一位有效；Length≥3DW 时首尾 BE 必须为连续有效位。1–2DW 才允许首部不连续 BE。
- 读请求 Length=1DW 且 BE全0时，Completer 可返回未定义数据；该模式可作为刷新机制，等待此前 Posted Write 排出。
- Transaction ID=Requester ID+Tag；TC 沿途不变并映射到每条链路的 VC；ID-based Ordering、Relaxed Ordering、No Snoop 属性随请求发送。
- 数据荷载首字节位于 Header 后最低地址；Message 无数据时 Length 保留，MsgD 才使用 Length。

# 原文引用
- 文档：PCI Express Technology 3.0 中文版；位置：§5.2.4.3–§5.2.4.5，PDF p.184–187，图5-4、图5-5。

# 适用条件与例外
- RCB由配置控制，具体边界可为64B或128B；非首尾 Completion 的分割规则只适用于单个 Memory 请求返回多个 TLP 的情形。

# 关联章节
- §5.2.5 具体 TLP；第6章流量控制；第7章QoS；第8章排序

# 待核验问题
- 无。
