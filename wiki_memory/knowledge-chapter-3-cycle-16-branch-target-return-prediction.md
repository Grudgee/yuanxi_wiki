---
name: knowledge-chapter-3-cycle-16-branch-target-return-prediction
description: 第3章本轮周期13，记录BTB、返回地址栈与控制转移取指。
---

# 知识点摘要

- Branch-Target Buffer 以当前取指地址查找分支预测信息和目标地址，使预测 taken 时可在更早阶段直接提供下一 PC。
- BTB miss、方向预测错误和目标错误具有不同 penalty；高发射宽度会放大每个错误周期损失的指令数。
- indirect jump 的多个可能目标难由单一 BTB 项覆盖；return address stack 通过 call 时压栈、return 时弹栈预测嵌套返回。

# 关键细节

- 周期：本轮周期 13/16。
- 来源范围：PDF p.230 下半页–234 / 书内 p.202 下半页–206。
- OCR 源字符数：10,479（小于 80,000）。
- 下一断点：PDF p.235 / 书内 p.207，return predictor 结果与 integrated instruction fetch unit。

# 原文引用

- 文档：《计算机体系结构：量化研究方法》扫描版 PDF。
- 版本/日期：未知。
- 位置：第3章 §3.9，PDF p.230–234 / 书内 p.202–206。
- 依据：图3.21–3.23说明 BTB 查找与不同命中/预测组合，随后正文解释 return address buffer 的栈式行为。

# 适用条件与例外

- 返回栈深度不足、非规范调用/返回或上下文切换会降低准确率。
- BTB 容量和索引 aliasing 会影响命中率。

# 关联章节

- §3.3 branch prediction；§3.9 integrated fetch。

# 待核验问题

- 图3.23 penalty 表的细小单元未逐项转录。
