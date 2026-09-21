---
name: knowledge_pcie_chapter_4_cycle_2_routing_all
description: PCIe第4章§4.4–§4.7完整学习：地址路由寄存器检查、TLP路由、ID/地址/隐式路由及DLLP边界。
---

# 学习范围

- 文档：PCI Express Technology 3.0 中文版
- 位置：第4章§4.4–§4.7，PDF p.147–170（书内约 p.27–50）
- 本周期源文本：约11,440字符；这是第4章剩余全部内容，因此作为最终周期允许低于15,000字符。

# 一、§4.4 地址路由寄存器检查

理解地址路由时必须区分两类寄存器：Bridge/Switch Port 自身的 BAR，以及描述下游范围的 P-MMIO、NP-MMIO、IO Base/Limit。自身 BAR 命中表示 TLP 的目标是当前端口；Base/Limit 命中表示目标位于下游，应转发而不是由端口本地消费。

- 多分支系统中，每个下游 Bridge 的窗口必须覆盖其叶子设备 BAR；父 Bridge 再把子窗口合并到自己的范围。
- P-MMIO、NP-MMIO、IO 三类窗口分别检查，不能用一个窗口代替另一类地址空间。
- 窗口重叠、范围空洞、Base 大于 Limit、对齐错误或 Bridge Decode 未启用都会造成路由错误、UR 或设备不可访问。
- 配置软件应检查 BAR、Base/Limit、Command Enable 和拓扑方向是否一致；不能只看 BAR 是否写入基址。

# 二、§4.5 TLP 路由基础

PCIe 接收入口要面对三类流量：Ordered Set、DLLP 和 TLP。Ordered Set 与 DLLP 只在当前链路本地处理；TLP Header 含有跨链路路由所需信息。RC、Switch 等多端口设备是 Routing Element，可以把 TLP 交给自身资源，也可以在入口和出口间转发。Endpoint 只有一条链路，只接受或拒绝，不负责转发。

## 三种路由方法

表4-7给出 TLP 类型和路由方法的对应关系：

- Memory Read/Read Lock、Memory Write、AtomicOp：Address Routing。
- IO Read/Write：Address Routing。
- Configuration Read/Write：ID Routing。
- Message/Message with Data：Address、ID 或 Implicit Routing。
- Completion/Completion with Data：ID Routing。

Message 是唯一标准上支持多种路由方式的类别；大多数标准 Message 使用隐式路由，厂商定义 Message 可按需要使用地址或 ID。

## Split、Posted 和 Non-Posted

- Non-Posted 请求需要 Completer 返回 Completion；Memory Read 可拆成多个 CplD。
- Posted 请求不要求 Completion；Memory Write 和许多 Message 属于 Posted，但仍经过数据链路层 Ack/Nak。
- Split Transaction 将请求与完成解耦，使 Requester 不必占用链路等待 Completer；Requester ID/Tag 用于把完成包关联回原请求。

# 三、§4.6.1 ID 路由

ID Routing 使用目标 BDF：Bus 8 bit、Device 5 bit、Function 3 bit。PCIe 理论上支持256条总线、每总线32个 Device、每 Device 8个 Function。配置请求、Completion 和部分 Message 使用 ID 路由。

## Endpoint 检查

- EP 比较 TLP Header 中目标 Bus/Device/Function 与自身 BDF；相等则接受并消费，否则拒绝。
- 外部 PCIe 链路上的 Endpoint 通常是 Device 0；ARI 可改变 Device/Function 的使用方式。
- EP 不转发目标为其他 Function 的 TLP。

## Switch/Bridge 两次检查

每个 Switch Port 都是独立的 Type 1 Bridge，拥有自己的配置空间和 BDF：

1. 先将目标 ID 与端口自身 BDF 比较；命中则端口本地消费。
2. 未命中时比较目标 Bus 是否落在该端口 Secondary–Subordinate 范围；在范围内则向下游转发。
3. 上行端口两项都不命中且目标不在其下游范围时按 UR 处理。
4. 下行端口收到非本端口目标时，各端口继续检查自己的下游范围；只有声明拥有目标范围的端口转发，其余忽略。

# 四、§4.6.2 地址路由

Address Routing 使用 TLP Header 的 Address 字段。32-bit Memory/IO 使用3DW Header；大于4GB的 Memory 使用4DW Header。IO 地址为32 bit，主要用于 Legacy 兼容。

## Endpoint 与下行端口

- EP 将地址与自身 BAR 范围比较，命中则消费；EP 不命中时不转发。
- Switch/Bridge 端口先检查自己的 BAR；不命中时，IO 检查 IO Base/Limit，Memory 分别检查 NP-MMIO 和 P-MMIO Base/Limit。
- 下行方向：自身 BAR 命中则消费；下游窗口命中则转发到 Secondary；两者都不命中且无其他端口声明拥有地址时，Primary 入口返回 UR。

## 上行端口与 P2P

- 下游接口收到地址 TLP 时，若命中自身 BAR则消费。
- 若地址落在该端口下游窗口，通常表示从下游方向返回了一个本应继续下行的目标，端口将其作为 UR；但 Switch 上行端口可能把这种窗口内事务视为 Peer-to-Peer，并转发到另一个下行端口。
- BAR/窗口均未命中时，TLP 向上游转发，因为当前 Bridge 及其下游都不是目标。
- PCIe 2.1 Multicast Capability 可定义独立多播地址范围；多播范围不一定出现在 BAR 或 Base/Limit 中，路由要按多播规则处理。

# 五、§4.6.3 隐式路由

Implicit Routing 通常用于 Message。路由元件利用 RC 在拓扑顶部、EP 在底部以及端口的上行/下行方向，不需要地址或 BDF 列表。

## Message 路由子字段

表4-10：Type[4:3]=10b 表示 Message；R[2:0] 为路由方法：

- 000b：隐式路由到 Root Complex。
- 001b：地址路由，Header byte8–15携带64-bit地址。
- 010b：ID路由，Header byte8–9携带目标ID。
- 011b：隐式向下广播。
- 100b：隐式本地终止于接收者。
- 101b：隐式收集后路由到 Root Complex。
- 110b–111b：保留，终止于接收者。

所有 Message TLP 使用4DW Header。EP 接受广播或终止于自身的 Message，不接受隐式目的地为 RC 的 Message。Switch 上行端口可接收广播并复制到所有下行端口；下行端口若收到向上广播则按 Malformed TLP 处理。下行端口可把目的地为 RC 的 Message 转发上行，上行端口不应接收这种会被错误向下转发的消息；本地终止消息由当前端口消费。

# 六、§4.7 DLLP 与 Ordered Set

DLLP 和 Ordered Set 不会被 Switch/RC 从入口端口路由到出口端口。DLLP 只在相邻端口的 Data Link Layer 之间传输，用于 Ack/Nak、流控 Credit 和链路电源管理；Ordered Set 只在相邻链路的 Physical Layer 处理，用于训练、对齐、时钟容忍和电气状态。只有 TLP 可以跨越多条链路由 Routing Element 转发。

# 适用条件与例外

- 路由结论依赖 TLP Type、端口方向、BAR/窗口配置和协议能力；不能把“地址未命中”简单等同于永远向上转发。
- RC 的 P2P 能力可选；Switch 必须支持 P2P 的基本路由。
- Message 的隐式路由与地址/ID 路由使用不同检查逻辑；方向错误可能是 Malformed TLP，而非普通 UR。

# 关联章节

- §4.1–§4.3地址分配；第5章TLP Header；第6章流控；第9章DLLP；第10章Ack/Nak；第15章错误报告。

# 待核验问题

- 无。本周期已完整重学§4.4–§4.7，第4章全部内容完成。
