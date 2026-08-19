---
name: knowledge_a8_additional_signaling
 description: AMBA4 AXI4 的 QoS、多 region 和 user-defined 信号
---

# 知识点摘要

- AXI4 可通过 `AWQOS`/`ARQOS` 提供 4-bit QoS 标识；规范建议把它作为优先级提示，高值表示高优先级，但不规定具体 QoS 算法。
- AXI4 可通过 `AWREGION`/`ARREGION` 提供 4-bit region 标识，最多区分 16 个 4KB 内 region；region 标识用于同一物理接口提供多个逻辑接口。
- User signals 是 AMBA4 引入的用户自定义信号；AMBA5 进一步统一了使用和配置，但协议不定义这些信号的功能，互操作性由系统设计者负责。

# 关键细节

## QoS

- `AWQOS` 在写地址通道发送，`ARQOS` 在读地址通道发送；统称 `AxQOS`。
- 默认 `0b0000` 表示接口不参与 QoS 方案。Manager 可为不同 traffic stream 生成不同值；不能生成自身 QoS 值的 Manager 必须使用默认值。
- 系统级默认行为是在没有 AXI 顺序约束的情况下，优先处理 QoS 值更高的事务；AXI ordering rules 优先于 QoS 的重排需求。
- QoS 需要系统各组件对方案有共同理解；interconnect 可以用可编程寄存器重新分配连接 Manager 的 QoS 值。

## Multiple region

- `AWREGION`/`ARREGION` 为 4-bit region ID，region ID 在 4KB 地址空间内必须保持不变，可编码高阶地址位。
- 一个 Subordinate 的单一物理接口可借助 region 提供多个逻辑接口，Subordinate 不必对不同逻辑接口间的地址做 decode。
- region 可用于：同一外设的数据路径和控制寄存器位于不同地址；同一 Subordinate 的不同 region 具有不同访问行为（如一个可读写、另一个只读）。
- Subordinate 必须保持不同 region 的正确协议和排序；同一 ID 对不同 region 的响应仍按正确顺序返回。
- region 信号只编码既有地址空间的 decode 结果，不创建新的独立地址空间；只有位于地址 decode 下游的接口才能提供 `AxREGION`。不支持的 region 必须返回错误或 alias 到受支持 region。

## User-defined signaling

- User signal 可附加到事务请求、事务响应或读/写事务的每个 beat；规范通常不建议使用，因为相同 User signal 被不同组件解释不一致会产生互操作问题。
- 配置属性：`USER_REQ_WIDTH` 范围 0–128，作用于 `AWUSER`/`ARUSER`；`USER_DATA_WIDTH` 范围 0–`DATA_WIDTH/2`，作用于 `WUSER`/`RUSER`；`USER_RESP_WIDTH` 范围 0–16，作用于 `BUSER`/`RUSER`。属性为 0 时对应信号不存在。
- 信号：`ARUSER` 是读请求属性，`AWUSER` 是写请求属性，`WUSER` 是写数据属性，`RUSER` 是读数据/响应属性，`BUSER` 是写响应属性；各自遵守对应通道的 VALID 规则。
- 可只在部分通道实现 User signals。为便于宽度转换，建议 `USER_DATA_WIDTH` 是字节宽度的整数倍；响应位在每个 beat 相同，`RUSER` 低位承载事务级响应信息，高位承载每 beat 数据属性。

# 原文引用

- 文档：`/home/yuanxi/yuanxi_cc/wiki_files/amba_prot/AXIACE5.pdf`
- 版本：ARM IHI 0022H.c
- PDF 物理页：101–106；印刷页 A8-101–A8-106。
- 章节：Chapter A8 `AMBA 4 Additional Signaling`；A8.1–A8.3。

# 适用条件与例外

- QoS 值是提示，不是独立的排序许可；协议规定的顺序保证优先。
- region ID 不是新的地址 decode 空间，只能在既有 decode 之后使用。
- User signals 的实际意义由实现定义，跨组件连接前必须定义一致的协议。

# 关联章节

- Chapter A4 Transaction Attributes
- Chapter A6 AXI Ordering Model
- Chapter A9 Default Signaling and Interoperability
- ACE5/AXI5 User signal configuration

# 待核验问题

- 继续学习 A9 的完整默认信号表和接口互操作规则。
