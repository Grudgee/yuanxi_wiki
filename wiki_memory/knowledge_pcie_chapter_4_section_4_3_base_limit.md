---
name: knowledge_pcie_chapter_4_section_4_3_base_limit
description: PCIe 第4章§4.3 Bridge Base/Limit 寄存器与下游地址范围路由。
---

# 知识点摘要

- Function 编程 BAR 后只知道自己的范围；上方每个 Bridge 还必须用 Base/Limit 描述其下游范围，才能把请求从 Primary 转发到 Secondary。
- Type 1 Header 有三类范围寄存器：P-MMIO、NP-MMIO、IO；它们只描述 Bridge 下方 Function 的空间，不包括 Bridge 自身 BAR。
- P-MMIO Base/Limit 可用低 32 bit 配合 Upper 32 bit 表示 64-bit 范围；NP-MMIO 只支持 32 bit；IO 可用低 16 bit，少见的 32-bit IO 译码需 Upper 寄存器。

# 关键细节

- P-MMIO 示例下游范围 2_4000_0000h–2_43FF_FFFFh：Base=4001h、Limit=43F1h，Upper Base/Limit 均为 00000002h；低 20 bit 被隐去，窗口按 1MB 对齐。
- NP-MMIO 示例 EP 实际仅 4KB（F900_0000h–F900_0FFFh），Bridge Base/Limit 却只能表示 1MB 窗口（F900_0000h–F90F_FFFFh），未使用的 1020KB 不能再分配给其他 EP。
- IO 示例 EP 实际 256B（4000h–40FFh），Bridge 表示 4KB（4000h–4FFFh）；低 12 bit 被隐去。x86 仅用 16-bit IO 时，Bridge 可区分的高 nibble 范围有限。
- 未使用的范围不能把 Base 和 Limit 都写 0，因为隐去低位后仍可能形成有效窗口；应使 Base 写入值大于 Limit 写入值使范围无效。原书该处存在 errata：文字曾把方向写反。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 版本/日期：PDF 元数据标注 2026年6月1日
- 位置：第4章 §4.3–§4.3.5，PDF p.136–p.146（书内 p.16–p.26）
- 依据：`pdftotext -layout` 提取 PDF p.136–p.147，核对 P/NP-MMIO、IO 字段示例、对齐粒度和无效化配置。

# 适用条件与例外

- Base/Limit 的可表示范围比下游 BAR 实际范围粗，会造成地址窗口浪费；这是路由正确性约束，不代表这些地址可安全使用。
- 64-bit P-MMIO、32-bit IO Upper 寄存器只有在 Bridge 支持相应译码时启用。

# 关联章节

- §4.2 BAR；§4.4 地址路由寄存器检查；§4.5 TLP 路由基础

# 待核验问题

- 无。§4.5–§4.6.3 已完成；Base/Limit 窗口与地址、ID、隐式路由的组合规则分别记录在本章 §4.5 及 §4.6 记忆中。
