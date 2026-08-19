---
name: knowledge_uvm_9_tlm2
description: UVM 1.2 TLM2 源码机制：transport interface、sockets、generic payload、time、extensions 与连接语义
metadata:
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T07:38:17.215Z
---

# UVM 1.2 TLM2 源码学习记忆

## 学习范围与入口

本节继续 [[knowledge_uvm_index]] 的 TLM2 阶段，聚焦 `src/tlm2` 的接口、ports/exports/imps、socket 基类、socket 连接语义、canonical time 和 generic payload。源码入口是 `src/tlm2/uvm_tlm2.svh`，其 include 顺序是：

1. `uvm_tlm2_defines.svh`
2. `uvm_tlm2_time.svh`
3. `uvm_tlm2_generic_payload.svh`
4. `uvm_tlm2_ifs.svh`
5. `uvm_tlm2_imps.svh`
6. `uvm_tlm2_ports.svh`
7. `uvm_tlm2_exports.svh`
8. `uvm_tlm2_sockets_base.svh`
9. `uvm_tlm2_sockets.svh`

源码依据：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm2/uvm_tlm2.svh:21-29`。

## 1. TLM2 的接口模型

TLM2 的核心接口在 `uvm_tlm_if #(T, P)`：

- `nb_transport_fw(T t, ref P p, input uvm_tlm_time delay)`
- `nb_transport_bw(T t, ref P p, input uvm_tlm_time delay)`
- `b_transport(T t, uvm_tlm_time delay)`

默认实现都会报错并返回占位值，因此真正的行为必须由 initiator/target 组件或 socket imp 提供。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm2/uvm_tlm2_ifs.svh:70-176`。

### 关键语义

- `b_transport` 表示阻塞事务；返回即表示事务完成。
- `nb_transport_fw/bw` 表示非阻塞前向/后向路径，靠 `phase` 和 `delay` 表示协议状态与时间点。
- `uvm_tlm_sync_e` 的三种返回值是协议握手结果：`ACCEPTED / UPDATED / COMPLETED`。
- `uvm_tlm_phase_e` 的阶段值是 `BEGIN_REQ / END_REQ / BEGIN_RESP / END_RESP`，加上 `UNINITIALIZED_PHASE`。

## 2. canonical time：`uvm_tlm_time`

`uvm_tlm_time` 不是简单的 `time` 包装，而是用于跨 timescale 的 canonical 时间容器。

关键点：

- 内部保存 `m_res`、`m_time`、静态默认分辨率 `m_resolution`。
- `incr/decr/get_realtime/get_abstime/set_abstime` 都显式接受 timescale 相关参数。
- `null` delay 会被 TLM2 宏拦下并报错。
- 负增量、非法 scale、减到负数都会报错。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm2/uvm_tlm2_time.svh:31-194`。

### 结论

TLM2 的 delay 不是“顺手传个 time”，而是为了让不同 timescale 的 initiator/target 能在同一协议里对齐时间语义。

## 3. generic payload：标准事务载体

`uvm_tlm_generic_payload` 继承 `uvm_sequence_item`，是 TLM2 的默认 transaction 类型。

关键字段：

- `m_address`：地址
- `m_command`：`UVM_TLM_READ_COMMAND / WRITE / IGNORE`
- `m_data[]`：数据字节数组
- `m_length`：数据长度
- `m_response_status`：响应状态
- `m_dmi`：DMI hint，UVM 子集里未真正支持
- `m_byte_enable[]` / `m_byte_enable_length`
- `m_streaming_width`
- `m_extensions`：按类型句柄存储的扩展

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm2/uvm_tlm2_generic_payload.svh:93-363`。

### 响应状态

`UVM_TLM_OK_RESPONSE` 为正值，其他错误响应为 0 或负值，因此：

- `is_response_ok()` 通过 `int'(m_response_status) > 0` 判断。
- `is_response_error()` 就是 `!is_response_ok()`。
- `get_response_string()` 把枚举转成可读字符串。

### Extensions

扩展机制是 generic payload 的重点：

- `uvm_tlm_extension_base` 定义 `get_type_handle()` / `get_type_handle_name()`。
- `uvm_tlm_extension#(T)` 通过静态单例 `ID()` 提供类型唯一句柄。
- 一个 payload 对同一扩展类型只能有一个实例。
- `set_extension/get_extension/clear_extension/clear_extensions` 通过类型句柄访问。

### 常见坑

- `m_command`、`m_length`、`m_response_status` 的默认值不是“业务正确值”，只是构造初值。
- `IGNORE_COMMAND` 用于零长度传输，不应靠 `m_length==0` 表达普通数据事务。
- byte enable 和 streaming width 都需要 target 按协议语义处理，否则应返回标准错误。

## 4. sockets：initiator / target / pass-through / termination

TLM2 的 socket 继承层次很有规律：

- termination socket：路径终点
- pass-through socket：跨层级连接用
- initiator/target：按 forward path 方向分类
- blocking / nonblocking：按协议能力分类

`uvm_tlm2_sockets_base.svh` 先定义 base class，derived class 再负责具体 connect 语义。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm2/uvm_tlm2_sockets_base.svh:21-193`、`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm2/uvm_tlm2_sockets.svh:21-431`。

### blocking socket

- `uvm_tlm_b_initiator_socket`：forward port，不能有 backward path。
- `uvm_tlm_b_target_socket`：forward imp，由组件实现 `b_transport`。
- `connect()` 只允许与兼容的 target / passthrough socket 连接，否则报类型错误。

### nonblocking socket

- `uvm_tlm_nb_initiator_socket`：forward port + backward imp。
- `uvm_tlm_nb_target_socket`：forward imp + backward port。
- pass-through initiator：`HAS-A bw_export`
- pass-through target：`HAS-A bw_port`

### 连接语义

连接不是纯类型匹配，而是“socket 类型 + 方向 + 是否 passthrough”的组合判断：

- initiator connect target/pass-through target/termination target
- target termination socket 不能再继续 connect
- nonblocking initiator 连到 pass-through/target 时，还会顺带把 backward path 的 bw imp/export 接上

这说明 TLM2 socket 把 forward path 和 backward path 分离管理，而不是像 TLM1 那样只靠单一 interface 角色。

## 5. TLM2 imps：宏把方法绑定到组件实现

`uvm_tlm2_imps.svh` 通过宏把 `nb_transport_fw/bw`、`b_transport` 包装成真正的类方法，并在入口处检查 `delay == null`。

关键宏：

- `UVM_TLM_NB_TRANSPORT_FW_IMP`
- `UVM_TLM_NB_TRANSPORT_BW_IMP`
- `UVM_TLM_B_TRANSPORT_IMP`

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm2/uvm_tlm2_imps.svh:41-149`。

关键点：

- socket/imp 只是把调用转交给 `m_imp`。
- 若 delay 为空，直接报 `UVM/TLM/NULLDELAY`。
- 这样把协议层错误尽量前移到调用入口。

## 6. 本轮最重要的结论

1. TLM2 的核心不是“更高级的 port”，而是“协议化的 transport 接口 + direction-aware socket”。
2. `uvm_tlm_time` 是 TLM2 能跨 timescale 协同的关键。
3. `uvm_tlm_generic_payload` 把地址、命令、数据、状态、byte enable、streaming、extension 统一成标准事务对象。
4. TLM2 socket 的 connect 规则体现了 forward/backward 路径分离和 pass-through 机制。

## 7. 下一步建议

继续学习两条线：

- **Sequencer / Sequence**：理解 sequence_item、sequence_base、sequencer_base、arbitration、default sequence。
- **更细的 TLM2 使用模式**：generic payload 的 extensions、阻塞/非阻塞协议如何映射到真实 master/target 组件。

其中 sequencer/sequence 是和 TLM2 之后最自然的下一层入口。
