---
name: knowledge_pcie_chapter_10_cycle_1_acknak_elements
description: PCIe第10章前段：Ack/Nak协议目标、发送/接收端元素、Replay Buffer、序列号、LCRC和定时器基础。
---

# 学习范围

- 位置：第10章§10.1–§10.3.1.3，PDF p.326–337
- 与第9章合计源文本约16,129字符；本文件覆盖章节分界后的第10章前段。

# 协议目标与模型

Ack/Nak 协议提供链路级 TLP 错误检测和恢复。发送端给每个 TLP 加序列号、计算 LCRC并保存副本；接收端检查 LCRC/序列号，成功则 Ack，失败则 Nak；发送端依据 Ack/Nak 决定释放或重放 Replay Buffer。

- Ack/Nak 只保证当前链路，TLP 经过每个 Switch/Bridge 时会重新形成下一跳的链路级保护。
- ECRC 是端到端可选校验，Ack/Nak 的 LCRC 是相邻链路校验，两者作用范围不同。
- 发送端和接收端同时具有 Tx/Rx 逻辑；书中用“发送设备/接收设备”描述当前 TLP 方向，不代表硬件只能单向工作。

# 发送端元素

## Sequence Number

发送端为每个 TLP 分配12 bit序列号，按发送顺序递增并回绕。序列号进入 LCRC 计算，也用于 Ack/Nak 指示边界。回绕时必须正确处理旧序列号和新序列号的窗口关系，不能仅用大小比较判断新旧。

## LCRC

LCRC 覆盖 TLP 的序列号、Header、Data 和 ECRC（若存在）。发送端把 LCRC 附在 TLP 后；下一个链路的出口会重新计算 LCRC，因此链路内部错误不会被同一 LCRC 跨越 Switch 掩盖。

## Replay Buffer

TLP 在发送前或发送时复制到 Replay Buffer，直到收到确认其序列号的 Ack 才释放。Buffer 必须覆盖尚未确认的 TLP 窗口；若收到 Nak，则从相应序列号开始重放，并保留尚未确认的后续副本。

## REPLAY_TIMER 与 REPLAY_NUM

REPLAY_TIMER 在存在已发出但尚未确认的 TLP 时运行；超时表示 Ack/Nak 可能丢失或链路异常，发送端重放未确认数据。REPLAY_NUM 是2 bit计数器，用于跟踪 Nak/超时后的重试次数；连续翻转通常意味着链路严重故障，需要物理层重新训练或报告错误。

## ACKD_SEQ 与 DLLP CRC

ACKD_SEQ 保存最近处理的 Ack/Nak 序列号；收到 DLLP 后先校验16 bit DLLP CRC，CRC错误的 Ack/Nak不能改变 Replay 状态，避免错误确认导致数据丢失。

# 接收端元素

- LCRC Error Check 重新计算并比较 TLP LCRC；错误时不把 TLP 上送事务层，并安排 Nak。
- NEXT_RCV_SEQ 保存期望的下一个 TLP 序列号；LCRC正确但序列号跳跃表示丢失/失序，需要 Nak 或等待重放。
- NAK_SCHEDULED 标志避免同一错误窗口重复调度不必要的 Nak；成功收到期望 TLP 后清除。
- AckNak_LATENCY_TIMER 确保成功接收的 TLP 在规定延迟内得到 Ack；超时会触发相应链路恢复动作。
- Ack/Nak Generator 根据 LCRC、序列号、重复包和错误状态生成 DLLP，而不是由事务层软件决定。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版；位置：第10章§10.1–§10.3.1.3，PDF p.326–337。

# 适用条件与例外

- Replay Buffer 大小是实现优化项，但必须足以覆盖协议要求的未确认窗口；过小会限制吞吐或触发协议违例。
- Ack/Nak 处理不能修复已经被事务层消费的错误数据；ECRC/错误报告负责端到端或软件可见层面的补充检查。

# 待核验问题

- 无。
