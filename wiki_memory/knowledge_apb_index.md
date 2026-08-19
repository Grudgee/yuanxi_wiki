---
name: knowledge_apb_index
description: AMBA APB Protocol Specification v2.0 学习索引与章节进度
metadata:
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T03:14:33.049Z
---

# AMBA APB Protocol Specification v2.0 知识库索引

## 文档信息

- 文档：AMBA APB Protocol Specification
- 版本：Version 2.0 / Issue C
- 编号：ARM IHI 0024C (ID042910)
- 发布：13 April 2010，Non-Confidential
- 本地来源：`file:///home/yuanxi/yuanxi_cc/wiki_files/amba_prot/APB.pdf`
- PDF 页数：28 页

## 章节结构

- Preface：文档使用说明、时序图和信号约定、反馈渠道。
- Chapter 1 Introduction：APB 协议定位与 APB2/APB3/APB4 修订差异。见 [[knowledge_apb_1_introduction]]。
- Chapter 2 Signal Descriptions：APB 信号表、信号方向、读写数据总线。见 [[knowledge_apb_2_signal_descriptions]]。
- Chapter 3 Transfers：写传输、写选通、读传输、错误响应、保护单元支持。见 [[knowledge_apb_3_transfers]]。
- Chapter 4 Operating States：APB `IDLE/SETUP/ACCESS` 工作状态机。见 [[knowledge_apb_4_operating_states_revisions]]。
- Appendix A Revisions：Issue A/B/C 技术修订差异。见 [[knowledge_apb_4_operating_states_revisions]]。

## 当前学习进度

- 已学习：封面、Release Information、目录、图表清单、Preface、Chapter 1、Chapter 2、Chapter 3、Chapter 4、Appendix A。
- 已存储记忆：4 个章节文件（[[knowledge_apb_1_introduction]]、[[knowledge_apb_2_signal_descriptions]]、[[knowledge_apb_3_transfers]]、[[knowledge_apb_4_operating_states_revisions]]）+ 本索引。
- 下一步：APB.pdf 已完整学习完毕；后续可按主题复习或与 AXI/AHB bridge 行为对照。

## 重要约定

- 时序图中阴影区域表示 bus/signal undefined，该时刻可以取任意值；实际电平不重要且不影响正常操作。
- 单比特信号在图中若画得类似总线变化，只要附文未特别说明，该画法不改变文字描述。
- 信号 asserted 的电平取决于 active-HIGH 或 active-LOW；信号名开头或结尾的小写 `n` 表示 active-LOW。
- APB transfer 至少包含 Setup phase 与 Access phase；`PENABLE` asserted 后进入 Access phase，`PREADY` 可扩展 transfer。
- `PSLVERR` 只在最后一个 transfer cycle 且 `PSEL`、`PENABLE`、`PREADY` 全 HIGH 时有效。
- `PPROT[2:0]` 编码与 AXI `AxPROT[2:0]` 对应：`[0]` privileged、`[1]` non-secure、`[2]` instruction。
- APB 状态机包含 `IDLE`、`SETUP`、`ACCESS`：`SETUP` 固定一个 clock cycle；`ACCESS` 由 `PREADY` 决定停留或退出。

## APB revision 快速索引

- APB2 / Issue A：基础 APB interface。
- APB3 / Issue B：增加 `PREADY` 和 `PSLVERR`。
- APB4 / Issue C：增加 `PPROT` 和 `PSTRB`。

## 关联知识

- [[knowledge_apb_1_introduction]]
- [[knowledge_apb_2_signal_descriptions]]
- [[knowledge_apb_3_transfers]]
- [[knowledge_apb_4_operating_states_revisions]]
- [[knowledge_a4_transaction_attributes]]：AXI protection 属性，可与 APB4 `PPROT` 对比。
