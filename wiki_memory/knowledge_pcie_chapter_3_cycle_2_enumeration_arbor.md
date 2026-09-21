---
name: knowledge_pcie_chapter_3_cycle_2_enumeration_arbor
description: PCIe第3章§3.11–§3.14：Function发现、UR/CRS、单/多RC枚举、热插拔和MindShare Arbor工具。
---

# 学习范围

- 文档：PCI Express Technology 3.0 中文版
- 位置：第3章§3.11–§3.14，PDF p.106–121（书内约 p.18–32）
- 本周期源文本约17,908字符；这是第3章剩余全部内容，达到15,000字符下限。

# 一、枚举目标与初始状态

系统上电或复位后，配置软件必须发现 PCIe 拓扑中的 Bus、Device、Function、Bridge 和 Endpoint，并为它们分配 Bus 号、BAR 和资源窗口。初始时软件通常只知道 Host/PCI Bridge 和其 Secondary Bus 0；各下游 Bridge 的 Subordinate Bus 可暂设为255，待深度优先扫描完成后再回填真实最大 Bus。

# 二、通过 Vendor ID 发现 Function

- 枚举软件遍历 Bus–Device–Function 组合，通常先读取配置 Header 的 Vendor ID。
- Vendor ID 为 FFFFh 表示没有 Function；PCIe 中不存在的设备实际可返回 Completion without Data/UR，RC 为兼容传统软件合成为 FFFFh。
- 枚举期间这种 UR 是预期探测结果，通常暂不启用错误报告，否则枚举会把正常的“空槽”误判为系统故障。
- 找到有效 Vendor ID 后读取 Header Type；Type 0 表示 Endpoint，Type 1 表示 PCI-to-PCI Bridge，bit7 Multifunction=1 时继续探测 Function1–7。

# 三、设备未准备好与 CRS

设备存在但尚未完成内部初始化时不能简单返回普通 Completion。PCIe Function 对配置请求返回 Configuration Request Retry Status（CRS）。CRS 只对 Configuration Request 合法，用在 Memory/IO 请求上会形成 Malformed TLP。

- 数据速率不高于5.0 GT/s：软件应在复位后至少等待100 ms再发起配置请求。
- 高于5.0 GT/s（Gen3）：应在链路训练完成后再等待100 ms；均衡可能约需50 ms。
- Function 初始化时间规范为1.0 s（+50%/-0%），可用于从串行 EEPROM 装载配置寄存器。
- Root Control 的 CRS Software Visibility=1时，若复位后第一次访问是完整2-byte Vendor ID读，RC 可返回伪 Vendor ID 0001h，软件据此知道设备存在但需要稍后重试；其他配置读/写由 RC 自动重新下达。

# 四、Header Type 与 EP/Bridge判断

Header Type Register 位于配置 Header 偏移0Eh。低7 bit：0表示非 Bridge/Endpoint，1表示 PCI-to-PCI Bridge，2表示 CardBus Bridge；bit7 为 Multifunction。Bridge Function 可能是多功能，每个 Function 可分别表现为虚拟 Bridge 或 Endpoint。

# 五、单 RC 深度优先枚举

典型流程如下：

1. Host/PCI Bridge Secondary=0，Subordinate=255，暂时把下方 Bus 范围视为0–255。
2. 从 Bus0 Device0 Function0 开始读取 Vendor ID，遍历 Device0–31；若 Multifunction，则扫描 Function1–7。
3. 发现 Bridge 后写 Primary Bus、Secondary Bus=下一个可用号、Subordinate=255。
4. 立即递归进入新 Secondary Bus，继续深度优先搜索；不要先返回父总线扫描其他 Device。
5. 叶子 Endpoint 扫描结束后，把上一级 Bridge 的 Subordinate 更新为该分支实际最大 Bus。
6. 返回父 Bus 继续扫描下一个 Device/Function；所有分支完成后，把 Host Bridge Subordinate 更新为整个树的最大 Bus。

这种算法保证任意 Bridge 的 Secondary/Subordinate 范围在配置路由时覆盖完整下游树，也让 Type 1 配置请求可以逐级转发到目标 Bus。

# 六、多 RC 枚举

多 Root Complex 各有自己的 CF8h/CFCh、ECAM、Host/PCI Bridge 和 Secondary/Subordinate 寄存器，但可能共享相同 IO 端口地址。枚举时先给主 RC 使用 Bus0并完成左侧树，再给次级 RC 分配不重叠的起始 Bus；书中示例把次级 RC 设为Bus64，其下游 Bridge 使用Bus65。64/128是常见软件习惯，不是协议强制值。

- 不同 RC 的总线范围必须不重叠，使共享配置端口访问只有一个 Host Bridge 响应。
- 次级 RC 枚举前，其 Host Bridge 忽略目标不在自身范围的配置访问；范围配置完成后才响应新的 Bus 区间。
- 枚举完成后回填每个 Bridge 的真实 Subordinate Bus；配置窗口和路由据此稳定。

# 七、热插拔注意事项

热插拔可能在枚举后增加设备或 Bridge。软件必须重新扫描受影响的 Bus/Port，分配新的 Bus 号和 BAR/窗口，并避免破坏现有设备的范围。预留的 Bus/Memory/IO 资源可减少热插拔时重排整个树的需要。

# 八、MindShare Arbor

§3.14 介绍 MindShare Arbor 作为 PCI/PCI-X/PCIe 的调试、验证、分析和学习工具。它不是协议硬件层，而是帮助软件工程师观察和修改配置/资源状态。

- 可扫描系统中的 PCI Function，读取或写入 Configuration、Memory 和 IO 地址。
- 可解码 PCI、PCI-X、PCIe 标准配置结构，以及部分 x86 结构和设备特定寄存器。
- 用户可用 XML 定义自定义寄存器译码，覆盖配置空间、Memory 空间和 IO 空间；扫描结果可保存为开放 XML 供其他系统查看。
- 工具适用于理解 BDF、Header、BAR、能力结构和当前路由配置，但不能替代硬件规范、平台固件或错误注入验证。
- 本书没有给出 Arbor 的具体版本号、授权条款或完整支持结构列表，这些信息不能从当前 PDF 臆造补全。

# 适用条件与例外

- FFFFh、UR、CRS 在枚举阶段有特定软件解释，不能推广为正常运行时所有配置访问的通用结果。
- 单/多 RC 的起始 Bus 选择是软件策略；协议只要求范围合法且不冲突。
- 热插拔重枚举的具体资源回收和操作系统策略不在本周期展开。

# 关联章节

- 第3章§3.1–§3.10配置访问；第4章Base/Limit和TLP路由；第16章电源管理；热插拔和错误报告章节。

# 待核验问题

- 无。本周期已重学§3.11–§3.14，第3章全部内容完成。
