---
name: knowledge_axiace5_relearn_index
description: 重新从 amba_prot/AXIACE5.pdf 建立的 AXI/ACE5 本地记忆索引
metadata:
  node_type: memory
  source: amba_prot/AXIACE5.pdf
  relearned: 2026-08-21
  pdf_version: ARM IHI 0022H.c
---

# AXIACE5 重新学习索引

## 清理说明

- 2026-08-21 已清除旧的迁移版 AXI/ACE 记忆文件，避免把早期不完整或推断性内容当作已验证知识继续使用。
- 本索引和同批新文件只以 `amba_prot/AXIACE5.pdf` 的重新抽取内容为依据。
- 若回答涉及尚未在新记忆中覆盖的章节，应重新从 PDF 抽取相应页，而不是引用旧迁移记忆。

## 新记忆文件

- `knowledge_axiace5_core_axi.md`：A3/A5/A6 的 AXI 基础事务、ID 与 ordering 模型。
- `knowledge_axiace5_transaction_attributes.md`：A4 的 `AxCACHE`、Modifiable/Non-modifiable、内存类型与可见性约束。
- `knowledge_axiace5_atomic_accesses.md`：A7 的 single-copy atomicity、multi-copy atomicity、exclusive access、locked access 与 `AxLOCK` 编码。
- `knowledge_axiace5_axi5_ace_boundary.md`：B1/C1/C2/D1–D5 的 AXI4-Lite、AXI5、AXI5-Lite、ACE 边界摘要和后续抽取入口。
- `knowledge_axiace5_cycles_01_05_d5_d8.md`：周期 01-05，PDF p.239-288，D5 Snoop 后半、D6 Interconnect、D7 Cache Maintenance、D8 Barrier 起始。
- `knowledge_axiace5_cycles_06_10_d8_d14.md`：周期 06-10，PDF p.289-338，D8 Barrier、D9 ACE Exclusive、D10-D14。
- `knowledge_axiace5_cycles_11_12_e1_atomic_stash.md`：周期 11-12，PDF p.339-358，E1 Atomic transactions、Cache stashing、Deallocating transactions。
- `knowledge_axiace5_cycles_13_17_e1_features.md`：周期 13-17，PDF p.359-408，E1 AMBA 5 additional features 后半。
- `knowledge_axiace5_cycles_18_20_e2_f3.md`：周期 18-20，PDF p.409-438，E2 Interface/data protection、F1 ACE5、F2 ACE5-Lite、F3 ACE5-LiteDVM 起始。

## 源文档范围

- 文档：`amba_prot/AXIACE5.pdf`
- 版本：Arm AMBA AXI and ACE Protocol Specification, ARM IHI 0022H.c, 2021
- 本轮重点抽取：PDF 物理页 39–100（A3–A7）、119–238（B1–D5 早段），以及自动学习 20 周期覆盖 PDF 物理页 239–438。
- 重点纠偏：A7 的 Atomic access signaling 在本文件覆盖范围内指 `AxLOCK` 正常/独占/锁定访问编码；不要把未重新抽取的 `AWATOP` 细节当作本地记忆结论。
- 最新断点：PDF p.439，继续 F3 ACE5-LiteDVM signal descriptions 与后续章节。

## 自动学习批次记录（2026-08-21）

- 监督规则：每周期约 10 页；单周期估算 19,113–40,215 字符，低于 `study_monitor` 的 80,000 字符阈值。
- 周期 01-05：D5/D6/D7/D8，阶段记忆 `knowledge_axiace5_cycles_01_05_d5_d8.md`。
- 周期 06-10：D8/D9/D10-D14，阶段记忆 `knowledge_axiace5_cycles_06_10_d8_d14.md`。
- 周期 11-12：E1.1-E1.4，重点补完 AMBA 5 Atomic transactions 与 `AWATOP`，阶段记忆 `knowledge_axiace5_cycles_11_12_e1_atomic_stash.md`。
- 周期 13-17：E1.5-E1.19 与 E2 入口，阶段记忆 `knowledge_axiace5_cycles_13_17_e1_features.md`。
- 周期 18-20：E2/F1/F2/F3 起始，阶段记忆 `knowledge_axiace5_cycles_18_20_e2_f3.md`。

## 回答原则

- 讲 AXI 原子操作时，优先从 `knowledge_axiace5_atomic_accesses.md` 出发。
- 讲 atomic 与 `AxCACHE` 的关系时，同时引用 `knowledge_axiace5_transaction_attributes.md`。
- 讲 AXI5 Atomic transactions / `AWATOP` 时，优先引用 `knowledge_axiace5_cycles_11_12_e1_atomic_stash.md`。
- 讲 ACE 或 AXI5 optional 属性时，先查本索引中对应周期文件；若未覆盖到字段级细节，应按页重新抽取 PDF。
