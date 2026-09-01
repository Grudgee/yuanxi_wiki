---
name: knowledge-superscalar-processor-design-cycle-43-wakeup-single-multicycle
description: 超标量处理器设计第43个学习周期，记录单周期与多周期指令的唤醒机制。
---

# 本周期总结

- 唤醒通过广播目的寄存器 tag，并与发射队列中所有源寄存器 tag 比较，把等待该结果的源操作数标记为 ready。
- 单周期 ALU 指令可在被选中后很快唤醒相关指令，实现背靠背执行。
- 多周期指令不能在选中当周期立即唤醒相关指令，需要使用 delayed tag broadcast 或 delayed wake up 等延迟机制。

# 关键细节

- 周期：第 43 个学习周期。
- 源文件：`books/超标量处理器设计.pdf`。
- 范围：第 8 章 §8.5.1–§8.5.2，PDF p.266–276 / 书内 p.254–264 左右。
- 批次规模：估计 12,000–18,000 源字符，低于 80,000 字符监督阈值。

# 来源依据

- 原文分步骤说明 tag bus 广播、源寄存器比较、ready 申请和仲裁响应；随后引入延迟广播和延迟唤醒。

# 下一断点

- 学习 load 指令的推测唤醒与 D-Cache/TLB 不确定性。

