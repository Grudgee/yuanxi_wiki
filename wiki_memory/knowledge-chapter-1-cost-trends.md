---
name: knowledge-chapter-1-cost-trends
description: 第1章 §1.6 Trends in Cost 的时间、规模、商品化与集成电路成本模型
metadata: 
  node_type: memory
  originSessionId: 74b02bc4-a492-4dcf-b44c-d04cf9991274
  modified: 2026-08-11T07:33:57.790Z
---

# 知识点摘要

- §1.6 说明成本意识对计算机体系结构决策的重要性，并讨论时间、产量、商品化和集成电路制造成本。
- 学习曲线使制造成本随时间下降；其关键指标是 yield，即通过测试的制造器件比例。产量翻倍通常可使成本约下降一半。
- 产量增加会加快学习曲线、降低采购和制造成本，并摊薄开发成本；商品化则通过多供应商竞争和规模效应降低成本。
- 集成电路成本必须区分晶圆、裸片、测试、封装和最终测试。良品率越高、每片晶圆上的裸片越多，单个裸片成本越低。
- 裸片面积不仅减少几何意义上的每片晶圆裸片数，还会显著降低 die yield；因此大裸片受到双重成本惩罚。
- 冗余可用来提高商品存储器的有效 yield：DRAM/SRAM 可加入冗余存储单元，以容纳一定数量的缺陷。

# 关键细节

- 集成电路成本公式：`Cost of integrated circuit = (Cost of die + Cost of testing + Cost of packaging and final test) / Final test yield`。
- 裸片成本公式：`Cost of die = Cost of wafer / (Dies per wafer × Die yield)`。
- 每片晶圆裸片数近似公式：`Dies per wafer = [π × (Wafer diameter / 2)^2 / Die area] − [π × Wafer diameter / √(2 × Die area)]`。
- 第二项用于修正圆形晶圆边缘不能放置完整矩形裸片的问题。
- 裸片良品率（Bose–Einstein 经验模型）：`Die yield = Wafer yield × 1 / (1 + defects per unit area × Die area)^N`。其中 wafer yield 是整片晶圆通过测试的比例，缺陷密度按单位面积计，`N` 是 process-complexity factor。
- 文中说明：2010 年缺陷密度通常为 0.1–0.3 defects/cm²；40 nm 工艺示例约为 0.016–0.057 defects/cm²；2010 年 40 nm 工艺的 `N` 约为 11.5–15.5。示例默认 wafer yield 为 100%。
- 良品率示例（PDF p.59 / 书内 p.31）：在 `Wafer yield = 1`、缺陷密度 `0.031/cm²`、`N=13.5` 的前提下，Bose–Einstein 公式计算得 2.25 cm² 裸片 yield ≈ `0.402`（记录时取 `0.40`）；1.00 cm² 裸片 yield ≈ `0.663`（记录时取 `0.66`）。
- 同一示例在 300 mm 晶圆上估算：2.25 cm² 裸片约 270 个几何裸片、约 109 个良品；1.00 cm² 裸片约 640 个几何裸片、约 424 个良品。这里良品数由 `dies per wafer × die yield` 得出，并非几何裸片数。
- 图 1.13 展示 Intel Core i7 裸片，尺寸为 13.9 mm × 13.6 mm（257 mm²），45 nm 工艺。
- 图 1.14 展示 Core i7 floorplan：四个核心、共享 L3 cache、memory controller、Misc I/O、QPI，以及单核心的执行单元、乱序调度/提交、指令解码、L1/L2 cache 等模块。
- 图 1.15 示例：300 mm 晶圆包含 280 个完整 Sandy Bridge 裸片；每个裸片 20.7 mm × 10.5 mm，32 nm 工艺，裸片面积 216 mm²。
- 2010 年先进工艺的 300 mm（12-inch）晶圆加工成本约为 5,000–6,000 美元；示例取 5,500 美元。按面积粗略估计，1.00 cm² 裸片成本约 13 美元，而 2.25 cm² 裸片成本约 51 美元（约四倍）。
- 生产成本还包括裸片测试、封装和封装后的最终测试；高密度工艺若有四到六层金属层，mask 成本可超过 100 万美元，低产量时原型和调试成本也可能显著。

# Why / How

- Why：体系结构特性若不考虑制造成本、产量和良品率，可能无法形成具有竞争力的产品；大裸片同时减少每片晶圆的裸片数并降低良品率。
- How：先由晶圆直径与裸片面积估计几何裸片数，再用缺陷密度、裸片面积和工艺复杂度因子估算 die yield，最后用 `good dies per wafer = dies per wafer × die yield` 估算每片晶圆的良品数，并纳入测试、封装、最终测试和 mask/开发成本。

# 原文引用

- 文档：计算机体系结构量化研究方法.pdf
- 版本/日期：第5版；日期未知
- 位置：PDF p.59–60 / 书内 p.31–32 / §1.6 “Trends in Cost”；p.60 底部出现下一小节标题 “Cost versus Price”，本批在该边界停止
- 依据：“The underlying principle that drives costs down is the learning curve—manufacturing costs decrease over time.”（PDF p.55 / 书内 p.27）
- 依据：“This Bose–Einstein formula is an empirical model developed by looking at the yield of many manufacturing lines.”
- 依据：“Die yield = Wafer yield × 1/(1 + Defects per unit area × Die area)^N”
- 依据：“The number of dies per wafer is approximately the area of the wafer divided by the area of the die.”（PDF p.57 / 书内 p.29）
- 依据：“The dimensions are 13.9 mm by 13.6 mm (257 mm²) in a 45 nm process.”（PDF p.57 / 书内 p.29 / 图1.13）
- 依据：“The bottom line is the number of good dies per wafer, which comes from multiplying the number of dies per wafer by die yield to incorporate the effects of defects.”
- 依据：“Processing of a 300 mm (12-inch) diameter wafer in a leading-edge technology cost between $5000 and $6000 in 2010.”

# 适用条件与例外

- 每片晶圆裸片数公式是近似值；实际值受边缘布局、切割道、工艺规则和可用面积影响。
- yield 模型是经验模型；示例还假设 wafer yield 为 100%，缺陷随机分布，且 `N` 取决于工艺成熟度/复杂度，不能无条件外推到其他工艺。
- “产量翻倍成本减半”是文中给出的经验性描述，不是所有产品、工艺或产量区间的严格定律。
- 裸片成本公式不包含测试、封装和最终测试；完整集成电路成本还要除以最终测试良品率。
- 本批没有把 p.60 底部之后的 “Cost versus Price” 内容纳入；PDF p.61 / 书内 p.33 开始出现 §1.7 “Dependability”，需按下一批边界复核。

# 关联章节

- 第1章 §1.5 “Trends in Power and Energy in Integrated Circuits”
- 第1章 §1.7 “Dependability”

# 待核验问题

- Cost versus Price 小节的完整内容及其与制造/运营成本的区分尚未核验。
- §1.7 Dependability 的 MTTF、MTTR、MTBF 和 availability 尚未学习。
