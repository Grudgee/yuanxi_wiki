---
name: knowledge_pcie_chapter_4_section_4_6_1_id_routing
description: PCIe 第4章§4.6.1 的 BDF/ID 路由规则与端口检查。
---

# 知识点摘要

- ID 路由以 Bus、Device、Function 组成的 BDF 指向拓扑中的逻辑 Function。
- PCIe 中配置包使用 ID 路由，完成包和部分 Message 也可使用；EP 做一次检查，Switch 每个端口做两次检查。

# 关键细节

- Bus 号 8 bit，系统最多 256 条总线（含 Switch 内部总线）；Device 号 5 bit，每总线最多 32 个设备；Function 号 3 bit，每设备最多 8 个 Function。
- 外部 PCIe 链路点到点；Switch/RC 下行端口强制外部设备 Device=0。启用 ARI 时，外部 Device 可不使用 Device 号。
- ID 路由 TLP 的 Type 指示使用 ID；Header 中 Bus/Device/Function 字段用于 routing check。3DW Header 可用于 ID 路由；4DW ID 路由仅可能出现在 Message 中。
- EP 将收到的 ID 与自身 BDF 比较；收到 Type 0 配置写时，会从 Header byte8–9 捕获 Bus/Device 并保存，之后可作为 Requester ID，使完成包返回时能被路由。
- Switch 端口是独立的 Type 1 Bridge 配置空间，每端口对入站 TLP 进行两次检查，因此不能只按整个 Switch 一个 BDF 推理。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 版本/日期：PDF 元数据标注 2026年6月1日
- 位置：第4章§4.6.1–§4.6.1.4，PDF p.157–p.160；书内 p.37–p.40
- 依据：`pdftotext -layout` 提取上述页，核对图4-15/4-16/4-17及 Bus/Device/Function 上限说明。

# 适用条件与例外

- Device=0 的外部 EP 规则受 ARI 例外影响；ARI 细节不在本周期展开。
- 端口是否接收/转发还受 TLP Type 及端口拓扑方向检查约束。

# 关联章节

- §3.6–§3.13 配置路由与枚举；§4.5 TLP 路由基础；§4.6.2 地址路由；第5章 TLP Header

# 待核验问题

- 无。图4-17（PDF p.160）核验结果：每个 Switch/Bridge 端口先将目标 ID 与自身 BDF 比较，命中则消耗；未命中再比较目标 Bus 是否落入该端口 Secondary–Subordinate 范围，命中则向下转发。上行端口两项均未命中时按 UR；下行端口对非本端口目标继续按各自下游范围检查，只有拥有目标范围的端口转发，其余忽略。
