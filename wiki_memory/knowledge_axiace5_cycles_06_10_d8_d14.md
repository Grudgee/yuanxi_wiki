---
name: knowledge_axiace5_cycles_06_10_d8_d14
description: AXIACE5.pdf 自动学习周期 06-10，D8 Barrier、D9 ACE Exclusive、D10-D14 ACE 扩展
metadata:
  node_type: memory
  source: amba_prot/AXIACE5.pdf
  learned: 2026-08-21
  cycles: 06-10
  source_pages: PDF 289-338
  estimated_source_chars: 151168
---

# 周期 06-10：D8 Barrier、D9 Exclusive、D10-D14

## 覆盖范围

- 周期 06：PDF p.289-298，D8.3-D8.4.5，进入 D9.1-D9.2.3。
- 周期 07：PDF p.299-308，D9.3-D9.6，进入 D10。
- 周期 08：PDF p.309-318，D10.3、D11 ACE-Lite、D12 Interface Control、进入 D13 DVM。
- 周期 09：PDF p.319-328，D13 DVM message 主体前段。
- 周期 10：PDF p.329-338，D13 DVM 后段与 D14 入口。

## D8 Barrier Transactions

- Barrier 约束 Manager、Subordinate、interconnect 在指定 domain 内的事务顺序与完成观察。
- Manager 发 barrier 前后要遵守自己的排序责任；Subordinate 和 interconnect 需要把 barrier 的响应与相关事务完成条件绑定。
- Device transaction ordering 与 barrier 有额外关系：Device 访问本身有更强顺序语义，barrier 不能被错误地弱化为普通请求。
- 对 Shareable locations，D8 明确关联 multi-copy atomicity 要求：shareable 写的可见性不能只对部分 observer 生效。

## D9 Exclusive Accesses from ACE Managers

- ACE Manager 的 exclusive load/store 不是简单复用 AXI4 exclusive 规则；它与 coherent cache line 的 unique 权限、PoS monitor 和 snoop 行为相关。
- Manager 角色包括：发起 exclusive load、从 exclusive load 到 exclusive store 的保持期、以及 exclusive store 是否成功。
- Interconnect 角色包括 PoS exclusive monitor。最小 PoS monitor 至少比较必要地址范围；实现可做额外地址比较，但过宽比较可能导致本可成功的 exclusive 被判失败。
- 多个 PoS monitor 和多个 exclusive thread 时，必须维护每个 thread 的独占状态，不得让不同线程或不同 ID 的状态相互污染。
- 来自 AXI components 的 exclusive access 接入 ACE 环境时，需要桥接 AXI exclusive 语义与 ACE coherent 权限模型。

## D10 Optional External Snoop Filtering

- 外部 snoop filter 用于减少不必要 snoop，但必须准确跟踪可能持有 cache line 的 Manager。
- 若 filter 需要跟踪额外 cache line，可增加存储；filter 不能因为优化而漏掉必须 snoop 的持有者。

## D11 ACE-Lite

- ACE-Lite 是 AXI4 加读/写地址通道上的一致性控制信号，适合 I/O coherent 组件。
- D11 给出 `ARSNOOP`、`ARBAR`、`ARDOMAIN` 和 `AWBAR`、`AWDOMAIN` 的合法组合。
- ACE-Lite 不是完整 ACE Manager；它可以参与一致性访问，但能力受限，不等于拥有完整 snoop cache 行为。

## D12 Interface Control

- Interface control signals 描述接口支持的事务能力组合。
- 这些控制信号帮助系统配置时判断一个组件可发起或响应哪些一致性/维护/DVM 事务。

## D13 Distributed Virtual Memory Transactions

- DVM transaction 用于传递维护虚拟内存系统的消息。
- DVM 有两类事务：DVM message 与 DVM Complete。
- DVM message 支持 TLB maintenance、branch predictor invalidate、instruction cache invalidate、synchronization 等操作。
- DVM Synchronization 需要 DVM Complete 响应，表示所需操作已经完成。
- DVM 支持能力由 `DVM_Message_Support` 等属性描述；接口可以支持发起和接收，也可以只接收。

## D14 入口

- 周期 10 到达 Chapter D14 入口；本阶段未完整展开 D14，应在后续周期继续从 D14 具体小节开始核验。

## 监督记录

- 单周期抽取字符估算：34,120；26,323；19,113；40,215；31,397，均低于 80,000 字符上限。
- 本阶段覆盖多个章节，按 D8-D14 阶段保存，D14 具体内容标记为后续待核验。
