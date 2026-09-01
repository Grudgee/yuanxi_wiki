---
name: knowledge_axiace5_axi5_ace_boundary
description: 重新从 AXIACE5.pdf 学习的 AXI4-Lite、AXI5、AXI5-Lite 与 ACE 边界摘要
metadata:
  node_type: memory
  source: amba_prot/AXIACE5.pdf
  relearned: 2026-08-21
  source_pages: PDF 119-238
---

# AXI4-Lite、AXI5、AXI5-Lite 与 ACE 边界

## AXI4-Lite

- AXI4-Lite 是 AXI4 的简化子集，用于简单寄存器式访问。
- 数据总线宽度固定为 32-bit 或 64-bit，所有事务宽度与数据总线一致。
- 不支持 burst，burst length 固定为 1。
- 不支持 AXI ID，事务必须按顺序处理；可选 ID reflection 用于与 full AXI 连接。
- 不支持 exclusive accesses。
- `AxCACHE` 被丢弃，所有事务按 Non-modifiable、Non-bufferable 处理。

## AXI5 / AXI5-Lite 边界

- C1/C2 引入 AXI5 与 AXI5-Lite 的 optional 属性和信号组合。
- 本轮边界摘要确认：AXI5 章节中出现 `Atomic_Transactions` 属性和 `AWATOP` 信号说明入口，但没有在本轮展开 E1 的 atomic transaction 详细语义。
- 因此回答 AXI5 `AWATOP`、compare/swap/fetch 类原子操作时，必须重新抽取 E1 对应页后再答，不能从旧迁移记忆或常识直接补全。
- AXI5-Lite 仍是 Lite 风格接口，强调简化访问；与 AXI4-Lite 一样不应默认支持 exclusive。

## ACE 边界

- ACE 在 AXI 基础上增加一致性相关通道、snoop 事务、domain 与 cache state 语义。
- ACE 中 `AxCACHE` 与 `AxDOMAIN` 的组合会影响合法性和 cache 访问范围。
- Device transaction（`AxCACHE[1]=0`）只能使用 System domain。
- Cacheable transaction（`AxCACHE[3:2] != 0`）不能使用 System domain。
- ACE 的 Clean/Make/Write/Evict、snoop data state、CleanUnique 等细节没有在本轮完整重写；若用户问 ACE 一致性，应重新抽对应 D 章页。

## 回答边界

- 可以直接回答：AXI4-Lite 不支持 exclusive、丢弃 `AxCACHE` 并按 Non-modifiable/Non-bufferable 处理。
- 可以直接回答：AXI5 文档有 `Atomic_Transactions`/`AWATOP` 入口，但本地重学记忆尚未覆盖 E1 详细语义。
- 不能直接回答：AXI5 atomic transaction 的完整编码、返回语义、操作集合；需继续抽取 E1。
