---
name: knowledge_pcie_chapter_3_configuration_basics
description: PCIe 第3章§3.1–§3.5配置基础，定义 Bus/Device/Function、设备类型和 PCI 兼容与扩展配置空间。
---

# 知识点摘要

- PCIe 配置寻址采用 Bus/Device/Function（BDF）层次：总线号标识总线，设备号标识总线上的设备，功能号标识设备内的 Function。
- 一个设备最多有 8 个 Function（0–7）；Function 是配置空间、BAR 和事务能力的基本软件可见实体。
- PCIe 总线是点到点链路构成的逻辑总线；Root Complex、Switch 和 Bridge 可提供总线分支，软件通过配置空间看到兼容 PCI 的层次结构。
- PCIe 设备包括 Root Complex、Switch、Bridge 和 Endpoint 等类型；配置 Header/Type 字段决定其能力及后续寄存器布局。
- 配置空间分为 PCI 兼容空间和扩展配置空间。前者保留传统 Header 和软件模型，后者扩展到更大的地址范围以容纳 PCIe 能力结构。

# 关键细节

- B/D/F 组合用于唯一定位 Function；Bus/Device/Function 的具体编码与配置请求路由在本章后续展开。
- PCI 兼容空间包含传统配置 Header、命令/状态、BAR、中断和能力指针等软件可见字段；扩展空间继续保留兼容空间，同时增加扩展能力结构。
- 配置空间的存在使枚举软件能够读取 Vendor/Device 信息、判断 Header 类型、分配总线号与资源，并据此建立系统拓扑。
- Root Complex 侧的 Host-to-PCI Bridge 配置寄存器连接处理器配置访问与 PCIe 层次；详细 Type 0/Type 1 请求和 ECAM 在后续小节说明。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 位置：第3章 §3.1–§3.5；PDF p.90–p.96 左右（书内第1–7页）
- 依据：从“总线/设备/功能的定义”至“配置地址空间”及其 PCI 兼容空间、扩展配置空间小节，在进入 §3.6 Host-to-PCI Bridge 前停止。

# 适用条件与例外

- 本文件只建立配置模型和术语；配置地址端口、ECAM、Type 0/Type 1 路由与枚举算法的核验结果记录在 `knowledge_pcie_chapter_3_configuration_routing_enumeration.md`。
- 具体寄存器偏移、位域和能力链应以后续章节及表格逐项核验。

# 关联章节

- 第3章 §3.6–§3.13：Host-to-PCI Bridge、配置请求、访问示例和枚举
- 第4章：地址空间与事务路由

# 待核验问题

- 无。配置请求逐跳路由、ECAM、Type 0/Type 1 与单/多 RC 枚举已在 `knowledge_pcie_chapter_3_configuration_routing_enumeration.md` 中完成核验。
