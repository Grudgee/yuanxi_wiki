---
name: knowledge_index_axiace5
description: AXIACE5.pdf 第19–276页知识记忆索引和学习进度
metadata: 
  node_type: memory
  originSessionId: 07d5e602-b8a4-4424-94dd-570763cdaafc
  modified: 2026-07-21T11:09:43.690Z
---

# AXIACE5 知识库索引

## 文档范围

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 目标范围：PDF 物理页 19–276（含首尾）
- 版本/日期：ARM IHI 0022H.c，Copyright 2003–2021
- 已学习范围：PDF 物理页 19–238
- 当前状态：进行中

## 章节索引

- 前言约定：`knowledge_preface_conventions.md`（PDF 物理页 19–21）
- Chapter A3 Single Interface Requirements：`knowledge_a3_axi_transactions.md`（PDF 物理页 39–60；印刷页 A3-39–A3-60）
- Chapter A4 Transaction Attributes：`knowledge_a4_transaction_attributes.md`（PDF 物理页 61–78；印刷页 A4-61–A4-78）
- Chapter A5 Transaction Identifiers：`knowledge_a5_transaction_identifiers.md`（PDF 物理页 79–82；印刷页 A5-79–A5-82）
- Chapter A6 AXI Ordering Model：`knowledge_a6_axi_ordering_model.md`（PDF 物理页 83–92；印刷页 A6-83–A6-92）
- Chapter A7 Atomic Accesses：`knowledge_a7_atomic_accesses.md`（PDF 物理页 93–100；印刷页 A7-93–A7-100）
- Chapter A8 AMBA 4 Additional Signaling：`knowledge_a8_additional_signaling.md`（PDF 物理页 101–106；印刷页 A8-101–A8-106）
- Chapter B1 AMBA AXI4-Lite、Chapter C1 AMBA AXI5：`knowledge_b1_c1_axi4lite_axi5.md`（PDF 物理页 119–138；印刷页 B1-119–B1-126、C1-129–C1-138）
- Chapter C2 AMBA AXI5-Lite、Chapter D1 About ACE：`knowledge_c2_d1_ace_overview.md`（PDF 物理页 139–158；印刷页 C2-139–C2-146、D1-149–D1-158）
- Chapter D1 ACE 事务与 Chapter D2/D3 信号：`knowledge_d1_d3_ace_transactions_signals.md`（PDF 物理页 159–178；印刷页 D1-159–D1-168、D2-169–D2-174、D3-175–D3-178）
- Chapter D3 ACE Channel Signaling（续）：`knowledge_d3_ace_channel_signaling.md`（PDF 物理页 179–198；印刷页 D3-179–D3-198）
- Chapter D4/D5 ACE 写事务、重叠写与 Snoop 通用要求：`knowledge_d4_d5_ace_write_snoop_requirements.md`（PDF 物理页 219–238；印刷页 D4-219–D4-230、D5-231–D5-238）
- Chapter D5 Snoop Transactions：待继续（本批已覆盖 D5-231–D5-238 的映射和通用规则）
- Chapter D1 ACE 事务与 Chapter D2/D3 信号：`knowledge_d1_d3_ace_transactions_signals.md`（PDF 物理页 159–178；印刷页 D1-159–D1-168、D2-169–D2-174、D3-175–D3-178）
- Chapter D3 ACE Channel Signaling（续）：`knowledge_d3_ace_channel_signaling.md`（PDF 物理页 179–198；印刷页 D3-179–D3-198）
- Chapter D3 CD通道、Chapter D4 一致性事务：`knowledge_d3_d4_ace_snoop_data_states.md`（PDF 物理页 199–218；印刷页 D3-199–D3-202、D4-203–D4-218）
- Chapter A1 Introduction：待整理独立摘要（本范围前段已读取章节目录和概览）
- Chapter A2 Signal Descriptions：待整理独立摘要（本范围前段已读取章节目录和信号表）

## 主题索引

- VALID/READY 握手、通道依赖、burst：A3 文件
- 读写响应和错误编码：A3 文件
- AxCACHE、Modifiable/Non-modifiable、内存类型：A4 文件
- AxPROT 访问权限：A4 文件
- 缓冲和属性不匹配：A4 文件
- 事务 ID、读写顺序：A5 文件
- AXI 排序、观察保证、提前响应：A6 文件
- 原子访问和独占访问：A7 文件
- QoS、多 region、User signals：A8 文件
- 默认信号和接口互操作：A9 文件
- AXI 信号定义：待整理/后续补充 A2 文件

## 交叉关系

- A2 信号定义 → A3 握手和事务结构
- A3 burst/事务结构 → A4 事务属性
- A4 AxCACHE 内存类型 → 后续排序、独占访问和 ACE5 一致性规则
- A5 事务 ID → A6 顺序模型和跨 Manager/interconnect 路由
- A6 排序保证 → A7 原子访问和后续 ACE5 一致性事务
- A7 原子访问 → A8/A9 接口信号与互操作约束

## 学习进度报告

- 已完成批次：19–38、39–58、59–78、79–98、99–118、119–138、139–158、159–178、179–198、199–218、219–238
- 已存储知识文件：15 个（含本索引）
- 下一个批次：PDF 物理页 239–258
- 上下文使用率：未知（无法精确测量）
- 待处理：继续读取后续章节，并在同一章节内容较短时整合到已有文件；避免按页机械拆分。
