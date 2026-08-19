---
name: knowledge-chapter-2-case-study-1-blocking
description: 第2章案例1周期一，矩阵转置的 blocking、cache 容量与关联度分析。
metadata:
  node_type: memory
  source: books/计算机体系结构量化研究方法.pdf
  modified: 2026-08-19
---

# 第2章案例1周期一：Blocking Matrix Transpose

> 范围：PDF p.159–160 / 书内 p.131–132。

- 普通矩阵转置让一个数组按行访问、另一个按列访问；loop interchange 只能交换哪个数组具有连续访问，不能同时改善两者。
- Blocking 把矩阵切成 `B×B` tile，使输入块和输出块在处理期间同时留在 L1，利用两边的时间/空间局部性。
- 最小 cache 容量应覆盖两个活动 tile，并考虑 cache line 粒度、write-allocate 带来的读入，以及循环中的其他活跃数据。
- blocked/unblocked miss 数量比较应分 compulsory、capacity、conflict miss；理想 fully associative 假设只能消除 conflict，不能消除工作集过大的 capacity miss。
- “最小关联度”问题用于检查两个数组因地址对齐映射到相同 set 的冲突；不能只根据总容量判断稳定性能。
- 实机结果还受 prefetcher、TLB、write buffer、编译器向量化和 cache replacement 影响，因此理论 miss count 与墙钟时间不一定严格对应。

## 下一周期

PDF p.161 / 书内 p.133：硬件与软件 prefetching 的提前量和 outstanding request 约束。
