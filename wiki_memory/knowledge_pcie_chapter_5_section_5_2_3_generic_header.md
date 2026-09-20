---
name: knowledge_pcie_chapter_5_section_5_2_3_generic_header
description: PCIe 第5章§5.2.3通用TLP Header字段、Fmt/Type、TC、属性和长度。
---

# 知识点摘要
- Fmt[2:0] 同时编码 Header 大小和是否有数据：3DW/4DW、无数据/有数据；低于4GB地址必须使用3DW。
- Type[4:0] 指定事务类型；TC[2:0] 指定流量类别；Attr 表达排序、No Snoop 等属性；Length[9:0] 以 DW 表示数据长度，0编码代表1024 DW。
- Header 还包括 Digest、TH、TD、EP、AT、First/Last DW Byte Enable 等字段。

# 关键细节
- Fmt=000/001 表示无数据的3DW/4DW；010/011表示有数据的3DW/4DW；100为TLP Prefix。
- TC0为默认，TC1–TC7用于差异化服务；AT 支持未转换、转换请求、已转换三种地址状态。
- 保留字段必须置0；4DW Header 用于低于4GB地址时行为未定义。

# 原文引用
- 文档：PCI Express Technology 3.0 中文版；位置：§5.2.3，PDF p.178–181，图5-3、表5-2。

# 适用条件与例外
- 具体事务合法的 Fmt/Type 组合需按表5-3和具体 TLP 类型判断。

# 关联章节
- §5.2.4 Header 详细说明；§5.2.5 具体 TLP 格式

# 待核验问题
- 无。
