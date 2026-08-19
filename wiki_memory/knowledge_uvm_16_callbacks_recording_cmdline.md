---
name: knowledge_uvm_16_callbacks_recording_cmdline
description: UVM 1.2 callbacks、transaction recording/database/stream/recorder 与命令行 plusargs 处理路径学习记忆
metadata:
  type: reference
  node_type: memory
  originSessionId: manual-2026-07-23
  modified: 2026-07-23T08:21:41.005Z
---

# UVM 1.2 查漏：callbacks / transaction recording / command-line plusargs

## 源码范围

本轮阅读：

- `src/base/uvm_callback.svh`
- `src/macros/uvm_callback_defines.svh`
- `src/base/uvm_transaction.svh`
- `src/base/uvm_tr_database.svh`
- `src/base/uvm_tr_stream.svh`
- `src/base/uvm_recorder.svh`
- `src/base/uvm_component.svh` recording 与 command-line report settings 片段
- `src/base/uvm_root.svh` run_test/build_phase/plusarg 处理片段

本主题补齐三个在 examples 中频繁出现、但此前知识库只分散提到的机制：callback 框架、transaction recording 框架、UVM-aware command-line 参数在 root/component/factory/config 中的落点。

## Callback 框架

### 基本角色

UVM callback 不是 SV language callback，而是 UVM 自己用 object queue + type checking 实现的 hook 机制。

核心类：

- `uvm_callback extends uvm_object`
  - 所有用户 callback class 的基类。
  - 内部有 `m_enabled`。
  - `callback_mode(int on=-1)` 类似 `rand_mode/constraint_mode`，可查询或启停 callback。
- `uvm_callbacks#(T,CB)`
  - 管理某种对象类型 `T` 与 callback 类型 `CB` 的合法 pairing、add/delete/display/iterator。
- `uvm_typed_callbacks#(T)`
  - 内部保存 typewide callback queue。
- `uvm_callbacks_base`
  - 全局 callback pool 与类型继承关系表。
- `uvm_callback_iter#(T,CB)`
  - 对 callback queue 进行 first/next/prev/last 迭代。
- `uvm_derived_callbacks#(T,ST,CB)`
  - 配合宏 `uvm_set_super_type` 建立 callback 类型继承关系。

### 注册与执行宏

`uvm_callback_defines.svh` 提供宏：

- `` `uvm_register_cb(T,CB) ``
  - 展开为 static local bit，调用 `uvm_callbacks#(T,CB)::m_register_pair("T","CB")`。
  - 作用：声明 callback 类型 `CB` 可以注册到对象类型 `T`。
- `` `uvm_set_super_type(T,ST) ``
  - 让 derived type `T` 继承 super type `ST` 的 typewide callbacks。
- `` `uvm_do_callbacks(T,CB,METHOD) ``
  - 等价于对 `this` 执行 `uvm_do_obj_callbacks`。
- `` `uvm_do_obj_callbacks(T,CB,OBJ,METHOD) ``
  - 创建 `uvm_callback_iter#(T,CB) iter = new(OBJ)`。
  - 从 `iter.first()` 开始遍历 enabled callback。
  - 对每个 callback 调 `cb.METHOD`。
- `` `uvm_do_callbacks_exit_on(T,CB,METHOD,VAL) ``
  - callback method 返回指定 `VAL` 时立即 `return VAL`。
  - 因为宏内部直接 `return`，只能放在返回 `bit` 的 function 中。

examples 中的 APB/VIP driver/monitor 正是这个模式：

```systemverilog
class apb_master_cbs extends uvm_callback;
  virtual task trans_received(apb_master xactor, apb_rw cycle); endtask
  virtual task trans_executed(apb_master xactor, apb_rw cycle); endtask
endclass

class apb_master extends uvm_driver#(apb_rw);
  `uvm_do_callbacks(apb_master, apb_master_cbs, trans_received(this,tr))
endclass
```

注意：APB 例子没有在 `apb_master` 中显式看到 `` `uvm_register_cb(apb_master, apb_master_cbs) ``，但 VIP driver/monitor 使用了 `` `uvm_register_cb ``。未注册 pair 时，`uvm_callbacks::add` 会 warning `CBUNREG`，但宏执行本身只是遍历 queue；如果没有 add 过 callback 就不会执行任何东西。

### add/delete/typewide/instance-specific

`uvm_callbacks#(T,CB)::add(obj, cb, ordering)`：

- `obj == null` 表示 typewide callback，添加到 `m_tw_cb_q`，并同步到已有实例 queue/derived types。
- `obj != null` 表示 instance-specific callback，queue 保存在 `m_pool[obj]`。
- `ordering` 为 `UVM_APPEND` 或 prepend，决定 callback 执行顺序。
- 先检查 callback 非 null，再检查 `m_base_inst.check_registration(obj,cb)`；未注册 pair 会 warning。
- 防重复：同一 callback object 已注册会 warning；同名不同 object 也会 warning。

`add_by_name(name, cb, root, ordering)`：

- 使用 `uvm_root::find_all(name,cq,root)` 找已有 components。
- 对匹配且可 `$cast` 为 T 的 component 调 add。
- 因为只查“已经存在”的 component，调用时机通常应在 build 之后或目标对象创建之后。

`delete/delete_by_name` 对应移除 callback。

### 迭代时只执行 enabled callback

`get_first/get_next` 中会 `$cast(cb, q.get(itr)) && cb.callback_mode()`，所以 disabled callback 会被跳过。

这解释了为什么 callback 对象不是简单函数指针：它是 UVM object，可命名、可启停、可被 factory/配置/显示/记录引用。

### tracing

若编译定义 `UVM_CB_TRACE_ON`，宏会通过 `UVMCB_TRC` id 输出 add/delete/execute trace。否则 trace 宏为空。`uvm_callbacks_base::m_tracing` 会在 display 时临时关闭，避免 display 自己产生 trace 噪音。

## Transaction 与 recording

### `uvm_transaction` 的时间戳/事件/记录

`uvm_transaction extends uvm_object`，但 UVM 1.2 注释明确：用户自定义 transaction 推荐继承 `uvm_sequence_item`，不是直接继承 `uvm_transaction`。

它新增：

- 时间戳：`accept_time`、`begin_time`、`end_time`
- initiator：`uvm_component initiator`
- event pool：`events`
  - 默认有 `begin_event = events.get("begin")`
  - `end_event = events.get("end")`
  - accept 时还会触发 `events.get("accept")`
- recording handle：`uvm_tr_stream stream_handle`、`uvm_recorder tr_recorder`
- transaction id：`m_transaction_id`

关键 API：

- `accept_tr(time accept_time=0)`
  - 设置 accept time。
  - 调 `do_accept_tr()`。
  - trigger accept event。
- `begin_tr(time begin_time=0)` / `begin_child_tr(begin_time,parent_handle)`
  - 设置 begin time。
  - 如果 `enable_recording(stream)` 已设置 stream，则在 stream 上 open recorder。
  - 有 parent recorder 时建立 parent-child link。
  - 调 `do_begin_tr()`。
  - trigger begin_event。
  - 返回 recorder handle；没开启 recording 时为 0。
- `end_tr(time end_time=0, bit free_handle=1)`
  - 设置 end time。
  - 调 `do_end_tr()`。
  - 如果 recording active，则 `record(tr_recorder)`、close、可选 free。
  - trigger end_event。
- `enable_recording(uvm_tr_stream stream)` / `disable_recording()`。
- `get_tr_handle()`。

重要语义：

- `get_next_item/item_done` 在 sequencer-driver pull port 上可自动触发 item begin/end recording；若 driver/layering sequence 要自己控制时间戳和 recording，应调用 sequencer interface 的 `disable_auto_item_recording()`。
- 对 pipeline protocol，sequence 的 `finish_item()` 返回不一定代表 item 真正完成；可等待 `item.end_event.wait_on()`。

### component-level recording

`uvm_component` 提供包装入口：

- `accept_tr(uvm_transaction tr, time accept_time=0)`
  - 调 `tr.accept_tr()`。
  - 调 component 的 `do_accept_tr(tr)` hook。
  - trigger component `event_pool.get("accept_tr")`。
- `begin_tr(tr, stream_name="main", label="", desc="", begin_time=0, parent_handle=0)`
  - 最终进入 `m_begin_tr`。
- `begin_child_tr(...)`
  - 与 begin 类似，但带 parent handle。
- `end_tr(tr,end_time,free_handle)`。
- `get_tr_stream(name, stream_type_name="")`。
- `free_tr_stream(stream)`。

`m_get_tr_database()`：如果 component 的 `tr_database` 为空，从 `uvm_coreservice_t::get().get_default_tr_database()` 获取默认 DB。

`get_tr_stream(name,type)`：

- 按 `m_streams[name][stream_type_name]` 缓存 stream。
- 若不存在，调用 `db.open_stream(name, this.get_full_name(), stream_type_name)`。

`m_begin_tr` 的核心：

1. 如果 transaction 是 `uvm_sequence_item`，尝试找 `seq.get_parent_sequence().m_tr_recorder` 作为 parent recorder。
2. 调 transaction 自己的 `begin_tr` / `begin_child_tr`，产生或关联 transaction 内部 recorder。
3. 若 component 的 `recording_detail != UVM_NONE`：
   - 使用 main stream 或指定 stream。
   - `stream.open_recorder(name, begin_time, kind)`。
   - 记录 label/desc。
   - 与 parent recorder 建立 parent-child link。
   - 与 transaction 自己的 recorder 建立 related link。
   - 将 recorder 存入 `m_tr_h[tr]`。
   - 调 `do_begin_tr(tr, stream_name, handle)`。
4. trigger component `begin_tr` event。
5. 返回 component recorder handle。

`end_tr`：

1. 调 `tr.end_tr()`。
2. 如果 component recording enabled 且 `m_tr_h.exists(tr)`：
   - 调 `do_end_tr(tr, handle)`。
   - `tr.record(recorder)` 把 field automation / do_record 内容写入 recorder。
   - `recorder.close(end_time)`。
   - 可选 `recorder.free()`。
3. trigger component `end_tr` event。

这解释了 `hello_world` 中的写法：

```systemverilog
p.enable_recording(get_tr_stream("packet_stream"));
accept_tr(p);
begin_tr(p);
end_tr(p);
```

transaction 自己和 component 都可能产生 recorder/link；UVM 通过 parent/related links 把 sequence、component stream、transaction recorder 关联起来。

## Database / stream / recorder 三层抽象

### `uvm_tr_database`

抽象表示 vendor/tool-specific transaction database。UVM 默认提供 text backend：`uvm_text_tr_database`。

核心 API：

- `open_db()`：若未打开则调用 `do_open_db()`。
- `close_db()`：关闭 DB，并隐式关闭/释放 streams。
- `open_stream(name, scope="", type_name="")`：打开 stream。
- `get_streams(ref q[$])`。
- `establish_link(uvm_link_base link)`：支持 stream/record 间 link，检查两端非 null、类型可接受、属于同一 DB。

`uvm_text_tr_database`：

- 默认文件名 `tr_db.log`。
- `do_open_db()` 用 `$fopen(filename,"a+")`，成功后锁定 filename DAP。
- `set_file_name(filename)` 只能在 DB open 前有效。
- `do_open_stream()` 创建 `uvm_text_tr_stream`。
- `do_establish_link()` 将 parent-child/related link 写入文本文件。

### `uvm_tr_stream`

表示 DB 中一条 record stream。

状态：

- opened / closed / freed。
- `m_records[uvm_recorder]` 保存 active/closed recorders。
- `m_cfg_dap` 保存 db/scope/stream_type_name。

关键 API：

- `close()`：先 `do_close()`，再 close 所有 open recorders。
- `free()`：若 open 先 close，再 `do_free()`，free 所有 recorders，清 DB/handle 状态。
- `open_recorder(name, open_time=0, type_name="")`：仅 stream open 时有效；调用 `do_open_recorder`，再 `recorder.m_do_open(this, time, type_name)`。
- `get_handle()` / `get_stream_from_handle(id)`。

`uvm_text_tr_stream` 在文本文件中打印：

- `CREATE_STREAM`
- `CLOSE_STREAM`
- `FREE_STREAM`
- `OPEN_RECORDER`

### `uvm_recorder`

`uvm_recorder` 同时是两件事：

1. DB 中一条 transaction record 的抽象。
2. object `record()` 过程使用的 policy object。

它保存：

- stream DAP
- open/close time
- recording knobs：`recording_depth`、`default_radix`、`physical`、`abstract`、`identifier`、`policy`
- handle 映射表

属性记录 API：

- `record_field(name,value,size,radix)`
- `record_field_int(...)`
- `record_field_real(...)`
- `record_object(name,value)`
- `record_string(name,value)`
- `record_time(name,value)`
- `record_generic(name,value,type_name)`

每个 public API 会先检查 `get_stream()!=null`，再调用对应 `do_record_*`。backend 必须实现 pure virtual `do_record_*`。

`uvm_text_recorder`：

- `do_open` 打印 `OPEN_RECORDER`。
- `do_close` 打印 `CLOSE_RECORDER`。
- `do_free` 打印 `FREE_RECORDER`。
- `do_record_*` 打印 `SET_ATTR`。
- `do_record_object` 会根据 `identifier` 和 `policy` 决定记录 object id 以及是否递归 `value.record(this)`，并用 cycle_check 防止递归环。

易错点：

- 默认 text DB 是 portable fallback；商业 simulator 常替换 default recorder/database 以写入 waveform transaction DB。
- handle 在 free 后失效；如果需要建立 link，不要太早 free。
- `recording_detail == UVM_NONE` 时 component-level recording 不会打开 recorder。

## Command-line plusargs 处理路径

前置：`uvm_cmdline_processor` 在构造时通过 DPI/VPI 收集 args，见 [[knowledge_uvm_13_dpi]]。本轮关注这些 args 在 UVM root/component 中何时生效。

### `run_test` 阶段：选择 test

`uvm_root::run_test(test_name="")`：

- 初始化 objection worker：`uvm_objection::m_init_objections()`。
- 若 DPI 打开：`clp.get_arg_values("+UVM_TESTNAME=", test_names)`。
- `+UVM_TESTNAME` 覆盖 `run_test()` 的参数。
- 多个 `+UVM_TESTNAME` 时取第一个并 warning。
- 如果 test_name 非空，用 factory：

```systemverilog
factory.create_component_by_name(test_name, "", "uvm_test_top", null)
```

- 如果创建失败，fatal `INVTST`。
- 如果没有任何 component，fatal `NOCOMP`。
- fork phase runner：`uvm_phase::m_run_phases()`。
- 等 `m_phase_all_done`，kill phase runner，report summary，按 `finish_on_completion` 决定 `$finish`。

### root build_phase：集中处理 UVM-aware settings

`uvm_root::build_phase` 调：

1. `super.build_phase(phase)`
2. `m_set_cl_msg_args()`：继承自 component，处理 report verbosity/action/severity。
3. `m_do_verbosity_settings()`：先校验 `+uvm_set_verbosity` 格式。
4. `m_do_timeout_settings()`：处理 `+UVM_TIMEOUT=<timeout>,<overridable>`。
5. `m_do_factory_settings()`：处理 factory override。
6. `m_do_config_settings()`：处理 config/default sequence。
7. `m_do_max_quit_settings()`：处理 report server quit count。
8. `m_do_dump_args()`：处理 `+UVM_DUMP_CMDLINE_ARGS`。

### factory settings

`m_do_factory_settings()` 用 regex 匹配大小写两种形式：

- `/^\+(UVM_SET_INST_OVERRIDE|uvm_set_inst_override)=/`
- `/^\+(UVM_SET_TYPE_OVERRIDE|uvm_set_type_override)=/`

然后分别处理：

- `+uvm_set_inst_override=<requested_type>,<override_type>,<instance_path>` → `factory.set_inst_override_by_name(...)`
- `+uvm_set_type_override=<requested_type>,<override_type>[,<replace>]` → `factory.set_type_override_by_name(..., replace)`

注意 substring 常量长度依赖 plusarg 前缀长度；这也是源码比较脆的地方。

### config/default sequence settings

`m_do_config_settings()`：

- `+uvm_set_config_int=<comp>,<field>,<value>`
  - 支持 `'b` / `0b` / `'o` / `'d` / `'h` / `'x` / `0x` base prefix。
  - 调 `uvm_config_int::set(m_uvm_top, comp, field, v)`。
- `+uvm_set_config_string=<comp>,<field>,<value>`
  - 调 `uvm_config_string::set(m_uvm_top, comp, field, value)`。
- `+uvm_set_default_sequence=<seqr>,<phase>,<type>`
  - 用 factory `find_wrapper_by_name(type)` 找 sequence wrapper。
  - 成功后：

```systemverilog
uvm_config_db#(uvm_object_wrapper)::set(this,
  {seqr, ".", phase}, "default_sequence", w);
```

这与 codec 例子中手写 default_sequence 的方式完全一致，只是来源从 plusarg 变成 root build 处理。

### report verbosity/action/severity settings

`uvm_component::m_set_cl_msg_args()` 会在每个 component build 时执行：

- `m_set_cl_verb()`
- `m_set_cl_action()`
- `m_set_cl_sev()`

`+uvm_set_verbosity=<comp>,<id>,<verbosity>,<phase|time>,<offset>`：

- 静态读取一次 values。
- 对每个 component，如果 `uvm_is_match(setting.comp, get_full_name())`：
  - build/time 且 offset 0 立即设置 verbosity。
  - 其他 phase 存入 `m_verbosity_settings`，后续 phase apply。
- 对 `phase == "time"` 的设置，在 top 中 fork 时间排序线程，按 offset 对匹配 components 设置 verbosity。

`+uvm_set_action=<comp>,<id>,<severity>,<action[|action]>`：

- 静态解析一次，保存到 `m_uvm_applied_cl_action`。
- 每个 component 根据 path match 应用。
- 支持 `_ALL_` id 和 `_ALL_` severity。
- run_phase 中 root 会检查 used==0 的设置并 warning，提示 pattern 未匹配任何 component。

`+uvm_set_severity=<comp>,<id>,<orig_severity>,<new_severity>`：

- 类似 action，设置 severity override。
- 同样支持 `_ALL_`。

### 其他 built-in switches

- `+UVM_VERBOSITY=<verbosity>`：`uvm_root::m_check_verbosity()` 取第一个，多次 warning，然后 `set_report_verbosity_level_hier(verbosity)`。
- `+UVM_TIMEOUT=<timeout>,<YES|NO>`：设置 global phase timeout 及是否可覆盖。
- `+UVM_MAX_QUIT_COUNT=<count>,<YES|NO>`：设置 report server max quit count。
- `+UVM_DUMP_CMDLINE_ARGS`：dump 所有 command-line args。
- `+UVM_PHASE_TRACE`：在 `uvm_phase` 中读取。
- `+UVM_OBJECTION_TRACE`：在 `uvm_objection` 中读取。
- `+UVM_RESOURCE_DB_TRACE` / `+UVM_CONFIG_DB_TRACE`：分别在 resource/config DB 中读取。

## 易错点总结

1. callback pair 需要 `` `uvm_register_cb(T,CB) `` 注册；否则 add 时 warning，但执行宏本身不会报错。
2. `add_by_name` 只作用于已经存在的 components；创建前调用找不到对象。
3. callback execution 只遍历 enabled callbacks；`callback_mode(0)` 会使对象留在 queue 但被跳过。
4. `uvm_do_callbacks_exit_on` 宏内部直接 `return`，只能在合适返回类型的 function 中使用。
5. transaction recording 有 transaction-level 和 component-level 两套 recorder/link；不要把 `tr.enable_recording(stream)` 与 `component.begin_tr(tr)` 混为一个动作。
6. recorder/stream/db 都有 open/close/free 生命周期；free 后 handle 不能再用于 link。
7. `recording_detail == UVM_NONE` 时 component-level recording 不打开 recorder。
8. `+UVM_TESTNAME` 覆盖 `run_test("...")` 参数，多个只取第一个。
9. command-line config/factory/default_sequence 多在 root build_phase 生效；目标类型必须已经注册到 factory，config 路径也要与后续 get 路径匹配。
10. report action/severity plusargs 会在 run_phase 检查是否匹配过 component，未匹配会 warning；这是调试 pattern 拼错的重要线索。

## 关联知识

- [[knowledge_uvm_3_factory_registry]]
- [[knowledge_uvm_4_policy_classes]]
- [[knowledge_uvm_5_component_phasing]]
- [[knowledge_uvm_6_reporting]]
- [[knowledge_uvm_7_config_resource_db]]
- [[knowledge_uvm_10_sequences_sequencer]]
- [[knowledge_uvm_13_dpi]]
- [[knowledge_uvm_14_examples]]
- [[knowledge_uvm_15_codec_example]]
