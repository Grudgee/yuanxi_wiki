---
name: knowledge_axiace5_core_axi
description: 重新从 AXIACE5.pdf 学习的 AXI 基础事务、ID 和 ordering 模型
metadata:
  node_type: memory
  source: amba_prot/AXIACE5.pdf
  relearned: 2026-08-21
  source_pages: PDF 39-60, 79-92
---

# AXI 核心事务、ID 与 Ordering

## A3 Single Interface Requirements

- AXI 事务由五个独立通道组成：读地址 `AR`、读数据 `R`、写地址 `AW`、写数据 `W`、写响应 `B`。
- 所有通道都基于 `VALID/READY` 握手；源端用 `VALID` 表示有效，接收端用 `READY` 表示可接收。
- 握手规则不能被原子性误读：`VALID/READY` 只保证一次通道传输被接收，不等价于内存访问原子完成。
- AXI4/AXI5 对写响应有更强依赖：Subordinate 必须等到对应地址和所有写数据均完成后才能给写响应。
- Burst 相关属性包括 `AxADDR`、`AxLEN`、`AxSIZE`、`AxBURST`；exclusive access 对 burst 还有额外限制，见 `knowledge_axiace5_atomic_accesses.md`。
- 响应编码中，`OKAY` 可表示普通访问成功，也可表示 exclusive 访问失败；`EXOKAY` 只用于 exclusive read/write 成功。

## A5 Transaction Identifiers

- AXI 用事务 ID 支持 outstanding 与乱序完成。
- 同一 ID 下的顺序约束比不同 ID 更强；不同 ID 事务通常可并行或乱序，但仍受目标、interconnect 和 ordering 规则限制。
- 原子 exclusive sequence 中，exclusive read 与 exclusive write 必须使用匹配的 ID；A7 明确要求同一序列的 `AxID` 相同。

## A6 Ordering Model

- AXI ordering 模型关注其他组件何时能观察到访问效果，而不只关注通道完成顺序。
- 写响应可能与最终可见性存在实现相关差异，因此回答 ordering 问题时要区分“响应返回”“到达最终目标”“被其他 agent 观察到”。
- A6 ordering 规则与 A7 原子访问相连：原子性要求不能让其他 Manager 观察到部分更新。

## 易错点

- `VALID/READY` 不是互斥锁。
- `OKAY` 不一定代表 exclusive 成功；exclusive 成功看 `EXOKAY`。
- 同一 exclusive sequence 的 ID 和多项属性必须一致，否则不能视为同一原子序列。
