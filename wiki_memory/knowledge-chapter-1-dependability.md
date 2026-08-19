---
name: knowledge-chapter-1-dependability
description: 第1章 §1.7 Dependability 的服务状态、可靠性、可用性、故障率、FIT、MTTF、MTTR、MTBF 与冗余计算。
metadata:
  node_type: memory
  originSessionId: 74b02bc4-a492-4dcf-b44c-d04cf9991274
  modified: 2026-08-12
---

# 知识点摘要

- §1.7 将 dependability 定义为系统提供其宣传服务的能力；服务状态分为按 SLA 正确完成（service accomplishment）和服务中断（service interruption）。
- Reliability 是系统或部件在给定时间内持续正确工作的概率；MTTF（Mean Time To Failure）是其常用度量。
- FIT（Failures In Time）表示每十亿小时的失效次数，恒定失效率下 `MTTF = 1 / failure rate`。
- Availability 是系统可用时间所占比例；非冗余系统 `Availability = MTTF / (MTTF + MTTR)`，且 `MTBF = MTTF + MTTR`。
- 对独立且服从指数分布的串联系统，总失效率为各模块失效率之和，因此总 MTTF 会因组件数量增加而降低。
- 冗余是应对失效的主要方法，可采用时间冗余（重试/重复执行）或资源冗余（备用组件接替）。

# 关键细节

- 服务状态转换由 failure 或 restoration 引起；failure 使交付服务偏离 SLA，restoration 使服务恢复到 SLA。
- 磁盘子系统示例（PDF p.62–63 / 书内 p.34–35）：10 个 MTTF=1,000,000 h 的磁盘、controller=500,000 h、power supply=200,000 h、fan=200,000 h、cable=1,000,000 h。独立指数模型下：
  - `λsystem = 10/1,000,000 + 1/500,000 + 1/200,000 + 1/200,000 + 1/1,000,000 = 23/1,000,000 failures/hour`；
  - 即约 `23,000 FIT`；
  - `MTTFsystem = 1/λ ≈ 43,500 h`，略低于 5 年。
- 两个独立冗余电源的近似示例（PDF p.63 / 书内 p.35）：`MTTFpair ≈ MTTFpower^2 / (2 × MTTRpower)`。当单电源 MTTF=200,000 h、MTTR=24 h 时，冗余对 MTTF 约 `830,000,000 h`，约为单电源的 4150 倍。
- MTTF 与 MTTR 的作用不同：提高 MTTF 减少失效发生频率，降低 MTTR 缩短服务中断；业务可用性不能只由 MTTF 推断。

# 原文引用

- 文档：`/home/mt/公共的/yuanxi_cc/wiki_files/books/计算机体系结构量化研究方法.pdf`
- 版本/日期：第5版；日期未知
- 位置：PDF p.62–63 / 书内 p.34–35 / §1.7 “Dependability”
- 依据：“Service accomplishment, where the service is delivered as specified”
- 依据：“Service interruption, where the delivered service is different from the SLA”
- 依据：“Transitions between these two states are caused by failures … or restorations …”
- 依据：“The primary way to cope with failure is redundancy, either in time … or in resources …”
- 依据：书中给出非冗余模块可用性公式 `Module availability = MTTF/(MTTF+MTTR)`，并用磁盘子系统和冗余电源例子计算系统可靠性。

# 适用条件与例外

- 串联系统失效率相加以及冗余电源公式均依赖独立性、恒定失效率/指数寿命等近似；相关故障、共因故障、检测延迟和切换失败会使结果偏乐观。
- Availability 的简单公式以平均 MTTF/MTTR 描述修复过程，不显式包含计划维护、后勤等待、降级服务或冗余切换时间。
- 磁盘和电源例子的参数是教材示例，不是一般硬件的保证值。
- 本次继续读取了 §1.7 后续至 PDF p.63 / 书内 p.35；同批还读取 PDF p.64–68 / 书内 p.36–40 的 §1.8 性能度量、benchmark 与 server benchmark 内容，但其主题应另建独立记忆，避免把 §1.7 与 §1.8 混合。

# 关联章节

- 第1章 §1.6 “Trends in Cost”
- 第1章 §1.8 “Measuring Performance”
- §1.8 后续：server benchmarks、功耗 benchmark 与并行性能

# 待核验问题

- §1.7：无（已覆盖 Figure 1.18 之后的 “The challenge is…” 及其后续至章节结束的主要内容）。
- §1.8 的 benchmark 细节尚未写入本文件；下一学习入口为 PDF p.69 / 书内 p.41。
