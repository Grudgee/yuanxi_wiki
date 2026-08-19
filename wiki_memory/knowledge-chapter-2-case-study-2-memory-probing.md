---
name: knowledge-chapter-2-case-study-2-memory-probing
description: 第2章案例2周期三，用工作集和 stride 实测 cache、TLB 与内存系统。
metadata:
  node_type: memory
  source: books/计算机体系结构量化研究方法.pdf
  modified: 2026-08-19
---

# 第2章案例2周期三：Memory-System Probing

> 范围：PDF p.161–164 / 书内 p.133–136。

- 测试程序对不同 array size 和 stride 构造指针链，并重复访问足够长时间；array size 控制 working set，stride 控制每条 cache line/page 的利用程度。
- 测量时先计时完整循环，再单独测循环开销并相减；输出 CSV，使用 log-log 图观察平台和跃迁。
- 小工作集平台对应 L1/L2 hit latency；工作集跨过 cache 容量后曲线抬升，可估 cache size 和 miss penalty；stride 跃迁可估 block size。
- 大 stride 跨 page 后可暴露 TLB capacity/miss penalty；更大工作集进入主存或 paging 区域，可估内存容量和 page-fault 时间。
- 地址映射、page coloring、预取器、NUMA、后台任务和动态频率会扰动曲线；教材建议单用户/低干扰运行，并指出 virtual address 不一定持续跟踪 physical address。
- 多进程并发版本可探索物理核与 SMT context、memory controller 数量和聚合带宽；instruction-cache 探测则需构造长度可控的简单指令序列。

## 下一周期

PDF p.164 / 书内 p.136 下半页：cache 组织、流水化、banking 与 write-buffer 习题。
