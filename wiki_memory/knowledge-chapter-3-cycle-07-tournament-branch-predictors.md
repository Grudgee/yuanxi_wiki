---
name: knowledge-chapter-3-cycle-07-tournament-branch-predictors
description: 第3章本轮周期4，记录局部、全局与锦标赛分支预测器的组合。
---

# 知识点摘要

- tournament predictor 同时维护局部和全局预测，并用选择器按分支近期表现决定采用哪一路。
- 局部预测可用每分支历史模式，global predictor 捕获跨分支相关；组合器让不同类型的分支使用更合适的预测源。
- Core i7 示例采用分级预测思路：快速小型一级预测满足周期约束，较大的后备预测提高准确率，并另设间接跳转与返回地址预测结构。

# 关键细节

- 周期：本轮周期 4/16。
- 来源范围：PDF p.193–195 上半页 / 书内 p.165–167 上半页。
- OCR 源字符数：5,544（小于 80,000）。
- 下一断点：PDF p.195 / 书内 p.167，§3.4 `Overcoming Data Hazards with Dynamic Scheduling` 标题处。

# 原文引用

- 文档：《计算机体系结构：量化研究方法》扫描版 PDF。
- 版本/日期：未知。
- 位置：第3章 §3.3，PDF p.193–195 / 书内 p.165–167，至 §3.4 标题前。
- 依据：原文比较 local 2-bit、correlating 与 tournament predictor，并描述局部历史表、饱和计数器和 Core i7 的多预测源组合。

# 适用条件与例外

- 原文图中的 SPEC89/SPEC CPU2006 结果不可直接外推到其他程序或现代实现。
- Intel 公开结构信息有限，原文明确将其描述为基于可见资料的高层组织。

# 关联章节

- §3.3 advanced branch prediction；§3.9 instruction delivery。

# 待核验问题

- 图3.4部分坐标文字 OCR 模糊，未转录逐点数值。
