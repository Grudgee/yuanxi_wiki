---
name: knowledge_axiace5_restart_from_p1
description: 2026-08-21 从 AXIACE5.pdf 第1页重新学习的 20 个周期进度日志
metadata:
  node_type: memory
  source: amba_prot/AXIACE5.pdf
  learned: 2026-08-21
  restart_from: PDF p.1
  total_cycles: 20
---

# 从第一页重新学习：20 个周期日志

## 周期 1

- 范围：PDF p.1-25。
- 内容：封面、版本历史、版权与许可、目录树、Part A-G 概览。
- 结论：先把文档结构和术语体系建立起来，确认本轮重学从第一页面开始。

## 周期 2

- 范围：PDF p.26-50。
- 内容：A1 Introduction、AXI 目标特性、A1 architecture、A1 terminology、A2 signal descriptions 开始、A3.1-A3.2。
- 结论：建立 AXI 的五通道模型、burst/beat/transaction 术语，以及完整信号族。

## 周期 3

- 范围：PDF p.51-75。
- 内容：A3.3-A3.4、A4.1-A4.4。
- 结论：补齐 VALID/READY 关系、transaction structure，以及 AXI3/AXI4 transaction attributes 和 memory types。

## 周期 4

- 范围：PDF p.76-100。
- 内容：A4.5-A4.9、A5、A6、A7.1-A7.4。
- 结论：补齐 mismatched memory attributes、buffering、permissions、ID、ordering、single-copy atomicity、exclusive / locked / atomic signaling。

## 周期 5

- 范围：PDF p.101-125。
- 内容：A8、A9、B1 开始。
- 结论：学习 QoS、multiple region、user 信号、默认信令与互操作，并进入 AXI4-Lite。

## 周期 6

- 范围：PDF p.126-150。
- 内容：B1 AXI4-Lite 细节、C1 AXI5 开始。
- 结论：确认 AXI4-Lite 的简化接口、固定宽度、无 burst、无 ID，以及 AXI5 的接口入口。

## 周期 7

- 范围：PDF p.151-175。
- 内容：C1/C2、D1 About ACE、D2 signal descriptions、D3 channel signaling 起始。
- 结论：开始 ACE 的一致性模型与通道级信号扩展。

## 周期 8

- 范围：PDF p.176-200。
- 内容：D3 继续、D4 coherency transactions on read/write address channels。
- 结论：学习 ACE 的 AR/AW 一致性事务和信号意义。

## 周期 9

- 范围：PDF p.201-225。
- 内容：D4 后半、D5 snoop 事务前半。
- 结论：理解 snoop 输入/输出和 cache line 状态变化的基本规则。

## 周期 10

- 范围：PDF p.226-250。
- 内容：D5 snoop 后半、D6 interconnect requirements 开始。
- 结论：补齐 snoop response、non-blocking 规则和 interconnect 的 coherent 责任。

## 周期 11

- 范围：PDF p.251-275。
- 内容：D6 interconnect、D7 cache maintenance 起始。
- 结论：学习 interconnect 的 ordering、response、main memory interaction、CMO 入口。

## 周期 12

- 范围：PDF p.276-300。
- 内容：D7 后半、D8 barrier、D9 exclusive accesses from ACE Managers 起始。
- 结论：学习 CMO、persistent CMO、write with CMO、barrier 和 ACE exclusive 基础。

## 周期 13

- 范围：PDF p.301-325。
- 内容：D9 结束、D10 snoop filtering、D11 ACE-Lite、D12 interface control、D13 DVM 起始。
- 结论：学习 ACE-Lite 能力、DVM 消息基础和过滤器职责。

## 周期 14

- 范围：PDF p.326-350。
- 内容：D13 DVM 后半、D14 recommendations、E1 Atomic transactions 开始。
- 结论：补齐 DVM 的 message / complete / synchronize，以及 AMBA 5 atomic transactions 的入口。

## 周期 15

- 范围：PDF p.351-375。
- 内容：E1 cache stashing、deallocating transactions、trace、loopback、QoS Accept、Wake-up、Coherency Connection、Untranslated transactions 开始。
- 结论：学习 AMBA 5 可选特性的控制面和互连面扩展。

## 周期 16

- 范围：PDF p.376-400。
- 内容：E1 untranslated、NSAID、read chunking、read interleaving、unique ID、MPAM、MTE、prefetch、WriteZero。
- 结论：补齐虚拟化、内存标签、预取和写零等 AMBA 5 能力。

## 周期 17

- 范围：PDF p.401-425。
- 内容：E1 additional interface properties、E2 interface and data protection、F1 ACE5、F2 ACE5-Lite 开始。
- 结论：学习 exclusive/shareable/size boundary 属性、Poison/Parity 保护，以及 ACE5/ACE5-Lite 的接口变更。

## 周期 18

- 范围：PDF p.426-450。
- 内容：F2 结束、F3 ACE5-LiteDVM、F4 ACE5-LiteACP 开始。
- 结论：学习 ACE5-LiteDVM 的 DVM 支持和 ACE5-LiteACP 的接口扩展。

## 周期 19

- 范围：PDF p.451-475。
- 内容：F5 changes in ACE5 and ACE5-Lite、G1-G3 appendices。
- 结论：学习 ACE5 家族变化与附录的信号矩阵、属性矩阵和 ID / response 编码。

## 周期 20

- 范围：PDF p.476-500。
- 内容：G4-G6 appendices、文档结尾。
- 结论：完成全书末尾的编码表与附录收束，形成从 p.1 到 p.500 的完整重学日志。

## 备注

- 这次重学从 PDF p.1 重新开始；此前已存在的 A3-A7 / D-E 内容记忆仍保留，但本日志把整本书重新按 20 个周期串起来。
- 若后续需要继续细化，可从本日志的周期 2 开始补写更细的 A1/A2 笔记，或从周期 14 开始重核 E1 Atomic transactions 的表格与编码。
