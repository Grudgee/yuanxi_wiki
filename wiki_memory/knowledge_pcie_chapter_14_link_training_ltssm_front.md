---
name: PCIe 第14章链路训练与 LTSSM 前段
description: 记录链路训练有序集、TS1/TS2 字段、LTSSM 总览，以及 Detect、Polling、Configuration 前段和链路宽度合并设计。
source: books/PCIe_Technology.pdf
source_pages: PDF p.482–510（书内 p.454–482）
---

# 学习范围

本周期读取第14章前段，源文本约43,950字符，覆盖 §14.1–§14.6.2，停在“支持链路合并的设备”自然小节边界。链路训练的目的不是枚举设备，而是先让物理层两端在存在性、速率、lane 数、lane 编号和工作参数上达成一致；训练完成后链路才进入 L0，随后配置空间访问和事务层业务才有可靠承载。

## 1. 有序集与训练字段（§14.1–§14.2，PDF p.482–493）

训练依赖 TS1/TS2 ordered sets。TS1/TS2 在所有候选 lane 上重复发送，使接收端能做 bit/symbol lock、极性识别、lane 对齐和参数交换。TS1 更偏向初始检测/协商，TS2 用于配置完成和进入正常工作前的确认。字段包含链路编号、通道编号、链路宽度/速度能力、训练控制、端口角色和其他规范定义的训练信息；字段不是普通 TLP payload，必须按 lane 和 ordered-set 边界解释。

训练有序集应在适当状态持续发送，状态转换以收到足够数量和一致字段为条件。若一端声称的 lane 编号、链路编号或宽度不一致，不能仅因“收到了 TS1”就进入 L0；需要回退、重新协商或降级。TS1/TS2 也承载 Gen3 equalization 控制入口，但具体四阶段均衡在后续 Recovery 讨论。

## 2. LTSSM 总览（§14.3，PDF p.493–505）

LTSSM（Link Training and Status State Machine）把链路从复位推进到 L0，并管理低功耗、恢复、速率/宽度改变、禁用和热复位等。主要状态包括 Detect、Polling、Configuration、L0、Recovery、Disabled、Hot Reset、Loopback 和 L2/L3 相关状态；很多状态又分 Quiet/Active、Configuration、Speed、Equalization 等次状态。状态机由发送/接收有序集、计时器、检测到的电气条件、训练成功/失败和软件请求驱动。

一个重要边界是：状态机负责链路级协商，不负责发现总线上的设备资源；后者是软件枚举。另一个边界是：状态进入条件通常要求“连续、足够数量且字段一致”的 TS1/TS2，单个偶然符号不能驱动全链路状态跳转。超时、失锁、坏序列或参数不兼容应使状态机按定义重新开始或降低目标，而不是停在半配置状态。

## 3. Detect 状态（§14.4，PDF p.505–514）

Detect 是复位后的入口。Detect.Quiet 保持发送器安静，等待复位条件稳定；除功能级复位等特殊情形外，链路会从这里开始。Detect.Active 通过发送器的接收器检测电路施加/采样规定的电气条件，以确认对端是否存在终端。检测成功才进入 Polling，失败则回到 Quiet/继续检测或等待复位。

检测是电气存在性而不是协议响应：没有对端接收器时不能通过发送 TS1“喊话”解决；有终端也不代表后续速率和 lane 配置一定兼容。发送器检测的电压、阻抗和时序受第13章电气规格约束。

## 4. Polling 状态（§14.5，PDF p.514–529）

Polling.Active 两端发送 TS1，建立 bit lock、symbol lock、极性、lane 对齐并交换初始能力。发送端必须把自己的 lane/链路信息放进 TS1；接收端只在收到满足数量、格式和一致性条件的 TS1 后进入下一步。若没有获得锁定、出现错误或超时，状态机会重试或回 Detect。

Polling.Configuration 把 TS1 切换为 TS2，表示基础同步已经成立，双方准备进入链路配置。TS2 中的字段仍需要检查 lane 编号、链路编号和能力。Polling.Compliance 是测试用途，发送特定测试码型，不代表正常系统训练成功；误把 Compliance 当作产品工作路径会造成“链路不进入 L0”的假故障。

Polling 的关键顺序是先建立可靠符号和 lane 关系，再协商配置；不能在未完成锁定时按猜测宽度拼包。宽链路也可以在 Polling 期间发现部分 lane 不可用，后续 Configuration 会决定能否降为较窄链路。

## 5. Configuration 状态前段（§14.6.1–§14.6.2，PDF p.529–510页范围内）

Configuration 负责确定链路编号、每个 lane 的编号和最终链路宽度。下游端口（DSP）通常发起编号协商，上游端口（USP）回应；TS1/TS2 中的 Link Number 和 Lane Number 使每条物理 lane 映射到逻辑链路顺序。配置必须处理 lane 反转、部分 lane 不可用和支持宽度集合不同等情况。

Configuration 的整体推进包括 Linkwidth、Lanenum、Complete、Idle 等次状态。Linkwidth 阶段先找到双方都支持的宽度；Lanenum 阶段为每条实际使用的 lane 分配逻辑编号；Complete 交换 TS2 并确认配置；Idle 等待进入 L0 的最后条件。每一阶段都以训练序列的数量、字段一致和计时器为前提。

设计支持链路合并的设备时，一个物理端口可能以多个较窄逻辑链路连接，也可能把多个窄连接合并为较宽连接。规范要求每个端口必须至少能作为 x1 工作；交换机内部每个端口对应逻辑桥，某些未被当前拓扑使用的逻辑桥可以闲置。LTSSM 在训练期间根据双方连接、支持的 lane 数和当前布线决定实现哪种连接，不能由“芯片有多少 lane”直接推断最终链路宽度。

## 6. 诊断要点与下一入口

Detect 失败优先检查终端/电气检测；Polling 失败优先检查参考时钟、极性、符号锁定和 TS1/TS2；Configuration 失败优先检查 lane 编号、链路编号、宽度能力和 lane skew。第14章后段将继续给出 Configuration 的三个训练示例、各次状态细节、L0、Recovery、速率/宽度切换和 Gen3 equalization。

来源：`books/PCIe_Technology.pdf` 第14章，PDF p.482–510（书内 p.454–482），§14.1–§14.6.2、图14-13/14-14。下一入口为 §14.6.3 Configuration 状态训练示例（PDF p.511）。
