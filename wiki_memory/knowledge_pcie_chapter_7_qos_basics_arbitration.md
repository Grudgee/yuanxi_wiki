---
name: knowledge_pcie_chapter_7_qos_basics_arbitration
description: PCIe 第7章§7.1–§7.5 QoS基础、TC/VC映射及VC/端口仲裁。
---

# 知识点摘要

QoS 为不同流量提供差异化优先级，目标是让视频、实时控制和等时数据获得可预测的延迟、带宽和抖动。PCIe 用 TLP 的 TC 字段标记流量，用 VC Buffer 隔离队列，再由 VC 仲裁和端口仲裁决定发送顺序；TC 只表达优先级，不能单独保证端到端带宽。

# §7.1–§7.2 基本模型

- 高级 QoS 同时依赖传输速率、低延迟、错误管理、硬件 Buffer、仲裁器和软件服务管理；任何一项不足都会破坏等时服务。
- TC[2:0] 提供 TC0–TC7，数值越大通常代表越高优先级；TC0 是默认值，传统 PCI 配置软件不识别 QoS 能力时默认使用 TC0/VC0。
- Configuration、IO 和 Message 等维护流量通常固定在 TC0/VC0，避免占据高优先级数据通道。

# §7.2.2 VC 与 TC/VC 映射

- VC 是端口输出方向上的硬件队列；每个端口必须有 VC0，最多可实现 VC0–VC7。
- TC 在 Requester 到 Completer 的路径上保持不变，但每一条链路可以把 TC 映射到不同 VC。VC Resource Control Register 的 TC/VC Map 为每个 TC 指定所属 VC。
- 多个 TC 可以映射到同一 VC；同一 VC 内的 TLP 共享 Buffer、Credit 和仲裁机会。VC0 是固定存在的默认通道，其他 VC 需能力结构和软件配置支持。
- 软件先检查链路两端可支持的 VC 数量，再分配 VC ID；不能使用对端未实现的 VC，也不能在未建立映射时发送高优先级流量。

# §7.3 VC 仲裁

- VC 仲裁从有足够 Flow Control Credit 的 VC 中选择下一个 TLP；没有 Credit 的 VC 即使优先级高也不能发送。
- Strict Priority 总是优先选择高优先级 VC，能降低高优先级延迟，但可能让低优先级 VC 饥饿。
- Group Arbitration 先在 VC 组之间选择，再在组内选择；组划分允许维护流量与实时流量隔离。
- Hardware-Fixed Arbitration 不需软件设置，硬件按固定规则选择；WRR 为不同 VC 配置权重，使服务份额近似与权重成比例。
- VAT（VC Arbitration Table）允许软件写入仲裁表，把 VC 或 VC 组按需要排列；表的周期性遍历影响可预测性和最大等待时间。

# §7.4 端口仲裁

- VC 仲裁选出候选 TLP 后，端口仲裁决定哪个 VC 的候选包占用物理出口；因此 QoS 受两个仲裁层共同影响。
- Hardware-Fixed Port Arbitration 提供固定优先级；WRR 用权重分配端口服务份额；TBWRR 用时间窗口/时隙控制等时流量。
- Port Arbitration Table 由配置软件加载，交换机每个出口可有独立策略；上游/下游端口的流量竞争可能不同。
- 仲裁只能在 Credit 足够、链路可发送且 TLP 合法时生效；它不能越过流控、排序或错误恢复约束。

# §7.5 多功能 Endpoint

- 多功能 Endpoint 的多个 Function 可能共享一个 PCIe Port 和输出资源，需要在 Function/VC 层面进行额外仲裁。
- 若某 Function 长期产生高优先级流量，硬件必须避免其独占共享端口；WRR、固定份额或表格策略可用于控制 Function 间公平性。

# 性能与验证要点

- TC 不变、VC 可变：调试时应同时查看 TLP Header 的 TC、每链路 TC/VC Map、VC Credit 和端口仲裁表。
- 评估等时服务应测量最大等待时间、带宽下限、缓冲溢出风险和错误恢复影响；仅证明“高优先级先发”不足以证明实时保证。
- 低优先级饥饿、Credit 不足、端口 WRR 权重错误和多级 Switch 累积延迟是常见验证场景。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版
- 位置：第7章§7.1–§7.5，PDF p.255–280；本周期源文本约 17,483 字符。
- 依据：QoS 动机、TC、VC、TC/VC 映射、VC 仲裁、端口仲裁、WRR/TBWRR 和多功能 Endpoint 小节。

# 适用条件与例外

- QoS/等时服务需要硬件 VC、Credit、仲裁和软件协同；TC 值本身不保证确定性延迟或带宽。
- 具体 TBWRR 时隙和等时代理流程在 §7.6 及后续小节，尚未包含在本周期。

# 关联章节

- 第6章流量控制；§7.6等时服务；第8章事务排序；第9章DLLP。

# 待核验问题

- 无。
