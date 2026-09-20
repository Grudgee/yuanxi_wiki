---
name: knowledge_pcie_chapter_1_background
description: PCI Express Technology 3.0 中文版第1章背景，涵盖 PCI/PCI-X 基础、事务模型、配置空间与并行总线向 PCIe 串行互连迁移的动机。
---

# 知识点摘要

- PCIe 的软件模型延续 PCI：配置空间、资源分配和驱动交互保持兼容，使旧 PCI 软件迁移到 PCIe 时改动较少；物理层则从共享并行总线转向高速串行互连。
- PCI 设备采用共享总线；设备可含最多 8 个 Function（编号 0–7），Function 可作为事务 Target，多数也可作为 Initiator/Bus Master。REQ#/GNT# 用于总线仲裁。
- 典型 PCI 事务由 FRAME#、IRDY#、TRDY#、DEVSEL#、STOP# 等握手控制；地址与数据复用 AD 总线，读事务需要 turn-around 周期，复用和等待态使流水化受限。
- PCI 的 PIO 由 CPU 搬运数据，开销大；DMA 由 DMA Engine/Bus Master 负责地址序列和协议，CPU 只需设置起始地址与字节数；Peer-to-peer 可绕过系统内存，但常受设备数据格式不一致限制。
- 共享并行总线的瓶颈来自反射波信号时序、信号/时钟歪斜、渡越时间、引脚与负载数量。33 MHz PCI 典型只能可靠承受约 10–12 个电气负载（插槽约计两个负载），66 MHz/64 bit 虽可达约 533 MB/s，却进一步压缩负载能力。
- PCI 的 Retry 用于 Target 尚未开始传输且需要较长延迟的情形；Disconnect 用于已传输至少一个双字但暂时无法继续。两者都释放共享总线，Master 至少等待 2 个时钟后重新仲裁并重试/续传。
- PCI 地址空间分为 Memory、I/O、Configuration 三类。传统 PCI Function 最多 256 B 配置空间，其中前 64 B 是 Header；Type 0 表示非 Bridge，Type 1 表示 Bridge。传统 x86 通过 CF8h–CFBh 配置地址端口和 CFCh–CFFh 配置数据端口间接访问。
- PCI-X 保持 PCI 软件/硬件和连接器兼容，通过 PLL 相移、输入寄存、属性阶段等方法提高时序和效率；支持 split transaction、MSI、No Snoop、Relaxed Ordering。PCI-X 2.0 使用源同步 strobe，支持 DDR/QDR，但高引脚数、点到点要求和成本限制了普及。
- PCIe 的核心动机是突破并行共享总线的实际带宽上限和负载/引脚成本约束，在物理层采用串行点到点模型，同时在软件层保留 PCI 兼容性。

# 关键细节

## PCI 总线周期与握手

- 空闲后，仲裁器通过 GNT# 选出下一个 Master；Master 在时钟沿驱动 FRAME# 和地址/命令。
- 读事务中，IRDY# 表示 Initiator 已准备好，Target 用 DEVSEL# 响应并用 TRDY# 表示数据准备好；IRDY# 与 TRDY# 同时有效时完成一个 data phase。
- FRAME# 仍有效表示还有后续 data phase；FRAME# 无效表示当前为最后一个 data phase。Initiator 或 Target 均可插入等待态；已开始传输后规范允许最多约 8 个连续等待周期，Target 从事务发起后迟迟不能响应时可用 Retry（最多约 16 个时钟等待后转用 STOP#）。
- AD 总线同时承载地址/数据；读事务方向切换需 turn-around 周期，避免两个 buffer 同时驱动总线。复用减少引脚，但阻碍当前数据与下一地址重叠流水化。

## PCI 体系与事务

- 传统平台由 North Bridge 连接处理器、系统内存、AGP 和中央 PCI 总线，South Bridge 连接 ISA 等遗留外设并提供复位、参考时钟和错误报告。
- PIO：CPU 从设备读入内部寄存器再写入内存，或反向搬运；每次传输约需两个总线周期且占用 CPU，但仍用于软件与设备交互。
- DMA：CPU 配置 DMA Engine 的 starting address 与 byte count，Engine/设备 Bus Master 自主执行块传输；完成后设备可产生中断。
- Peer-to-peer：一个 Bus Master 直接访问另一 PCI 设备，不占用系统内存路径；若两设备数据格式不同，仍需经内存和 CPU 转换。
- 仲裁算法未由 PCI 规范规定，但必须公平；仲裁可在当前事务结束前“隐藏”进行，降低下一个所有者的额外延迟。

## 错误、中断与地址映射

- PCI 设备可对地址/数据做偶校验。数据校验错误通过 PERR# 报告，某些读事务可由软件重发恢复；地址校验错误通过 SERR# 报告，错误地址和误命中的 Target 不可确定，传统系统可能停止运行。
- 传统中断使用 INTA#–INTD# 四条边带信号之一；后续 APIC 将中断作为消息发往 CPU。PCI-X/PCIe 采用 MSI 的消息写方式：向预定义地址写入中断向量，避免共享引脚和设备扫描。
- Memory 空间可支持 32/64 位寻址；I/O 空间设备可支持 32 位，但 x86 常把 I/O 限制在 64 KB。Configuration 空间在传统 x86 上不能直接访问。
- 配置地址由 Bus（最多 256）、Device（每 Bus 最多 32）和 Function（每 Device 最多 8）以及 64 个 dword 内偏移组成；理论最大配置空间为 256 B × 8 × 32 × 256 = 16 MiB。
- 每个 Function 前 64 B 为 Header，其余 192 B 可放置可选能力；固件先枚举/分配资源，操作系统加载后可能再次配置。

## PCI-X 改进与限制

- PCI-X 输入在 Target 引脚寄存，配合 PLL 相移改善 setup budget；PCI-X 事务属性阶段提前携带总数据量和 Requester B:D:F，因此第一个 data phase 后不允许插入等待态，常以 128 B block 突发传输。
- Split transaction 中 Completer 保存地址、类型、总长度和 Requester ID，先返回 Split Response 释放总线；数据准备好后再仲裁并发送 Split Completion，Requester 无需轮询，提升总线利用率（文中约 PCI-X 85% 对 PCI 50%–60%）。
- No Snoop 跳过已知不可缓存区域的 cache snoop；Relaxed Ordering 允许无依赖事务在 Bridge buffer 中提前调度，避免强排序造成的性能损失。
- PCI-X 2.0 源同步模型用随数据同路径的 strobe 采样，降低公共时钟歪斜和长走线渡越时间影响，支持 DDR/QDR；但高速设计需要点到点、更多引脚和更高成本，并以 ECC 提升可靠性。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 版本/日期：2026 年 6 月 1 日构建；内容标注 PCI Express Technology 3.0 中文版
- 位置：PDF p.11–p.40；第 1 章“背景（Background）”、§1.1–§1.5.4.2（书内页码约 p.1–p.29）
- 依据：本周期使用 `pdftotext -layout` 提取上述页码；章节末明确说明并行总线走到终点，新模型“就是我们所知道的 PCI Express（PCIe）”。

# 适用条件与例外

- 本记忆描述书中 PCI/PCI-X 背景模型，不等同于 PCIe 3.0 规范的完整字段和实现要求；PCIe 分层、配置和事务细节从第 2 章起展开。
- “约 10–12 个电气负载”“约 85%/50%–60% 利用率”等是本书背景说明中的典型值，不能替代具体平台 SI/时序分析。
- 传统 PCI 配置端口 CF8h/CFCh 是 x86 兼容访问模型；PCIe 还支持将扩展配置空间映射到内存地址空间，后续章节再核验其完整机制。

# 关联章节

- 第 2 章 PCIe 体系结构概述：分层端口设计与各层职责
- 第 3 章 PCIe 配置概述：Bus/Device/Function、配置空间与枚举
- 第 4 章 地址空间与事务路由：BAR、路由和 TLP
- 第 5–6 章：TLP 元素与流量控制

# 待核验问题

- 无。表 1-1 已于 PDF p.14 通过页面图像逐项复核：PCI 33 MHz 为 133–266 MB/s、4–5 插槽；PCI 66 MHz 为 266–533 MB/s、1–2 插槽；PCI-X 1.0 66 MHz 为 266–533 MB/s、4 插槽；PCI-X 1.0 133 MHz 为 533–1066 MB/s、1–2 插槽；PCI-X 2.0 DDR 133 MHz 为 1066–2132 MB/s、1 个点到点插槽；PCI-X 2.0 QDR 133 MHz 为 2132–4262 MB/s、1 个点到点插槽。
- PCIe Gen1–Gen3 速率、编码、分层和基本事务已在第2章相关记忆中完成核验。
