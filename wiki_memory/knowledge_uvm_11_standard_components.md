---
name: knowledge_uvm_11_standard_components
description: UVM 1.2 标准组件源码机制：driver/monitor/agent/env/test/subscriber/scoreboard/comparators 与 analysis 连接
metadata:
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T07:54:25.213Z
---

# UVM 1.2 Standard Components 源码学习记忆

## 学习范围与入口

本节继续 [[knowledge_uvm_index]]，聚焦 `src/comps` 中最常用的标准组件骨架与比较器。入口文件 `src/comps/uvm_comps.svh` 依次 include：

1. `uvm_pair.svh`
2. `uvm_policies.svh`
3. `uvm_in_order_comparator.svh`
4. `uvm_algorithmic_comparator.svh`
5. `uvm_random_stimulus.svh`
6. `uvm_subscriber.svh`
7. `uvm_monitor.svh`
8. `uvm_driver.svh`
9. `uvm_push_driver.svh`
10. `uvm_scoreboard.svh`
11. `uvm_agent.svh`
12. `uvm_env.svh`
13. `uvm_test.svh`

源码依据：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/comps/uvm_comps.svh:23-36`。

## 1. 这些基类本身很薄，但语义很重要

`uvm_monitor`、`uvm_scoreboard`、`uvm_env`、`uvm_test` 基本都是“标识性基类”：

- 它们几乎不加行为，只提供稳定的类型层次和未来扩展点。
- `uvm_env` 是层次容器。
- `uvm_test` 是 `run_test` 的入口类型，用于 `+UVM_TESTNAME` 选择测试。
- `uvm_scoreboard` 只是 scoreboarding 的语义基类。
- `uvm_monitor` 是采样/观察组件的语义基类。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/comps/uvm_monitor.svh:24-52`、`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/comps/uvm_scoreboard.svh:23-54`、`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/comps/uvm_env.svh:23-51`、`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/comps/uvm_test.svh:23-80`。

## 2. `uvm_driver`: sequencer-facing request/response shell

`uvm_driver #(REQ,RSP)` 是 driver 的标准基类，它本身不实现 bus 行为，只提供 sequencer 连接端口与常用字段：

- `seq_item_port`：`uvm_seq_item_pull_port #(REQ,RSP)`，用于从 sequencer 拉取 request。
- `seq_item_prod_if`：`seq_item_port` 的别名。
- `rsp_port`：`uvm_analysis_port #(RSP)`，用于把 response 送回 sequencer 的 analysis export，或连接外部观察者。
- `req` / `rsp`：常用类型化成员，供派生 driver 保存当前 request/response。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/comps/uvm_driver.svh:27-87`。

### 关键理解

driver 的作用不是“自己决定发什么”，而是作为 sequencer 拉取机制的执行端：

- 正常路径是 `driver.seq_item_port.connect(sequencer.seq_item_export)`。
- 如果 driver 需要发 response，可以用 `rsp_port`，或通过 `item_done(response)` / `put(response)` 让 sequencer 路由回 sequence。

## 3. `uvm_agent`: active/passive 的结构分界

`uvm_agent` 是最重要的标准结构基类之一。它本身只保存 `is_active`，但 `build_phase()` 会从 resource pool 读取配置并把它转成 `uvm_active_passive_enum`。

要点：

- 默认 `is_active = UVM_ACTIVE`。
- `build_phase()` 使用 `uvm_resource_pool::lookup_name(get_full_name(), "is_active", null, 0)`。
- 它接受多种类型的配置值：`uvm_active_passive_enum`、`uvm_integral_t`、`uvm_bitstream_t`、`int`、`int unsigned`、`string`。
- `get_is_active()` 默认直接返回 `is_active`，但可覆盖为更复杂逻辑。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/comps/uvm_agent.svh:25-134`。

### 关键理解

active agent 通常包含：driver + sequencer + monitor；passive agent 通常只保留 monitor。

这不是约束性的硬规则，而是 UVM 推荐的标准组织方式。

## 4. `uvm_subscriber`: analysis 终端

`uvm_subscriber #(T)` 是 analysis 流的标准终点基类：

- 内部持有 `uvm_analysis_imp #(T,this_type) analysis_export`。
- 子类必须实现 `write(T t)`。
- 常用于 coverage collector、简单观察器或轻量 sink。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/comps/uvm_subscriber.svh:23-66`。

关键点：subscriber 的语义是“订阅 analysis 事务”，而不是普通组件；它直接暴露 `analysis_export` 给 monitor 或其他 publisher 连接。

## 5. Comparators：从 analysis 流直接构建检查链

### 5.1 `uvm_in_order_comparator`

`uvm_in_order_comparator #(T,comp_type,convert,pair_type)` 是两个按顺序到达的数据流比较器：

- `before_export` / `after_export` 都是 `uvm_analysis_export #(T)`。
- 内部使用两个 `uvm_tlm_analysis_fifo #(T)` 缓冲数据。
- `run_phase()` 中阻塞取出一对事务，调用 `comp_type::comp(b,a)` 比较。
- 每对结果会通过 `pair_ap` 发送出去。
- `flush()` 清零统计计数，FIFO 清理由 `uvm_tlm_fifo::flush()` 负责。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/comps/uvm_in_order_comparator.svh:23-259`。

### 5.2 `uvm_algorithmic_comparator`

`uvm_algorithmic_comparator #(BEFORE,AFTER,TRANSFORMER)` 是对 in-order comparator 的封装：

- `before_export` 接收 BEFORE 类型。
- `after_export` 接收 AFTER 类型。
- `m_transformer.transform(b)` 把 BEFORE 转成 AFTER。
- 然后复用内部的 `uvm_in_order_class_comparator #(AFTER)`。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/comps/uvm_algorithmic_comparator.svh:23-134`。

### 关键理解

- comparator 不是 passive “日志器”，而是会在 run_phase 持续消费两个流并做同步比较。
- analysis_fifo 是这里的关键缓冲器，因为两个流可能不同步到达。
- algorithmic comparator 体现了“先转换、后比较”的常见验证模式。

## 6. `uvm_scoreboard`

`uvm_scoreboard` 本身几乎没有行为，只是一个语义基类。真正的 scoreboard 实现通常会：

- 从 monitor 或 subscriber 接收 analysis 事务；
- 做预测值与实际值比较；
- 记录错误计数与覆盖信息；
- 在结束时汇总结果。

它的存在更多是为了类型区分和未来扩展，而不是提供现成算法。

## 7. 本轮最重要的结论

1. 标准组件基类大多很薄，但它们定义了 UVM 推荐的结构角色。
2. `uvm_driver` 只提供 sequencer 连接 shell，不定义具体总线行为。
3. `uvm_agent` 是 active/passive 模式的配置入口，且 `is_active` 是通过 resource pool 读取的。
4. `uvm_subscriber` 是 analysis 流的标准终点，必须实现 `write()`。
5. comparator 家族展示了 analysis port + analysis FIFO 的典型检查链：双流同步、缓冲、比较、统计、发布 pair。

## 8. 下一步建议

继续学习 `push driver` 与 `random stimulus`，然后转向 register model。`push driver` 可以补齐 sequencer-driver 通信的另一种风格；reg model 则是前面大部分机制的综合应用。
