---
name: knowledge_pcie_chapter_10_cycle_3_errors_timers_passthrough
description: PCIe第10章后段：Ack/Nak错误示例、调度优先级、不同速率定时器和Switch直通模式。
---

# 学习范围

- 位置：第10章§10.4–§10.8.4，PDF p.351–359
- 本周期源文本约10,219字符；这是第10章剩余全部内容，作为最终周期允许低于15,000字符。

# 错误示例

## 丢失 TLP与错误确认

丢失 TLP 会表现为 NEXT_RCV_SEQ 与收到序列号不一致，接收端发送 Nak，发送端从缺失序列号开始 Replay。错误确认包括 Ack DLLP 被破坏、Ack 序列号超出发送窗口或 Ack/Nak 状态不一致；这些情况不能让发送端错误释放 Replay Buffer。

## 损坏的 Nak

如果 Nak DLLP 的 CRC 损坏，发送端不能确认 Nak 的序列号和原因，应依据 Replay Timer 或其他链路状态保护机制继续保留副本。错误的 Nak 不得导致错误的起始序列号重放。

# Ack/Nak协议错误处理

- 发送端必须保存足够 Replay 状态，直到 Ack 确认；接收端必须避免重复 Ack/Nak 造成状态机倒退。
- 物理层错误、DLLP CRC错误、TLP LCRC错误、序列号跳跃和定时器超时分别记录；不能把所有错误折叠成普通 UR。
- 错误恢复应优先保证链路状态一致，再恢复事务层流量；持续错误可能触发 Link Recovery、重新训练或上报不可校正错误。

# 推荐调度优先级

Ack/Nak DLLP、流控 DLLP 等链路维护包必须获得足够优先级，避免被普通 TLP 或低优先级维护流量阻塞。调度优先级需要同时考虑：

1. Ack/Nak 的确认延迟限制；
2. Flow Control Update 的 Credit 释放；
3. Power Management DLLP 的状态转换；
4. 正常 TLP 和重放 TLP 的链路占用。

如果 Ack/Nak 被延迟到 Replay Timer 超时，会产生不必要重放和吞吐下降；如果流控更新被延迟，则发送端可能停顿等待 Credit。

# 不同速率的定时器

第10章给出 Gen1（2.5 GT/s）、Gen2（5.0 GT/s）和 Gen3（8.0 GT/s）下 AckNak_LATENCY_TIMER 与 REPLAY_TIMER 的参考值。实现应使用对应速率和 Symbol Time 表格，结合最大 Payload、链路宽度、端口延迟和编码效率装载计数器，不能把某一代的值直接复制到另一代。

- 速率提高通常缩短单个符号传输时间，但更高 Payload、宽链路和更复杂均衡/缓冲会改变总延迟。
- 配置速率变化或链路宽度变化后，相关定时器必须使用新参数重新评估。

# Switch直通模式

Switch Pass-Through Mode 是延迟优化选项：在确认 TLP 足以通过本端口且不会违反流控/错误检查时，Switch 可减少完整存储后转发的等待。直通不能跳过必要的 LCRC、序列号、路由、Credit 或错误检查。

- 直通模式可能降低首字节延迟，但要求出口端口及时可用；若出口阻塞，Switch 仍需缓存或暂停。
- 发生错误时，已经部分进入出口的数据必须遵循链路错误恢复，不能因为直通而绕过 Replay/NAK 机制。
- 直通模式应在验证中覆盖：下游 Credit 不足、TLP 边界、Ack/Nak 到达、连续重放、多个 VC 竞争和出口切换。

# 原文引用

- 文档：PCI Express Technology 3.0 中文版；位置：第10章§10.4–§10.8.4，PDF p.351–359。

# 适用条件与例外

- 本周期已读完第10章；下一入口是第11章 Gen1/Gen2 物理层逻辑。
- 具体定时器数值应以对应速率表格和实现参数为准；本 memory 记录规则和影响，不复制整张表。

# 待核验问题

- 无。
