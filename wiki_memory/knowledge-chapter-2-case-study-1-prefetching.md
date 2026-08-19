---
name: knowledge-chapter-2-case-study-1-prefetching
description: 第2章案例1周期二，矩阵转置的硬件和软件预取方法。
metadata:
  node_type: memory
  source: books/计算机体系结构量化研究方法.pdf
  modified: 2026-08-19
---

# 第2章案例1周期二：Prefetching

> 范围：PDF p.161 / 书内 p.133 上半页。

- 最简单硬件 prefetcher 只跟随 unit-stride miss stream；矩阵列访问属于 non-unit stride，需要 stride detector 或软件显式计算未来地址。
- 稳态需要的 outstanding prefetch 数约等于“预取延迟 ÷ 每次内层迭代时间”，并向上取整；这是用并发请求覆盖 latency 的基本关系。
- 预取必须足够早，否则不能隐藏 miss penalty；过早又会延长数据在 cache 中的驻留时间，增加 eviction、pollution 和资源占用。
- blocked transpose 与 software prefetch 可以组合：blocking 控制 working set，prefetch 隐藏仍然存在的首次 block miss。
- 性能估算必须同时计入 prefetch 指令开销、可支持的 outstanding misses、L2 吞吐、cache pollution 和无用预取。

## 下一周期

PDF p.161 / 书内 p.133 下半页：Case Study 2 的 stride-based memory-system probing。
