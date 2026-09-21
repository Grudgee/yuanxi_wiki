---
name: knowledge_pcie_chapter_4_cycle_1_address_spaces_bars_windows
description: PCIe第4章§4.1–§4.3完整学习：地址空间、BAR评估与编程、Resizable BAR以及Bridge Base/Limit窗口。
---

# 学习范围

- 文档：PCI Express Technology 3.0 中文版
- 位置：第4章§4.1–§4.3，PDF p.122–146（书内约 p.2–26）
- 本周期源文本：约18,602字符，达到15,000字符下限且低于80,000上限。

# 一、§4.1 地址空间

PCIe Function 可能需要三类软件可见地址资源：Configuration Space、Memory Space 和 I/O Space。配置空间用于发现和配置 Function；Memory/I/O 空间用于设备运行期寄存器和数据访问。Root Complex 或 Bridge 根据 TLP 类型和地址将请求送往目标 Function。

## Memory 与 I/O

- Memory-Mapped I/O（MMIO）把设备寄存器映射到处理器 Memory 地址空间；32-bit Memory 地址位于4GB以下，64-bit Memory 地址可位于4GB以上。
- I/O 空间是传统 PCI 兼容资源，地址宽度较小，Native PCIe 设备通常优先使用 MMIO；IO TLP 使用地址路由和 Bridge 的 IO Base/Limit 窗口。
- Memory 空间分为 Prefetchable MMIO（P-MMIO）和 Non-Prefetchable MMIO（NP-MMIO）。P-MMIO 允许读合并、预取或写合并，前提是设备寄存器没有读副作用、读写顺序要求或外部状态变化；NP-MMIO 用于读有副作用、访问顺序敏感或不能被合并的寄存器。
- Software/firmware 必须把 P-MMIO 与 NP-MMIO 分开分配，Bridge 也分别维护两类下游窗口。把不可预取寄存器放进可预取窗口可能导致重复读、顺序变化或状态清除错误。

# 二、§4.2 BAR

Base Address Register（BAR）由 Function 配置 Header 提供，用来声明一个 IO、NP-MMIO 或 P-MMIO 地址范围。Type 0 Header 最多有6个32-bit BAR；Type 1 Header（Bridge、Switch/RC Port）有2个自身 BAR。Bridge 的下游地址范围不是自身 BAR，而由 Base/Limit 寄存器描述。

## BAR 评估流程

1. 软件暂时清除 Command Register 中对应的 Memory Space 或 IO Space Enable，避免评估期间设备响应错误地址。
2. 向 BAR 的可写位写全1，再读回。设备将固定为0的位保留为0，可写位读回1；最低有效位显示资源类型和对齐粒度。
3. 由读回掩码推导请求大小：对齐位越低，所需地址范围越大；软件据此在系统资源中分配不重叠基址。
4. 把分配的基址写回 BAR，恢复相应 Enable。Function 才会接受落入该范围的 TLP。

## 32-bit、64-bit 和 IO BAR

- 32-bit Memory BAR 的低位包含 Memory 类型、可预取属性和大小掩码；地址基址必须按请求大小对齐。
- 64-bit P-MMIO 使用连续两个 BAR：低位 BAR 声明类型和低32位，高位 BAR 保存地址高32位。软件评估低位 BAR 后必须把下一个 BAR 当作高位部分，不能将其当独立资源。
- IO BAR 的 bit0=1 表示 IO 请求，bit1保留为0；IO BAR 的有效地址位从 bit2 起，评估结果的低位决定对齐和大小。书中 IO 示例请求256B，分配4000h–40FFh。
- 所有 BAR 必须按 BAR0、BAR1、BAR2……顺序评估，包括未实现 BAR。未实现 BAR 应固定为0；64-bit BAR 占用的高位 BAR 不能再次独立分配。

## 书中评估实例

- 32-bit NP-MMIO：最低可写位 bit12 表示4KB范围，示例基址 F900_0000h，响应范围 F900_0000h–F900_0FFFh。
- 64-bit P-MMIO：低/高 BAR 组成一个连续地址值，软件必须用64-bit资源分配器处理，且满足设备要求的自然对齐。
- IO BAR：读回掩码表明请求256B，基址4000h，启用 IO 译码后设备响应4000h–40FFh。

## Resizable BAR

PCIe 2.1 引入 Resizable BAR 能力。Function 可以通过扩展能力结构公布多个可接受大小，软件根据平台资源选择2GB、1GB、512MB或256MB等大小。改变大小前应清除 Memory Enable；改变后原 BAR 内容丢失，必须重新写入基址。

- 能力寄存器 bits[4:23] 分别表示1MB、2MB、4MB……512GB是否支持。
- 控制寄存器 BAR Index 取0–5；Number of Resizable BARs 只在第0组定义；BAR Size 编码0=1MB、1=2MB、2=4MB，递增到19=512GB。
- 软件应选择系统能支持的最大大小，但设备不应声明超过实际有效使用范围的大小。

# 三、§4.3 Base/Limit

Bridge 的 Base/Limit 寄存器把下游地址空间汇总成连续窗口。每个 Bridge 根据下游所有 Function 的 BAR 需求分配 P-MMIO、NP-MMIO 和 IO 范围，并将窗口向上游逐级汇总。

## P-MMIO 窗口

- Prefetchable Base/Limit 描述下游可预取 Memory 范围，通常可使用64-bit扩展寄存器以覆盖大地址空间。
- NP-MMIO Base/Limit 描述不可预取 Memory 范围；窗口边界必须满足设备和 Bridge 的粒度对齐。
- 一个 TLP 若命中下游窗口，Bridge 向 Secondary Interface 转发；未命中任一合法窗口则不能随意转发。

## IO 窗口与无效范围

- IO Base/Limit 描述下游传统 IO 地址范围，寄存器具有类型/地址扩展位；平台若不支持 IO，可把窗口置为无效范围。
- Base 大于 Limit 或窗口未启用时表示没有可转发资源；软件不能把无效窗口当作“全部地址”窗口。
- 下游多个 Bridge 的范围必须合并为上游可表示的窗口；若资源不连续，软件可能需要扩大窗口并承担地址空洞，或重新分配 BAR 使其连续。

## 配置顺序与路由关系

1. 枚举先发现 Endpoint BAR 需求。
2. 从叶子 Bridge 向上设置 Secondary/Subordinate Bus 和各类 Base/Limit。
3. 上游 Bridge 的窗口必须覆盖下游所有需要转发的范围。
4. 最后启用 Bridge Command 中的 Memory/IO Decode，使窗口和 BAR 正式生效。

Base/Limit 只描述 Bridge 下游地址范围，不代表 Bridge 自身 BAR；TLP 到达端口后应先检查端口自身 BAR，再检查下游窗口。

# 适用条件与例外

- 地址空间的可预取属性必须与设备寄存器行为一致；不能仅按性能偏好设置。
- 资源分配受平台地址宽度、固件策略、BAR 对齐、Bridge 窗口粒度和热插拔空间预留影响。
- 本周期覆盖地址分配与窗口形成；TLP 进入端口后的三种路由方法在下一周期处理。

# 关联章节

- 第3章配置与枚举；第4章§4.4–§4.7路由；第5章TLP Header；第20章Resizable BAR扩展说明。

# 待核验问题

- 无。本周期已完整重学§4.1–§4.3。
