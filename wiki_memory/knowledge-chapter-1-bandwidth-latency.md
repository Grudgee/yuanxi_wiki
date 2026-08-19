---
name: knowledge-chapter-1-bandwidth-latency
description: 第1章 §1.4 带宽相对延迟的技术趋势、跨技术比较和设计启示
metadata: 
  node_type: memory
  originSessionId: 74b02bc4-a492-4dcf-b44c-d04cf9991274
  modified: 2026-08-11T06:20:12.710Z
---

# 知识点摘要

- §1.4 的核心观察是：微处理器和网络的性能主要由带宽区分，因此两者带宽增益约为 10,000–25,000 倍、延迟改善约为 30–80 倍；存储器和磁盘更看重容量，带宽增益约为 300–1200 倍，而延迟只改善约 6–8 倍。
- 跨技术的经验法则是，带宽增长大致为延迟改善的平方；计算机设计者应按这一趋势规划系统，而不能把带宽和延迟当成同速指标。
- 图 1.9 以 log-log 图展示微处理器、网络、存储器和磁盘的带宽—延迟里程碑；图 1.10 给出各技术的代表性产品/介质及容量、带宽、延迟等数据。

# 关键细节

- 微处理器里程碑（图 1.10）从 1982 年 16-bit bus、Intel 80286，到 2010 年四核 Turbo；示例带宽从 2 MIPS 增至 50,000 MIPS，延迟从 320 ns 降至 4 ns。
- DRAM 模块示例从 1980 年 16-bit、0.06 Mbits/DRAM chip、225 ns，到 2010 年 64-bit、2048 Mbits/chip、37 ns；带宽由 13 MB/s 增至 16,000 MB/s。
- 局域网络示例从 1978 Ethernet（10 Mbit/s、3000 μs）到 2010 100 Gigabit Ethernet（100,000 Mbit/s、100 μs）。
- 硬盘示例从 1983 年 3600 RPM、0.6 MB/s、48.3 ms，到 2010 年 15,000 RPM、204 MB/s、3.6 ms；此处容量通常比性能更重要。

# Why / How

- Why：系统瓶颈不能只看单次操作延迟；当连续数据流或吞吐量主导工作负载时，带宽增长对系统价值更大。
- How：比较技术时分别记录容量、带宽和延迟，并使用相对改善或 log-log 图观察趋势；在系统规划中按带宽约随延迟改善平方增长的经验法则配置路径、缓冲和并行度。

# 原文引用

- 文档：计算机体系结构量化研究方法.pdf
- 版本/日期：第5版；日期未知
- 位置：PDF p.47 / 书内 p.19 / §1.4 “Trends in Technology”；PDF p.48 / 书内 p.20 / Figure 1.10
- 依据：“Performance is the primary differentiator for microprocessors and networks, so they have seen the greatest gains: 10,000–25,000x in bandwidth and 30–80X in latency.”
- 依据：“A simple rule of thumb is that bandwidth grows by at least the square of the improvement in latency.”

# 适用条件与例外

- 这些数字是图表所列的历史里程碑和近似趋势，不是对任意新器件或任意工作负载的保证。
- 容量优先适用于文中所述存储器和磁盘语境；微处理器和网络则主要按性能（带宽/吞吐及延迟）评价。
- 图表数据的单位和产品名称应与图 1.10 一起解释，不能把 MIPS、MB/s、μs、ns、ms 混作同一指标。

# 关联章节

- 第1章 §1.5 “Trends in Power and Energy in Integrated Circuits”
- 第2–5章（存储器层次、并行性及缓存等交叉主题）

# 待核验问题

- 无
