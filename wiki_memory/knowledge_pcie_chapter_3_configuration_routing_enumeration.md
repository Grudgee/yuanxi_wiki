---
name: knowledge_pcie_chapter_3_configuration_routing_enumeration
description: PCIe 第3章配置访问、Type 0/1路由和单/多Root Complex枚举的原文核验记录。
---

# 知识点摘要

- 只有 Root Complex 可以发起配置请求；配置请求以目标 BDF 路由且只向拓扑下游传播，不支持 Peer-to-Peer 配置访问。
- 传统机制使用 CF8h 地址端口与 CFCh 数据端口两步间接访问；ECAM 把每个 Function 的 4 KB 配置空间映射到一个 256 MB 对齐窗口，实现单步、线程间不易被地址寄存器覆盖的内存访问。
- Bridge 比较目标 Bus 与 Secondary/Subordinate 范围：等于 Secondary 时发送 Type 0；大于 Secondary 且不超过 Subordinate 时发送/继续转发 Type 1；抵达目标 Bus 的上游 Bridge 时转成 Type 0。
- 枚举读取 Vendor ID 探测 Function，读取 Header Type 区分 Endpoint/Bridge，并以深度优先方式临时把 Subordinate 置 255，完成分支扫描后回填实际最大总线号。

# 关键细节

## 传统配置访问

- CF8h 地址端口布局：bit31 Enable；[23:16] Bus；[15:11] Device；[10:8] Function；[7:2] 兼容配置空间 DW；[1:0] 固定为0；[30:24] 保留为0。
- CFCh–CFFh 数据端口接受 1/2/4-byte IO 访问。一次配置访问需要先写完整 32-bit CF8h，再访问 CFCh；多线程必须避免地址端口在第二步前被覆盖。
- 传统示例读取 Bus4/Device0/Function0/Vendor ID：RC 先发 Type 1，沿包含 Bus4 的 Bridge 范围下传；最后一级 Bridge 把请求转换为 Type 0，目标 Function 用 First DW Byte Enable 返回 Vendor ID。

## ECAM

- ECAM 窗口总容量 256 MB，每 Function 占 4 KB。地址字段为：[63:28] 256 MB 对齐基址，[27:20] Bus，[19:15] Device，[14:12] Function，[11:2] DW，[1:0] Byte Offset。
- 书中示例基址 E0000000h，地址 E0400000h 选择 Bus4/Device0/Function0/DW0/Byte0。
- 软件应避免跨 DW 边界访问以及依赖 RC 未声明支持的总线锁定操作。

## Type 0/Type 1

- Type 0 的 Type 字段为 00100b，用于当前 Secondary Bus 上的目标；设备继续检查 Device、Function、Register Number 与 First DW Byte Enable。外部链路 Endpoint 总是 Device 0。
- Type 1 的 Type 字段为 00101b，仅 Bridge 处理；Endpoint 忽略。Bridge 若发现目标 Bus 等于自己的 Secondary，则转换为 Type 0，否则在目标位于 Subordinate 范围时继续以 Type 1 下传。

## 枚举与异常响应

- 不存在的 Function：下游 Bridge 返回 Completion without Data、状态 UR；RC 为传统软件合成为 Vendor ID FFFFh。枚举期间这是预期探测结果，通常不启用错误报告。
- 暂未准备好的 Function：配置请求返回 CRS。复位后的初始化窗口为 1.0 s（+50%/-0%）；在不高于5.0 GT/s时软件至少等待复位后100 ms，高于5.0 GT/s时至少等待链路训练完成后100 ms。
- 若 Root Control 的 CRS Software Visibility=1，且访问是完整 2-byte Vendor ID 读，RC 可向软件返回伪 Vendor ID 0001h；其他配置访问由 RC 自动重试。
- Header Type 偏移 0Eh：低7 bit为0表示 Endpoint，1表示 PCI-to-PCI Bridge，2表示 CardBus Bridge；bit7表示 Multifunction。
- 单 RC 枚举从 Bus0 开始，对 Device0–31 的 Function0 读取 Vendor ID；若 Multifunction=1再扫描 Function1–7。发现 Bridge 后分配 Secondary，新分支 Subordinate 暂设255，深度优先扫描完成后回填最大实际 Bus。
- 多 RC 系统给各 RC 分配不重叠的总线范围。书中示例在主 RC 枚举完成后把次级 RC 的 Secondary/Subordinate 都设为64，发现其 Bridge 后分配 Bus65，并在完成后把次级 RC 的 Subordinate 更新为65；64/128只是常见软件习惯，不是协议硬性值。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 版本/日期：PDF 元数据标注 2026年6月1日
- 位置：第3章 §3.6–§3.13.3；PDF p.94–p.118 左右（书内 p.6–p.30 左右）
- 依据：使用 `pdftotext -layout` 提取 PDF p.93–p.120，并核对配置端口、ECAM、Type 0/1、配置访问示例和枚举正文。

# 适用条件与例外

- 本文件记录该书 PCIe 3.0 语境下的配置机制概述；平台固件的 MCFG/ECAM 基址和多 RC 总线范围由具体平台提供。
- 枚举期间 UR/CRS 的软件呈现有明确适用条件，不应推广到正常运行期的任意访问。

# 关联章节

- 第4章地址空间和事务路由；PCIe能力寄存器、错误报告、热插拔与链路训练章节

# 待核验问题

- 无。
