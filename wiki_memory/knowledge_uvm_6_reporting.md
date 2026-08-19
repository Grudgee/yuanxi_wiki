---
name: knowledge_uvm_6_reporting
description: UVM 1.2 reporting 源码机制：report macros/object/handler/server/message/catcher 与退出链路
metadata:
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T06:57:49.162Z
---

# UVM 1.2 Reporting 源码学习记忆

## 学习范围与入口

本节继续 [[knowledge_uvm_index]] 的第六阶段，聚焦 UVM reporting，而暂不展开 config/resource DB。Reporting 在 `src/base/uvm_base.svh` 中的 include 顺序位于 callback 之后、transaction/phasing/component 之前：

- `base/uvm_report_message.svh`
- `base/uvm_report_catcher.svh`
- `base/uvm_report_server.svh`
- `base/uvm_report_handler.svh`
- `base/uvm_report_object.svh`

源码依据：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_base.svh:77-83`。这个顺序说明：message/catcher/server/handler/object 是 component/phasing 的基础能力，`uvm_component` 继承 reporting 后才能在 build/run 等 phase 中统一报错、计数、退出。

相关类型定义在 `uvm_object_globals.svh`：

- `uvm_severity`：`UVM_INFO/WARNING/ERROR/FATAL`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_object_globals.svh:231-248`。
- `uvm_action` 是 `int`，具体 bit 在 `uvm_action_type` 中：`UVM_DISPLAY/LOG/COUNT/EXIT/CALL_HOOK/STOP/RM_RECORD`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_object_globals.svh:253-284`。
- `uvm_verbosity`：`UVM_NONE=0`、`UVM_LOW=100`、`UVM_MEDIUM=200`、`UVM_HIGH=300`、`UVM_FULL=400`、`UVM_DEBUG=500`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_object_globals.svh:287-310`。

## 1. 用户入口：report macros 与 global reporting functions

最常用入口不是直接调用 handler/server，而是宏：

- `` `uvm_info(ID, MSG, VERBOSITY)``：先调用 `uvm_report_enabled(VERBOSITY,UVM_INFO,ID)`，通过后才构造/求值消息参数并调用 `uvm_report_info(..., report_enabled_checked=1)`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/macros/uvm_message_defines.svh:103-115`。
- `` `uvm_warning``、`` `uvm_error``、`` `uvm_fatal``：使用 `UVM_NONE` 做 enabled 检查并传给对应函数，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/macros/uvm_message_defines.svh:118-166`。
- 宏自动带入 `` `uvm_file`` / `` `uvm_line``，它们默认映射到 `` `__FILE__`` / `` `__LINE__``，也可由 `UVM_REPORT_DISABLE_FILE_LINE` 等宏关闭，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/macros/uvm_message_defines.svh:36-51`。

宏的关键价值：

1. 对 info 先做 verbosity/action 过滤，避免 `$sformatf` 等字符串构造成本。
2. 自动携带文件/行号。
3. 对 warning/error/fatal 强制 `UVM_NONE`，避免用户误以为降低 verbosity 可以关闭错误；真正关闭要改 action。

package-scope 的 global functions 是给 module/interface 或静态上下文使用的便利入口，它们最终转发给 `uvm_root`：

- `uvm_get_report_object()` 返回 `uvm_root`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_globals.svh:106-117`。
- `uvm_report_enabled()` 调 `top.uvm_report_enabled(...)`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_globals.svh:120-139`。
- `uvm_report()` 和 `uvm_report_info/warning/error/fatal()` 都通过 core service 找 root 并转发，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_globals.svh:141-248`。
- `uvm_process_report_message()` 可把已构造的 `uvm_report_message` 直接交给 `uvm_top`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_globals.svh:251-266`。

## 2. `uvm_report_object`：reporting 的对象级 facade

`uvm_report_object` 继承 `uvm_object`，是 UVM reporting 的对象级接口。`uvm_component` 因继承它而获得 `uvm_report_*`、`set_report_*` 等方法。

关键源码：

- 类定义持有 `uvm_report_handler m_rh`，构造函数中用 factory 创建 handler，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_object.svh:80-93`。
- `uvm_report_enabled()` 比较 `get_report_verbosity_level(severity,id)` 和消息 verbosity；阈值小于消息 verbosity 时过滤，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_object.svh:111-124`。
- `uvm_report()` 若没有预先 checked，会再调用 `uvm_report_enabled()`；随后 `uvm_report_message::new_report_message()`、`set_report_message()`、`uvm_process_report_message()`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_object.svh:126-146`。
- `uvm_process_report_message()` 只补上 report object，然后委托给 handler：`report_message.set_report_object(this); m_rh.process_report_message(report_message);`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_object.svh:242-251`。
- 大量 `set_report_*` 只是 thin wrapper：verbosity/action/file/severity override 最终都写入 handler，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_object.svh:259-439`。
- `set_report_handler()` 允许多个 report object 共享同一个 handler；`reset_report_handler()` 调 handler 的 `initialize()` 清空配置，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_object.svh:446-473`。

默认 action 注释也在 `uvm_report_object` 顶部说明：INFO/WARNING 默认 `UVM_DISPLAY`，ERROR 默认 `UVM_DISPLAY|UVM_COUNT`，FATAL 默认 `UVM_DISPLAY|UVM_EXIT`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_object.svh:56-67`。

## 3. `uvm_report_message`：报文在整条 reporting pipeline 中流动的载体

UVM 1.2 把 report 表示成对象，而不是只传散落参数。

关键源码：

- `uvm_report_message` 保存 `_report_object/_report_handler/_report_server` 三个基础设施引用，以及 `_severity/_id/_message/_verbosity/_filename/_line/_context_name/_action/_file`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_message.svh:473-490`。
- `new_report_message()` 会保存并恢复当前 process 的 randstate，保证创建 report message 不破坏随机稳定性，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_message.svh:503-522`。
- `set_report_message()` 一次写入 severity/id/message/verbosity/file/line/context，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_message.svh:824-843`。
- message 可携带额外 element container，并支持 `add_int/add_string/add_object`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_message.svh:815-933`。

对应的宏入口：`` `uvm_info_begin`` / `` `uvm_info_end`` 等先创建 `__uvm_msg`，中间可用 `` `uvm_message_add_tag/int/string/object`` 追加结构化字段，最后 `uvm_process_report_message(__uvm_msg)`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/macros/uvm_message_defines.svh:236-319` 和 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/macros/uvm_message_defines.svh:477-533`。

与 [[knowledge_uvm_4_policy_classes]] 的关联：这些附加 element 在 compose 时会用 `uvm_default_printer` 的 `sprint()` 形成多行输出；在 record 时会走 `uvm_recorder`，因此 reporting 直接复用了 printer/recorder policy。

## 4. `uvm_report_handler`：每个 reporter 的配置和优先级决策

`uvm_report_handler` 是 `uvm_report_object` 背后的配置表。

关键状态：

- `m_max_verbosity_level`
- `id_verbosities`、`severity_id_verbosities`
- `id_actions`、`severity_actions`、`severity_id_actions`
- `sev_overrides`、`sev_id_overrides`
- `default_file_handle`、`id_file_handles`、`severity_file_handles`、`severity_id_file_handles`

见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_handler.svh:54-78`。

初始化默认值：

- `default_file_handle=0`
- `m_max_verbosity_level=UVM_MEDIUM`
- INFO/WARNING action=`UVM_DISPLAY`
- ERROR action=`UVM_DISPLAY | UVM_COUNT`
- FATAL action=`UVM_DISPLAY | UVM_EXIT`
- 各 severity file 默认都是 default file

见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_handler.svh:367-391`。

优先级规则：

- verbosity：`(severity,id)` 优先于 `id`，否则用 `m_max_verbosity_level`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_handler.svh:429-453`。
- action：`(severity,id)` 优先于 `id`，否则用 `severity_actions[severity]`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_handler.svh:456-479`。
- file：先尝试 `(severity,id)` / `id` / `severity` / default；但 `get_file_handle()` 里会把 0 当作“继续找更低优先级”，因此显式设置为 0 不一定等同于终止查找，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_handler.svh:482-511`。
- severity override：先 id-specific override，再 generic severity override，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_handler.svh:309-332`。

处理入口 `process_report_message()` 的顺序很重要：

1. 取得全局 server：`uvm_report_server::get_server()`。
2. 根据 id/severity 做 severity override。
3. 根据最终 severity/id 设置 file、report_handler、action。
4. 调 server 的 `process_report_message()`。

见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_handler.svh:304-334`。

## 5. `uvm_report_server` / `uvm_default_report_server`：全局执行者

`uvm_report_server` 是抽象全局 server，默认实现是 `uvm_default_report_server`：

- 抽象类声明 quit count、severity/id count、message database、`process_report_message()`、`execute_report_message()`、`compose_report_message()`、`report_summarize()` 等 pure virtual 方法，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_server.svh:36-171`。
- `set_server()` / `get_server()` 是对 core service report server 的包装；`set_server()` 会先 copy 旧 server 的统计计数到新 server，再替换，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_server.svh:185-234`。
- `uvm_default_report_server` 保存 quit count、max quit count、severity/id counts、message DB、record streams，以及 `record_all_messages/show_verbosity/show_terminator` 等开关，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_server.svh:244-276`。

server 的处理分两段：

### `process_report_message()`：catch/hook/compose 调度

默认 server：

1. 把 report server 设置到 message。
2. 若 action 含 `UVM_CALL_HOOK`，先调用 deprecated hook 链。
3. 调 `uvm_report_catcher::process_all_report_catchers(report_message)`。
4. 若 action 是 `UVM_NO_ACTION`，取消处理。
5. 若需要 display/log，先 compose；再 execute。

见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_server.svh:555-632`。

### `execute_report_message()`：真正执行 action

执行顺序：

1. 增加 severity/id 统计计数。
2. 若 `record_all_messages`，给 action OR 上 `UVM_RM_RECORD`。
3. `UVM_RM_RECORD`：打开/复用 message stream，并用 `uvm_recorder` 记录 report message。
4. `UVM_DISPLAY`：`$display(composed_message)`。
5. `UVM_LOG`：写到 file handle，且对 stdout/mcd 做特殊处理避免重复写 stdout。
6. `UVM_COUNT`：若 max quit count 非 0，则增加 quit count；达到上限时给 message 加 `UVM_EXIT`。
7. `UVM_EXIT`：通过 core service 拿 root 并调用 `l_root.die()`。
8. `UVM_STOP`：执行 `$stop`。

见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_server.svh:640-738`。

### `compose_report_message()`：默认格式

默认输出格式大致为：

```text
<SEVERITY>(<VERBOSITY optional>) <file(line)> @ <time>: <reporter><@@context optional> [<id>] <message><terminator optional>
```

若 message 有 element container，则追加 `el_container.sprint()` 的多行内容；compose 会临时把 `uvm_default_printer.knobs.prefix` 改成 `" +"` 再恢复，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_server.svh:741-809`。

### summary

`report_summarize()` 先打印 report catcher summary，再打印 severity count 和 id count，最后用 `` `uvm_info("UVM/REPORT/SERVER", ...)`` 输出，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_server.svh:812-847`。

## 6. `uvm_report_catcher`：callback 形式的报文拦截/改写

`uvm_report_catcher` 本质上是 report object 上的 callback：

- `typedef uvm_callbacks#(uvm_report_object, uvm_report_catcher) uvm_report_cb;`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_catcher.svh:25-31`。
- 用户扩展类必须实现 `catch()`，返回 `THROW` 继续传递，返回 `CAUGHT` 停止后续 processing；可改 severity/id/action/verbosity/message，也可 `issue()` 立即发出，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_catcher.svh:43-69` 和 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_catcher.svh:417-426`。
- 当前 message 状态通过 static `m_modified_report_message` 暴露给 getter/setter，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_catcher.svh:108-132` 和 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_catcher.svh:145-355`。

`process_all_report_catchers()` 的细节：

1. 防递归：`in_catcher` 为 1 时直接返回。
2. 从 `uvm_report_cb::get_first(iter,l_report_object)` 开始按注册顺序迭代。
3. 跳过 callback_mode 关闭的 catcher。
4. 每个 catcher 调 `process_report_catcher()`，内部调用用户 `catch()`。
5. 如果 catcher 修改了 severity 且没有显式 set_action，且当前 action 仍等于旧 severity 的默认 action，则自动替换为新 severity 的默认 action。
6. `CAUGHT` 时统计 caught count 并停止。
7. 结束时统计 fatal/error/warning 被 demote 的次数。

见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_report_catcher.svh:571-656`。

这解释了一个常见现象：在 catcher 中把 `UVM_ERROR` demote 成 `UVM_INFO` 时，如果没显式设置 action，UVM 会尝试把 action 从 error 默认 action 调整为 info 默认 action；因此 demote 后不一定还会保留 `UVM_COUNT`。

## 7. 退出与 run_test 生命周期关联

Reporting 与 phasing/root 直接相连：

- `uvm_root::new()` 构造 root 后命名 report handler 为 `reporter`，调用 `report_header()`，再 `m_check_verbosity()` 处理全局 verbosity 设置，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_root.svh:333-346`。
- `report_header()` 会输出版本/版权/release notes；`+UVM_NO_RELNOTES` 可关闭该 notice，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_root.svh:348-392`。
- `run_test()` 在 phase 全部完成后调用 `uvm_report_server::get_server().report_summarize()`，然后按 `finish_on_completion` 决定 `$finish`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_root.svh:499-517`。
- 当 report server 执行 `UVM_EXIT` 时调用 `uvm_root::die()`；die 先 bottom-up 调所有 component 的 `pre_abort()`，再 report summary，最后 `$finish`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_root.svh:118-136`。
- `uvm_component::pre_abort()` 是用户可覆盖的 abort 前回调；底层递归实现 `m_do_pre_abort()` 先对子节点递归，再调用本节点 `pre_abort()`，即 bottom-up，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_component.svh:1368-1386` 和 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_component.svh:3631-3638`。

与 [[knowledge_uvm_5_component_phasing]] 的关联：reporting 是 phasing 的异常出口和收尾统计机制。build/connect 等 phase 中产生的 ERROR 默认带 `UVM_COUNT`，root 在 end_of_elaboration 检查 error count，若已有 build error 会 fatal 停止，相关逻辑在 `uvm_root::phase_started()` 中读取 server severity count，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_root.svh:249-262`。

## 8. 一条消息的完整源码路径

典型 `` `uvm_error("ID", "msg")`` 路径：

1. 宏展开：先 `uvm_report_enabled(UVM_NONE,UVM_ERROR,"ID")`，再 `uvm_report_error(..., report_enabled_checked=1)`。
2. 若在 component 内，调用 `uvm_report_object::uvm_report_error()`；若在 module/global 上下文，global function 先转发给 `uvm_top`。
3. `uvm_report_object::uvm_report()` 创建 `uvm_report_message`，填 severity/id/message/verbosity/file/line/context。
4. `uvm_report_object::uvm_process_report_message()` 设置 report_object，并交给该 object 的 `m_rh`。
5. `uvm_report_handler::process_report_message()` 做 severity override，计算 file/action，设置 report_handler，然后交给全局 server。
6. `uvm_default_report_server::process_report_message()` 跑 hook/catcher，必要时 compose，然后 execute。
7. `execute_report_message()` 更新统计，display/log/record/count，必要时 die/stop。

简图：

```text
`uvm_* macro / uvm_report_* global
        ↓
uvm_report_object facade
        ↓ creates/fills
uvm_report_message
        ↓
uvm_report_handler（override + verbosity/action/file 决策）
        ↓
uvm_report_server（hook/catcher + compose + execute）
        ↓
$display / $fdisplay / recorder / count / root.die() / $stop
```

## 常见误区

1. **以为降低 verbosity 可以关闭 error/fatal。** 宏层面 warning/error/fatal 使用 `UVM_NONE`，且注释明确 verbosity 对 warnings/errors/fatals 不应作为关闭机制；要关闭或改变行为，应设置 action 或用 catcher，但这通常要谨慎。
2. **忘记 macro 不带分号的风格。** `uvm_message_defines.svh` 注释说明 macros 是 text substitutions，不应被当成普通 statement 额外处理，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/macros/uvm_message_defines.svh:94-95`。
3. **混淆 handler 和 server。** handler 是每个 report object 的配置表；server 是全局执行者。多个 report object 可共享 handler，但全局 server 通常只有一个，由 core service 管理。
4. **以为 `UVM_COUNT` 本身一定退出。** 只有 max quit count 非 0，且 quit count 达到上限时，server 才给 message 加 `UVM_EXIT`，见 `uvm_report_server.svh:714-723`。
5. **catcher 中 `issue()` 可能重复计数。** 源码注释说明 issuing 当前消息会更新 report_server stats；若消息之后继续 THROW，还可能再次执行。
6. **file handle 0 的优先级语义容易误解。** handler 的 `get_file_handle()` 会在某些优先级返回 0 时继续查找下一级，因此 0 不是所有层级上的“显式禁用文件输出”。
7. **report message 创建保持随机稳定性。** `new_report_message()` 和 element add 方法保存/恢复 randstate；不要用普通对象创建行为来类比它对随机流的影响。

## 后续学习建议

下一主题建议按索引继续学习 Config/resource DB：`uvm_resource.svh`、`uvm_resource_db.svh`、`uvm_config_db.svh`、`uvm_resource_specializations.svh`。Reporting 学完后再看 config/resource 很合适，因为 config/resource 源码大量用 report/warning/error 暴露 lookup、type mismatch、deprecated API 等行为，并且它在 `uvm_base.svh` 中的 include 顺序早于 reporting，但运行时诊断依赖 reporting 基础设施。
