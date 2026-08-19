---
name: knowledge-chapter-3-cycle-08-scoreboarding-dynamic-scheduling
description: 第3章本轮周期5，记录动态调度、记分牌和乱序执行基础。
---

# 知识点摘要

- dynamic scheduling 由硬件在运行时重排执行，以容忍编译期未知的 latency，并在保持依赖的前提下允许独立指令越过停顿指令。
- 简化记分牌把流水过程分为 issue、read operands、execution、write result；各阶段检查 structural、RAW、WAR 与 WAW hazard。
- 乱序完成会带来非精确异常问题：发生异常时，较晚指令可能已改变体系结构状态。

# 关键细节

- 周期：本轮周期 5/16。
- 来源范围：PDF p.195 下半页–198 上半页 / 书内 p.167 下半页–170 上半页。
- OCR 源字符数：8,989（小于 80,000）。
- 下一断点：PDF p.198 / 书内 p.170，`Dynamic Scheduling Using Tomasulo's Approach` 标题处。

# 原文引用

- 文档：《计算机体系结构：量化研究方法》扫描版 PDF。
- 版本/日期：未知。
- 位置：第3章 §3.4，PDF p.195–198 / 书内 p.167–170。
- 依据：原文先说明动态调度的动机，再以 CDC 6600 scoreboard 的四阶段控制说明 hazard 检测和乱序执行代价。

# 适用条件与例外

- 动态调度不能消除真实 RAW 依赖，只能让无关指令绕过等待。
- 非精确异常是否可接受取决于 ISA 与系统软件要求。

# 关联章节

- §3.4 Tomasulo；§3.6 hardware-based speculation。

# 待核验问题

- 无。
