---
name: knowledge_pcie_chapter_3_cycle_1_bdf_config_access
description: PCIe第3章§3.1–§3.10：BDF、设备/功能、配置空间、Host Bridge、CF8h/CFCh、ECAM、Type 0/1及访问示例。
---

# 学习范围

- 文档：PCI Express Technology 3.0 中文版
- 位置：第3章§3.1–§3.10，PDF p.90–105（书内约 p.1–17）
- 本周期源文本约17,375字符，达到15,000字符下限且低于80,000字符上限。

# 一、B/D/F基本模型

PCIe 配置软件用 Bus、Device、Function（BDF）定位一个可配置 Function。Bus 号标识拓扑中的逻辑总线；Device 号标识该总线上的设备位置；Function 号标识设备内部独立的配置/事务实体。每个 Device 最多有8个 Function（0–7），每个 Function 都有自己的配置空间、BAR、能力结构和错误/电源状态。

- Endpoint 通常在树末端；Bridge/Switch Port 使用 Type 1 Header 并连接上下游总线；Root Complex 的 Root Port 是 CPU/DRAM 与 PCIe 树的接口。
- 软件通过读取 Vendor ID、Device ID、Header Type、Class Code 和能力结构判断 Function 类型，并据此分配 Bus 号、BAR 和窗口。
- Header Type 低7 bit 为0表示 Endpoint、1表示 PCI-to-PCI Bridge、2表示 CardBus Bridge；bit7 表示 Multifunction。

# 二、PCIe Bus、Device 与 Function

- PCIe 物理连接是点到点，但软件仍把 Root Complex、Switch 和 Bridge 抽象为 PCI 风格的逻辑 Bus 层次，以保持旧软件兼容。
- 一个 Switch 的每个 Port 都可被软件视为一个 Bridge；上游 Port 连接父总线，下游 Port 创建子总线。
- Function 是资源分配和配置的最小单位；多 Function Device 的 Function 0 Header bit7 指示是否需要继续扫描 Function 1–7。
- 配置访问只针对一个 Function 的某个 DW/字节范围，不把同一 Device 的多个 Function 当成一个寄存器空间。

# 三、配置空间

传统 PCI 兼容空间每个 Function 为256 B，前64 B 为标准 Header，剩余空间承载能力结构。PCIe 把每 Function 配置空间扩展到4 KB：前256 B仍保留兼容空间，新增960 DW扩展区由 Extended Configuration Access Mechanism 访问，旧 PCI 软件不可见。

- 必备能力包括 PCI Express Capability、电源管理以及 MSI/MSI-X（具体能力依 Function 类型和实现而定）。
- 配置空间用于读取设备身份、分配/启用 BAR、设置 Command/Status、建立能力链、配置错误报告和电源管理。
- 配置空间不是普通 MMIO；CPU 不能直接把任意 Memory Read 当作配置读，必须由 RC 将配置访问转换成配置请求。

# 四、Host-to-PCI Bridge 与配置请求发起

- Host-to-PCI Bridge 的配置寄存器通常映射在平台特定 Memory/IO 地址中，但寄存器布局遵循 PCI Type 0 模板。
- 只有 Root Complex 能发起配置请求。限制其他设备发起配置是为了防止任意 Function 修改系统拓扑和资源配置；因此 PCIe 不支持 Peer-to-Peer Configuration Request。
- 目标 BDF 作为路由信息写入配置请求；RC 根据目标 Bus 和各 Bridge 的 Secondary/Subordinate 范围生成 Type 0 或 Type 1。

# 五、Legacy PCI 配置机制（CF8h/CFCh）

传统 x86 使用两个32-bit IO 端口：Configuration Address Port 为 CF8h–CFBh，Configuration Data Port 为 CFCh–CFFh。一次访问必须先写地址端口，再访问数据端口，属于两步间接访问。

## CF8h 地址字段

- bit31：Enable；为1时后续 CFCh 读/写被转换成配置访问。
- bits[23:16]：目标 Bus（0–255）。
- bits[15:11]：目标 Device（0–31）。
- bits[10:8]：目标 Function（0–7）。
- bits[7:2]：兼容配置空间目标 DW/寄存器号。
- bits[1:0]：固定为0，要求 DW 对齐；bits[30:24] 保留为0。

CFCh 数据端口接受1、2或4 byte IO 访问。Host Bridge 比较目标 Bus 是否位于自己的 Secondary–Subordinate 范围：目标等于 Secondary 时生成 Type 0；目标在下游范围内但不是 Secondary 时生成 Type 1；范围外不转发。

## 多 Host 注意

多 RC 系统可共享相同 CF8h/CFCh IO 地址，但 Host Bridge 必须用不重叠的总线范围决定谁响应。枚举软件先配置活跃 RC，再给另一个 RC 分配不重叠 Bus 范围，避免多个 Host Bridge 同时响应同一配置数据访问。

# 六、ECAM/增强配置访问

ECAM 把所有 Function 的4 KB配置空间映射到一个256 MB对齐的 Memory 窗口。与 CF8h/CFCh 两步访问相比，ECAM 是单步 Memory Read/Write，不存在多线程在地址寄存器第二步前覆盖目标的问题。

- 地址[63:28]：256 MB 对齐的 ECAM 基址。
- 地址[27:20]：Bus。
- 地址[19:15]：Device。
- 地址[14:12]：Function。
- 地址[11:2]：Function 配置空间 DW。
- 地址[1:0]：DW 内 byte offset。

每 Function 占4 KB，256 Bus×32 Device×8 Function×4 KB=256 MB。访问应保持 DW 对齐，避免跨 DW 边界；除非 RC 明确支持，软件也不应依赖配置空间上的总线锁定操作。

# 七、Type 0/Type 1 配置请求

- Type 0 的 Type 编码为00100b，发往当前 Secondary Bus；设备检查 Device、Function、Register Number 和 First DW Byte Enable。
- Type 1 的 Type 编码为00101b，只由 Bridge 处理；目标 Bus 仍在下游范围时继续转发，抵达目标 Secondary 的 Bridge 把 Type 1 转换成 Type 0。
- Endpoint 忽略目标不是本总线的 Type 1；Bridge 根据 Secondary/Subordinate 范围决定继续下传或停止。

# 八、配置访问示例

- 传统示例先向 CF8h 写入 Enable、Bus4、Device0、Function0、DW0，再从 CFCh 读取 Vendor ID 的2 byte；Host Bridge 先生成 Type 1，经过各级 Bridge 后转为 Type 0，Completion 沿 Requester ID 返回。
- ECAM 示例使用预留窗口基址 E0000000h，访问 E0400000h 即选择 Bus4/Device0/Function0/DW0/byte0，再由 RC 转成相同的配置读请求。

# 适用条件与例外

- CF8h/CFCh 是传统 x86 兼容机制；现代平台主要使用 ECAM，但平台仍需提供合法 MCFG/ECAM 窗口。
- Type 0/1 不是任意 TLP 的通用路由标签，只有 Configuration Request 使用这两种配置类型。

# 关联章节

- §3.11–§3.14枚举与工具；第4章地址空间/路由；第5章Configuration TLP；第15章错误报告。

# 待核验问题

- 无。本周期已重学§3.1–§3.10。
