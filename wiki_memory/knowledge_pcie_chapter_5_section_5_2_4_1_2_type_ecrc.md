---
name: knowledge_pcie_chapter_5_section_5_2_4_1_2_type_ecrc
description: PCIe 第5章§5.2.4.1–§5.2.4.2 TLP Format/Type编码与Digest/ECRC规则。
---

# 知识点摘要
- 表5-3给出 MRd/MWr/IO/Cfg/Message/Completion/AtomicOp/Prefix 的 Fmt 与 Type 组合。
- Digest/TD 位表示 TLP 是否带 ECRC；ECRC 是端到端校验，需设备支持并由软件启用 AER 相关能力。

# 关键细节
- MRd/MRdLk 为无数据 3DW/4DW；MWr 为有数据 3DW/4DW；IO/Cfg 使用3DW；Message 固定4DW；Cpl/CplD 使用3DW；AtomicOp 可用3DW或4DW有数据格式。
- ECRC 在跨 Fabric 转发时保持不变；LCRC则在每条链路出口重新计算。Switch 可检查通过自身的 ECRC，但错误不改变其转发。
- 配置事务穿越 Bridge 时 Type 1 可能转换为 Type 0，Type bit0 可合法改变；EP/Error Poisoned bit 也可能因错误转发改变。

# 原文引用
- 文档：PCI Express Technology 3.0 中文版；位置：§5.2.4.1–§5.2.4.2，PDF p.182–183，表5-3。

# 适用条件与例外
- ECRC 是可选端到端能力，不能与链路级 LCRC 混同。

# 关联章节
- §4.6 路由；第8章错误处理；第10章 Ack/Nak

# 待核验问题
- 无。
