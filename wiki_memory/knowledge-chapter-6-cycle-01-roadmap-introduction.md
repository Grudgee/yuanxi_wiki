---
name: knowledge-chapter-6-cycle-01-roadmap-introduction
description: 第6章自动学习周期1/15，记录仓库级计算机的章节路线、定义、规模特征及与HPC和传统数据中心的区别。
---

# 知识点摘要

- Warehouse-Scale Computer（WSC）把数万台服务器、网络、存储、供电、制冷和系统软件作为一台计算机共同设计，其主要并行来源是大量独立请求形成的 request-level parallelism。
- WSC 与 HPC 集群的目标不同：HPC偏向低延迟、紧耦合通信和定制高速网络；WSC偏向吞吐量、商品化硬件、服务级可用性和规模经济。
- WSC 也不同于传统企业数据中心：硬件和软件更同质、应用由运营者共同设计，目标是让整座仓库表现为统一系统，而不是隔离并整合异构业务。

# 关键结论

- WSC 的体系结构边界扩展到建筑级基础设施；服务器、网络、软件、供电和制冷的局部优化可能破坏系统级成本与能效。
- 低单机利用率并不意味着系统无效，它反映交互服务的波动、延迟约束和峰值容量要求。

# 关键细节

- 批次周期：1/15。
- 范围：第6章目录、标题页和 §6.1 导论；PDF p.458 OCR第1行至 p.464 OCR第46行 / 书内 p.430–436。
- OCR源字符数：15854，严格小于80000。

# 边界依据

- 从第6章目录页开始，完整覆盖标题页、WSC定义、规模经济、与HPC及传统数据中心的比较。
- 在 PDF p.464 OCR第47行 `Programming Models and Workloads for Warehouse-Scale Computers` 标题前停止。

# 待核验问题

- 目录页部分节号被OCR识别为 `63/64/65`，节标题和后续正文边界已交叉核验，逐字符引用目录节号时仍应查看页面图像。

# 下一精确断点

- PDF p.464 / 书内 p.436，OCR第47行，§6.2 `Programming Models and Workloads for Warehouse-Scale Computers` 标题。
