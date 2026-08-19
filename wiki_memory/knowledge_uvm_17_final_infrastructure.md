---
name: knowledge_uvm_17_final_infrastructure
description: UVM 1.2 最后一轮源码查漏：event/barrier/pool/queue/DAP/links/traversal/misc/global helpers
metadata:
  type: reference
  node_type: memory
  originSessionId: manual-2026-07-23
  modified: 2026-07-23T08:53:20.791Z
---

# UVM 1.2 Final Infrastructure 源码查漏

本轮回到源码做最后一轮查漏，覆盖此前主线之外、但会被多个子系统复用的基础设施：`uvm_event` / `uvm_barrier`、`uvm_pool` / `uvm_queue`、DAP、transaction link、visitor traversal、misc/global helper、spell checker 与 object globals。它们不像 factory、phase、TLM、reg 那样构成单独大系统，但经常解释 UVM 源码中的“小魔法”。关联：[[knowledge_uvm_2_object_model]]、[[knowledge_uvm_4_policy_classes]]、[[knowledge_uvm_5_component_phasing]]、[[knowledge_uvm_6_reporting]]、[[knowledge_uvm_7_config_resource_db]]、[[knowledge_uvm_13_dpi]]、[[knowledge_uvm_16_callbacks_recording_cmdline]]。

## 源码范围

- `src/base/uvm_event.svh`
- `src/base/uvm_event_callback.svh`
- `src/base/uvm_barrier.svh`
- `src/base/uvm_pool.svh`
- `src/base/uvm_queue.svh`
- `src/dap/uvm_dap.svh`
- `src/dap/uvm_set_get_dap_base.svh`
- `src/dap/uvm_simple_lock_dap.svh`
- `src/dap/uvm_get_to_lock_dap.svh`
- `src/dap/uvm_set_before_get_dap.svh`
- `src/base/uvm_links.svh`
- `src/base/uvm_traversal.svh`
- `src/base/uvm_misc.svh`
- `src/base/uvm_globals.svh`
- `src/base/uvm_object_globals.svh`
- `src/base/uvm_spell_chkr.svh`

## `uvm_event`：带状态、数据与回调的 SV event 包装

`uvm_event_base extends uvm_object` 包装 SystemVerilog `event`，并维护：

- `num_waiters`：当前等待者数量。
- `on`：事件是否处于 on 状态。
- `trigger_time`：最近一次触发时间。
- `callbacks[$]`：事件本地 callback 列表。

等待 API 的关键区别：

- `wait_on(delta=0)`：如果 `on` 已经为 1，会立即返回；`delta` 为 1 时额外 `#0`，用于错开同一 delta 的竞争。
- `wait_off(delta=0)`：如果当前不在 on 状态，立即返回。
- `wait_trigger()`：等待 `@m_event`。如果调用发生在同一个 time slice 的 trigger 之后，可能错过事件。
- `wait_ptrigger()`：先检查 `m_event.triggered`，避免同一 time slice 中 `wait_trigger` 的竞争问题。
- `reset(wakeup=0)`：可以先唤醒等待旧 event 的进程，再创建新的 SV event 句柄；同时清除 waiters/on/trigger_time。
- `cancel()`：用于等待者取消等待时减少计数。

`uvm_event#(T) extends uvm_event_base` 增加 `trigger_data`：

- `trigger(T data=null)`：先调用每个 callback 的 `pre_trigger(e,data)`；任一 callback 返回 1 时跳过真正触发，且不会调用 `post_trigger`。否则执行 `->m_event`、调用 `post_trigger`、清零 waiters、置 `on=1`、记录时间和 data。
- `wait_trigger_data` / `wait_ptrigger_data`：等待触发后返回当前 `trigger_data`。
- `get_trigger_data()`：读取最近一次触发数据。
- `add_callback(cb, append=1)` / `delete_callback(cb)`：管理事件私有 callback 队列。

理解点：`uvm_event` 是 UVM 内部常用同步原语，但它不只是 SV event；它带状态、等待者计数、数据传递和可 veto 的前置 callback。

## `uvm_event_callback`：事件本地 callback，不等于全局 callback 框架

`uvm_event_callback#(T) extends uvm_object` 提供：

- `pre_trigger(uvm_event#(T) e, T data)`：返回 1 表示阻止这次 trigger。
- `post_trigger(e,data)`：真正 trigger 后执行。

它不是 `uvm_callbacks#(T,CB)` 那套 typewide/instance-specific callback 框架，而是 `uvm_event` 对象自己维护的 callback list。源码阅读时不要把这两套机制混淆。关联 [[knowledge_uvm_16_callbacks_recording_cmdline]]。

## `uvm_barrier`：阈值同步器

`uvm_barrier extends uvm_object` 是基于 `uvm_event` 的多进程 barrier：

- `threshold`：达到多少等待者后释放。
- `num_waiters`：当前等待数。
- `at_threshold`：非 auto-reset 模式下，阈值已达到后的保持状态。
- `auto_reset`：默认开启。

核心行为：

- `wait_for()`：
  1. 如果 `at_threshold` 已经为 1，直接返回。
  2. 否则增加 waiter 数。
  3. 当 `num_waiters >= threshold` 时触发内部 event；若 `auto_reset` 关闭，则设置 `at_threshold=1`。
  4. 未达到阈值者等待内部 event。
- `reset(wakeup=1)`：清除状态；可选择是否唤醒当前 waiters。
- `set_threshold(threshold)`：如果新 threshold 小于等于当前 waiters，会触发 reset/wakeup 行为。
- `cancel()`：通过 event cancel 更新等待计数。
- `m_trigger()`：触发 event 后清零 waiters，并有一个 `#0`，让被唤醒的进程先恢复。

理解点：auto-reset 开启时，barrier 到阈值后自动进入下一轮；关闭时，相当于 latch，一旦达到阈值，后续 waiter 会直接通过，直到显式 reset。

## `uvm_pool` / `uvm_queue`：class 形式的容器包装

`uvm_pool#(KEY,T) extends uvm_object` 是 associative array 的 class 包装：

- 每个参数化类型有自己的 static global pool：`get_global_pool()` / `get_global(key)`。
- `get(key)` 若 key 不存在会创建默认值并插入；对 object handle 类型来说，默认值可能是 `null`，除非子类改写。
- 提供 `add`、`exists`、`delete`、`num`、`first/last/next/prev`、`do_copy`、`do_print`。

`uvm_object_string_pool#(T=uvm_object)` 继承自 `uvm_pool#(string,T)`，覆盖 `get(string key)`，若不存在则 `new(key)` 创建对象。常用 typedef：

- `uvm_barrier_pool`
- `uvm_event_pool`

`uvm_queue#(T) extends uvm_object` 是 SV queue 的 class 包装：

- 每个参数化类型有 static global queue。
- `get(index)` 越界时 warning 并返回默认值，不会自动扩容。
- `insert(index,item)` 对 `index >= size()` 会 warning，因此不能用 `insert(size())` 表示 append；append 应使用 `push_back`。
- `delete(index=-1)` 默认清空整个 queue。
- 提供 `push_front/back`、`pop_front/back`、`size`、`do_copy`、`convert2string`。

理解点：这些容器让 UVM 能在需要 object API、factory/policy/printing 或 global singleton 时使用关联数组/队列。

## DAP：Data Access Policy 小型状态机

`src/dap/uvm_dap.svh` 只是聚合入口，包含 4 个文件。DAP 的共同接口是 `uvm_set_get_dap_base#(T)`：

- `set(T value)`：策略允许时写入，否则可报错。
- `try_set(T value)`：尝试写入，失败返回 0，不报错。
- `get()`：策略允许时读取，否则可报错。
- `try_get(output T value)`：尝试读取，失败返回 0，不报错。

三个具体策略：

### `uvm_simple_lock_dap#(T)`

- 允许任意次 `set`，直到显式 `lock()`。
- `get()` 任意时刻都允许。
- `set()` 在 locked 时报错；`try_set()` 在 locked 时返回 0。
- 可 `unlock()`，`is_locked()` 查询。
- UVM 用它保护 `uvm_text_tr_database` 的 file name。

### `uvm_get_to_lock_dap#(T)`

- `get()` 会把 DAP 锁住；之后禁止 `set()`。
- `try_get()` 总是成功，并同样锁住。
- 用于保护 `uvm_sequence_base` 中 starting phase、automatic objection 等“第一次读取后不应再改”的值。关联 [[knowledge_uvm_10_sequences_sequencer]]。

### `uvm_set_before_get_dap#(T)`

- 必须先 `set()` / `try_set()`，再 `get()`。
- `get()` 在未 set 时报告错误；`try_get()` 在未 set 时返回 0。
- 典型用途是传递“稍后才可用”的引用，例如 build 阶段先把 DAP 传给 virtual sequencer，connect 阶段再 set 真正 sequencer。

所有 DAP 都刻意不支持 `copy/pack/unpack`，因为这些自动化操作可能绕过或破坏访问策略。

## Transaction links：record 间关系的类型安全封装

`uvm_links.svh` 定义 `uvm_link_base extends uvm_object`，用于在 `uvm_tr_database` 记录之间建立关系。它抽象出：

- `set_lhs` / `get_lhs`
- `set_rhs` / `get_rhs`
- `set(lhs,rhs)`
- 由子类实现 `do_set_lhs` / `do_get_lhs` / `do_set_rhs` / `do_get_rhs`

具体 link 类型：

- `uvm_parent_child_link`：parent/child 关系。
- `uvm_cause_effect_link`：cause/effect 关系。
- `uvm_related_link`：泛化的 related 关系。

每个类提供 `static get_link(lhs,rhs,name)` 便于一行创建预填 link。数据库实现可通过 `$cast` 识别 link 类型，而不是依赖魔法字符串。关联 [[knowledge_uvm_16_callbacks_recording_cmdline]]。

## Traversal：visitor 模式与组件名检查

`uvm_traversal.svh` 提供通用 visitor 框架：

- `uvm_visitor#(NODE)`：`begin_v()`、`visit(NODE node)`、`end_v()`。
- `uvm_structure_proxy#(STRUCTURE)`：抽象如何取得 immediate children。
- `uvm_visitor_adapter#(STRUCTURE,VISITOR)`：抽象 traversal。
- `uvm_top_down_visitor_adapter`：先访问父节点，再递归子节点。
- `uvm_bottom_up_visitor_adapter`：先递归子节点，再访问父节点。
- `uvm_by_level_visitor_adapter`：按层 BFS。
- `uvm_component_proxy`：对 component 调 `get_children(children)`。

`uvm_component_name_check_visitor` 用 DPI regex 检查组件名：

- 正则约束大意：允许字母数字、`[](){}_:-`、空格等，但不允许路径分隔符导致 `get_full_name()` 难以解析。
- `begin_v()` 获取 `uvm_root`，root 自身不检查。
- 需要 DPI；`UVM_NO_DPI` 下只输出提示。关联 [[knowledge_uvm_13_dpi]]。

## Misc helpers：scope、随机 seed、字符串/数组/配置辅助

`uvm_misc.svh` 包含一批底层工具：

- `uvm_void`：所有 UVM class 的最底层抽象基类，类似 C 的 void pointer 概念；直接继承它不会获得 `uvm_object` 功能。
- `uvm_apprepend`：`UVM_APPEND` / `UVM_PREPEND`，供顺序敏感 API 使用。
- `uvm_scope_stack`：维护自动化方法中的层级字段路径，支持 `down`、`up`、`down_element`、`set_arg`，用于 print/compare/pack/record 的字段路径。
- `uvm_status_container`：field automation 的内部状态容器，保存 comparer/packer/recorder/printer、cycle check、临时字段值、scope stack 等。
- `uvm_global_random_seed`、`uvm_seed_map`、`uvm_random_seed_table_lookup`：支撑 deterministic per-type/per-instance seed。
- `uvm_instance_scope()`：计算 UVM 包/实例所在 scope。
- `uvm_oneway_hash()` / `uvm_create_random_seed()`：根据 type/instance 生成并更新随机种子。
- `uvm_object_value_str()`：将 object handle 打成 `@inst_id` 风格字符串。
- `uvm_leaf_scope()`：从层级名取叶子 scope，支持 bracket 匹配。
- `uvm_bitstream_to_string()` / `uvm_integral_to_string()`：按 radix 转字符串。
- `uvm_get_array_index_int()` / `uvm_get_array_index_string()` / `uvm_is_array()`：解析 `field[index]` 形式。
- `uvm_has_wildcard()`：识别 glob 或 `/regex/`。
- `uvm_utils#(TYPE,FIELD)`：提供 `find_all`、`find`、`create_type_by_name`、`get_config`，是用户侧偶尔可见的类型化便捷工具。
- `m_uvm_string_queue_join()`：内部 string queue 拼接，兼容不同 simulator。

## Global functions：package-scope 入口委托到 core service/root

`uvm_globals.svh` 提供包级 API，多数只是委托给 `uvm_coreservice_t::get()` 取得 root/factory/report server 等：

- `run_test(test_name="")`：调用 `uvm_root::run_test`。
- reporting 入口：`uvm_report_enabled`、`uvm_report`、`uvm_report_info/warning/error/fatal`、`uvm_process_report_message`。
- `m__uvm_report_dpi`：导出给 DPI-C 使用的 report wrapper。
- `uvm_string_to_severity()`、`uvm_string_to_action()`：把字符串转为 severity/action。
- deprecated config 入口：`set_config_int/object/string`，内部警告用户改用 `uvm_config_db`。
- `uvm_is_match(expr,str)`：glob 转 regex 后调用 `uvm_re_match`。
- `uvm_string_to_bits()` / `uvm_bits_to_string()`。
- `uvm_wait_for_nba_region()`：等待到 NBA region；若禁用 NBA 实现，则退化为若干 `#0`。
- `uvm_split_string()`。
- `uvm_enum_wrapper#(T)`：为 enum 提供 `from_name()`，大小写敏感。

理解点：UVM 的很多“全局函数”不是全局状态直接实现，而是 core service/root/factory/report handler 的薄包装。

## Object globals：全局 typedef、enum、默认 policy 对象

`uvm_object_globals.svh` 定义许多贯穿全库的类型与常量：

- `uvm_bitstream_t`、`uvm_integral_t`。
- radix：`UVM_BIN/DEC/UNSIGNED/OCT/HEX/STRING/TIME/ENUM/REAL/...`。
- copy recursion policy：`UVM_DEEP`、`UVM_SHALLOW`、`UVM_REFERENCE`。
- agent active/passive：`UVM_ACTIVE`、`UVM_PASSIVE`。
- field automation flags：`UVM_COPY/NOCOPY`、`UVM_COMPARE/NOCOMPARE`、`UVM_PRINT/NOPRINT`、`UVM_RECORD/NORECORD`、`UVM_PACK/NOPACK`、`UVM_PHYSICAL/ABSTRACT`、`UVM_READONLY` 等。
- reporting：`uvm_severity`、`uvm_action`、`uvm_verbosity`。
- TLM port kind：`UVM_PORT`、`UVM_EXPORT`、`UVM_IMPLEMENTATION`。
- sequence arbitration/state/library mode。
- phase type/state/wait op。
- objection event。
- 默认 policy 对象：`uvm_default_table_printer`、`uvm_default_tree_printer`、`uvm_default_line_printer`、`uvm_default_printer`、`uvm_default_packer`、`uvm_default_comparer`。

这些定义解释了为什么 policy、reporting、phasing、sequence、field automation 的 flags 可以在全库共享。

## Spell checker：config/resource 名称建议

`uvm_spell_chkr#(T)` 主要用于资源/config 相关提示：

- `check(ref tab_t strtab, string s)`：若 key 存在返回 1；否则遍历所有 key，计算 Levenshtein distance，找最小距离候选并用 `UVM/CONFIGDB/SPELLCHK` 信息提示“did you mean ...”。
- 如果表为空，提示没有候选。
- 算法不高效，需要遍历整个 string table，但 config/resource 表通常不大。

这解释了 UVM 在 config/resource 查找失败时能给出近似拼写建议的来源。关联 [[knowledge_uvm_7_config_resource_db]]。

## 总结：最后一轮查漏后的源码地图

完成本轮后，UVM 1.2 源码主干基本闭环：

1. `uvm_pkg.sv` / include 组织定义编译顺序。
2. `uvm_object` + macros + object globals + misc/status container 构成对象和自动化方法底座。
3. factory/core service/reporting/config/phasing/objection 构成 testbench 控制面。
4. TLM、components、sequence、reg model 构成验证建模与通信主线。
5. DPI、recording、callback、command line、event/barrier/container/DAP/link/traversal 等补齐可观测性、同步、策略保护、外部接口和辅助基础设施。

实际调试时，如果遇到 UVM 行为“不像普通 SV”的地方，优先判断它是否落在这些基础设施之一：event/barrier 的 delta 语义、DAP 的 get/set 锁定、global function 的 root 委托、field automation 的 scope/status container、或者 config/resource 的 wildcard/spell checker。
