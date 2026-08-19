---
name: knowledge-chapter-2-exercises-system-evaluation
description: 第2章习题周期六，虚拟化、OOO 延迟隐藏和 VTune 层次测量。
metadata:
  node_type: memory
  source: books/计算机体系结构量化研究方法.pdf
  modified: 2026-08-19
---

# 第2章习题周期六：System Evaluation

> 范围：PDF p.170–172 / 书内 p.142–144；第2章案例与习题已通读，数值题未逐题求解。

- 嵌套虚拟化要求 L0 VMM 向 L1 VMM 再提供可虚拟化的 VT-x/AMD-V 接口，并正确组合多级控制状态、interrupt 和 nested address translation。
- Intel VT-x 与 AMD-V 比较应关注 guest/host transition、nested paging、TLB tagging 和 I/O virtualization，而不是只比较指令集名称。
- IOMMU 把 device DMA address 转换并限制到 VM 被授权的物理页，既提升 direct device assignment 性能，也维持隔离和安全。
- Alpha 21264 题把 issue queue/register mapper 看作“隐藏不可预测 memory latency 的硬件”；用其面积换更大 L1 并不必然获益，因为容量不能替代 OOO 对独立指令的调度能力。
- VTune 实验用 8 KB、128 KB、4 MB、32 MB 工作集和 64-byte stride 跨越 L1/L2/L3/主存，结合各级 MPKI 与 IPC/CPI 推断 cache size、miss penalty 和 latency hiding。
- 比较 8 KB 与 128 KB 工作集时，即使 L1 misses 大增，OOO 核的 CPI 可能增加较少；差值反映 memory-level parallelism、speculation 和 independent instruction overlap。

## 下一断点

第2章案例与习题通读完成至 PDF p.172 / 书内 p.144。PDF p.173–175 为第3章目录/引导页；下一正文入口是 PDF p.176 / 书内 p.148 的 §3.1 `Instruction-Level Parallelism: Concepts and Challenges`。
