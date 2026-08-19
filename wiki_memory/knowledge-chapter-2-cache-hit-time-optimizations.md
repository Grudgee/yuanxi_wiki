---
name: knowledge-chapter-2-cache-hit-time-optimizations
description: 第2章 §2.2 高级 cache 优化 1–2：小型简单 L1、关联度权衡、CACTI 评估、way prediction 与 way selection。
metadata:
  node_type: memory
  type: reference
  modified: 2026-08-18
---

# 第2章：降低 Cache Hit Time 的高级优化

> 来源：`books/计算机体系结构量化研究方法.pdf`，第5版，第2章 §2.2。
> 本批核心学习范围：PDF p.107–110 / 书内 p.79–82；PDF p.106 / 书内 p.78 用于承接五类优化框架。
> OCR 源字符数：13,371（PDF p.106–110），低于单批 80,000 字符上限。

## 十项高级优化的五类目标

1. 降低 hit time：小而简单的 L1、way prediction；通常也降低功耗。
2. 提高 cache bandwidth：流水化、多 bank、nonblocking cache；功耗影响各异。
3. 降低 miss penalty：critical word first、合并 write buffer；功耗影响较小。
4. 降低 miss rate：编译器优化；通常同步改善功耗。
5. 以并行性降低 miss penalty 或 miss rate：硬件/编译器预取；无用预取通常增加功耗。

总体趋势是后续优化的硬件复杂度更高，部分还依赖复杂编译器技术（PDF p.106–107 / 书内 p.78–79）。

## 优化一：小而简单的 L1 Cache

### 原理

- L1 同时承受高时钟频率和功耗预算压力，因此容量不能随片上总 cache 容量同比增长。
- set-associative hit 的关键路径通常包括：用 index 访问 tag、比较 tag、控制多路选择器选出正确 data way。
- direct-mapped cache 可让 tag 比较与数据传输重叠；较低关联度通常只激活更少 cache line，因此 hit time 和功耗更低。
- 近代设计常选择提高关联度而非继续增大 L1；原因包括 virtually indexed L1 的容量约束、多线程导致的 conflict miss，以及 cache 访问已占多个周期后关联度延迟可能不再决定周期时间（PDF p.107–108 / 书内 p.79–80）。

### CACTI 的用途与限制

- CACTI 用工艺节点、容量、关联度、读写端口等参数估算 CMOS cache 的访问时间和能量，可在流片前缩小设计搜索空间。
- 书中对应技术条件下，two-way 约比 four-way 快 1.2 倍，four-way 约比 eight-way 快 1.4 倍；这些数字依赖工艺、布局和模型假设，不能跨实现直接套用。
- 关联度增加需要并行读取更多 tag 和 data，能量开销可能显著；图 2.4 中 eight-way 的能量惩罚尤其明显（PDF p.107–109 / 书内 p.79–81）。

### 32 KB L1 例题

设 two-way L1 hit time 为 1，L2 miss penalty 为较快 L1 hit time 的 15 倍：

```text
AMAT_2way = 1 + 0.038 × 15 = 1.57

AMAT_4way = 1.4 + 0.037 × 10 ≈ 1.77
```

注：PDF 原页明确印刷 `1 + 0.038 × 15 = 1.38`，但按公式计算应为 `1.57`，属于教材原页的算术或排版错误，不是 OCR 误识别。原页还印刷 `15 / 1.4 = 10.1`，实际约为 `10.71`，随后按 10 简化；当前 four-way 结果 `1.4 + 0.037 × 10 = 1.77` 与教材采用的简化值一致。例题结论仍是 four-way 的较小 miss-rate 改善不足以抵消 1.4 倍 hit-time，因而在该假设下 two-way AMAT 更低。现代 cache 常被流水化，关联度对时钟周期的真实影响仍需结合流水级评估（PDF p.108 / 书内 p.80）。

## 优化二：Way Prediction

### 机制

- way prediction 为每个 block 保存预测位，提前选择下一次访问最可能命中的 way。
- 预测正确时，只访问/比较预测 way，可获得接近 direct-mapped 的快速 hit；预测错误时，下一周期检查其他 way、更新预测器，并付出额外周期。
- 书中模拟结果：two-way 预测准确率超过 90%，four-way 约 80%；I-cache 通常高于 D-cache。two-way 若预测路径至少快 10%，通常即可降低平均访问时间。
- MIPS R10000 较早采用该技术；ARM Cortex-A8 在 four-way cache 上使用。高速处理器必须能把误预测额外代价控制在约 1 周期，否则收益下降（PDF p.109 / 书内 p.81）。

### Way Selection 的功耗取舍

- 扩展形式 way selection 用预测位决定实际只读哪个 way，不只是提前设置 mux，因此预测正确时节能更多；预测错误时必须重新执行整个访问，延迟更大，也不利于 cache 流水化。
- SPEC95 估计中，four-way way selection 使 I-cache/D-cache 平均访问时间变为 1.04/1.13 倍，但功耗降至普通 four-way 的 0.28/0.35。
- 若 I-cache 和 D-cache 分别占处理器功耗 25% 和 15%，且 D-cache 访问量约为 I-cache 一半：总功耗约降至 0.88，性能约降至 0.90，性能/焦耳比约 `0.90 / 0.88 = 1.02`，仅略有改善。
- 因而 way selection 更适合功耗优先而非性能优先的处理器（PDF p.110 / 书内 p.82）。

## 设计结论

- 不能仅凭更低 miss rate 选择更高关联度；必须联合比较 hit time、miss penalty、功耗、TLB/虚拟索引约束和流水化结构。
- way prediction 以预测正确率换取快速常见路径；way selection 进一步以误预测延迟换取功耗下降。
- 所有 CACTI 和历史处理器数字都是特定技术条件下的证据，适用于方法论和相对权衡，不是现代实现的固定参数。

## 下一断点

PDF p.110 / 书内 p.82 的第三项优化 `Pipelined Cache Access to Increase Cache Bandwidth`；其正文从该页下半部开始，下一批应从这里继续。
