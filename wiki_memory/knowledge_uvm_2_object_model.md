---
name: knowledge_uvm_2_object_model
description: UVM 1.2 uvm_object 基类、对象生命周期、数据方法、field automation 与相关全局类型
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T03:41:53.479Z
---

# UVM 1.2 Object 模型源码学习

## 知识点摘要

`uvm_object` 是 UVM 数据对象和层次化类的公共基类，继承自更底层的 `uvm_void`。它定义了 UVM 对象的统一能力：命名、实例 ID、随机稳定 seeding、factory proxy 查询、create/clone/copy、compare、print/sprint、record、pack/unpack，以及通过 field automation 宏进行自动数据操作。

UVM object 模型的核心设计模式是：用户不要重载固定入口方法（如 `copy`、`compare`、`print`、`pack`），而是重写对应的 `do_*` hook（如 `do_copy`、`do_compare`、`do_print`、`do_pack`）。field macros 会生成 `__m_uvm_field_automation()`，固定入口方法先执行宏生成的自动字段逻辑，再调用用户自定义 hook。

## 继承与基础类型

### `uvm_void`

源码位置：`src/base/uvm_misc.svh`

```systemverilog
virtual class uvm_void;
endclass
```

- `uvm_void` 是所有 UVM class 的最底层基类。
- 它本身没有数据成员和函数。
- 作用类似 C 语言中的 `void*`，使不同 UVM 对象可以放进泛型容器。
- 直接继承 `uvm_void` 的用户类不会获得 `uvm_object` 的 copy/compare/print/factory 等能力。

### `uvm_object`

源码位置：`src/base/uvm_object.svh`

```systemverilog
virtual class uvm_object extends uvm_void;
```

`uvm_object` 是 UVM 数据对象和层次化类的公共基类。源码注释明确说它的主要职责是定义 common operations：

- `create`
- `copy`
- `compare`
- `print`
- `record`

## 命名与实例 ID

### 构造函数

```systemverilog
function uvm_object::new (string name="");
  m_inst_id = m_inst_count++;
  m_leaf_name = name;
endfunction
```

关键点：

- 每个 `uvm_object` 分配时获得唯一 `m_inst_id`。
- `m_inst_count` 是静态计数器，记录已分配的 `uvm_object` 数量。
- `m_leaf_name` 保存对象局部名称。

### 名称 API

- `set_name(string name)`：覆盖对象名。
- `get_name()`：返回构造或 `set_name()` 设置的 leaf name。
- `get_full_name()`：默认等同 `get_name()`。

普通 `uvm_object` 不天然属于组件层次，所以默认没有真正 hierarchy。`uvm_component` 会 override `get_full_name()`。某些非 component 对象也可能 override，例如 sequence 可把 sequencer full name 与 sequence name 拼接，方便 debug。

## Seeding 机制

源码要点：

```systemverilog
static bit use_uvm_seeding = 1;
function void uvm_object::reseed();
  if(use_uvm_seeding)
    this.srandom(uvm_create_random_seed(get_type_name(), get_full_name()));
endfunction
```

含义：

- `use_uvm_seeding` 控制是否启用 UVM seeding。
- 启用时，对象 seed 基于 type name 和 full hierarchical name，而不是线程中的 allocation order。
- 这样可提高 random stability，尤其适合 instance name 唯一的类型，例如 `uvm_component`。
- 相关辅助函数在 `uvm_misc.svh`：`uvm_create_random_seed()`、`uvm_oneway_hash()`、`uvm_random_seed_table_lookup`。

## Factory 相关接口

### `get_type()`

`uvm_object::get_type()` 默认报错并返回 null：

- 用户派生类必须通过手写或 `uvm_*_utils` 宏实现。
- factory 的 type-based override 和 create API 使用 `uvm_object_wrapper` 作为类型代理。

### `get_object_type()`

默认实现：

1. 通过 `uvm_coreservice_t::get()` 获取 core service。
2. 通过 core service 获取 factory。
3. 若 `get_type_name()` 是 `"<unknown>"`，返回 null。
4. 否则通过 factory `find_wrapper_by_name(get_type_name())` 查找 wrapper。

使用 `uvm_object_utils` 等宏时，会生成更直接的 `get_object_type()`，返回 `type_id::get()`。

### `get_type_name()`

默认返回 `"<unknown>"`。每个派生类应定义自己的 type name。`uvm_object_utils` 会生成：

```systemverilog
const static string type_name = "T";
virtual function string get_type_name();
  return type_name;
endfunction
```

## 创建、克隆与复制

### `create()`

默认：

```systemverilog
virtual function uvm_object create(string name=""); return null; endfunction
```

派生类必须实现。`uvm_object_utils` 会生成 `create()`，调用构造函数：

- name 为空时 `new()`；
- name 非空时 `new(name)`；
- 若定义 `UVM_OBJECT_DO_NOT_NEED_CONSTRUCTOR`，则允许 `new()` 后再 `set_name(name)`。

### `clone()`

实现流程：

```systemverilog
tmp = this.create(get_name());
if(tmp == null) warning;
else tmp.copy(this);
return tmp;
```

含义：

- `clone()` = `create()` + `copy()`。
- 如果 `create()` 未实现或失败，则无法 clone。
- 因此 object 想支持 clone，必须正确实现 `create()`，通常用 `uvm_object_utils` 宏。

### `copy()` 与 `do_copy()`

`copy(rhs)` 是固定入口，不应在派生类中重载。派生类应重写 `do_copy(rhs)`。

源码流程：

1. 若 `rhs == null`，报 `NULLCP` warning 并忽略。
2. 使用静态 `uvm_global_copy_map` 做 cycle checking，避免递归对象图复制时无限循环。
3. 调用 `__m_uvm_field_automation(rhs, UVM_COPY, "")` 执行 field macros 生成的字段复制逻辑。
4. 调用 `do_copy(rhs)` 执行用户自定义字段复制。
5. 顶层复制完成后清空 copy map。

用户实现 `do_copy` 必须：

- 调用 `super.do_copy(rhs)`；
- 将 `rhs` `$cast` 到派生类型；
- 再复制派生类字段。

## 比较模型

### `compare(rhs, comparer)`

`compare()` 是固定入口。派生类应重写 `do_compare(rhs, comparer)`。

关键源码行为：

- 若未传入 comparer，则使用全局 `uvm_default_comparer`。
- 顶层 compare 会清理 `compare_map`、`result` 和 `miscompares`，并建立 scope。
- 若 `rhs == null`，记录 object mismatch。
- 若 compare_map 发现 rhs 已比较过，避免递归对象图无限 compare。
- 若 `comparer.check_type` 为真且左右对象 `get_type_name()` 不同，会记录类型 mismatch。
- 然后执行：
  1. `__m_uvm_field_automation(rhs, UVM_COMPARE, "")`
  2. `dc = do_compare(rhs, comparer)`
- 返回条件：`comparer.result == 0 && dc == 1`。

用户实现 `do_compare` 必须：

- 调用 `super.do_compare(rhs, comparer)`；
- `$cast` rhs 到派生类型；
- 使用 `uvm_comparer` API 比较字段，而不是直接 `==` 后自己打印。

## 打印与字符串转换

### `print()` / `sprint()` / `do_print()`

- `print(printer)`：固定入口，不应重载；默认使用 `uvm_default_printer`，通过 `$fwrite(printer.knobs.mcd, sprint(printer))` 输出。
- `sprint(printer)`：返回字符串；如果不是 top-level print，会直接运行 field automation + `do_print` 后返回空字符串；顶层则调用 `printer.print_object(get_name(), this)`，最后 `emit()`。
- `do_print(printer)`：用户 hook，用于补充自定义字段。

用户 `do_print()` 不应直接 `$display` 或手动拼接格式，而应调用 `printer` API，以保持格式一致。

### `convert2string()`

- 默认返回空字符串。
- 是完全自定义的用户字符串转换 hook。
- 与 `print/sprint` 不同，不要求使用 `uvm_printer`。
- field macros 声明的字段不会自动出现在 `convert2string()` 里。

## Recording

### `record(recorder)` / `do_record(recorder)`

- `record()` 是固定入口，不应重载。
- 如果 `recorder == null`，源码直接返回。
- 设置 `__m_uvm_status_container.recorder`，增加 `recording_depth`。
- 先执行 field automation 的 `UVM_RECORD`，再调用 `do_record(recorder)`。
- `do_record()` 是用户 hook。

recording 是 vendor-specific 机制的抽象，用户通过 `uvm_recorder` policy 使用 simulator 的 recording 能力。

## Pack / Unpack

### Pack 固定入口

- `pack(ref bit bitstream[])`
- `pack_bytes(ref byte unsigned bytestream[])`
- `pack_ints(ref int unsigned intstream[])`

内部都调用 `m_pack(packer)`：

1. 选择传入 packer 或 `uvm_default_packer`。
2. `packer.reset()`。
3. `packer.scope.down(get_name())`。
4. 执行 `__m_uvm_field_automation(null, UVM_PACK, "")`。
5. 调用 `do_pack(packer)`。
6. `packer.set_packed_size()`。
7. scope 回退。

### Unpack 固定入口

- `unpack(ref bit bitstream[])`
- `unpack_bytes(ref byte unsigned bytestream[])`
- `unpack_ints(ref int unsigned intstream[])`

内部流程：

1. `m_unpack_pre()` 选择 packer 并 reset。
2. 把输入 bit/byte/int stream 放入 packer。
3. `m_unpack_post()`：
   - 记录 provided size。
   - scope down。
   - 执行 field automation 的 `UVM_UNPACK`。
   - 调用 `do_unpack(packer)`。
   - scope up。
   - 如果 unpack 消费 bit 数和 provided size 不一致，报 `BDUNPK` warning。
4. 设置 packed size 并返回。

### Pack/unpack 约束

源码注释强调：

- unpack 顺序必须与 pack 顺序完全一致。
- 对动态数据结构，如果希望 unpack 恢复等价结构，pack 时必须包含 metadata：
  - queue/dynamic array/associative array：元素个数，通常 32 bit。
  - string：末尾 null byte。
  - object：对象存在性 4 bit，null 为 `4'b0000`，非 null 为 `4'b0001`。
- field macros 会在 `uvm_packer::use_metadata` 打开时自动加入 metadata。

## Local set/get 配置入口

`uvm_object` 定义三个 local setter：

- `set_int_local(field_name, value, recurse=1)`
- `set_string_local(field_name, value, recurse=1)`
- `set_object_local(field_name, value, clone=1, recurse=1)`

它们用于 command-line debugging 和 auto-configuration，不建议用户直接调用。

实现机制：

- 清理 cycle check 状态。
- 把待设置值写入静态 `__m_uvm_status_container`。
- 调用 `__m_uvm_field_automation()`，分别传 `UVM_SETINT`、`UVM_SETSTR`、`UVM_SETOBJ`。
- field macros 用 `uvm_is_match()` 匹配字段名，因此 `field_name` 支持 wildcard。
- `set_object_local()` 默认 clone 输入对象；clone 后会把 clone 的 name 设置为 `field_name`。
- 如果启用 warning 且没有匹配字段，会报 `NOMTC`。

## `uvm_object_globals.svh` 中相关全局类型

源码位置：`src/base/uvm_object_globals.svh`

### 关键 typedef

- `uvm_bitstream_t`：`logic signed [UVM_STREAMBITS-1:0]`，用于传递大位宽 integral field。
- `uvm_integral_t`：`logic signed [63:0]`，用于 64 bit 或以下 integral field。

### Radix

`uvm_radix_enum` 定义 print/record 格式：

- `UVM_BIN`
- `UVM_DEC`
- `UVM_UNSIGNED`
- `UVM_UNFORMAT2`
- `UVM_UNFORMAT4`
- `UVM_OCT`
- `UVM_HEX`
- `UVM_STRING`
- `UVM_TIME`
- `UVM_ENUM`
- `UVM_REAL`
- `UVM_REAL_DEC`
- `UVM_REAL_EXP`
- `UVM_NORADIX`

`uvm_radix_to_string()` 将这些 radix 转成格式字符，例如 `UVM_HEX -> "h"`。

### Field macro flags

常用 flags：

- 操作控制：`UVM_COPY`、`UVM_COMPARE`、`UVM_PRINT`、`UVM_RECORD`、`UVM_PACK`。
- 反向关闭：`UVM_NOCOPY`、`UVM_NOCOMPARE`、`UVM_NOPRINT`、`UVM_NORECORD`、`UVM_NOPACK`。
- 对象递归策略：`UVM_DEEP`、`UVM_SHALLOW`、`UVM_REFERENCE`。
- 字段分类：`UVM_PHYSICAL`、`UVM_ABSTRACT`。
- 配置控制：`UVM_READONLY`。
- 默认：`UVM_DEFAULT`、`UVM_ALL_ON`。

源码里 `UVM_DEFAULT` 当前等价于打开主要操作，但注释建议使用 `UVM_DEFAULT`，因为未来可能增加推荐默认关闭的新 flags。

### 默认 policy object

`uvm_object_globals.svh` 中定义全局默认 policy：

- `uvm_default_table_printer`
- `uvm_default_tree_printer`
- `uvm_default_line_printer`
- `uvm_default_printer = uvm_default_table_printer`
- `uvm_default_packer`
- `uvm_default_comparer`

`uvm_object` 的 print/pack/compare 在未传 policy 时使用这些默认对象。

## `uvm_misc.svh` 中相关工具

### `uvm_scope_stack`

用于维护字段操作过程中的 scope 路径：

- `down()` / `up()` 进入/退出字段层次。
- `down_element()` / `up_element()` 处理数组元素下标。
- `set_arg()` / `unset_arg()` 处理当前字段。
- `get()` 返回类似 `a.b[0].c` 的完整 scope。

field automation、comparer、packer、printer 都依赖 scope stack 生成字段路径。

### `uvm_status_container`

内部状态容器，服务 field automation：

- 保存 set/get 中的临时值：bitstream、string、object。
- 保存当前 policy：comparer、packer、recorder、printer。
- 保存 scope stack。
- 记录 field duplication check 状态。
- 维护 object cycle check，避免 field automation 对递归对象图无限下降。

`uvm_object` 中有一个静态：

```systemverilog
static uvm_status_container __m_uvm_status_container = new;
```

这解释了为什么很多 field macro 通过 `uvm_object::__m_uvm_status_container` 交换状态。

### 随机 seed 工具

- `uvm_global_random_seed = $urandom`
- `uvm_seed_map`
- `uvm_random_seed_table_lookup`
- `uvm_oneway_hash()`
- `uvm_create_random_seed(type_id, inst_id)`

这些函数让对象 seed 与 type name + instance name 关联，提高随机稳定性。

## Object utility macros

源码位置：`src/macros/uvm_object_defines.svh`

### Utils 宏族

每个继承自 `uvm_object` 的用户类通常应使用 utils 宏：

- 简单非参数化 object，无 field macros：
  ```systemverilog
  `uvm_object_utils(TYPE)
  ```
- 非参数化 object，带 field macros：
  ```systemverilog
  `uvm_object_utils_begin(TYPE)
    `uvm_field_*(...)
  `uvm_object_utils_end
  ```
- 参数化 object，无 field macros：
  ```systemverilog
  `uvm_object_param_utils(TYPE)
  ```
- 参数化 object，带 field macros：
  ```systemverilog
  `uvm_object_param_utils_begin(TYPE)
    `uvm_field_*(...)
  `uvm_object_utils_end
  ```

非参数化 utils 做的事：

1. 实现 `get_type_name()`。
2. 实现 `create()`。
3. 用字符串 type name 注册到 factory。
4. 实现静态 `get_type()`。
5. 实现虚函数 `get_object_type()`。

参数化 utils 不提供字符串 type name 注册，因此 parameterized class 不能通过 factory name-based lookup 创建。

### Field automation 入口

`uvm_field_utils_begin(T)` 生成：

```systemverilog
function void __m_uvm_field_automation(uvm_object tmp_data__, int what__, string str__);
```

它会：

- 定义 `local_data__` 并在 copy/compare 时 cast `tmp_data__`。
- 对 set 操作做 cycle check。
- 调用 `super.__m_uvm_field_automation(...)`，保证基类字段先参与操作。
- 后续每个 `uvm_field_*` 宏根据 `what__` 执行对应操作。

## Field macros 机制

field macros 用一个统一的 `what__` 分发不同操作：

- `UVM_COPY`
- `UVM_COMPARE`
- `UVM_PACK`
- `UVM_UNPACK`
- `UVM_RECORD`
- `UVM_PRINT`
- `UVM_SETINT`
- `UVM_SETSTR`
- `UVM_SETOBJ`
- `UVM_CHECK_FIELDS`

### 常用字段宏

标量：

- `uvm_field_int`
- `uvm_field_real`
- `uvm_field_enum`
- `uvm_field_object`
- `uvm_field_event`
- `uvm_field_string`

数组/队列：

- `uvm_field_sarray_*`：静态数组。
- `uvm_field_array_*`：动态数组。
- `uvm_field_queue_*`：queue。

关联数组：

- `uvm_field_aa_*_string`
- `uvm_field_aa_*_int`
- `uvm_field_aa_int_key(KEY, ARG, FLAG)` 等。

### `uvm_field_object` 的重要行为

- Copy：
  - 若 flag 有 `UVM_REFERENCE` 或源对象为 null，则只复制 handle。
  - 否则调用源对象 `clone()` 深拷贝，再 cast 回目标字段类型。
  - clone 失败会 `uvm_fatal("FAILCLN", ...)`。
- Compare：调用 comparer `compare_object()`。
- Pack/unpack：若非 `UVM_REFERENCE` 且未 `UVM_NOPACK`，递归 pack/unpack object。
- Print：`UVM_REFERENCE` 时只打印 object header，否则递归 print object。
- Set：若非 readonly/reference，可能递归进入子对象字段。

### Component handle 的坑

源码注释明确警告：

- `uvm_component` 不应被 `uvm_field_object` 注册，除非 flag 包含 `UVM_REFERENCE`。
- 否则 field macro 会尝试 deep copy component，这是非法操作。
- 可能导致 FATAL，或打印 topology 时出现重复条目。

### Static array 与 dynamic array 差异

- static array 不能 resize；set 操作匹配整个数组名时会 warning。
- dynamic array/queue 在 pack metadata 或 set size 时可 resize。
- `M_UVM_ARRAY_RESIZE` 使用 `ARG = new[sz](ARG)`。
- `M_UVM_QUEUE_RESIZE` 使用 push/pop 调整 size。
- `M_UVM_SARRAY_RESIZE` 什么也不做。

### Associative array 注意点

- 对 string key 和 int/integral key 有不同宏展开。
- compare 时会先比较数量，再逐 key 比较内容。
- object associative array 在非 `UVM_REFERENCE` 时 copy 通常会 clone 每个元素。

## 推荐使用方式与源码约束

### 推荐

- 每个直接或间接继承 `uvm_object` 的用户类都使用合适的 `uvm_object_utils*` 宏，除非确实要手写 factory/create/type_name。
- 对需要自动 copy/compare/print/pack 的字段使用 field macros，但性能敏感或行为复杂的对象应手写 `do_*`。
- 派生类实现 `do_copy`、`do_compare`、`do_print`、`do_pack`、`do_unpack` 时都要先调用 `super.do_*`。
- `compare` 中应使用 `uvm_comparer`，`print` 中应使用 `uvm_printer`，`pack` 中应使用 `uvm_packer`。

### 不推荐/风险

- 不要重载 `copy`、`compare`、`print`、`pack`、`unpack` 这些固定入口。
- 不要忘记实现 `create()`，否则 `clone()` 会失败。
- 不要用 `uvm_field_object` 对 component handle 做 deep copy；需要时使用 `UVM_REFERENCE`。
- 参数化类使用 `uvm_object_param_utils` 后不能依赖 factory name-based lookup。
- `convert2string()` 不会自动包含 field macro 字段，必须手写。

## 本批次未深入内容

本批次聚焦 object 模型与 object macros。以下内容只建立关联，未展开源码：

- `uvm_factory.svh` / `uvm_registry.svh`：factory proxy 和 override 机制。
- `uvm_printer.svh` / `uvm_comparer.svh` / `uvm_packer.svh` / `uvm_recorder.svh`：policy class 细节。
- `uvm_component.svh`：component 如何继承并扩展 object。
- `uvm_sequence_item.svh`：sequence item 如何建立在 object/transaction 之上。

## 原文位置

- `src/base/uvm_object.svh`：`uvm_object` 类声明与实现。
- `src/base/uvm_object_globals.svh`：field flags、radix、默认 policy objects、相关 enum/typedef。
- `src/base/uvm_misc.svh`：`uvm_void`、`uvm_scope_stack`、`uvm_status_container`、seed/hash 工具。
- `src/macros/uvm_object_defines.svh`：object/component utils、field automation、field macros。

## 关联知识

- [[knowledge_uvm_index]]
- [[knowledge_uvm_1_source_architecture]]
- [[knowledge_uvm_3_factory_registry]]
- [[knowledge_uvm_4_policy_classes]]
- [[knowledge_uvm_5_component_phasing]]
