---
name: knowledge-chapter-6-cycle-03-network-architecture
description: 第6章自动学习周期3/15，记录WSC网络层次、带宽超额订阅、存储组织和故障规模效应。
---

# 知识点摘要

- WSC网络像存储层次一样分层：服务器连接机架交换机，再连接更高层聚合交换结构；越过机架的带宽通常低于机架内总注入带宽。
- Oversubscription降低成本，却使任务放置、数据复制和通信局部性成为软件必须显式处理的体系结构约束。
- 大规模系统中组件故障是常态，可靠性依赖冗余数据、任务重试、快速检测和服务级容错，而不是假设所有硬件持续正常。

# 关键结论

- WSC网络不是透明互连；其拓扑、带宽和故障模式会直接塑造编程模型和数据布局。
- 商品化服务器的单节点成本优势只有在网络、存储和容错软件能够规模化时才成立。

# 关键细节

- 批次周期：3/15。
- 范围：§6.3；PDF p.469 OCR第20行至 p.474 OCR第37行 / 书内 p.441–446。
- OCR源字符数：10639，严格小于80000。

# 边界依据

- 从§6.3标题开始，完整覆盖网络层次、超额订阅、数据存储和故障频率讨论。
- 在 PDF p.474 OCR第38行 `Physical Infrastructure and Costs of Warehouse-Scale Computers` 标题前停止。

# 待核验问题

- 网络图中的链路速率、端口数和超额订阅比例需要在引用具体数值时重新核图。

# 下一精确断点

- PDF p.474 / 书内 p.446，OCR第38行，§6.4 `Physical Infrastructure and Costs of Warehouse-Scale Computers` 标题。
