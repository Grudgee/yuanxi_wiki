---
name: knowledge_uvm_4_policy_classes
description: UVM 1.2 policy classes：uvm_printer、uvm_comparer、uvm_packer、uvm_recorder 及其与 uvm_object 固定入口的配合
metadata:
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T05:42:30.069Z
---

# UVM 1.2 Policy Classes 源码学习

## 知识点摘要

UVM 的 `print/compare/pack/record` 并不是写在 `uvm_object` 里的一坨逻辑，而是通过 policy class 拆开：

- `uvm_printer` 决定如何格式化输出。
- `uvm_comparer` 决定如何比较、如何记录 miscompare。
- `uvm_packer` 决定如何把对象编码进 bit/byte/int stream。
- `uvm_recorder` 决定如何把对象写入 transaction database。

`uvm_object` 的固定入口方法会先执行 field automation，再调用对应 `do_*` hook；这些 hook 必须通过 policy API 追加字段，不能直接 `$display` / 手工拼字符串，否则会破坏格式、scope 和递归控制。

## 这批学习的源码范围

- `src/base/uvm_printer.svh`
- `src/base/uvm_comparer.svh`
- `src/base/uvm_packer.svh`
- `src/base/uvm_recorder.svh`
- 回看 `src/base/uvm_object.svh` 中的 `print/sprint/compare/pack/unpack/record` 入口。

## `uvm_object` 与 policy class 的连接点

### 固定入口与 hook

`uvm_object.svh` 中的关键流程：

- `print(printer)` / `sprint(printer)` → `do_print(printer)`，默认 printer 是 `uvm_default_printer`。
- `compare(rhs, comparer)` → `do_compare(rhs, comparer)`，默认 comparer 是 `uvm_default_comparer`。
- `pack(...)` / `pack_bytes(...)` / `pack_ints(...)` → `m_pack(packer)` → `do_pack(packer)`。
- `unpack(...)` / `unpack_bytes(...)` / `unpack_ints(...)` → `m_unpack_pre()` + stream 写入 packer + `m_unpack_post()` → `do_unpack(packer)`。
- `record(recorder)` → `do_record(recorder)`。

源码位置：`src/base/uvm_object.svh:1134-1321`。

### field automation 的共同依赖

这些 policy class 都依赖 `uvm_scope_stack` 与 `__m_uvm_status_container` 来维护当前字段路径、递归层次、cycle check 和 policy 状态。也就是说，field macro 不是“展开成死代码就结束了”，而是把状态交给统一容器，再调用 policy API。

## `uvm_printer`

源码位置：`src/base/uvm_printer.svh:38-1232`

### 类层次

- `uvm_printer`：抽象基类，定义打印 API 与 rows 收集机制。
- `uvm_table_printer`：表格格式。
- `uvm_tree_printer`：树形格式。
- `uvm_line_printer`：单行树形格式。
- `uvm_printer_knobs`：通用 knob 容器。

### 核心行为

`uvm_printer` 并不直接输出文本，而是先把字段抽成 `uvm_printer_row_info`，存入 `m_rows`：

- `print_field` / `print_field_int`
- `print_object` / `print_object_header`
- `print_string`
- `print_time`
- `print_real`
- `print_generic`
- `print_array_header` / `print_array_range` / `print_array_footer`

之后由 `emit()` 统一生成字符串。

### 重要实现细节

- `print_object()` 会对 `uvm_component` 的 children 做特殊处理：若对象可 cast 成 component，会先遍历 children，再打印对象自身。
- `cycle_check` 防止递归对象图无限打印。
- `adjust_name()` 根据 `show_root`、`full_name`、`identifier` 和 scope separator 决定打印 leaf 还是 full name。
- `knobs.reference` 为真时，对 object handle 打印引用字符串而不是 `-`。

### `uvm_printer_knobs`

常用 knob：

- `header` / `footer`
- `full_name` / `identifier`
- `type_name` / `size`
- `depth`
- `reference`
- `begin_elements` / `end_elements`
- `prefix`
- `indent`
- `show_root`
- `separator`
- `show_radix` / `default_radix`

`get_radix_str()` 会把 radix enum 映射成 `'h`、`'d`、`'b`、`'o` 之类的前缀。

### 子类输出风格

- `uvm_table_printer::emit()`：先算列宽，再拼 header/body/footer。
- `uvm_tree_printer::emit()`：按层级和 separator 生成树状结构。
- `uvm_line_printer`：仅把换行符和缩进改成单行输出。

### 实际意义

`do_print()` 中应该调用 printer API，而不是自己 `$display`。这样 `sprint()`、`print()`、table/tree/line 格式、knob 控制才能全部一致。

## `uvm_comparer`

源码位置：`src/base/uvm_comparer.svh:23-423`

### 角色

`uvm_comparer` 是比较 policy 对象，保存：

- `policy`：deep / reference / shallow
- `show_max`
- `verbosity`
- `sev`
- `miscompares`
- `physical` / `abstract`
- `check_type`
- `result`

### 比较 API

- `compare_field()`：比较大位宽 integral。
- `compare_field_int()`：≤64 bit 的快速路径。
- `compare_field_real()`：real 比较。
- `compare_string()`：string 比较。
- `compare_object()`：对象比较，依据 `policy` 决定 reference/深层递归。

### 关键行为

- `compare_field()` 和 `compare_field_int()` 在 miscompare 时会构造按 radix 显示的 lhs/rhs 字符串，并调用 `print_msg()`。
- `compare_object()`：
  - `policy == UVM_REFERENCE` 时只做句柄/引用比较。
  - null 检查会直接报 mismatch。
  - 非 reference 时会递归调用 `lhs.compare(rhs, this)`。
- `print_msg()` 会递增 `result`，并把信息追加到 `miscompares`。
- `print_rollup()` 在顶层 compare 结束时打印总 miscompare 数。

### 与 `uvm_object::compare` 的关系

`uvm_object::compare()` 会：

1. 选定 comparer。
2. 清理 `result`、`miscompares`、`compare_map`。
3. 做 type check、cycle check。
4. 执行 field automation 的 `UVM_COMPARE`。
5. 调用 `do_compare()`。
6. 顶层做 rollup。

因此 `do_compare()` 应始终调用 `super.do_compare()`，然后用 comparer API 比较字段。

## `uvm_packer`

源码位置：`src/base/uvm_packer.svh:25-1038`

### 角色

`uvm_packer` 是 pack/unpack policy object，负责把对象编码为连续 bit 流，并支持 byte / int 视图。

### 关键配置

- `physical` / `abstract`
- `use_metadata`
- `big_endian`
- `policy`
- `count`、`m_bits`、`m_packed_size`
- `scope`
- `nopack`

### Pack 方向

主要 API：

- `pack_field()`
- `pack_field_int()`
- `pack_bits()`
- `pack_bytes()`
- `pack_ints()`
- `pack_string()`
- `pack_time()`
- `pack_real()`
- `pack_object()`

### Unpack 方向

主要 API：

- `unpack_field()`
- `unpack_field_int()`
- `unpack_bits()`
- `unpack_bytes()`
- `unpack_ints()`
- `unpack_string()`
- `unpack_time()`
- `unpack_real()`
- `unpack_object()`
- `is_null()`

### 重要实现细节

- `pack_field()` / `pack_field_int()` 按 `big_endian` 选择 bit 顺序。
- `pack_string()` 在 `use_metadata==1` 时末尾额外加 null byte。
- `pack_object()` 在 `use_metadata==1` 时为非 null object 预置 4-bit header；null object 写 0。
- `pack_object()` 和 `unpack_object()` 都有 cycle check。
- `put_bits()` / `put_bytes()` / `put_ints()` 会把外部 stream 装进内部 `m_bits`，并重置 `count`。
- `get_bits()` / `get_bytes()` / `get_ints()` 则把内部 bitstream 导出为外部数组。
- `enough_bits()` 和 `index_error()` 是所有 unpack/get 路径的基本保护。

### metadata 语义

源码注释明确规定：

- string：末尾 null byte。
- object：4-bit 标头，0 表示 null，1 表示 non-null。
- queue / dynamic array / associative array：先编码 size。

这和 `uvm_object` 的 field automation 是配套的。

## `uvm_recorder`

源码位置：`src/base/uvm_recorder.svh:25-909`

### 角色

`uvm_recorder` 同时承担两个角色：

1. `uvm_tr_stream` 里的 record 抽象句柄。
2. 把字段记录到该 record 的 policy object。

### 生命周期与状态

关键状态：

- `m_stream_dap`
- `m_is_opened`
- `m_is_closed`
- `m_open_time`
- `m_close_time`
- `recording_depth`
- `default_radix`
- `physical` / `abstract`
- `identifier`
- `policy`

关键流程：

- `m_do_open(stream, open_time, type_name)`：初始化 recorder，并触发 `do_open()`。
- `close()`：若 opened，触发 `do_close()`，再标记 closed。
- `free()`：必要时先 close，再 `do_free()`，清内部状态，并从 stream 释放。

### Recording API

- `record_field()`
- `record_field_int()`
- `record_field_real()`
- `record_object()`
- `record_string()`
- `record_time()`
- `record_generic()`

这些 public 方法都会先检查 stream 是否存在，再调用对应 `do_record_*`。

### Mandatory backend hooks

`uvm_recorder` 把真正的后端写入留给纯虚函数：

- `do_record_field`
- `do_record_field_int`
- `do_record_field_real`
- `do_record_object`
- `do_record_string`
- `do_record_time`
- `do_record_generic`

这意味着 vendor 只需实现 backend，不必改动 object 层。

### `uvm_text_recorder`

源码里给了默认文本后端实现：`uvm_text_recorder`。

它把 recorder 操作映射成文本 transaction database 输出，例如：

- `OPEN_RECORDER`
- `CLOSE_RECORDER`
- `FREE_RECORDER`
- `SET_ATTR`

`do_record_object()` 会先记录 object 的 instance id（如果 `identifier` 打开），再根据 `policy` 决定是否递归 `value.record(this)`。

## 这批学习得到的统一模式

1. `uvm_object` 固定入口只负责流程控制，真正可定制点是 `do_*`。
2. policy class 是“行为对象”，不是简单工具类。
3. scope、cycle check、metadata、depth 控制由统一状态容器协调。
4. printer/comparer/packer/recorder 共享相同的字段自动化入口，但各自负责不同输出目标。
5. 对象图递归时必须关注 cycle check，否则打印、比较、pack、record 都可能无限递归。

## 与后续主题的衔接

这一批学完后，下一步最自然的是：

- `uvm_component` / `uvm_root`：看 object policy 如何扩展到 component tree。
- phasing / objection / domain：看 component 生命周期怎样驱动 testbench。
- report object / handler / server：看 comparer 和 recorder 之外的消息体系。

## 原文位置

- `src/base/uvm_printer.svh`
- `src/base/uvm_comparer.svh`
- `src/base/uvm_packer.svh`
- `src/base/uvm_recorder.svh`
- `src/base/uvm_object.svh`

## 关联知识

- [[knowledge_uvm_index]]
- [[knowledge_uvm_2_object_model]]
- [[knowledge_uvm_3_factory_registry]]
- [[knowledge_uvm_5_component_phasing]]
