---
name: knowledge-chapter-2-exercises-memory-virtualization
description: 第2章习题周期五，DRAM、Flash、可靠性与虚拟化题目的分析框架。
metadata:
  node_type: memory
  source: books/计算机体系结构量化研究方法.pdf
  modified: 2026-08-19
---

# 第2章习题周期五：DRAM、Flash 与虚拟化

> 范围：PDF p.167–169 / 书内 p.139–141；本轮归纳解题方法，未逐题提交数值答案。

- DDR 题先用 `clock = transfers/second ÷ 2`，再组合 `tRCD + CL + burst`；open-row hit 可省 activation，close-page/open-page policy 的平均值取决于 row locality。
- DIMM 峰值带宽由 transfer rate、总数据总线宽度和 channel 数决定；ECC 位增加芯片与传输开销，但有效数据带宽只计算数据位。
- 多核带宽配置题从 `instruction rate × misses/instruction × bytes/miss` 得平均带宽，再乘峰均比并除以单通道可持续带宽。
- DRAM power 题区分 activation、read/write、standby；低密度/高密度芯片改变芯片数量、每次激活范围和总待机功耗。
- DRAM hibernation 到 Flash 必须比较迁移能量与节省的 idle power，break-even time 为迁移总能量除以功率差。
- Flash 题需考虑 block erase/rewrite、读写非对称、耐久性和 wear leveling；ECC/Chipkill 题则比较校验开销、可纠正故障粒度和系统规模。
- 虚拟化性能题按 privileged instructions、TLB misses、traps、I/O events 分解 VMM invocation；比较 native、full virtualization、paravirtualization 时要区分事件频率与单次处理成本。

## 下一周期

PDF p.170 / 书内 p.142：嵌套虚拟化、IOMMU、OOO 延迟隐藏与 VTune 实测习题。
