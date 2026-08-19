---
name: knowledge-chapter-1-section-1-6-cost-foundations
description: 第1章§1.6前段关于时间、产量、商品化及集成电路成本组成的可追溯知识。
metadata: 
  node_type: memory
  status: superseded
  superseded_by: knowledge-chapter-1-cost-trends.md
  originSessionId: 74b02bc4-a492-4dcf-b44c-d04cf9991274
  modified: 2026-08-11T07:15:04.448Z
---

# 知识点摘要

> 状态：本文件是 §1.6 前段的阶段性旧稿，内容已完整并入 `knowledge-chapter-1-cost-trends.md`。保留用于溯源，不再作为当前索引入口。

- §1.6标题为“Trends in Cost”，从PDF p.55/书内p.27开始；已核验前一节§1.5止于PDF p.54/书内p.26。
- 计算机架构决策应考虑成本；本节先把成本变化归因于时间、产量与商品化（commoditization）。
- 学习曲线（learning curve）指制造成本随时间下降；文中以yield定义为“survive the testing procedure”的制造器件百分比，并给出经验规则：产量翻倍时成本约减半。
- 产量提高还会提高采购与制造效率、摊薄每台产品需分摊的开发成本。商品化产品由多个供应商以大批量制造，标准DRAM、Flash memory、磁盘、显示器和键盘均被列举为例子。
- §1.6随后转向“Cost of an Integrated Circuit”：完整封装集成电路成本包含裸片、测试、封装/最终测试，且要考虑最终测试良率；本批仅学至PDF p.57的图1.14，未学习下一页的每晶圆裸片数公式推导。
- 图1.13所示Intel Core i7裸片为13.9 mm × 13.6 mm（257 mm²），45 nm工艺；图1.14给出其floorplan，包括四个核心、shared L3 cache、memory controller、QPI与Misc I/O。

# 关键细节

- 完整成本关系：`Cost of integrated circuit = (Cost of die + Cost of testing + Cost of packaging and final test) / Final test yield`。
- 本批页内给出裸片成本关系：`Cost of die = Cost of wafer / (Dies per wafer × Die yield)`。
- 这两个关系说明：仅以晶圆价格或几何裸片数不能得到已封装、通过最终测试的单颗集成电路成本。

# Why / How

- Why：成本敏感的设计日益重要；架构功能是否加入设计，取决于其成本是否合理。裸片尺寸和良率会进入单颗裸片成本，测试、封装和最终测试又决定最终交付成本。
- How：评估时先把成本拆成裸片、测试、封装/最终测试，并以最终测试良率调整；对于裸片成本，后续需用每晶圆裸片数和die yield共同计算，不能只看晶圆面积。

# 原文引用

- 文档：`/home/mt/公共的/yuanxi_cc/wiki_files/books/计算机体系结构量化研究方法.pdf`
- 版本/日期：第5版；日期未知。
- 位置：PDF p.55/书内p.27/§1.6 “Trends in Cost”；PDF p.56/书内p.28/“Cost of an Integrated Circuit”；PDF p.57/书内p.29/图1.13、图1.14。
- 依据（短引）：PDF p.55/书内p.27/§1.6：“The underlying principle that drives costs down is the learning curve”。
- 依据（短引）：PDF p.56/书内p.28/“Cost of an Integrated Circuit”：“Cost of die = Cost of wafer / (Dies per wafer × Die yield)”。
- 依据（短引）：PDF p.57/书内p.29/图1.13：“The dimensions are 13.9 mm by 13.6 mm (257 mm²) in a 45 nm process.”

# 适用条件与例外

- “产量翻倍、成本约减半”是本书在学习曲线语境下的经验法则，不应视为跨产品、工艺和产量范围的精确公式。
- 本批没有核验die yield模型、每晶圆裸片数公式的完整解释或数值示例；它们从PDF p.58/书内p.30继续。
- 页码均由实际页面图像核验；PDF物理页和书内印刷页须同时保留。

# 关联章节

- 第1章 §1.5 “Trends in Power and Energy in Integrated Circuits”
- 第1章 §1.6 “Trends in Cost”后续：每晶圆裸片数与die yield

# 待核验问题

- PDF p.58/书内p.30的每晶圆裸片数公式及边缘修正解释。
- 后续die yield经验模型、测试/封装成本的数值示例与§1.6结束边界。
