---
name: knowledge-chapter-1-classes-of-computers
description: 第1章 §1.2 的服务器、集群/WSC、嵌入式计算机、并行性层次与 Flynn 分类。
metadata: 
  node_type: memory
  summary: 第1章 §1.2 的服务器、集群/WSC、嵌入式计算机及并行性分类。
  originSessionId: ea9d9683-f2fb-426d-8f4a-20026b18b9d1
  modified: 2026-08-11T02:46:05.258Z
---

# 知识点摘要

- 服务器在企业计算中替代传统大型机，首要特征是可用性、可扩展性和吞吐量；单次请求响应仍重要，但多数服务器以单位时间可处理请求数为关键指标。
- 集群由通过局域网连接的桌面机或服务器组成，每个节点运行自己的操作系统；最大规模的集群称为仓库级计算机（WSC），目标是可容纳数万台服务器。
- WSC 的核心约束是性价比和功耗；其规模要求利用廉价、冗余组件，并以软件层捕获和隔离故障。WSC 的可扩展性主要由本地网络连接计算机实现，而不是依靠集成式计算机硬件。
- 嵌入式计算机广泛存在于微波炉、洗衣机、网络交换机和汽车等设备中；性能范围从低成本 8/16 位处理器到可执行每秒数十亿指令的高端网络交换机处理器。设计目标通常是在最低价格下满足性能需求，而不是追求最高性能。
- 应用并行性分为数据级并行（DLP）和任务级并行（TLP）；硬件支持路径包括 ILP、向量/GPU、线程级并行和请求级并行。
- Flynn 分类：SISD、SIMD、MISD、MIMD。该分类是粗粒度模型，实际并行处理器可能是混合体。

# 关键细节

## Server（§1.2）

- 可靠性：银行 ATM、航空订票等服务器全天候运行；服务器故障的后果可能远大于桌面机单机故障。
- 可扩展性：要随服务需求或功能需求增长而扩展计算容量、内存、存储和 I/O 带宽。
- 吞吐量：整体性能以每分钟/每秒处理的事务或 Web 页面数衡量。
- 图 1.3 用每小时停机成本和不同可用性（1%、0.5%、0.1% downtime）量化不可用系统损失；表中示例包括 brokerage operations、信用卡授权、包裹运输、家庭购物、目录销售、航空订票、蜂窝服务、在线网络费用和 ATM 服务。

## Clusters/Warehouse-Scale Computers（§1.2）

- SaaS（搜索、社交网络、视频共享、多人游戏、在线购物等）的增长促进了集群计算。
- 每个节点拥有自己的 OS，节点通过网络通信；WSC 是最大规模的集群。
- 书中例子：一个 9000 万美元仓库约 80% 成本与电源和冷却相关；计算机及网络设备另需约 7000 万美元并且数年更换。价格/性能提升 10% 可节省约 700 万美元（按 7000 万美元设备成本计）。
- WSC 需要高可用性；以 Amazon.com 为例，书中用 2010 年第四季度约 130 亿美元销售额、约每小时 560 万美元平均收入说明服务中断的经济后果。此数字是书中历史案例，不应当作当前业务数据。
- Supercomputer 与 WSC 对比：超级计算机昂贵且成本可达数亿美元，强调浮点性能，运行可持续数周的计算密集批处理；WSC 更强调交互式应用、大规模存储、可靠性和高 Internet 带宽。

## Embedded Computers（§1.2）

- PMD 单独列类，因为 PMD 可运行外部开发软件；其他嵌入式设备的硬件和软件更受限。
- 价格是嵌入式市场设计关键因素；通常目标是以最低成本满足性能，而非以更高价格获得最高性能。
- 书中范围示例：8/16 位处理器可能低于 10 美分；执行每秒 1 亿条指令的 32 位微处理器低于 5 美元；网络交换机高端处理器约 100 美元且可执行每秒数十亿条指令。
- 作者指出本书许多设计、使用和性能内容适用于嵌入式处理器，但嵌入式数据驱动的定量设计与评估尚未充分扩展，因此将相关材料整理到附录 E；这一处理属于本版的组织选择。

## Parallelism classification（§1.2）

- DLP：同时存在许多可并行操作的数据项。
- TLP：创建可独立、基本并行运行的工作任务。
- 四种硬件利用方式：
  1. ILP：编译器辅助流水线，较高层次可用推测执行等。
  2. 向量架构与 GPU：以一条指令作用于一组数据。
  3. 线程级并行：数据级或任务级并行，依靠紧耦合硬件模型让并行线程交互。
  4. 请求级并行：程序员或 OS 指定的、基本解耦的任务之间并行。
- SISD 是单处理器、单指令流/单数据流；SIMD 以同一指令处理多数据流，适合 DLP；MISD 无商业多处理器实例；MIMD 各处理器取自己的指令、处理自己的数据，适合 TLP，通常比 SIMD 灵活但开销更大。松耦合 MIMD（集群/WSC）天然适合 RLP。

# 原文引用

- 文档：计算机体系结构：量化研究方法（第5版）
- 版本/日期：第5版；书中服务器/WSC 示例含 2010 年数据
- 位置：PDF p.35；书内 p.7；§1.2 Servers
- 依据：“For servers, different characteristics are important. First, availability is critical.”
- 位置：PDF p.35；书内 p.7；图1.3题注
- 依据：“Figure 1.3 Costs rounded to nearest $100,000 of an unavailable system are shown by analyzing the cost of downtime...”
- 位置：PDF p.36；书内 p.8；§1.2 Clusters/Warehouse-Scale Computers
- 依据：“Clusters are collections of desktop computers or servers connected by local area networks to act as a single larger computer.”
- 位置：PDF p.36；书内 p.8；§1.2 WSC
- 依据：“The largest of the clusters are called warehouse-scale computers (WSCs), in that they are designed so that tens of thousands of servers can act as one.”
- 位置：PDF p.36–37；书内 p.8–9；§1.2 Embedded Computers
- 依据：“Embedded computers are found in everyday machines: microwaves, washing machines, most networking switches, and all cars contain simple embedded microprocessors.”
- 位置：PDF p.37；书内 p.9；§1.2 Classes of Parallelism and Parallel Architectures
- 依据：“There are basically two kinds of parallelism in applications: 1. Data-Level Parallelism (DLP) ... 2. Task-Level Parallelism (TLP) ...”
- 位置：PDF p.37–38；书内 p.9–10；§1.2 Flynn taxonomy
- 依据：“This taxonomy is a coarse model, as many parallel processors are hybrids of the SISD, SIMD, and MIMD classes.”

# 适用条件与例外

- 双页码映射仅对本次已视觉核对的 PDF p.30–38 有依据；本批使用逐页页眉/页码核验，不沿用旧版 PDF 页码。
- WSC 成本、Amazon 销售额和停机损失均是教材中的历史示例，不能外推为当前数值。
- Flynn 分类不是对所有现代异构处理器的完整描述；书中明确称其为粗粒度模型。
- 嵌入式数据驱动的定量评估在本书正文中不充分，需关联附录 E（本地是否包含附录内容待后续核验）。

# 关联章节

- 第1章 §1.3 Defining Computer Architecture
- 第3章 Instruction-Level Parallelism
- 第4章 Data-Level Parallelism、向量和 GPU
- 第5章 Thread-Level Parallelism 和多处理器
- 第6章 Warehouse-Scale Computers
- 附录 E（嵌入式系统，待核验）

# 待核验问题

- §1.2 中图1.3的全部数字需在后续整理时逐项转录并核对表头。
- 本地 PDF 是否包含在线附录 E 及其物理页范围尚未核验。
