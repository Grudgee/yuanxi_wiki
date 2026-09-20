---
name: knowledge_pcie_chapter_4_section_4_6_2_address_routing
description: PCIe 第4章§4.6.2 的 Memory/IO 地址路由与 EP、Switch 检查。
---

# 知识点摘要

- 地址路由用 TLP Header 的 Address 字段匹配 Memory 或 IO 地址空间。
- 小于4GB的 Memory 请求使用3DW Header；大于4GB的 Memory 请求使用4DW Header；IO 地址为32 bit，且主要为传统兼容用途。
- EP 以 BAR 接受或拒绝；Switch 先检查自身 BAR，再检查下游 Base/Limit 窗口，最后决定向下转发、向上转发或 UR。

# 关键细节

- 32-bit Memory 与 IO 请求的目标寄存器位于4GB边界内；64-bit Memory 请求目标可位于4GB以上。
- EP 将 Address 与自身每个 BAR 的范围比较，匹配则接收；EP 无转发能力。
- Switch 端口自身是 Bridge：先以端口 Type 1 Header 的两个 BAR 检查是否为自身目标；不匹配时，IO 检查 IO Base/Limit，Memory 检查不可预取及可预取 Base/Limit。
- 下行方向：自身 BAR 命中则消耗；下游窗口命中则转发至次级接口；都不命中则主接口按 UR 处理。上行方向则按次级接口 BAR/窗口和上行路径判定；窗口内但不属于本端口的目标通常向上转发，具体例外见原文步骤。
- PCIe 2.1 引入 Multicast Capability：指定地址范围内的 TLP 按多播规则接受/转发，即使该范围不在 BAR 或 Base/Limit 中。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 版本/日期：PDF 元数据标注 2026年6月1日
- 位置：第4章§4.6.2–§4.6.2.4，PDF p.161–p.165；书内 p.41–p.45
- 依据：`pdftotext -layout` 提取上述页，核对图4-18至图4-21及下行/上行路由步骤。

# 适用条件与例外

- IO 请求仅在支持传统旧功能时使能；地址宽度必须与 Header Format/Type 一致。
- 多播是 PCIe 2.1 新增能力，不能用普通 BAR/Base/Limit 命中规则替代。

# 关联章节

- §4.1 地址空间；§4.2 BAR；§4.3 Base/Limit；§4.6.1 ID路由；§4.6.3 隐式路由

# 待核验问题

- 无。图4-21及§4.6.2.3（PDF p.164–165）核验结果：下行入口先检查本端口 BAR，命中则消耗；否则命中 IO/NP-MMIO/P-MMIO Base/Limit 就向 Secondary 转发；均未命中且无其他下游桥声明拥有该地址时由 Primary 入口报 UR。上行入口命中自身 BAR 则消耗；命中窗口通常报 UR，因为目标在本端口下游不应从下游方向返回，但若该端口是 Switch 上行端口，窗口内事务可作为 P2P 转发到另一下行端口；两者均未命中则向上游转发。
