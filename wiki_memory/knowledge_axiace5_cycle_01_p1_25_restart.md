---
name: knowledge_axiace5_cycle_01_p1_25_restart
description: 从头重学 AXIACE5.pdf 的第 1 个周期，覆盖前言、目录、A1/A2 起始与 A3 入口
metadata:
  node_type: memory
  source: amba_prot/AXIACE5.pdf
  learned: 2026-08-21
  restart_from: PDF p.1
  cycle: 1
  source_pages: PDF 1-25
  estimated_source_chars: 68249
---

# 周期 1：前言与目录

## 覆盖范围

- PDF p.1-25。
- 包含封面、版本历史、版权许可、前言目录、Part A-G 结构与 A1/A2/A3 起始目录页。

## 关键内容

- 文档标题为 `AMBA AXI and ACE Protocol Specification`，版本号为 `ARM IHI 0022H.c`。
- 版本历史显示 AXI3、AXI4、AXI5、AXI5-Lite、ACE5、ACE5-Lite、ACE5-LiteDVM、ACE5-LiteACP 的逐步演进。
- Part A 到 Part G 的结构已经明确：AXI、AXI4-Lite、AXI5/AXI5-Lite、ACE/ACE-Lite、AMBA 5 features、ACE5 family、appendices。
- A1 目录显示 AXI protocol 的三个核心入口：`About the AXI protocol`、`AXI Architecture`、`Terminology`。
- A2 目录开始列出全局信号和五个通道信号：`AW*`、`W*`、`B*`、`AR*`、`R*`。
- A3 目录确认后续基础事务章节包含 clock/reset、basic read/write transactions、channel relationships、transaction structure。

## 当前理解

- AXI 文档的前 25 页主要是结构导览和章节地图，不是协议细节本体。
- 这一周期的目标是把文档版图和术语入口固定下来，方便后续按章节推进。

## 监督记录

- 本周期抽取字符估算约 68,249，低于 `study_monitor` 的 80,000 字符阈值。
- 章节边界停在 A1/A2/A3 的目录入口，未跨越自然章节正文边界。
