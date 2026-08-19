---
name: knowledge-chapter-2-fallacies-pitfalls
description: 第2章 §2.7 的 cache 测量、带宽与虚拟化谬误和陷阱。
metadata:
  node_type: memory
  source: books/计算机体系结构量化研究方法.pdf
  modified: 2026-08-19
---

# 第2章：Fallacies and Pitfalls

> 来源：`books/计算机体系结构量化研究方法.pdf`，第5版，第2章 §2.7。
> 本批学习范围：PDF p.153–156；书内 p.125–128。PDF p.157 / 书内 p.129 开始 §2.8。

## 1. 不能用一个程序预测另一个程序

- 不同程序的 instruction/data locality 差异可以跨越多个数量级；同一 cache 容量下，三个 SPEC2000 程序的数据 MPKI 和指令 MPKI 相差巨大。
- integer 与 floating-point 程序也不存在稳定、可直接套用的相对 miss-rate 规律；数据库等商业工作负载在大容量 L2 中仍可能保持显著 miss rate。
- cache 结论必须绑定具体 workload、input、cache 组织和统计口径，不能从少量 benchmark 泛化为普遍规律。

## 2. 必须模拟足够长的执行区间

- 小 trace 很难预测大 cache，因为短执行片段可能尚未覆盖完整 working set。
- 程序 locality 会随执行阶段变化；warm-up 后的稳定阶段、初始化阶段和不同算法阶段可能有完全不同的 miss 行为。
- locality 还依赖输入。书中 `perl` 的五种输入在前 1.9 billion instructions 看似接近，但完整执行后的平均 instruction misses 可从约 2.4 到 7.9 per 1000 references。
- 因此不能只凭早期 running average 宣称 cache 性能已收敛；应覆盖代表性 phase，或者使用经过验证的 phase sampling。

## 3. 低平均延迟不等于高带宽

- Cache 能显著降低 average memory access latency，但对流式、大工作集或高并行访问，数据仍需频繁到达主存。
- 如果 cache 后方没有足够的 memory channels、bank parallelism 和传输带宽，应用仍会受 bandwidth 限制。
- 体系结构必须同时设计 latency hierarchy 与 sustained bandwidth，不能把高 hit rate 当作主存带宽不足的替代方案。

## 4. 在不可虚拟化 ISA 上实现 VMM

- 若读取/修改敏感机器状态的指令在 user mode 不 trap，VMM 就无法可靠地截获并模拟 guest OS 的行为。
- 早期 80x86 有两类问题指令：一类可在 user mode 读取敏感控制状态而不 trap；另一类隐含按最高 privilege level 做 segmentation protection check，guest OS 降权运行时语义不正确。
- 书中列出 18 条相关 80x86 指令，包括 SGDT、SIDT、SMSW、PUSHF/POPF、LAR/LSL、VERR/VERW、far CALL/RET/JMP、INT 和 segment-register 操作。
- 早期 80x86 TLB 缺少 Process ID tags，VMM 与 guest OS 切换地址空间时通常需要 flush TLB，进一步增加虚拟化开销。
- Intel VT-x/AMD SVM 通过新的 guest mode、敏感寄存器 shadow/mask、受控 trap 和 nested page tables 改善问题；nested page tables 可避免软件维护 shadow page tables。

## 方法论结论

- cache 模拟必须同时说明 workload、input、trace 长度、phase、warm-up、cache 参数和 miss-rate 分母。
- 体系结构功能若未来可能被更高权限软件虚拟化，所有敏感操作都应具有可捕获、可模拟且语义明确的路径。
- 性能优化要区分 latency、bandwidth 和 virtualization overhead，三者不能用单一 miss-rate 指标替代。

## 下一断点

PDF p.157 / 书内 p.129：进入 §2.8 `Concluding Remarks: Looking Ahead`。
