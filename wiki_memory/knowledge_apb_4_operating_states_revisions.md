---
name: knowledge_apb_4_operating_states_revisions
description: APB 工作状态机 IDLE/SETUP/ACCESS 与 Issue A/B/C 修订差异
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T03:14:32.930Z
---

# APB Chapter 4 与 Appendix A：Operating States / Revisions

## 知识点摘要

APB 的操作活动可以抽象为 3 个状态：

- `IDLE`
- `SETUP`
- `ACCESS`

核心规则是：有 transfer 请求时从 `IDLE` 进入 `SETUP`；`SETUP` 只持续一个 clock cycle，然后总是进入 `ACCESS`；`ACCESS` 是否退出由 slave 驱动的 `PREADY` 控制。

Appendix A 记录了 APB 各 issue 的技术变更：Issue B 添加 `PREADY` 和 `PSLVERR`，Issue C 添加 `PPROT` 和 `PSTRB`。

## Operating states

### IDLE

`IDLE` 是 APB 的默认状态。

典型状态条件：

- `PSELx = 0`
- `PENABLE = 0`

如果没有 transfer，状态机保持在 `IDLE`。

### SETUP

当需要一次 transfer 时，bus 进入 `SETUP` 状态。

关键规则：

- appropriate select signal `PSELx` 被 asserted。
- `PENABLE = 0`。
- bus 只在 `SETUP` 状态停留一个 clock cycle。
- 在下一个 `PCLK` 上升沿，状态总是进入 `ACCESS`。

`SETUP` 对应 Chapter 3 transfer 描述中的 Setup phase。见 [[knowledge_apb_3_transfers]]。

### ACCESS

`ACCESS` 状态中：

- `PSELx = 1`
- `PENABLE = 1`

在 `ACCESS` 状态中，address、write、select 和 write data signals 必须在从 `SETUP` 到 `ACCESS` 的 transaction 期间保持稳定。

退出 `ACCESS` 由 slave 的 `PREADY` 控制：

- 如果 `PREADY` 为 LOW，peripheral bus 保持在 `ACCESS` 状态。
- 如果 `PREADY` 为 HIGH，则退出 `ACCESS`。
  - 若没有后续 transfer，bus 回到 `IDLE`。
  - 若紧接着有另一笔 transfer，bus 直接进入 `SETUP`。

这和 Chapter 3 中 `PREADY` 扩展 wait states 的规则一致。见 [[knowledge_apb_3_transfers]]。

## 状态转换总结

| 当前状态 | 条件 | 下一状态 |
| --- | --- | --- |
| `IDLE` | no transfer | `IDLE` |
| `IDLE` | transfer | `SETUP` |
| `SETUP` | 固定一个 clock cycle 后 | `ACCESS` |
| `ACCESS` | `PREADY = 0` | `ACCESS` |
| `ACCESS` | `PREADY = 1` 且 no transfer | `IDLE` |
| `ACCESS` | `PREADY = 1` 且 transfer | `SETUP` |

## Appendix A：Revisions

### Issue A

- First release。

### Issue B 相对 Issue A 的变化

Issue B 添加 APB3 相关信号：

- `PREADY` added。
  - 影响 Table 2-1、write transfers、read transfers、error response、operating states。
  - 作用是允许 slave 通过 wait states 扩展 transfer。
- `PSLVERR` added。
  - 影响 Table 2-1 和 error response。
  - 作用是支持 transfer error response。

### Issue C 相对 Issue B 的变化

Issue C 添加 APB4 相关内容：

- 增加记录各 revision 变化的 section。
- `PPROT` added。
  - 影响 Table 2-1 和 protection unit support。
  - 作用是提供 protection attribute，包括 privileged、secure/non-secure、instruction/data。见 [[knowledge_apb_3_transfers]]。
- `PSTRB` added。
  - 影响 Table 2-1 和 write strobes。
  - 作用是支持 write data bus 上的 sparse data transfer。见 [[knowledge_apb_3_transfers]]。

## 与前面章节的关系

- Chapter 4 的 `IDLE/SETUP/ACCESS` 状态机是 Chapter 3 transfer timing 的抽象形式。
- `PREADY` 是 APB3 引入的关键扩展信号，直接影响 wait states、error response 的有效采样，以及 operating states 中 `ACCESS` 的退出条件。
- `PPROT` 和 `PSTRB` 是 APB4 引入的关键扩展信号，分别对应 protection unit support 与 write strobes。

## 原文位置

- 文档：AMBA APB Protocol Specification，Version 2.0 / Issue C，ARM IHI 0024C。
- Chapter 4 Operating States，4.1 “Operating states”，文档页 4-2，PDF 物理页约 26。
- Appendix A Revisions，Table A-1/A-2/A-3，文档页 A-1 至 A-2，PDF 物理页约 27 至 28。

## 关联知识

- [[knowledge_apb_index]]
- [[knowledge_apb_1_introduction]]
- [[knowledge_apb_2_signal_descriptions]]
- [[knowledge_apb_3_transfers]]
