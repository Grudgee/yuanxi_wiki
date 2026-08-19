---
name: knowledge-chapter-3-ilp-cycle-3-speculation-entry
description: 第3章 §3.1 周期三，软件 speculation、liveness 与 §3.2 调度入口。
metadata:
  node_type: memory
  source: books/计算机体系结构量化研究方法.pdf
  modified: 2026-08-19
---

# 第3章周期三：Speculation 与编译器入口

> 范围：PDF p.184–185 / 书内 p.156–157。

- 若提前执行的指令不会改变 exception behavior，也不会改变最终 data flow，就可以有条件地违反 control dependence。
- 例中若分支之后写入的寄存器在 join 点之后为 dead，编译器可把计算移到 branch 前；分支被采取时该计算虽然无用，但不影响结果，这属于 software speculation。
- software speculation 是编译器对分支结果或执行条件下注；hardware speculation 则由处理器预测并在确认后提交，两者必须明确区分。
- 控制 hazard detection 会产生 control stalls；分支预测、编译器调度和 speculation 可减少停顿，但不能牺牲异常语义和数据流正确性。
- §3.2 的基本编译器技术面向 static issue/static scheduling，目标是让依赖指令在源指令 latency 完成后及时执行，并通过循环变换暴露更多 ILP。
- 后续分析应把“程序中存在的 ILP”与“编译器/硬件能否发现并调度的 ILP”分开；依赖、功能单元延迟、结构资源和分支都会限制可兑现并行度。

## 下一断点

PDF p.185 / 书内 p.157 下半页开始 §3.2 `Basic Compiler Techniques for Exposing ILP`，下一完整正文页为 PDF p.186 / 书内 p.158。
