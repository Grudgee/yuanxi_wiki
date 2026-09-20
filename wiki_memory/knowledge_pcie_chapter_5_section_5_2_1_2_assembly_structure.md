---
name: knowledge_pcie_chapter_5_section_5_2_1_2_assembly_structure
description: PCIe 第5章§5.2.1–§5.2.2 TLP组包拆包流程与TLP基本结构。
---

# 知识点摘要
- Device Core 提供目标地址/ID、Requester ID/Tag、事务类型、长度、数据、TC 和属性；事务层组装 Header、数据和可选 ECRC。
- 数据链路层分配序列号、计算 LCRC、保存 Replay Buffer；物理层条带化、扰码、编码和串行化。
- 接收端执行逆向处理，链路层校验 LCRC/序列号并返回 Ack 或 Nak，事务层解码后交给 Device Core。

# 关键细节
- TLP 由 Header（3或4 DW）、可选 Data（1–1024 DW）和可选 Digest/ECRC（1 DW）组成。
- Gen1/2 使用 STP/END；Gen3 使用 STP token，不添加 END。
- 数据链路层校验成功后剥除 LCRC/序列号并上送；ECRC 在最终接收者可选检查。

# 原文引用
- 文档：PCI Express Technology 3.0 中文版；位置：§5.2.1–§5.2.2，PDF p.174–177。

# 适用条件与例外
- TLP 的具体 Type/Format 与字段语义由后续通用 Header 和具体 TLP 小节定义。

# 关联章节
- §5.2.3 通用 Header；§5.2.4 Header 字段；第10章 Ack/Nak

# 待核验问题
- 无。
