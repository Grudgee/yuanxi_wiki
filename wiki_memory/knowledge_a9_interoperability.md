---
name: knowledge_a9_interoperability
 description: AXI 接口互操作类别、可选信号、默认值和地址/事务约束
---

# 知识点摘要

- AXI 组件应支持所有合法输入组合，但不必生成所有输出组合；例如 Subordinate 可以支持所有 burst 长度，而 Manager 只生成自己需要的 burst 类型。
- 输出信号只有在组件可能需要偏离默认值时才必须存在；输入信号在 Manager/Subordinate 不需要观察时可以省略。Interconnect 也可省略始终为默认值或目的地不使用的信号。
- 读写接口包含 AR、R、AW、W、B 五通道；只读接口仅 AR/R，不支持 exclusive；只写接口仅 AW/W/B，也不支持 exclusive。
- Memory Subordinate 必须正确处理所有事务类型；Peripheral Subordinate 只需正确处理其数据手册规定的访问方式，其他访问仍必须协议合规完成以避免死锁，但不要求继续正确工作。

# 关键细节

## 默认信号

- Manager 写通道常见默认值：`AWID=0`、`AWREGION=0`、`AWLEN=0`（长度 1）、`AWSIZE=数据总线宽度`、`AWBURST=INCR`、`AWLOCK=0` 普通访问、`AWCACHE=0b0000`、`AWQOS=0`、`WSTRB=全 1`。`ACLK`、`ARESETn`、`AWADDR`、`AWPROT`、`AWVALID`、`AWREADY`、`WDATA`、`WLAST`、`WVALID`、`WREADY` 等通常为必需信号。
- Manager 读通道常见默认值：`ARID=0`、`ARREGION=0`、`ARLEN=0`（长度 1）、`ARSIZE=数据总线宽度`、`ARBURST=INCR`、`ARLOCK=0`、`ARCACHE=0b0000`、`ARQOS=0`；`ARADDR`、`ARPROT`、`ARVALID`、`ARREADY`、`RDATA`、`RVALID`、`RREADY` 等为必需信号。
- Memory Subordinate 输入中，`BRESP`/`RRESP` 可选时默认 `0b00=OKAY`；其他信号的省略和默认应按 A9 表格与接口类别决定。

## 地址宽度

- Manager 没有最小地址位数要求。系统地址比 Manager 更宽时，高位补 0；系统更窄时，Manager 高位地址可不连接。典型 Manager 提供 32 位地址，可选支持 64 位。
- Subordinate 没有最小地址位数要求；可假设低位地址为 0 以支持数据总线宽度 decode，interconnect 提供的更高地址位也可默认补 0。典型 Memory Subordinate 至少能完整 decode 4KB 地址范围。

## 信号省略的约束

- Memory Subordinate 可以不使用 `AxLOCK`，除非支持 exclusive；可以不使用 `AxCACHE`，前提是没有 cache 或所有事务以相同方式缓存。
- Manager 如果始终执行完整数据宽度写，可以不使用 `WSTRB`，默认全 1；Subordinate 可以不使用 `WLAST`，因为可由 `AWLEN[7:0]` 推导最后 beat；Manager 可以不使用 `RLAST`，因为可由 `ARLEN[7:0]` 推导。
- Manager 可省略 `RRESP`/`BRESP` 输入，条件是不用 exclusive 且不需要事务错误通知。Subordinate 可省略这些响应输出，条件是不支持 exclusive 且不产生错误响应。
- 不区分 Secure/Non-secure 且无需额外保护的 Subordinate 可省略 `AxPROT`，但错误设置 `AxPROT[1]` 会导致系统安全属性错误，必须谨慎。

# 原文引用

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 版本：ARM IHI 0022H.c
- PDF 物理页：107–118；印刷页 A9-107–A9-116，Part B 起始页 A9-117–A9-118。
- 章节：Chapter A9 `Default Signaling and Interoperability`；A9.1–A9.3.7。

# 适用条件与例外

- 默认值和信号是否 Required/Optional 与接口类型、Manager/Subordinate 方向有关；不能把 Manager 的默认值直接套到 Subordinate 输入。
- 只读/只写接口不支持 exclusive access。
- 本批次已覆盖 A9-107–A9-116；A9 表格后续页如有补充需继续核对。

# 关联章节

- Chapter A2 Signal Descriptions
- Chapter A7 Atomic Accesses
- Chapter A8 AMBA4 Additional Signaling
- Part B AMBA AXI4-Lite Interface Specification

# 待核验问题

- 继续核对 A9 后续页以及 Part B AXI4-Lite 的接口限制。
