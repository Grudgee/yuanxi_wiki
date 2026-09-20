---
name: knowledge_pcie_chapter_4_section_4_4_address_routing_registers
description: PCIe 第4章§4.4 对 BAR 与 Bridge 地址路由寄存器关系的理解检查。
---

# 知识点摘要

- §4.4 通过扩展示例拓扑检查 BAR 与 Base/Limit 的职责：EP 的 BAR 定义自身范围，Bridge 的 Base/Limit 只汇总下游范围。
- Type 1 Header 中 Bridge 自身也有 2 个 BAR，但这些 BAR 不应计入该 Bridge 的下游 Base/Limit 范围。
- 配置正确的目标是让请求沿点到点链路经过每个相关路由元件，最终到达声明其 BAR 地址范围的 Function。

# 关键细节

- 示例在 Switch Port A 下增加另一个 EP，用来同时检查多个下游分支的范围汇总。
- Base/Limit 是“下游可达地址窗口”，不是设备自身资源表；软件需要把不同下游 Function 的地址范围合并后设置相应窗口，并考虑寄存器粒度造成的空洞。
- 本节是对 §4.2–§4.3 的 sanity check，下一节 §4.5 才正式进入 TLP 路由基础和入口端口的接收/转发/拒绝决策。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 版本/日期：PDF 元数据标注 2026年6月1日
- 位置：第4章 §4.4，PDF p.147–p.148（书内 p.27–p.28 起）
- 依据：`pdftotext -layout` 提取 PDF p.147–p.149，核对图4-11及其对 Type 1 BAR、下游 Base/Limit 范围的说明。

# 适用条件与例外

- 图示是理解检查示例，不是覆盖全部桥接组合的算法规范；复杂拓扑仍需逐个 Bridge 核对。
- 事务是否真正被转发还取决于 Command Register 译码使能及后续 TLP 路由规则。

# 关联章节

- §4.2 BAR；§4.3 Base/Limit；§4.5 TLP 路由基础

# 待核验问题

- 无。§4.5–§4.6 已核验三种 TLP 路由、端口入口检查、窗口命中和 UR/Malformed TLP 条件；详见对应章节记忆。
