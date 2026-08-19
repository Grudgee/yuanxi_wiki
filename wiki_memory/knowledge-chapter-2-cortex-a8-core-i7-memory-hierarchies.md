---
name: knowledge-chapter-2-cortex-a8-core-i7-memory-hierarchies
description: 第2章 §2.6 的 ARM Cortex-A8 与 Intel Core i7 内存层次结构、访问路径和性能分析。
metadata:
  node_type: memory
  source: books/计算机体系结构量化研究方法.pdf
  modified: 2026-08-19
---

# 第2章：Cortex-A8 与 Core i7 内存层次实例

> 来源：`books/计算机体系结构量化研究方法.pdf`，第5版，第2章 §2.6。
> 本批学习范围：PDF p.141–152；书内 p.113–124。PDF p.153 / 书内 p.125 开始 §2.7。

## 1. Cortex-A8 内存层次

- Cortex-A8 是 ARMv7 IP core，可发射每周期两条指令，目标频率最高约 1 GHz。hard core 面向特定工艺优化，性能和面积较好；soft core 更易移植和修改。
- L1 为分离的 I-cache/D-cache，各可配置为 16 KB 或 32 KB、four-way set associative，使用 way prediction 和 random replacement，目标是 single-cycle hit 与 one-cycle load-to-use。
- 可选 L2 为 unified、eight-way set associative，容量 128 KB–1 MB，可分成 1–4 个 bank；L1/L2 block size 均为 64 bytes。
- L1 使用 virtually indexed、physically tagged；L2 使用 physically indexed、physically tagged。32 KB D-cache 配 4 KB page 时可能出现 synonym/alias，硬件在 miss 时检测并避免同一物理页形成不一致副本。
- I-TLB 与 D-TLB 各 32 项、fully associative，支持 4 KB、16 KB、64 KB、1 MB、16 MB page，round-robin replacement；TLB miss 由硬件 page-table walker 处理。

## 2. Cortex-A8 性能观察

- 书中使用 32 KB L1、1 MB eight-way L2 和 Minnespec integer benchmarks。Minnespec 缩小输入会显著改变 cache footprint，不能与完整 SPEC2000 的绝对 miss rate 直接比较。
- I-cache miss rate 对多数程序接近 0，全部低于 1%；D-cache miss 更依赖应用工作集，`mcf` 是明显的 cache buster。
- 1 GHz Cortex-A8 示例中 L1 miss penalty 为 11 cycles，L2 miss penalty 为 60 cycles。即使 L2 global miss rate 较低，因 penalty 超过 L1 的五倍，仍可能贡献可观的平均内存停顿。
- 评估层次结构不能只看 miss rate，还要计算 `miss rate × miss penalty`，并明确 local/global miss rate 的分母。

## 3. Core i7 的层次参数

- 书中 i7 为四核、out-of-order、每核最高每周期执行四条 x86 指令，并支持 simultaneous multithreading；三个 DDR3-1066 memory channels 的峰值带宽略高于 25 GB/s。
- 地址为 48-bit virtual、36-bit physical。L1 I-TLB 128 项、L1 D-TLB 64 项，均 four-way；共享 L2 TLB 512 项、four-way。L1 TLB hit 约 1 cycle，L2 TLB hit 约 6 cycles，页表访问可能需要数百 cycles。
- 每核 L1 为 32 KB I-cache/32 KB D-cache，I-cache four-way、D-cache eight-way，hit latency 4 cycles；每核 L2 为 256 KB、eight-way、10 cycles；共享 L3 按每核 2 MB、16-way、约 35 cycles。
- 三级 cache 都为 write-back、64-byte block、nonblocking，并允许多个 outstanding writes。L3 inclusive of L1/L2；L1 write miss 不分配 cache line，而进入 merging write buffer。

## 4. Core i7 的取指与数据访问路径

- L1 I-cache 为 virtually indexed、physically tagged。虚拟地址的 index 与 page offset 可先访问 cache，同时 I-TLB 产生 physical page number，再用 physical tag 比较，以重叠转换和 cache 访问。
- 书中 I-cache 需要 7-bit index 和 6-bit block offset，合计 13 bits，超过 4 KB page offset 的 12 bits，因此有 1 个 index bit 来自 virtual page number，可能产生 instruction alias；硬件必须保证副本一致。
- I-TLB miss 先查 512-entry L2 TLB；再 miss 时由硬件遍历 page table；最坏情况下触发 page fault，由操作系统从磁盘调页。
- L1 miss 后用 physical address 查每核 L2，再查共享 L3，最后进入片上 memory controller 和 DRAM。L2 返回速率约 8 bytes/cycle，L3 返回速率约 16 bytes/cycle。
- 3.3 GHz CPU 配 DDR3-1600 的示例中，确认 L3 miss 约 35 CPU cycles，DRAM 到首个 16 bytes 约 100 cycles，总 critical-word miss penalty 约 135 cycles；剩余 64-byte block 继续填充约需 45 cycles。
- L2/L1 间和 L3/内存方向的 merging write buffer 保存 dirty evictions，并可被新 miss snoop；若所需 block 仍在 buffer 中，可直接由 buffer 满足。
- L1 D-cache eight-way 的 index 完全落在 page offset 中，因此没有 I-cache 的 alias 问题。store miss 不 write-allocate，而先进入 write buffer；store hit 也需等到指令确认不会被错误推测后才更新 cache。
- i7 对 L1/L2 做硬件 prefetch，通常预取 next block；限制预取层级可避免无用请求一直到达高代价主存。

## 5. Core i7 性能统计的解释

- L1 I-cache 在 SPECCPU2006 中 miss rate 约 0.1%–1.8%，平均略高于 0.4%，与有效的 instruction prefetch 有关。
- L1 D-cache 的 miss rate 不能脱离分母解释：以 retired/graduated loads 为分母，平均约 9.5%；以所有 L1 D-cache references（含 speculative accesses、prefetch 和不产生 miss 的 writes）为分母，平均约 5.9%，两者相差约 1.6 倍。
- 平均 L2 data miss rate 约为全部 L1 data references 的 4%，L3 约 1%。由于主存 miss 超过 100 cycles，L3 把 miss rate 从约 4% 降到约 1% 对 CPI 至关重要。
- 若没有 L3，并假设约一半指令是 load/store，书中估算 L2 miss 可能给 CPI 增加约 2 cycles；因此大容量低层 cache 即使 hit latency 更高，仍能显著降低总停顿。

## 设计要点

- Cortex-A8 追求简单、低延迟和可配置；Core i7 通过更深层次、更多并行未决请求、更复杂 TLB 与预取来支撑高发射率和乱序执行。
- VIPT 的价值是重叠 TLB 与 L1 访问，但 cache index 超出 page offset 时会产生 alias 风险；容量、关联度和 page size 必须联合设计。
- 任何 miss-rate 图都必须先确认分母、是否计入 speculative/prefetch/write，以及是 local 还是 global miss rate。

## 下一断点

PDF p.153 / 书内 p.125：进入 §2.7 `Fallacies and Pitfalls`，从“用一个程序预测另一个程序的 cache performance”开始。
