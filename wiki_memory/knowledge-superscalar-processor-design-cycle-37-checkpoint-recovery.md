---
name: knowledge-superscalar-processor-design-cycle-37-checkpoint-recovery
description: 超标量处理器设计第37个学习周期，记录RAT Checkpoint恢复机制。
---

# 本周期总结

- Checkpoint 的目的，是在分支预测、异常等事件发生前保存 RAT 等关键状态，使错误路径可快速恢复。
- 书中比较串行访问 GC 和随机访问 GC：串行方式主存储负载较小但恢复慢，随机访问方式恢复快但面积、驱动和延迟压力更大。
- 带 Checkpoint 的 RAT 在电路层面增加 CBC 等备份单元；cRAT 与 sRAT 的 checkpoint 开销不同，设计时要权衡面积和速度。

# 关键细节

- 周期：第 37 个学习周期。
- 源文件：`books/超标量处理器设计.pdf`。
- 范围：第 7 章 §7.5.1，PDF p.226–237 / 书内 p.214–225 左右。
- 批次规模：估计 12,000–20,000 源字符，低于 80,000 字符监督阈值。

# 来源依据

- 原文介绍 GC、SAGC/RAGC、MBC/CBC、Allocation/Restore，并指出 checkpoint 是为了快速保存和恢复处理器状态。

# 下一断点

- 学习使用 WALK 进行较低成本但较慢的 RAT 状态恢复。

