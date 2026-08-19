---
name: knowledge_uvm_10_sequences_sequencer
description: UVM 1.2 sequence/sequencer 源码机制：sequence_item、start/start_item/finish_item、arbitration、driver pull interface、response routing、default sequence
metadata:
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T07:48:27.171Z
---

# UVM 1.2 Sequences / Sequencer 源码学习记忆

## 学习范围与入口

本节继续 [[knowledge_uvm_index]]，聚焦 `src/seq` 与 `src/tlm1` 中 sequencer-driver 连接相关源码。`src/seq/uvm_seq.svh` 的 include 顺序是：

1. `seq/uvm_sequence_item.svh`
2. `seq/uvm_sequencer_base.svh`
3. `seq/uvm_sequencer_analysis_fifo.svh`
4. `seq/uvm_sequencer_param_base.svh`
5. `seq/uvm_sequencer.svh`
6. `seq/uvm_push_sequencer.svh`
7. `seq/uvm_sequence_base.svh`
8. `seq/uvm_sequence.svh`
9. `seq/uvm_sequence_library.svh`
10. `seq/uvm_sequence_builtin.svh`

并在末尾 typedef 默认 sequence/sequencer/driver 类型。

源码依据：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/seq/uvm_seq.svh:23-40`。

## 1. `uvm_sequence_item`：item 与 sequence 共用的上下文基类

`uvm_sequence_item` 继承 `uvm_transaction`，不仅是用户 request/response item 的基类，也是 `uvm_sequence_base` 的基类。因此 sequence 本身也是一种 sequence item，只是 `is_item()` 返回 0。

关键成员：

- `m_sequence_id`：由 sequencer 分配，用于 response 路由。
- `m_sequencer`：当前默认 sequencer。
- `m_parent_sequence`：父 sequence。
- `m_depth`：sequence 嵌套深度。
- `m_use_sequence_info`：控制打印/拷贝/记录时是否带 sequence 上下文。

关键 API：

- `set_item_context(parent_seq, sequencer)`：设置父 sequence、sequencer、深度并 reseed。
- `set_id_info(item)`：把 request 的 `sequence_id` 和 `transaction_id` 拷到 response；driver 应使用它初始化 response。
- `get_full_name()`：优先基于 parent sequence，其次基于 sequencer 拼出名字。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/seq/uvm_sequence_item.svh:28-259`。

## 2. `uvm_sequence_base::start()` 生命周期

`uvm_sequence_base` 是真正的 sequence 执行状态机。`start(sequencer,parent_sequence,priority,call_pre_post)` 的主要流程：

1. `set_item_context(parent_sequence, sequencer)`。
2. 检查当前状态必须是 `CREATED / STOPPED / FINISHED`。
3. 若有 parent sequence，登记到 parent 的 children array。
4. 确定 priority：root 默认 100，child 默认继承 parent priority。
5. 清空 response queue。
6. 如有 sequencer，注册 sequence 并开启 transaction recording。
7. 状态进入 `PRE_START`，fork 内依次执行：
   - `pre_start()`
   - `pre_body()`，仅当 `call_pre_post==1`
   - parent 的 `pre_do(0)` / `mid_do(this)`，如果有 parent
   - `body()`
   - parent 的 `post_do(this)`，如果有 parent
   - `post_body()`，仅当 `call_pre_post==1`
   - `post_start()`
8. 如启用 automatic phase objection，则 start 前 raise、结束后 drop。
9. 结束 transaction recording，通知 sequencer `m_sequence_exiting()` 清理队列。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/seq/uvm_sequence_base.svh:23-132`、`:258-397`、`:400-498`。

### 关键结论

- `pre_start/post_start` 总会执行。
- `pre_body/post_body` 只在 `call_pre_post==1` 时执行。
- parent sequence 的 `pre_do/mid_do/post_do` 是对子 sequence 或 item 的包裹回调；被启动的 child 自己的这些 do 回调不是主角。

## 3. item 发送协议：`start_item()` / `finish_item()`

`start_item(item, priority, sequencer)` 与 `finish_item(item)` 共同完成一次 request 发送：

`start_item()`：

1. 检查 item 非空。
2. 禁止对 sequence 调 `start_item()`，sequence 必须用 `seq.start()`。
3. 确定 sequencer：参数优先，其次 item 自带，再其次当前 sequence 的 `m_sequencer`。
4. 调 `item.set_item_context(this, sequencer)`。
5. 调 `sequencer.wait_for_grant(this, priority)`。
6. 如启用 auto item recording，开始 child transaction。
7. 调当前 sequence 的 `pre_do(1)`。

`finish_item()`：

1. 从 item 取 sequencer。
2. 调当前 sequence 的 `mid_do(item)`。
3. `sequencer.send_request(this,item)`。
4. `sequencer.wait_for_item_done(this,-1)`。
5. 结束 auto recording。
6. 调 `post_do(item)`。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/seq/uvm_sequence_base.svh:950-1036`。

关键坑：`finish_item` 注释明确要求 start_item 与 finish_item 之间不能有 delay 或 delta-cycle；只能做 randomize 等函数性操作。真正会让 driver 取到 item 的是 `finish_item()` 中的 `send_request()`，不是 `start_item()`。

## 4. sequencer-driver pull interface

sequencer 与 driver 之间不是普通 analysis 连接，而是专门的 `uvm_sqr_if_base #(REQ,RSP)`：

- driver 侧通常持有 `uvm_seq_item_pull_port`。
- sequencer 侧 `uvm_sequencer` 持有 `seq_item_export`，类型是 `uvm_seq_item_pull_imp #(REQ,RSP,this_type)`。
- 接口方法包括 `get_next_item / try_next_item / item_done / get / peek / put / put_response / wait_for_sequences / has_do_available`。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm1/uvm_sqr_ifs.svh:30-252`、`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/tlm1/uvm_sqr_connections.svh:23-83`、`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/seq/uvm_sequencer.svh:61-86`。

典型 driver pull 语义：

- `get_next_item(req)`：peek 当前 request；之后必须调用 `item_done()`。
- `get(req)`：取 request 后自动 `item_done()`，因此 driver 不应再调用 item_done。
- `peek(req)`：保持当前 item，不移除。
- `item_done(rsp)`：移除当前 request，并可携带 response。
- `put(rsp)`：单独发 response。

## 5. `uvm_sequencer_base`：仲裁、锁、sequence 注册

`uvm_sequencer_base` 是非参数化控制核心。关键状态：

- `arb_sequence_q[$]`：等待仲裁的 sequence request 队列。
- `lock_list[$]`：当前持有 lock/grab 的 sequence。
- `reg_sequences[int]`：sequence_id 到 sequence 的映射。
- `m_arbitration`：默认 `UVM_SEQ_ARB_FIFO`。
- 全局递增 `g_request_id / g_sequence_id / g_sequencer_id`。

`wait_for_grant(sequence_ptr, item_priority, lock_request)` 会：

1. 注册 sequence，拿到 per-sequencer sequence id。
2. 如 lock_request 为真，先把 lock request 推入仲裁队列。
3. 再把普通 REQ request 推入队列。
4. 等 `m_wait_for_arbitration_completed(request_id)`。
5. grant 后递增 `m_wait_for_grant_semaphore`，用于检查后续 `send_request()` 是否合法。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/seq/uvm_sequencer_base.svh:41-68`、`:600-643`、`:1048-1097`。

## 6. 仲裁机制

`m_select_sequence()` 会循环：

1. `wait_for_sequences()`，默认进入 NBA region。
2. `m_choose_next_request()`。
3. 若无可用 sequence，则 `m_wait_for_available_sequence()` 等队列变化或 non-relevant sequence 的 `wait_for_relevant()` 返回。
4. 有结果后设置 arbitration completed，删除该队列项。

`m_choose_next_request()` 会过滤：

- 已死进程的 request。
- 非普通 REQ。
- 被 lock/grab 阻塞的 sequence。
- `is_relevant()==0` 的 sequence。

仲裁模式：

- `UVM_SEQ_ARB_FIFO`
- `UVM_SEQ_ARB_WEIGHTED`
- `UVM_SEQ_ARB_RANDOM`
- `UVM_SEQ_ARB_STRICT_FIFO`
- `UVM_SEQ_ARB_STRICT_RANDOM`
- `UVM_SEQ_ARB_USER`

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_object_globals.svh:349-370`、`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/seq/uvm_sequencer_base.svh:709-976`。

常见坑：如果 `wait_for_relevant()` 在零时间内反复返回，sequencer 会通过 `m_max_zero_time_wait_relevant_count` 检测 zero-time loop 并 fatal。

## 7. request 进入 FIFO 与 response 路由

`uvm_sequencer_param_base #(REQ,RSP)` 把非参数化 sequencer 扩展成强类型 REQ/RSP：

- 内部 `m_req_fifo` 保存被 grant 后真正发给 driver 的 request。
- `send_request()` 检查必须先 `wait_for_grant()`；必要时 rerandomize；自动分配 transaction_id；设置 sequence_id；再 `try_put()` 到 `m_req_fifo`。
- 若 driver 并发调用 `get_next_item()` 导致 FIFO put 失败，会 fatal 提醒 driver 应用 semaphore 串行化。
- `put_response()` 根据 response 的 sequence_id 找到原 sequence，若启用 response_handler 则直接调用，否则放入 sequence 的 response queue。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/seq/uvm_sequencer_param_base.svh:31-47`、`:213-232`、`:264-344`。

`uvm_sequence_base` 的 response queue 默认深度是 8；溢出会报错，可设置为 -1 表示无限深，或开启 handler 模式。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/seq/uvm_sequence_base.svh:148-150`、`:1100-1236`。

## 8. `uvm_sequencer` 的 driver-facing API

`uvm_sequencer #(REQ,RSP)` 提供 driver 看到的接口实现：

- `get_next_item()`：若尚未 request item，则选择 sequence；设置 `sequence_item_requested/get_next_item_called`，peek FIFO。
- `try_next_item()`：选择可用 sequence，给它一个 NBA 产生 item；没产生则返回 null 并报错。
- `item_done()`：清 flag、从 FIFO 取走 request、记录 sequence_id/transaction_id 以释放 waiters；若带 rsp 则 put_response。
- `get()`：选择/peek 后立即 `item_done()`。
- `peek()`：选择并 peek，但不移除。
- `put()`：单独发送 response。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/seq/uvm_sequencer.svh:30-139`、`:203-343`。

## 9. default sequence

`uvm_sequencer_base::start_phase_sequence(phase)` 通过 resource pool 查：

- scope：`{sequencer_full_name, ".", phase_name, "_phase"}`
- name：`default_sequence`

支持两种资源类型：

- `uvm_config_db#(uvm_sequence_base)`：直接给 sequence instance，优先级更高。
- `uvm_config_db#(uvm_object_wrapper)`：给 sequence type，由 factory 创建。

找到后会：

1. `seq.set_sequencer(this)`。
2. `seq.reseed()`。
3. `seq.set_starting_phase(phase)`。
4. 若允许，先 randomize。
5. fork 一个进程执行 `seq.start(this)`，并登记到 `m_default_sequences[phase]`。

`stop_phase_sequence()` 会 kill 该 phase 的默认 sequence。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/seq/uvm_sequencer_base.svh:116-170`、`:1429-1535`。

## 10. 本轮最重要的结论

1. sequence 机制的核心握手是：sequence `wait_for_grant()` → sequencer 仲裁 → sequence `send_request()` → driver `get_next_item/get` → driver `item_done/put` → sequence `wait_for_item_done/get_response`。
2. `sequence_id` 用于把 response 路由回 sequence；`transaction_id` 用于 sequence 内部关联具体 request/response。
3. `start_item()` 只拿 grant 和执行 pre_do；真正发送 item 的是 `finish_item()`。
4. driver 使用 `get_next_item()` 后必须 `item_done()`；使用 `get()` 后不能再 `item_done()`。
5. default sequence 本质是 resource/config DB 驱动的 phase 级自动启动。

## 11. 下一步建议

继续学习 `src/comps` 中的标准组件，尤其 `uvm_driver`、`uvm_monitor`、`uvm_agent`、`uvm_subscriber`、`uvm_scoreboard`，把 sequence/sequencer 与真实 driver/agent 结构连接起来。
