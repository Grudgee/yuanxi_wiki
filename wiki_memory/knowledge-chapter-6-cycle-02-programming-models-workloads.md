---
name: knowledge-chapter-6-cycle-02-programming-models-workloads
description: 第6章自动学习周期2/15，记录WSC编程模型、MapReduce、工作负载特性和软件栈选择。
---

# 知识点摘要

- WSC服务以搜索、内容处理和大规模数据分析为代表，存在海量独立请求与数据分片，适合通过请求级并行扩展。
- MapReduce把计算拆成 map 和 reduce 两阶段，由运行时负责分区、任务调度、数据局部性、失败重试和慢任务处理，使普通程序员能够使用数千台机器。
- WSC软件通常自行开发并运行在Linux等低许可成本平台上，以便适配规模、故障和数据布局，而不是直接采用昂贵的传统数据库软件栈。

# 关键结论

- WSC的高可扩展性来自编程模型与运行时共同暴露并管理并行度，而不是依赖单机ILP或单一共享存储映像。
- 数据局部性和调度器对慢节点、失败节点的容忍是WSC吞吐量的重要组成部分。

# 关键细节

- 批次周期：2/15。
- 范围：§6.2；PDF p.464 OCR第47行至 p.469 OCR第19行 / 书内 p.436–441。
- OCR源字符数：12005，严格小于80000。

# 边界依据

- 从§6.2标题开始，完整覆盖MapReduce模型、Google工作负载增长、调度与软件成本讨论。
- 在 PDF p.469 OCR第20行 `Computer Architecture of Warehouse-Scale Computers` 标题前停止。

# 待核验问题

- 图6.2的年度作业数、机器数和字节数属于书中历史快照；精确引用时需核对图表图像。

# 下一精确断点

- PDF p.469 / 书内 p.441，OCR第20行，§6.3 `Computer Architecture of Warehouse-Scale Computers` 标题。
