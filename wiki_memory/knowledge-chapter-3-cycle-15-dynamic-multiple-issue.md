---
name: knowledge-chapter-3-cycle-15-dynamic-multiple-issue
description: 第3章本轮周期12，记录动态多发射与推测的组合及同周期依赖检查。
---

# 知识点摘要

- 现代动态多发射把分支预测、register renaming、reservation station、ROB 和多功能部件结合，在每周期发射多条并保持按序提交。
- 同一 issue bundle 内可能存在依赖；实现可把一个周期拆成顺序子步骤，或构建更宽的组合逻辑同时检查依赖和资源。
- 资源不足时 bundle 必须拆分，并保持原程序顺序；宽度增加使依赖组合与资源分配复杂度近似快速增长。

# 关键细节

- 周期：本轮周期 12/16。
- 来源范围：PDF p.225–230 上半页 / 书内 p.197–202 上半页。
- OCR 源字符数：14,910（小于 80,000）。
- 下一断点：PDF p.230 / 书内 p.202，§3.9 `Advanced Techniques for Instruction Delivery and Speculation` 标题处。

# 原文引用

- 文档：《计算机体系结构：量化研究方法》扫描版 PDF。
- 版本/日期：未知。
- 位置：第3章 §3.8，PDF p.225–230 / 书内 p.197–202，至 §3.9 标题前。
- 依据：图3.17–3.20给出双发射推测处理器、成对指令发射步骤以及有无 speculation 的循环时序对比。

# 适用条件与例外

- 实际吞吐受发射宽度、ROB/保留站容量、功能部件和分支准确率共同限制。
- 表中时序建立在原文给定 latency 与双发射限制上。

# 关联章节

- §3.6 ROB speculation；§3.7 multiple issue；§3.9 instruction delivery。

# 待核验问题

- 图3.19–3.20部分周期数字 OCR 密集，未逐格转录。
