---
name: knowledge-chapter-5-cycle-07-symmetric-memory-performance-basics
description: 自动学习周期 10/30，记录《计算机体系结构：量化研究方法》对称共享存储性能基础。
---

# 知识点摘要

- 共享存储性能需把单处理器miss与通信引起的coherence miss分开，并区分true sharing与false sharing。
- false sharing由同块不同字的写共享触发，可通过数据布局或块粒度分析识别。

# 关键细节

- 批次周期：10/30（严格串行）。
- 节号/主题：§5.3，对称共享存储性能基础。
- 源范围起点：PDF p.394 / 书内 p.366，OCR第9行。
- 源范围终点：PDF p.397 / 书内 p.369，OCR第48行（含）。
- 下一锚点（不含）：PDF p.397 / 书内 p.369，OCR第49行，Performance Measurements of the Commercial Workload标题。
- OCR源字符数：9078（按实际分段文本逐字符统计，严格小于80,000）。
- OCR方法：扫描页转140 DPI JPEG，以Tesseract eng逐页识别；标题、图表讨论与章节边界同时对照页面图像。

# 核心概念

- 共享存储性能需把单处理器miss与通信引起的coherence miss分开，并区分true sharing与false sharing。

# 关键结论

- false sharing由同块不同字的写共享触发，可通过数据布局或块粒度分析识别。

# 边界依据

- 起点采用PDF p.394 OCR第9行的已核验语义锚点；终点完整覆盖当前主题至PDF p.397 OCR第48行。
- 在下一独立标题、完整主题段或章节硬边界PDF p.397 OCR第49行前停止；下一周期从同一锚点开始，因而不切断连续正文或图表讨论。

# 原文引用

- 文档：books/计算机体系结构量化研究方法.pdf
- 版本/日期：第5版；具体印次未知。
- 位置：§5.3；PDF p.394 / 书内 p.366，OCR第9行至PDF p.397 / 书内 p.369，OCR第48行。
- 依据：上述范围的实际OCR与关键页面图像；不以常识替代扫描原文。

# 适用条件与例外

- 体系结构、处理器和性能数据按本书写作时点理解，不外推为当前产品规格。
- 例题时间步、表格数字及true/false sharing分类需引用时核图。

# 关联章节

- §5.3；与周期9和周期11按锚点连续。

# 待核验问题

- 例题时间步、表格数字及true/false sharing分类需引用时核图。

# 下一精确断点

- PDF p.397 / 书内 p.369，OCR第49行，Performance Measurements of the Commercial Workload标题。
