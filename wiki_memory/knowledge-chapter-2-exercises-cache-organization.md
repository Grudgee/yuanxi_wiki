---
name: knowledge-chapter-2-exercises-cache-organization
description: 第2章习题周期四，cache 组织、way prediction、banking、miss handling 与写缓冲。
metadata:
  node_type: memory
  source: books/计算机体系结构量化研究方法.pdf
  modified: 2026-08-19
---

# 第2章习题周期四：Cache Organization

> 范围：PDF p.164–166 / 书内 p.136–138；本轮归纳解题方法，未逐题提交数值答案。

- CACTI 题要求把 capacity、associativity、bank 数和 block size 转换为 access/cycle time，再与 miss rate、miss penalty 合成 AMAT。
- Way prediction 题同时比较正确预测、错误预测和主存 miss 三条路径；若 cycle time 也改变，还需把 cycles 转为真实时间或整体执行时间。
- Way selection/serial tag-data access 用更长 hit time 换较低能耗；评价指标应包括 performance、energy/access 和 energy-delay，而非只看命中率。
- Pipelined cache 提高吞吐但不必降低单次 latency；banked cache 的收益取决于访问是否均匀分散，以及 bank conflict 概率。
- Nonblocking cache、critical-word-first、early restart 和 merging write buffer 分别利用 miss-level parallelism、优先返回所需字、提前恢复处理器和合并同一 block 写入。
- 写缓冲宽度应匹配下一级接口传输粒度；blocking/nonblocking L1 会改变同时需要容纳的 pending writes 数量。

## 下一周期

PDF p.167 / 书内 p.139：DRAM timing/bandwidth/power、Flash 与虚拟化习题。
