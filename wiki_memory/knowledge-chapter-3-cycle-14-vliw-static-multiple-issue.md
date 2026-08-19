---
name: knowledge-chapter-3-cycle-14-vliw-static-multiple-issue
description: 第3章本轮周期11，记录静态多发射、VLIW与软件流水权衡。
---

# 知识点摘要

- 多发射主要路线包括静态调度 superscalar、VLIW/EPIC 和动态调度 superscalar；差别集中在发射结构、hazard 检测与调度责任归属。
- VLIW 把可并行操作打包为固定或显式并行的长指令，编译器负责功能部件分配、依赖检查与空槽填充。
- loop unrolling 可形成长直线调度区；software pipelining 能跨迭代重叠操作，并降低纯展开造成的代码膨胀。

# 关键细节

- 周期：本轮周期 11/16。
- 来源范围：PDF p.220 下半页–224 / 书内 p.192 下半页–196。
- OCR 源字符数：12,413（小于 80,000）。
- 下一断点：PDF p.225 / 书内 p.197，§3.8 `Exploiting ILP Using Dynamic Scheduling, Multiple Issue, and Speculation`。

# 原文引用

- 文档：《计算机体系结构：量化研究方法》扫描版 PDF。
- 版本/日期：未知。
- 位置：第3章 §3.7，PDF p.220–224 / 书内 p.192–196。
- 依据：图3.15比较多发射路线，正文以展开循环构造 VLIW 包并讨论编码、代码尺寸与二进制兼容性。

# 适用条件与例外

- 静态包对功能部件数量和 latency 敏感，跨实现二进制兼容困难。
- 空操作槽、寄存器压力和分支会降低有效发射率。

# 关联章节

- §3.2 loop unrolling；Appendix H software pipelining；§3.8 dynamic multiple issue。

# 待核验问题

- 图3.16的部分指令槽 OCR 对齐不稳定，未保存逐周期包布局。
