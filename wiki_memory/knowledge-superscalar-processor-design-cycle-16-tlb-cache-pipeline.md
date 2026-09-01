---
name: knowledge-superscalar-processor-design-cycle-16-tlb-cache-pipeline
description: 超标量处理器设计第16个学习周期，记录TLB和Cache加入流水线后的协同。
---

# 知识点摘要

- 直接每次访问页表会使地址转换延迟过高，因此处理器使用 TLB 缓存最近使用的 PTE。
- TLB 命中时可快速得到物理页号和属性；TLB 缺失时需要查页表，若页表项无效则进一步触发 Page Fault。
- Cache 与 TLB 加入流水线后，取指和访存路径要同时处理地址转换、权限检查、cacheable 属性和 Cache 命中判定。

# 关键细节

- 周期：第 16 个学习周期。
- 源文件：`books/超标量处理器设计.pdf`。
- 已读范围：第 3 章 §3.4 起点，书内 p.68 以后。
- 批次规模：估计 5,000–12,000 源字符，低于 80,000 字符监督阈值。

# 原文依据

- 位置：第 3章“加入 TLB 和 Cache”，§3.4.1 TLB 的设计起点。
- 依据：正文说明借鉴 Cache 思想缓存最近使用的页表项，并把 MMU/TLB/Cache 放入处理器访问路径。

# 适用条件与限制

- 当前周期只覆盖 §3.4 起点，后续还需继续学习 TLB 与 I-Cache/D-Cache 的具体流水线时序。

# 下一断点

- 继续第 3 章 §3.4，细化 TLB、I-Cache、D-Cache 与流水线阶段的配合。
