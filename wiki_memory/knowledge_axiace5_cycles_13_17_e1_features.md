---
name: knowledge_axiace5_cycles_13_17_e1_features
description: AXIACE5.pdf 自动学习周期 13-17，E1 AMBA 5 additional features 后半
metadata:
  node_type: memory
  source: amba_prot/AXIACE5.pdf
  learned: 2026-08-21
  cycles: 13-17
  source_pages: PDF 359-408
  estimated_source_chars: 165453
---

# 周期 13-17：E1 AMBA 5 Additional Features 后半

## 覆盖范围

- 周期 13：PDF p.359-368，E1.5-E1.8。
- 周期 14：PDF p.369-378，E1.9-E1.11.1。
- 周期 15：PDF p.379-388，E1.11.2-E1.15.2。
- 周期 16：PDF p.389-398，E1.15.3-E1.17.3。
- 周期 17：PDF p.399-408，E1.18-E1.19，进入 E2.1-E2.3。

## User Loopback、QoS Accept、Wake-up

- User Loopback signaling 用于把用户定义信号在接口路径中回传或关联，便于端到端追踪实现自定义信息。
- QoS Accept signaling 允许下游反馈 QoS 接收能力或接受状态，帮助上游调节请求发出。
- Wake-up signaling 包括 `AWAKEUP` 和 `ACWAKEUP` 等规则，用于低功耗或接口休眠场景下通知对端恢复活动。

## Coherency Connection signaling

- Coherency Connection signaling 描述一致性连接状态和握手，帮助接口在一致性能力开启/关闭或连接变化时保持协议安全。
- 该机制与 DVM message 相关：若连接状态不支持 DVM，就不能假定 DVM 消息能安全传递。
- 不兼容的 Coherency Connection 支持需要被显式识别，避免把能力不同的组件直接组合。

## Untranslated transactions

- Untranslated transactions 允许携带尚未经过最终地址转换的信息，适用于 SMMU/PCIe 等复杂 I/O 虚拟化路径。
- 相关信令包含虚拟化/安全/标识字段；可选信号有默认值。
- 与 ACE5 结合时要检查协议规则，不能简单把 untranslated 事务当作普通物理地址访问。
- `StashTranslation` 与 stash/translation 流程有关，提示 stash 目标和地址转换可能互相依赖。

## NSAID、Read data chunking、Read interleaving、Unique ID

- Non-secure access identifier（NSAID）用于区分非安全访问来源或上下文；cache 行为可能依赖 NSAID。
- Read data chunking 允许读数据按 chunk 组织或返回，需遵守 chunking protocol rules 和互操作规则。
- Read interleaving property 描述接口是否允许或支持读数据交织。
- Unique ID indicator 用于声明某些 ID 使用方式的唯一性，辅助 ordering 和 routing 判断。

## MPAM

- MPAM（Memory Partitioning and Monitoring）用于内存分区和监控。
- E1.14 覆盖 MPAM signaling 与组件交互，核心作用是把资源分配、监控或服务质量相关属性随事务传播。

## Memory tagging (MTE)

- MTE support 和 MTE signaling 为内存标签传输提供协议级支持。
- 标签可被 cache、传输、读写事务携带或访问。
- E1.15 单独说明 MTE 与 Atomic transactions、Prefetch、Poison 的交互：Atomic transaction 不应只按普通写处理，必须考虑标签检查/携带语义。
- MTE interoperability 规定不同组件支持能力不一致时的互操作处理。

## Prefetch 与 WriteZero

- Prefetch request/response 允许显式请求预取数据，并可对预取结果给出响应。
- Prefetch 不应被误读为一定改变架构可见状态；其作用是性能优化和提前数据准备。
- WriteZero transaction 写入零值且不携带普通写数据，用于减少带宽或表达特殊清零操作；需要查看其配置、事务行为和互操作要求。

## Additional interface properties 与 E2 入口

- E1.18 收束若干接口属性：exclusive accesses、shareable transactions、maximum transaction size and boundary、consistent DECERR response。
- E1.19 讨论 signal width properties。
- 周期 17 进入 E2 Interface and data protection，开始 Poison、Parity 和接口保护配置。

## 监督记录

- 单周期抽取字符估算：32,730；38,395；31,273；35,450；24,665，均低于 80,000 字符上限。
- 本阶段 E1 完整跨越多个可选特性；已按功能主题保存，未复制原文表格。
