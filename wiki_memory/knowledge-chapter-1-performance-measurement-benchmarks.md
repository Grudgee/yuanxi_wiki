---
name: knowledge-chapter-1-performance-measurement-benchmarks
description: 第1章 §1.8 Measuring, Reporting, and Summarizing Performance：事务处理基准、性能报告、SPECratio 与几何平均。
metadata:
  node_type: memory
  modified: 2026-08-12
---

# 知识点摘要

- 事务处理（TP）benchmark 衡量系统处理数据库访问与更新事务的能力；典型场景包括航空预订、银行 ATM 与复杂数据库/决策支持。
- TPC 基准以 transactions per second 衡量性能，并要求响应时间约束；只有满足响应时间限制的事务吞吐量才计入性能。
- TPC benchmark 还应模拟真实系统：包含多个用户、较大数据库，并计入系统成本；TPC 对定价策略和结果验证设有规则，以便进行 cost-performance 比较。
- 性能报告的指导原则是可复现性（reproducibility）：其他实验者应能据报告重现结果。报告需说明硬件、软件、编译器及 flags、baseline/optimized tuning，并同时给出实际执行时间及表格/图形结果。
- 由多个 benchmark 汇总性能时，简单算术平均可能被运行时间较长的程序支配；加权平均需要选择权重，而权重选择可能产生争议。
- SPECratio 用参考计算机执行时间除以被测计算机执行时间，将执行时间归一化为无量纲比值；比值越大表示性能越好。
- 对一组 SPECratio，几何平均适合汇总比值，因为“比值的几何平均 = 几何平均的比值”，且参考计算机选择不会影响比较结果。

# 关键细节

## TPC benchmark

- TPC-A 于 1985 年发布，之后被多个 benchmark 替代/增强；TPC-C 模拟复杂查询环境；TPC-H 模拟决策支持；TPC-E 模拟在线事务处理（OLTP）工作负载中的经纪公司客户账户。
- 所有 TPC benchmark 以 transactions per second 报告吞吐量，同时要求 response time limit；较真实的模型还包括多用户、较大数据库及完整系统成本。

## 性能报告

- SPEC 报告需要给出 compiler 与 compiler flags，以及 baseline 和 optimized results 的发布信息。
- TPC 报告还包含实际性能时间、benchmark 审计与成本信息；这些报告可用于比较真实系统成本和 cost-performance。

## SPECratio 与 Figure 1.17

- 定义：
  - `SPECratio = Execution time_reference / Execution time_test`
  - 它等于被测计算机相对于参考计算机的性能比。
- 书中 Figure 1.17（PDF p.71 / 书内 p.43）用 Ultra 5、AMD Opteron、Intel Itanium 2 的 SPEC CPU2006 执行时间和 SPECratio 展示：参考机执行时间的选择不影响相对性能。
- 示例中 AMD Opteron 与 Itanium 2 相对于 Ultra 5 的几何平均 SPECratio 分别为 20.86 和 27.12；二者比值 `27.12 / 20.86 ≈ 1.30`，与各 benchmark SPECratio 比值的几何平均相同。
- 若 `SPECratio_A / SPECratio_B = 1.25`，则：
  - `SPECratio_A / SPECratio_B = Execution time_B / Execution time_A`
  - `= Performance_A / Performance_B`
  - 因此计算机 A 比 B 快 1.25 倍。
- SPECratio 是比值而非绝对执行时间，因此不能对其使用算术平均；几何平均没有单位。

## 几何平均性质

设第 i 个程序的 `sample_i` 为该程序的 SPECratio，则：

`Geometric mean = (∏(sample_i), i=1..n)^(1/n)`

书中列出的两个性质：

1. 比值的几何平均等于几何平均的比值。
2. 性能比值的几何平均之比等于所有 benchmark 性能比值的几何平均，因此参考计算机选择无关紧要。

# 原文引用

- 文档：`/home/mt/公共的/yuanxi_cc/wiki_files/books/计算机体系结构量化研究方法.pdf`
- 版本/日期：第5版；日期未知
- 位置：PDF p.69–72 / 书内 p.41–44 / §1.8 “Measuring, Reporting, and Summarizing Performance”
- 依据：“Transaction-processing (TP) benchmarks measure the ability of a system to handle transactions that consist of database accesses and updates.”
- 依据：“All the TPC benchmarks measure performance in transactions per second.”
- 依据：“The guiding principle of reporting performance measurements should be reproducibility”
- 依据：“Rather than pick weights, we could normalize execution times to a reference computer”
- 依据：`SPECratio = Execution time_reference / Execution time_test`
- 依据：书中说明几何平均使“the choice of the reference computer is irrelevant”。
- 图表：Figure 1.17 “SPEC CPU2006 execution times ... and SPECratios ...”位于 PDF p.71 / 书内 p.43。

# 适用条件与例外

- TPC 的 transactions-per-second 只有在满足规定响应时间限制时才有意义；不同 TPC 基准的工作负载和规则不能随意横向混合。
- SPECratio 适用于相对性能比较；它不是绝对执行时间，算术平均 SPECratio 通常不具备所需的比例不变性。
- 几何平均默认 benchmark 集合具有代表性；benchmark suite 若不能代表目标应用，数学汇总仍不能保证实际应用预测准确。
- 本记忆覆盖 §1.8 至其自然小节“Summarizing Performance Results”结束：PDF p.69–72 / 书内 p.41–44。PDF p.73 / 书内 p.45 已进入 §1.9 “Quantitative Principles of Computer Design”，未纳入本文件。

# 关联章节

- 第1章 §1.8 前文：response time、throughput、CPU time、benchmark suites 与 SPEC/TPC 服务器 benchmark（已在前批预读）
- 第1章 §1.9 “Quantitative Principles of Computer Design”
- 第1章 §1.9 后续：parallelism、locality、common case 与 Amdahl’s Law

# 待核验问题

- 无（本批 §1.8 的性能报告与汇总小节已覆盖至 §1.9 起始边界）。
