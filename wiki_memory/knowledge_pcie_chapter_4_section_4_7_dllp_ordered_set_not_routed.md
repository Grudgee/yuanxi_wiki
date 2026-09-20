---
name: knowledge_pcie_chapter_4_section_4_7_dllp_ordered_set_not_routed
description: PCIe 第4章§4.7 对 DLLP 与 Ordered Set 链路本地性的说明。
---

# 知识点摘要

- DLLP 和 Ordered Set 不会由 Switch/RC 入口端口路由到出口端口；它们只在相邻端口间经过物理层链路传输。
- DLLP 在对端物理层后到达数据链路层并被处理；Ordered Set 在对端物理层即被处理，两者都不会上行到事务层。
- 只有起源于事务层、结束于事务层的 TLP 会被 Switch 和 RC 路由。

# 关键细节

- DLLP 路径：源端口数据链路层→物理层→链路→对端物理层→对端数据链路层；到达数据链路层即消耗。
- Ordered Set 路径：源端口物理层→链路→对端物理层；在物理层处理和消耗。
- 因为二者没有跨端口继续向事务层传播的路径，所以不存在像 TLP 那样的拓扑路由。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 版本/日期：PDF 元数据标注 2026年6月1日
- 位置：第4章§4.7，PDF p.168–p.170；书内 p.48–p.50
- 依据：`pdftotext -layout` 提取上述页，核对 §4.7 开头及 DLLP/Ordered Set 处理段落；PDF p.171 起进入第5章。

# 适用条件与例外

- 本结论针对 PCIe 分层数据路径；不应把 DLLP/Ordered Set 当作跨交换结构的端到端事务。
- §4.7 后 PDF p.171 起为第5章 TLP Elements，不属于本周期范围。

# 关联章节

- §2.2.3 DLLP；§2.2.4.5 Ordered Sets；§4.5 TLP 路由基础；第5章 TLP 元素

# 待核验问题

- 无；若继续学习，下一入口为第5章 TLP Elements 标题页/正文。
