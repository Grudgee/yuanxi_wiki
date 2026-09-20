---
name: knowledge_pcie_chapter_4_section_4_2_bars
description: PCIe 第4章§4.2 BAR 地址空间请求、评估、编程和 Resizable BAR。
---

# 知识点摘要

- 设备通过配置 Header 中的 BAR 向系统软件请求 IO、NP-MMIO 或 P-MMIO 地址范围；软件负责评估需求、分配基址并写回 BAR。
- Type 0 Header（非 Bridge，如 EP）有 6 个 32-bit BAR；Type 1 Header（Bridge、Switch/RC 端口）有 2 个 BAR。
- 评估流程是向可写位写全 1、读取回值判断最低可写位和类型、分配并写入对齐的基地址；启用 Command Register 的相应译码后范围才响应事务。

# 关键细节

- 32-bit NP-MMIO 示例：最低可写位 bit12 表示 4KB，分配基址 F900_0000h 后响应 F900_0000h–F900_0FFFh。
- 64-bit P-MMIO 使用连续 BAR pair；低位 BAR 表示类型/大小，高位 BAR 提供地址高 32 bit，软件应把二者作为一个请求评估。
- IO 示例：BAR3 请求 256B，基址 4000h，启用 IO译码后响应 4000h–40FFh。
- 所有 BAR 必须按 BAR0、BAR1…顺序评估，即使中间 BAR 未实现；未实现 BAR 固定为 0。64-bit BAR 会占用连续的下一个 BAR，不能把高位 BAR 当独立请求。
- Resizable BAR 能力允许 Function 公布多个可用大小，软件按系统资源选择，例如 2GB 理想大小不足时可降为 1GB、512MB 或 256MB。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 版本/日期：PDF 元数据标注 2026年6月1日
- 位置：第4章 §4.2–§4.2.6，PDF p.125–p.135（书内 p.5–p.15）
- 依据：`pdftotext -layout` 提取 PDF p.125–p.136，核对 BAR0/1/2/3 示例、顺序评估和 Resizable BAR。

# 适用条件与例外

- BAR 低位固定位由设备设计声明类型/对齐需求，高位可写位由软件分配；具体位编码应结合对应 Header 定义。
- 设备请求的范围可能因平台资源不足而无法满足；Resizable BAR 不是所有 Function 都实现。

# 关联章节

- §4.1 地址空间；§4.3 Bridge Base/Limit；第3章 Type 0/Type 1 Header

# 待核验问题

- 已核验。书中 PCIe 2.1 Resizable BAR 说明位于 PDF p.911–913（第20.5.2），引用的能力/控制寄存器图20-20至图20-22。能力寄存器 bits[4:23] 分别表示 1MB、2MB、4MB … 512GB；控制寄存器 BAR Index 取0–5，Number of Resizable BARs 只在第0组定义，BAR Size 编码 0=1MB、1=2MB、2=4MB，依次到 19=512GB。修改大小前应清除 Memory Enable；改变大小会丢失 BAR 内容，需重新编程。该书节选未给出完整 PCIe Capability 链中的绝对扩展配置偏移，故“偏移”本身仍以图示和能力定位为准，不能臆造固定地址。
