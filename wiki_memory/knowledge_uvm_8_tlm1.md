---
name: knowledge_uvm_8_tlm1
description: UVM 1.2 TLM1 源码机制：ports/exports/imps、analysis port、FIFO、req/rsp channel 与连接语义
metadata:
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T07:31:26.587Z
---

# UVM 1.2 TLM1 源码学习记忆

## 学习范围与入口

本节继续 [[knowledge_uvm_index]] 的 TLM1 阶段，聚焦 `src/tlm1` 的端口、导出、imp、analysis port、FIFO 和 request/response channel。源码入口是 `src/tlm1/uvm_tlm.svh`，其 include 顺序是：

1. `uvm_tlm_ifs.svh`
2. `uvm_sqr_ifs.svh`
3. `base/uvm_port_base.svh`
4. `uvm_tlm_imps.svh`
5. `uvm_imps.svh`
6. `uvm_ports.svh`
7. `uvm_exports.svh`
8. `uvm_analysis_port.svh`
9. `uvm_tlm_fifo_base.svh`
10. `uvm_tlm_fifos.svh`
11. `uvm_tlm_req_rsp.svh`
12. `uvm_sqr_connections.svh`

源码依据：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm1/uvm_tlm.svh:23-39`。

## 1. TLM1 的三类连接件：port / export / imp

TLM1 的核心是把同一种接口能力分成三种角色：

- `port`：上游使用者，需要连接到下游实现。
- `export`：中继/提升子层实现。
- `imp`：终点实现，直接委托给组件自身的方法。

`uvm_ports.svh` 与 `uvm_exports.svh` 都是围绕 `uvm_port_base #(uvm_tlm_if_base #(…))` 生成具体类型；`uvm_imps.svh` 则把调用落到 `m_imp` 或 `m_req_imp/m_rsp_imp` 上。所有这些类都靠宏把接口方法展开到成员上，而不是手写重复的委托代码。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm1/uvm_ports.svh:32-261`、`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm1/uvm_exports.svh:32-258`、`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm1/uvm_imps.svh:32-315`。

### 关键语义

- `port` 由主动发起方持有，默认必须连到至少一个实现。
- `export` 用来把下层实现“向上推广”。
- `imp` 只能作为终点，不能再往别的 port/export 连接。
- `min_size/max_size` 仍由 `uvm_port_base` 统一检查，具体 TLM 类只是设定 mask 和方法展开。

## 2. 分层委托的真正落点：`uvm_tlm_if_base`

`uvm_analysis_port`、`uvm_put_port`、`uvm_get_port`、`uvm_*_imp` 等类最终都通过 `uvm_tlm_if_base` 抽象调用接口方法。连接建立后，`write/put/get/peek/transport` 之类调用都先经 `get_if(i)` 或 `m_if`，再落到具体实现。

这意味着 TLM1 的行为不是“类层级继承”决定的，而是“端口图 + 运行时分发”决定的。

## 3. Analysis port：广播而不是单点发送

`uvm_analysis_port` 与其他 port 最大的区别是广播：`write()` 会遍历所有连接的 interface，依次调用每个订阅者。

源码要点：

- 构造函数把 `max_size` 设为 `UVM_UNBOUNDED_CONNECTIONS`。
- `write()` 里按 `this.size()` 遍历每个连接，若某个槽为 null 就 fatal。
- `uvm_analysis_export` 也做同样的广播，因此 analysis 链可以多级透传。
- `uvm_analysis_imp` 只是把 `write()` 委托给父组件的 `write()` 方法。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm1/uvm_analysis_port.svh:35-153`。

常见坑：analysis transaction 是“广播副本/引用语义”的消费者接口，subscriber 的 `write()` 不应修改传入对象，否则会影响其他订阅者。

## 4. FIFO：用 mailbox 作为阻塞存储

`uvm_tlm_fifo#(T)` 是最基础的事务缓冲：内部用 `mailbox #(T)` 存数据，`put/get/peek/try_*` 全都围绕 mailbox 封装。

关键行为：

- `put()` → `m.put(t)` 后调用 `put_ap.write(t)`。
- `get()` → `m.get(t)` 后调用 `get_ap.write(t)`，并用 `m_pending_blocked_gets` 跟踪阻塞 get。
- `try_put()` / `try_get()` 成功后也会触发对应 analysis port。
- `can_put/can_get/can_peek` 直接反映 mailbox 状态和 pending blocked get 数量。
- `flush()` 通过循环 `try_get()` 清空 FIFO，若存在 blocked gets 会报错。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm1/uvm_tlm_fifos.svh:46-186`。

`uvm_tlm_fifo_base` 则只定义接口和默认错误实现：它把 `put_export`、`get_peek_export`、`put_ap`、`get_ap` 建好，再通过 `blocking_*` / `nonblocking_*` 别名提供兼容入口。`build_phase()` 里只调用 `build()`，明确关闭自动配置语义。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm1/uvm_tlm_fifo_base.svh:32-252`。

## 5. `uvm_tlm_analysis_fifo`

`uvm_tlm_analysis_fifo#(T)` 是一个分析接收缓冲：

- 继承 `uvm_tlm_fifo#(T)`。
- 构造时把容量设为 0，表示无界。
- 提供 `analysis_export`，类型是 `uvm_analysis_imp #(T, uvm_tlm_analysis_fifo #(T))`。
- `write()` 直接 `try_put()`，因为无界 FIFO 理论上不会失败。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm1/uvm_tlm_fifos.svh:189-238`。

## 6. req/rsp channel：把多个 TLM 端口组合成一组 FIFO + analysis 视图

`uvm_tlm_req_rsp_channel#(REQ,RSP)` 是典型的组合型 TLM 组件：它把 request FIFO 和 response FIFO 封装在一个 component 里，并同时暴露多种接口。

结构：

- `put_request_export` / `get_peek_response_export`
- `get_peek_request_export` / `put_response_export`
- `request_ap` / `response_ap`
- `master_export` / `slave_export`
- 兼容别名 `blocking_*` / `nonblocking_*`

构造函数会：

1. new 两个内部 FIFO。
2. new 两个 analysis port。
3. new 四个主导出口。
4. 用 `create_aliased_exports()` 建立所有旧名字别名。
5. 关闭 connection error 的 report action。

connect_phase 里把 export 接到内部 FIFO，再把 FIFO 的 `put_ap` 接到外部 analysis port。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm1/uvm_tlm_req_rsp.svh:32-289`。

### 关键理解

- 这个 channel 把“请求进入 FIFO”、“响应从 FIFO 取出”、“观察请求/响应流”三件事都整合了。
- `master_export` 和 `slave_export` 用 `uvm_master_imp` / `uvm_slave_imp` 组合 request 与 response 两侧接口。
- alias 不是新行为，只是旧 API 名称的重绑定。

## 7. 本轮最重要的结论

1. TLM1 的本质是“接口角色 + 端口图 + 运行时委托”，不是单纯的类继承。
2. `analysis_port` 是广播型，`put/get/peek` 是单点型；两者在连接和调用语义上完全不同。
3. `uvm_tlm_fifo` 依赖 mailbox 做阻塞同步，`analysis_fifo` 只是无界 FIFO 的分析包装。
4. `uvm_tlm_req_rsp_channel` 是理解 UVM TLM1 组合模式的最好例子：它把多个基础接口拼成一个可复用 channel。

## 8. 下一步建议

继续学 `tlm2`，重点看：

- `uvm_tlm_if` 中的 `b_transport` / `nb_transport_fw` / `nb_transport_bw`
- initiator/target、pass-through/termination socket 的继承与 HAS-A 组合
- `uvm_tlm_generic_payload` 的 command/status/extension 语义

这部分会把 TLM1 的“单点/广播/FIFO”扩展到标准 TLM2 socket 与协议语义。
