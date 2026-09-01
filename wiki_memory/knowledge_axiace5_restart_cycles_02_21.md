---
name: knowledge_axiace5_restart_cycles_02_21
description: 从 AXIACE5.pdf 第 1 页重新学习后的第 2-21 个周期进度日志
metadata:
  node_type: memory
  source: amba_prot/AXIACE5.pdf
  learned: 2026-08-21
  restart_from: PDF p.1
  cycles: 02-21
  note: cycle 1 stored separately in knowledge_axiace5_cycle_01_p1_25_restart.md
---

# 周期 2-21 进度日志

## 周期 2：PDF p.26-35

- A1.1-A1.3：AXI protocol 目标、架构和术语。
- A2.1-A2.2：全局信号与写地址通道信号。
- 结论：建立 AXI 五通道模型和基础信号入口。

## 周期 3：PDF p.36-45

- A2 剩余通道信号开始进入读/写数据与响应通道。
- A3.1-A3.2：clock/reset、basic read/write transactions。
- 结论：确认通道独立性和握手基线。

## 周期 4：PDF p.46-55

- A3.3-A3.4：channel relationships、transaction structure。
- A4.1-A4.2：transaction types 与 AXI3 memory attribute signaling。
- 结论：从基础事务过渡到 `AxCACHE` / `AxPROT` 语义入口。

## 周期 5：PDF p.56-65

- A4.3 开始：AXI4 changes to memory attribute signaling。
- 学到 `Modifiable` / `Non-modifiable`，以及 `AxCACHE[1]` 的行为变化。
- 结论：第一次把 `AxCACHE` 和事务可修改性绑定起来。

## 周期 6：PDF p.66-75

- A4.4-A4.7：memory types、mismatched memory attributes、transaction buffering、access permissions。
- 结论：`AxCACHE` 不只是缓存提示，还决定系统可见性与属性一致性。

## 周期 7：PDF p.76-85

- A4.8-A4.9：legacy considerations、usage examples。
- A5.1-A5.2：transaction identifiers 与 ID signals。
- A6.1-A6.2：ordering model 概览、memory locations/peripheral regions。
- 结论：建立 ID 与 ordering 的框架。

## 周期 8：PDF p.86-95

- A6.3-A6.7：transactions and ordering、observation/completion、manager guarantees、ordering requirements、response before endpoint。
- A7.1：single-copy atomicity size。
- 结论：区分“完成响应”与“可见性”，并进入原子性粒度。

## 周期 9：PDF p.96-105

- A7.2-A7.4：exclusive accesses、locked accesses、atomic access signaling。
- A8.1-A8.2：QoS、multiple region signaling。
- 结论：学会 `AxLOCK` 的 atomic / exclusive / locked 编码与约束。

## 周期 10：PDF p.106-115

- A8.3：user-defined signaling。
- A9.1-A9.3：interoperability principles、major interface categories、default signal values。
- 结论：AXI4 的默认信号与互操作规则开始成体系。

## 周期 11：PDF p.116-125

- A9.3 继续：Manager/Subordinate 默认地址相关规则。
- B1.1 开始：AXI4-Lite definition、signal list、bus width、write strobes、optional signaling。
- 结论：AXI4-Lite 的简化接口进入主线。

## 周期 12：PDF p.126-135

- B1.2-B1.4：interoperability、conversion rules、conversion/protection/detection。
- C1.1-C1.2 开始：AXI5 protocol overview 和 signal descriptions。
- 结论：AXI4-Lite 互操作与 AXI5 的接口升级一起建立。

## 周期 13：PDF p.136-145

- C1.2 继续：AXI5 additional signaling 的入口。
- C2 开始：AXI5-Lite 概览与接口边界。
- 结论：AXI5/AXI5-Lite 的可选特性轮廓开始出现。

## 周期 14：PDF p.146-155

- C2 继续：AXI5-Lite 的信号和兼容性。
- D1 开始：ACE about、ACE 基本一致性模型入口。
- 结论：从 AXI 过渡到 ACE coherent protocol。

## 周期 15：PDF p.156-165

- D1/D2：ACE 事务与信号描述。
- D3 开始：channel signaling。
- 结论：开始理解 ACE 的 snoop / coherency 通道基础。

## 周期 16：PDF p.166-175

- D3 继续：ACE channel signaling 的更完整规则。
- D4 开始：coherency transactions on read/write address channels。
- 结论：AXI 地址通道被赋予 ACE coherence 语义。

## 周期 17：PDF p.176-185

- D4 继续：coherency transactions 细节。
- D5 开始：snoop transactions。
- 结论：读写地址事务和 snoop 协同形成 coherent 行为。

## 周期 18：PDF p.186-195

- D5 snoop transactions 继续。
- D6 开始：interconnect requirements。
- 结论：理解 snoop response、WasUnique、non-blocking 与 interconnect 责任。

## 周期 19：PDF p.196-205

- D6 继续：sequencing、issuing snoops、transaction responses、interactions with main memory。
- D7 开始：cache maintenance。
- 结论：interconnect 与主存/缓存状态的协作规则开始成形。

## 周期 20：PDF p.206-215

- D7 cache maintenance 继续：CMO transactions、actions on receiving a CMO、cache maintenance attributes 和 propagation。
- 结论：学习 CMO 在 ACE 中如何传播和落地。

## 周期 21：PDF p.216-225

- D7 后半：persistent CMO、write with cache maintenance、ACE Managers and CMOs。
- D8 开始：barrier transactions。
- 结论：cache maintenance 与 barrier 语义连接起来，进入更强顺序控制。

## 总结

- 从 PDF p.1 重新开始后，第 2-21 个周期已经覆盖 AXI 结构、信号、事务属性、ID、ordering、atomic/exclusive、AXI4-Lite、AXI5、ACE 以及 cache maintenance / barrier 的前半主线。
- 下一步应继续从 D8 barrier 往后推进，进入 D9-D14、E1-E2 以及 F 章。
