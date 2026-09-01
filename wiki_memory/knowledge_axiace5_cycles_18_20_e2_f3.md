---
name: knowledge_axiace5_cycles_18_20_e2_f3
description: AXIACE5.pdf 自动学习周期 18-20，E2 Interface/data protection、F1 ACE5、F2 ACE5-Lite、F3 ACE5-LiteDVM 开始
metadata:
  node_type: memory
  source: amba_prot/AXIACE5.pdf
  learned: 2026-08-21
  cycles: 18-20
  source_pages: PDF 409-438
  estimated_source_chars: 97554
---

# 周期 18-20：E2、F1、F2、F3 开始

## 覆盖范围

- 周期 18：PDF p.409-418，E2.4-E2.6，进入 F1.1。
- 周期 19：PDF p.419-428，F1.1-F1.2.2，进入 F2.1-F2.2.1。
- 周期 20：PDF p.429-438，F2.2.2，进入 F3.1。

## E2 Interface and data protection

- E2 覆盖 Poison、AMBA 中 parity 的使用、接口保护配置、byte parity check signals、error detection behavior 和 parity check signals。
- Poison 用于标记数据或事务存在错误/不可安全使用状态；下游组件必须按接口保护配置处理。
- Parity 相关信号为通道数据、控制或字节粒度信息提供错误检测能力。
- Error detection behavior 规定发现保护错误后如何响应、传播或报告；不能把 parity 只当作调试信号。

## F1 AMBA ACE5

- ACE5 是 ACE 在 AMBA 5 中的接口形态，基于既有 ACE 一致性模型并引入 AMBA 5 optional features 的信号扩展。
- F1.2 描述现有 ACE 通道的变化和 additional signaling。
- 回答 ACE5 问题时，应区分：ACE 的基本一致性事务来自 D 章，ACE5 的新增/变更信号来自 F1。

## F2 AMBA ACE5-Lite

- ACE5-Lite 是 ACE5 的 Lite 形态，面向 I/O coherent 或非完整缓存一致性 Manager。
- F2.2 描述 ACE5-Lite signal descriptions，包括既有 ACE-Lite 通道变化和 additional signaling。
- ACE5-Lite 可与 E1 的部分 AMBA 5 可选特性组合，但具体支持能力由接口属性声明。

## F3 AMBA ACE5-LiteDVM 开始

- ACE5-LiteDVM 在 ACE5-Lite 基础上增加 DVM 支持，面向包含 SMMU 功能并需要接收 DVM transaction 的 I/O coherent 组件。
- ACE5-LiteDVM Manager 必须能在 AC channel 接收 DVM messages；对于 DVM Synchronization，还必须能在 AR channel 发送 DVM Complete。
- Interconnect 的 ACE5-LiteDVM interface 可在 AC channel 发出 DVM messages，并能在 AR channel 接收 DVM Complete。
- F3.1 属性表显示可选能力包括 `Atomic_Transactions`、不同版本的 DVM support、Cache Stash、Exclusive_Accesses、Shareable_Transactions 等。

## 当前断点

- 已读至 PDF p.438 / F3.1 ACE5-LiteDVM 属性说明早段。
- 下一学习断点：PDF p.439，继续 F3 ACE5-LiteDVM signal descriptions 与后续章节。

## 监督记录

- 单周期抽取字符估算：26,672；37,694；33,188，均低于 80,000 字符上限。
- 本阶段完成 E2 并进入 F3，停在自然章节内部断点，已记录下一页继续。
