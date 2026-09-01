---
name: knowledge_axiace5_transaction_attributes
description: 重新从 AXIACE5.pdf 学习的 A4 Transaction Attributes 与 AxCACHE 规则
metadata:
  node_type: memory
  source: amba_prot/AXIACE5.pdf
  relearned: 2026-08-21
  source_pages: PDF 61-78
---

# A4 Transaction Attributes：`AxCACHE` 与可见性

## `AxCACHE` 的定位

- `AxCACHE` 是 `ARCACHE` 与 `AWCACHE` 的统称。
- A4 明确说这些信号控制两件事：事务如何在系统中前进，以及系统级 cache 如何处理事务。
- 因此 `AxCACHE` 不是单纯“缓存提示”，也会影响事务能否被缓冲、预取、合并、拆分或改变形态。

## AXI3 定义

- `AxCACHE[0]`：Bufferable。置位后，interconnect 或组件可以延迟事务到达最终目的地；通常主要影响写。
- `AxCACHE[1]`：Cacheable。为 0 时禁止 allocation；为 1 时允许 allocation，并且最终到达目的地的事务特性不必完全等同原事务。
- `AxCACHE[2]`：Read-Allocate 建议位；只有 Cacheable 为 1 时才允许置位。
- `AxCACHE[3]`：Write-Allocate 建议位；只有 Cacheable 为 1 时才允许置位。

## AXI4 的关键变化

- AXI4 将 `AxCACHE[1]` 从 Cacheable 重命名为 Modifiable，以强调该位控制事务特性能否被修改。
- `AxCACHE[1]=0`：Non-modifiable。
- `AxCACHE[1]=1`：Modifiable。

## Non-modifiable 事务

- Non-modifiable 事务不能被拆成多个事务，也不能与其他事务合并。
- 固定参数包括 `AxADDR`、`AxSIZE`、`AxLEN`、`AxBURST`、`AxLOCK`、`AxPROT` 等。
- `AxCACHE` 只能从 Bufferable 转为 Non-bufferable，不能做其他改变。
- burst length 大于 16 的 Non-modifiable 事务允许拆分，但生成事务只能缩短 burst 并适当调整地址。
- Non-modifiable 的 exclusive access 允许修改 `AxSIZE` 和 `AxLEN`，前提是总访问字节数保持不变。

## Modifiable 事务

- Modifiable 事务可以被拆分、合并。
- 读事务可以多取数据；写事务可以访问更大地址范围，但必须用 `WSTRB` 保证只更新目标位置。
- 生成事务可修改 `AxADDR`、`AxSIZE`、`AxLEN`、`AxBURST`。
- 不得修改 `AxLOCK` 和 `AxPROT`。
- `AxCACHE` 可以被修改，但不能降低其他组件对事务的可见性，不能阻止事务传播到必须到达的点，也不能取消本应进行的 cache lookup。
- 同一地址范围生成事务的内存属性必须一致。
- 禁止把事务改到原事务之外的另一个 4KB 地址空间。
- 禁止把对 single-copy atomicity sized region 的单次访问变成多次访问。

## Allocate 位在 AXI4 中的含义

- AXI4 中 `AxCACHE[3:2]` 不只是“分配建议”，还表达是否必须 cache lookup。
- 只要 `AxCACHE[3:2] != 2'b00`，事务必须在 cache 中查找。
- `AxCACHE[3:2] == 2'b00` 时，不要求 cache lookup。
- 读写通道对 Allocate / Other Allocate 的具体位定义不同：读通道 `ARCACHE[2]` 是 Allocate、`ARCACHE[3]` 是 Other Allocate；写通道 `AWCACHE[3]` 是 Allocate、`AWCACHE[2]` 是 Other Allocate。

## 与原子操作的关系

- `AxCACHE` 不编码“原子操作类型”；A7 中 atomic access signaling 由 `AxLOCK` 表示 normal/exclusive/locked。
- 但 `AxCACHE` 会影响原子访问是否能正确到达负责监视的 Subordinate。
- 若 `AxCACHE` 允许上游 buffer/cache 提前响应，exclusive monitor 可能看不到访问；A7 因此要求 exclusive access 的 `AxCACHE` 必须保证事务到达监视独占访问的 Subordinate。
- Modifiable 也不能破坏 single-copy atomicity；即使允许拆分/合并，也不能把一个原子粒度内的单次访问变成多个可被观察的访问。
