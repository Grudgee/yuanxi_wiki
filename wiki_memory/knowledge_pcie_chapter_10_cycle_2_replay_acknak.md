---
name: knowledge_pcie_chapter_10_cycle_2_replay_acknak
description: PCIe第10章中段：TLP重放、Nak响应、Replay Timer、接收端LCRC/序列号处理和AckNak延迟。
---

# 学习范围

- 位置：第10章§10.3.1.4–§10.3.2.6.2，PDF p.338–350
- 本周期源文本约15,938字符，达到15,000字符下限。

# TLP重放流程

当发送端收到 Nak 或 REPLAY_TIMER 超时，它暂停从事务层接收新的 TLP，定位 Replay Buffer 中的起始序列号，并按原发送顺序重放未确认 TLP。重放期间仍必须处理新收到的 Ack/Nak DLLP，否则可能无法释放已经成功接收的副本。

- Ack 清除其序列号及更早的 Replay 副本；Nak 保留需要重放的副本。
- 重放不重新生成事务层语义，但物理层/数据链路层会按当前链路重新发送并重新进行 LCRC/编码。
- 重放完成后才恢复接受新的事务层 TLP，避免新旧序列号窗口混淆。

# 重复重放与 REPLAY_NUM

- 每次收到 Nak，REPLAY_NUM 递增；达到计数器回绕/协议规定的重复重放条件，说明链路持续不可靠。
- 重复重放可能由固定物理损坏、错误的 Ack/Nak、接收端状态失步或定时器设置不当引起。
- 协议可触发物理层重新训练，而不是无限重放；软件应通过错误状态寄存器记录并采取降级或复位策略。

# REPLAY_TIMER

REPLAY_TIMER 在存在未确认 TLP 时启动或继续运行。超时值与最大 Payload、链路宽度、速率、编码和往返 Ack 延迟相关：宽链路/大 Payload 需要更长的发送和确认时间。定时器不能过短，否则正常链路会被误判；也不能过长，否则真实丢包恢复延迟过大。

- 重放期间计时器和 Ack/Nak 处理必须协同；收到有效 Ack 后可清除已确认副本并重新评估定时器。
- 表格中的 Gen1/Gen2/Gen3 符号时间值是规范参考，实际实现还需考虑端口延迟和最大包配置。

# 接收端 LCRC 与序列号

1. Physical Layer 先检查符号、组帧和解码错误。
2. Data Link Layer 重新计算 TLP LCRC；失败则丢弃 TLP，并调度 Nak。
3. LCRC 正确后比较序列号与 NEXT_RCV_SEQ。
4. 等于期望值：转发事务层、递增 NEXT_RCV_SEQ，并调度 Ack。
5. 小于期望值：可能是重复重放；按协议确认/丢弃，避免重复交给事务层。
6. 大于期望值：出现缺失或失序，调度 Nak，请发送端从缺失序列号重放。

# AckNak_LATENCY_TIMER

接收端在成功接收尚未确认的 TLP 后运行 AckNak_LATENCY_TIMER，确保 Ack 不被无限推迟。多个连续 TLP 可按协议合并确认，但确认延迟不能超过规定窗口。定时器值取决于链路速率、最大 TLP、接收处理延迟和 DLLP 返回时间。

- Ack/Nak DLLP 自身有16 bit CRC；坏 DLLP不能改变 NEXT_RCV_SEQ、Replay Buffer 或 AckD_SEQ。
- 收到错误 Nak、重复 Ack 或不符合序列号窗口的 DLLP 时，链路层需按错误规则处理，而不能直接释放任意副本。

# 示例逻辑

- 丢失 TLP：接收端期待序列号N+1，却收到N+2；发送 Nak(N)，发送端重放N及后续，接收端在正确收到后重新 Ack。
- 损坏 TLP：LCRC失败，即使序列号正确也丢弃并 Nak；发送端无需事务层参与即可恢复。
- 重复 TLP：重放副本在接收端已处理后再次到达；接收端根据序列号判断重复，不再次上送事务层。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版；位置：第10章§10.3.1.4–§10.3.2.6.2，PDF p.338–350。

# 适用条件与例外

- 定时器值不能只按固定纳秒数估计，应根据速率、Payload、Lane 宽度和规范表格计算。
- Replay/ACK 状态是链路本地的；跨 Switch 时每一跳独立维护。

# 待核验问题

- 无。
