---
name: knowledge-chapter-2-compiler-cache-optimizations
description: 第2章 §2.2 高级 cache 优化 8：用 loop interchange 和 blocking 改善空间/时间局部性并降低 cache miss。
metadata:
  node_type: memory
  type: reference
  modified: 2026-08-19
---

# 第2章：编译器驱动的 Cache Miss 优化

> 来源：`books/计算机体系结构量化研究方法.pdf`，第5版，第2章 §2.2 优化 8。
> 核心范围：PDF p.116–118 / 书内 p.88–90；PDF p.119 / 书内 p.91 仅用于核验优化 9 的入口。
> OCR 页面源字符数约 8.1k（Tesseract `eng`、PSM 3，对既有约 140 dpi 页面图像提取；PDF p.116–119 以 `wc -m` 统计为 8,118）。低于 80,000 字符上限。

## 编译器优化的定位

- 前七项优化主要改变硬件；优化 8 通过变换程序访问顺序，在不改变 cache 硬件的情况下减少 instruction/data miss。
- 关键原则不是减少指令数量，而是让访问顺序更符合 cache block 布局，并在数据被替换前尽可能重复使用。
- 变换必须保持程序语义和数据依赖；只有不存在阻止重排的依赖关系时，编译器才能安全应用（语义约束是编译器变换的一般前提，指定页面以可重排示例说明）。

## Loop Interchange

### 原理

- 多维数组通常按 row-major 或 column-major 连续布局；嵌套循环的内外层顺序若与布局不匹配，会以大 stride 跨 block 访问。
- 交换循环嵌套次序，使最内层循环沿内存连续方向移动，可在离开一个 cache block 前使用其中更多元素，从而改善空间局部性。
- 书中 row-major 二维数组示例中，原循环以约 100-word stride 访问；交换循环后按相邻 word 前进。
- 该优化不改变执行的核心计算或指令总量，主要减少 cache miss，因此性能收益来自 memory behavior 而非计算量减少（PDF p.116–117 / 书内 p.88–89）。

### 适用条件

- 数组工作集不能完全驻留 cache 时收益更明显；若全部数据已命中，重排的 miss-rate 收益有限。
- 需要确认循环迭代间无阻止交换的依赖，并考虑语言数组布局。

## Blocking / Tiling

### 为什么仅交换循环不够

- 矩阵运算常同时按行和列访问不同数组；无论选择 row-major 还是 column-major，总有一部分访问方向与布局正交。
- loop interchange 只能改善某一个访问方向，无法同时为所有矩阵提供良好局部性。

### 机制

- blocking 不一次处理完整行/列，而是把迭代空间切成 `B × B` 子矩阵，在切换到下一 tile 前尽可能复用当前 tile 数据。
- `B` 是 blocking factor，应使活跃子块及必要的行/列数据能够同时驻留目标 cache。
- 它同时利用空间局部性和时间局部性：书中矩阵乘示例里，`y` 的访问主要受益于空间局部性，`z` 的重复访问主要受益于时间局部性（PDF p.117–118 / 书内 p.89–90）。

### 量化效果

只考虑 capacity miss/内存 word 访问，未 blocking 的最坏情况约为：

```text
2N^3 + N^2
```

使用 `B × B` blocking 后约为：

```text
2N^3 / B + N^2
```

主导项因此约改善 `B` 倍。实际收益还受 conflict miss、cache 关联度、block size、替换策略、边界 tile 和指令开销影响（PDF p.117–118 / 书内 p.89–90）。

### 跨层次复用

- blocking 不只适用于 cache；若选择更小的 block 使其驻留寄存器，也可减少 load/store，帮助寄存器分配。
- 不同层次可使用多级 tiling：寄存器 tile、L1 tile、L2/L3 tile。指定页面只明确说明可把小 block 放入寄存器，多级 tiling 是该原则的推广。
- 对以矩阵为主要数据结构的应用，cache blocking 是获得良好 cache-based processor 性能的关键技术，第4章 §4.8 会继续应用（PDF p.118 / 书内 p.90）。

## 设计结论与限制

- loop interchange 主要修复访问顺序与物理布局不匹配；blocking 主要缩小活跃工作集并提高跨迭代复用。
- 二者收益依赖数据布局、cache 容量和依赖关系；不能仅凭源码循环形状判断，需结合 miss/CPI 或执行时间测量。
- 扫描页代码 OCR 存在变量和循环上界识别噪声，本记忆保留变换原理与公式，不把 OCR 代码当作可直接编译的准确转录。

## 下一断点

PDF p.119 / 书内 p.91 的优化 9：`Hardware Prefetching of Instructions and Data to Reduce Miss Penalty or Miss Rate`。
