---
name: knowledge-chapter-2-cache-bandwidth-optimizations
description: 第2章 §2.2 高级 cache 优化 3–5：cache 流水化、nonblocking cache 与 multibanked cache 的带宽和延迟权衡。
metadata:
  node_type: memory
  type: reference
  modified: 2026-08-19
---

# 第2章：提高 Cache Bandwidth 的高级优化

> 来源：`books/计算机体系结构量化研究方法.pdf`，第5版，第2章 §2.2。
> 本批核心范围：PDF p.110–114 / 书内 p.82–86 的优化 3–5；在同页优化 6 标题前停止。
> OCR 页面源字符数：约 13.7k（本轮以 Tesseract `eng`、PSM 3 对既有约 140 dpi 页面图像提取，并以 `wc -m` 统计为 13,707；不同分辨率或 OCR 参数会有小幅差异）。首尾页还包含优化 2 收尾和优化 6 开头，因此本记忆只吸收优化 3–5，批次仍低于 80,000 字符上限。

## 优化三：Pipelined Cache Access

- cache 访问流水化允许一次 L1 hit 跨多个时钟周期完成，以增加同时在途的访问数，从而获得更短的处理器时钟周期和更高吞吐带宽；代价是单次 hit latency 增大。
- 书中历史示例：Pentium 的 I-cache 访问约 1 周期，Pentium Pro 到 Pentium III 为 2 周期，Pentium 4 与当时 Core i7 为 4 周期。这些数字是历史实现参数，不应直接用于现代处理器。
- 更深的 cache pipeline 会增加分支误预测代价，并拉长 load 发出到数据可用之间的周期数；另一方面，它更容易容纳高关联度 cache 的 tag/data 访问延迟。
- 设计时必须区分 **latency** 与 **bandwidth**：流水化主要提高单位时间可接受/完成的访问数量，并不让单次访问更快（PDF p.110 / 书内 p.82）。

## 优化四：Nonblocking Cache

### 基本机制

- 在乱序处理器中，data cache miss 不必让整个处理器停顿。nonblocking/lockup-free cache 允许 miss 未完成时继续服务其他命中，即 **hit under miss**。
- 更复杂的实现允许多个 miss 重叠，即 **hit under multiple misses** 或 **miss under miss**。只有下层 cache/内存能够并行处理多个请求时，这种能力才有价值。
- 高性能处理器通常支持多种 nonblocking 行为；书中 ARM Cortex-A8 示例只在 L2 提供有限支持（PDF p.111 / 书内 p.83）。

### 性能证据

- 早期 8 KB cache、14-cycle miss penalty 研究中，允许一个 hit under miss 使 SPECINT92/SPECFP92 的有效 miss penalty 分别降低约 20%/30%。
- 基于单核 Intel Core i7 风格三级 cache 和 SPEC2006 的更新研究中，一个 hit under miss 使 integer/FP 的 data-cache access latency 平均降低约 9%/12.5%；第二个 hit 提升到约 10%/16%，再扩大到 64 个的边际收益很小。
- CPI 研究结果：integer 程序在 1 个和 64 个 hit-under-miss 下分别降低约 7% 和 12.7%；FP 程序分别降低约 12.7% 和 17.8%（PDF p.111–113 / 书内 p.83–85）。

### 与关联度的比较

32 KB D-cache 示例假定 L2 miss penalty 为 10 cycles：

```text
FP direct-mapped stall = 0.052 × 10 = 0.52
FP two-way stall       = 0.049 × 10 = 0.49
two-way/direct ratio   = 0.49 / 0.52 ≈ 94%

Integer direct-mapped stall = 0.035 × 10 = 0.35
Integer two-way stall       = 0.032 × 10 = 0.32
two-way/direct ratio        = 0.32 / 0.35 ≈ 91%
```

- FP 中，one-hit-under-miss 将访问延迟降至 blocking cache 的约 87.5%，优于只把 direct-mapped 改为 two-way 的约 94%。
- Integer 中，one-hit-under-miss 的约 9% 改善与 two-way 关联度的约 9% 改善大致相当。
- 结论依赖工作负载的 miss 行为，不能把 nonblocking 与关联度的胜负推广到所有程序（PDF p.111–112 / 书内 p.83–84）。

### 评价困难与在途请求数量

- nonblocking cache 下，单个 miss 不一定造成 stall；有效 miss penalty 应计算处理器真正停顿且无法与其他执行重叠的时间，而不是简单累加所有 miss latency。
- 支持多少 outstanding miss 取决于 miss stream 的时空局部性、下层带宽、各级 cache 的在途请求容量和内存延迟。低层要支持 N 个在途 miss，上层至少也要能承载相应请求。
- 例题：16 GB/s、64-byte block 的内存每秒最多传输 `16e9 / 64 = 250M` 个 block；36 ns latency 下维持峰值带宽需要约 `250e6 × 36e-9 = 9` 个独立在途请求。若约一半请求因地址冲突暂时不能发出，为保持 9 个可执行请求，支持规模约需翻倍到 18。
- 这体现带宽-延迟乘积：高带宽、高延迟内存需要足够多的独立在途请求才能填满数据通路（PDF p.112–113 / 书内 p.84–85）。

## 优化五：Multibanked Cache

- 将单体 cache 划分为多个可独立访问的 bank，可同时服务多个落在不同 bank 的请求。
- 书中示例：ARM Cortex-A8 L2 支持 1–4 个 bank；Intel Core i7 L1 有 4 个 bank，以支持每周期最多两个内存访问，L2 有 8 个 bank。这些是书中对应时代的实现数据。
- banking 的收益取决于访问是否均匀分散，地址到 bank 的映射决定冲突概率。
- 简单的 sequential interleaving 按 block address 对 bank 数取模：四 bank 时，`bank = block_address mod 4`。连续 block 分散到相邻 bank，适合连续访问模式。
- 原文明确指出多 bank 也能降低 cache 和 DRAM 功耗；一种合理的机制解释是只激活目标 bank，但这是基于 bank 组织的推论，指定页面未展开该机制（PDF p.113–114 / 书内 p.85–86）。

## 综合结论

- cache 流水化用更高单次 latency 换更高请求吞吐；nonblocking 用乱序执行和多个在途请求隐藏 miss；banking 用物理并行增加同时访问能力。
- 三者解决的都是 bandwidth/overlap 问题，但瓶颈位置不同：流水级、未决 miss 状态、bank 冲突与下层带宽都可能限制收益。
- 评价时应使用 execution time/CPI 和真实非重叠 stall，而不能仅比较静态 AMAT 或峰值带宽。

## 下一断点

PDF p.114 / 书内 p.86 的优化 6：`Critical Word First and Early Restart to Reduce Miss Penalty`。
