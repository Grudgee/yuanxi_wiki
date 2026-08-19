---
name: knowledge-chapter-2-prefetch-optimizations
description: 第2章 §2.2 高级 cache 优化 9–10：硬件与编译器预取、nonfaulting prefetch、时机/带宽/功耗权衡及十项优化总结。
metadata:
  node_type: memory
  type: reference
  modified: 2026-08-19
---

# 第2章：硬件与编译器 Cache Prefetch

> 来源：`books/计算机体系结构量化研究方法.pdf`，第5版，第2章 §2.2 优化 9–10 与总结。
> 核心范围：PDF p.119–124 / 书内 p.91–96；PDF p.124 下半页开始 §2.3，只用于确认下一断点。
> OCR 页面源字符数约 14.0k（Tesseract `eng`、PSM 3，对既有约 140 dpi 页面图像提取；`wc -m` 为 14,005）。低于 80,000 字符上限。

## 优化九：Hardware Prefetching

### 基本机制

- prefetch 在处理器正式请求前把指令或数据取入 cache 或更快的外部 stream buffer，以计算/执行重叠内存访问。
- 常见指令预取策略在 miss 时同时请求需求 block 和下一个连续 block；需求 block 进入 I-cache，预取 block 进入 instruction stream buffer。
- 后续请求若命中 stream buffer，则取消原 cache 请求、直接读取 buffer，并继续发出下一次预取，形成顺序流。
- 数据也可使用多个 stream buffer；书中研究中 8 个 stream buffer 能捕获两个 64 KB four-way I/D cache 的约 50%–70% miss（PDF p.119 / 书内 p.91）。

### 收益与风险

- Core i7 示例支持向 L1/L2 硬件预取，常见模式为 next-line；更激进策略曾使部分应用变慢，说明 prefetch 不是越多越好。
- prefetch 借用原本空闲的 memory bandwidth；若与 demand miss 争夺带宽、污染 cache 或逐出有用数据，会降低性能。
- 有效且被使用的预取对功耗影响可能很小；无用预取和错误替换会浪费传输与 cache 能量，显著恶化功耗。
- Figure 2.10 只展示受益较大的 SPEC2000 子集，未展示的 15 个程序加速低于 15%；不能把筛选后的高收益图当作整体平均（PDF p.120 / 书内 p.92）。

## 优化十：Compiler-Controlled Prefetching

### 指令语义

- **Register prefetch** 把值装入寄存器；**cache prefetch** 只把数据带入 cache。
- prefetch 可为 faulting 或 nonfaulting。最有效的 prefetch 对程序语义不可见：不改变寄存器/内存可见状态，也不因虚拟地址或保护问题抛出异常。
- 书中采用 nonfaulting/nonbinding cache prefetch；若地址会导致异常，该操作成为 no-op。因此编译器可在循环尾发出可能越过数组边界的预取，但仅在体系结构保证 nonfaulting 时安全（PDF p.120–122 / 书内 p.92–94）。

### 时机与结构条件

- prefetch 必须足够早，使数据在真实使用前到达；太晚只能缩短部分 penalty，太早则增加被替换或无用的风险。
- cache 必须能够在 prefetch 未完成时继续服务请求，通常需要 nonblocking cache。
- miss penalty 较小时可通过少量 loop unrolling 调度未来迭代的预取；penalty 大时需要更深的 unrolling 或 software pipelining。
- prefetch 指令本身有执行开销，编译器应集中于高概率 miss，避免为本来会 hit 的访问增加指令和带宽（PDF p.121 / 书内 p.93）。

### 数组例题

书中 8 KB direct-mapped、16-byte block、write-back/write-allocate 示例：

- `a` 为连续写入，8-byte double 每个 16-byte block 容纳两个元素，因此约 150 miss。
- `b` 的访问缺少空间局部性但具有时间局部性，约 101 miss。
- 原程序合计约 251 data-cache miss。
- 提前 7 次迭代预取后，仅剩约 19 个未预取 miss，以 400 条 prefetch 指令避免约 232 miss。
- 原循环约 `2100 + 251×100 = 27,200` cycles；两个预取循环含剩余 miss 约 `2000 + 2400 = 4,400` cycles。在假设预取完全重叠时，加速约 `27,200/4,400 = 6.2`。

该结果依赖非常理想的重叠、无 conflict/capacity miss、足够带宽和固定 100-cycle miss penalty，不能直接当作一般收益（PDF p.121–123 / 书内 p.93–95）。

### 指针结构

- 编译器预取不局限数组。递归数据结构研究中，10 个程序有一半提升 4%–31%，其余保持在原性能约 2% 范围内。
- 关键问题是预取地址是否已在 cache，以及是否足够早到达；pointer chasing 的依赖链使提前量更难获得（PDF p.123 / 书内 p.95）。

## 十项优化总结

- cache 优化针对 hit time、bandwidth、miss penalty、miss rate 和 power，但改善一项往往伤害另一项。
- 小型 cache 与 way prediction 主要改善 hit time；流水化、nonblocking 和 banking 提高 bandwidth；critical word 与 write merging 降低 penalty；编译器布局变换降低 miss rate；软硬件 prefetch 可能降低 miss rate，也可能只缩短 penalty。
- prefetch 最依赖 nonblocking、空闲带宽和准确性，并可能增加功耗；硬件/软件复杂度在十项技术中较高。
- Figure 2.11 的复杂度 0–3 是主观等级，用于相对比较，不是可移植的定量成本模型（PDF p.123–124 / 书内 p.95–96）。

## 下一断点

PDF p.124 / 书内 p.96 下半页开始 §2.3 `Memory Technology and Optimizations`；下一完整正文页为 PDF p.125 / 书内 p.97。
