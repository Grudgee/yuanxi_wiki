---
name: knowledge_uvm_3_factory_registry
description: UVM 1.2 factory、registry proxy、override 查找、create 路径与 core service 关系
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T05:34:36.283Z
---

# UVM 1.2 Factory / Registry 源码学习

## 知识点摘要

UVM factory 用于制造 `uvm_object` 和 `uvm_component`，核心目的不是简单替代 `new`，而是在创建点保留“可替换性”：用户代码请求创建基类或默认类型，factory 根据 type override / instance override 决定实际生成的类型。

UVM 1.2 的 factory 体系由三层构成：

1. `uvm_object_wrapper`：抽象代理接口，能创建 object/component，并返回 type name。
2. `uvm_object_registry#(T,Tname)` / `uvm_component_registry#(T,Tname)`：具体类型的轻量 singleton proxy，负责注册到 factory 并调用 `new` 创建对象。
3. `uvm_factory` / `uvm_default_factory`：保存注册类型、override 表，执行 override 查找和实际 create。

`uvm_coreservice_t` 是全局服务入口，factory 不是直接全局变量，而是通过 core service 取得；UVM 1.2 release notes 中提到 factory 可替换，源码中 `uvm_default_coreservice_t::set_factory()` 正是替换入口。

## 相关源码

- `src/base/uvm_factory.svh`：`uvm_factory` 抽象接口、`uvm_default_factory` 实现、override 数据结构和查找算法。
- `src/base/uvm_registry.svh`：`uvm_object_registry` / `uvm_component_registry` 类型代理。
- `src/base/uvm_coreservice.svh`：`uvm_coreservice_t` 和 `uvm_default_coreservice_t`，提供 factory/report server/root/default tr database 的集中服务入口。
- `src/macros/uvm_object_defines.svh`：`uvm_object_utils` / `uvm_component_utils` 展开为 registry typedef、`get_type()`、`get_object_type()`、`create()` 等。

## Core service 与 factory 获取

`uvm_factory::get()` 只是 convenience wrapper：

```systemverilog
static function uvm_factory get();
  uvm_coreservice_t s;
  s = uvm_coreservice_t::get();
  return s.get_factory();
endfunction
```

`uvm_coreservice_t::get()` 返回 singleton core service，实际类型由宏 `UVM_CORESERVICE_TYPE` 决定，默认是 `uvm_default_coreservice_t`。

`uvm_default_coreservice_t::get_factory()`：

- 如果本地 `factory == null`，创建 `uvm_default_factory`。
- 返回当前 factory。

`set_factory(f)` 允许替换当前 factory。源码注释说明：用户若替换 factory，需要自己保存原 factory 内容或 delegate 到原 factory。

## Registry proxy 的目的

源码明确说 factory 不保存每个已注册类型的真实对象实例，而是保存轻量 wrapper/proxy。原因是：

- 避免为每个注册类型创建真实 component/object 实例。
- factory 需要创建某个类型时，调用 proxy 的 `create_object()` 或 `create_component()`。

抽象 proxy：

```systemverilog
virtual class uvm_object_wrapper;
  virtual function uvm_object create_object(string name=""); return null; endfunction
  virtual function uvm_component create_component(string name, uvm_component parent); return null; endfunction
  pure virtual function string get_type_name();
endclass
```

## `uvm_object_registry#(T,Tname)`

### 角色

`uvm_object_registry#(T,Tname)` 是 object 类型 `T` 的 lightweight proxy，继承 `uvm_object_wrapper`。

### 创建对象

```systemverilog
virtual function uvm_object create_object(string name="");
  T obj;
  if (name=="") obj = new();
  else obj = new(name);
  return obj;
endfunction
```

若定义 `UVM_OBJECT_DO_NOT_NEED_CONSTRUCTOR`，则允许 `new()` 后再 `set_name(name)`。

### Singleton proxy

```systemverilog
local static this_type me = get();
static function this_type get();
  if (me == null) begin
    factory = uvm_coreservice_t::get().get_factory();
    me = new;
    factory.register(me);
  end
  return me;
endfunction
```

关键含义：

- 每个注册类型有一个 singleton proxy。
- `get()` 首次调用时会创建 proxy 并注册到当前 factory。
- type-based factory 操作依赖每个类型只有一个 proxy instance。

### 静态 `create()`

```systemverilog
static function T create(string name="", uvm_component parent=null, string contxt="");
```

流程：

1. 若 `contxt == ""` 且 `parent != null`，使用 `parent.get_full_name()` 作为 context。
2. 调用 `factory.create_object_by_type(get(), contxt, name)`。
3. 将返回的 `uvm_object` cast 为 `T`。
4. cast 失败则 `uvm_report_fatal("FCTTYP", ...)`。

## `uvm_component_registry#(T,Tname)`

与 object registry 类似，但创建 component：

```systemverilog
virtual function uvm_component create_component(string name, uvm_component parent);
  T obj;
  obj = new(name, parent);
  return obj;
endfunction
```

静态 `create(name, parent, contxt="")` 调用：

```systemverilog
factory.create_component_by_type(get(), contxt, name, parent)
```

component create 需要 `name` 和 `parent` 两个构造参数，因此 `uvm_component_utils` 要求 component 构造函数形如：

```systemverilog
function new(string name, uvm_component parent=null);
```

## Utils 宏与 registry 的关系

`uvm_object_utils(TYPE)` 展开后核心包括：

- `typedef uvm_object_registry#(TYPE,"TYPE") type_id;`
- `static function type_id get_type(); return type_id::get(); endfunction`
- `virtual function uvm_object_wrapper get_object_type(); return type_id::get(); endfunction`
- `create()`
- `get_type_name()`

`uvm_component_utils(TYPE)` 展开后核心包括：

- `typedef uvm_component_registry#(TYPE,"TYPE") type_id;`
- `get_type()`
- `get_object_type()`
- `get_type_name()`

参数化宏 `uvm_object_param_utils` / `uvm_component_param_utils` 不提供字符串 `Tname` 注册，也不自动生成 `get_type_name()`，所以不能依赖 factory name-based lookup。

## Factory 抽象接口

`uvm_factory` 是抽象类，声明 pure virtual 方法，主要分组：

### 注册

- `register(uvm_object_wrapper obj)`

### Override 设置

- `set_type_override_by_type(original_type, override_type, replace=1)`
- `set_type_override_by_name(original_type_name, override_type_name, replace=1)`
- `set_inst_override_by_type(original_type, override_type, full_inst_path)`
- `set_inst_override_by_name(original_type_name, override_type_name, full_inst_path)`

### 创建

- `create_object_by_type(requested_type, parent_inst_path, name)`
- `create_component_by_type(requested_type, parent_inst_path, name, parent)`
- `create_object_by_name(requested_type_name, parent_inst_path, name)`
- `create_component_by_name(requested_type_name, parent_inst_path, name, parent)`

### 查询/调试

- `find_override_by_type(requested_type, full_inst_path)`
- `find_override_by_name(requested_type_name, full_inst_path)`
- `find_wrapper_by_name(type_name)`
- `debug_create_by_type(...)`
- `debug_create_by_name(...)`
- `print(all_types=1)`

## Name-based vs type-based

源码注释明确推荐 type-based interface：

- type-based：错误更少，很多错误可在编译期发现。
- name-based：大量 string 参数，容易拼错、顺序传错；错误可能到运行时才发现；对 parameterized classes 跨 simulator 不可移植。

源码甚至说明：type-based factory methods 的主要目的之一就是支持 parameterized types，并消除 string-based factory 的错误来源。因此：

> 新代码优先使用 `TYPE::type_id::create()`、`set_type_override_by_type()`、`set_inst_override_by_type()` 这类 type-based 方式。

## `uvm_default_factory` 内部数据结构

`uvm_default_factory` 保存：

- `m_types[uvm_object_wrapper]`：所有已注册 proxy，以 wrapper handle 为 key。
- `m_type_names[string]`：name-based lookup 表，type name -> wrapper。
- `m_lookup_strs[string]`：曾用于 name lookup/override 的字符串。
- `m_type_overrides[$]`：type override 队列。
- `m_inst_override_queues[uvm_object_wrapper]`：按 requested wrapper 分组的 instance override 队列。
- `m_inst_override_name_queues[string]`：original type 尚未注册时，用名字暂存的 instance override 队列。
- `m_wildcard_inst_overrides[$]`：original type name 带 wildcard 的 instance override。
- `m_override_info[$]`：当前 create/debug 查找路径记录，用于 debug 和递归 loop 检测。

`uvm_factory_override` 保存一条 override：

- `full_inst_path`
- `orig_type_name`
- `ovrd_type_name`
- `orig_type`
- `ovrd_type`
- `selected`
- `used`

构造时如果 `ovrd_type == null` 会 fatal。

## 注册流程 `register()`

`uvm_default_factory::register(obj)`：

1. `obj == null` 时 fatal。
2. 如果 `obj.get_type_name()` 非空且不是 `<unknown>`：
   - 若同名已注册，报 `TPRGED` warning，表示同名多类型不支持 string-based lookup。
   - 否则写入 `m_type_names[type_name] = obj`。
3. 如果 `m_types` 已有该 wrapper，可能报重复注册 warning。
4. 否则加入 `m_types[obj] = 1`。
5. 如果之前有基于 type name 暂存的 instance override，搬到 `m_inst_override_queues[obj]`。
6. 如果已有 wildcard instance override，则检查是否匹配该 type name，并加入该类型的 override queue。

这解释了为什么 name-based override 可以在原类型未注册时被暂存；但 override type 本身仍必须已注册。

## Type override 设置

### by type

`set_type_override_by_type(original_type, override_type, replace=1)`：

- original 与 override 相同会 warning。
- 若 original/override 未注册，会自动 `register()`，这是 type-based 方法优势之一。
- 若已有针对 original 的 type override：
  - `replace=0`：保留旧 override 并 info。
  - `replace=1`：替换旧 override 并 info。
- 若不存在旧 override，添加一条 `full_inst_path="*"` 的 override 到 `m_type_overrides`。

### by name

`set_type_override_by_name(original_type_name, override_type_name, replace=1)`：

- override type name 必须已注册，否则报 `TYPNTF` error 并返回。
- original type name 可以未注册；未注册时记录到 `m_lookup_strs`。
- original 与 override 名字相同时 warning 并忽略。
- replace 语义同 by type。

## Instance override 设置

### by type

`set_inst_override_by_type(original_type, override_type, full_inst_path)`：

- 自动注册 original/override wrapper。
- 检查相同 original/override/path 的重复 override。
- 把 override push 到 `m_inst_override_queues[original_type].queue`。

### by name

`set_inst_override_by_name(original_type_name, override_type_name, full_inst_path)`：

- override type name 必须已注册，否则报 `TYPNTF`。
- original type name 可以未注册：
  - 若 original name 带 `*`/`?`，对当前已注册类型尝试匹配，并存入 `m_wildcard_inst_overrides` 以供后续注册类型使用。
  - 否则存入 `m_inst_override_name_queues[original_type_name]`，等 original type 注册后搬迁。
- 若 original type 已注册，则直接加入该 wrapper 的 instance override queue。

## Create 路径

### Full instance path 构造

create API 会把 `parent_inst_path` 和 `name` 拼成 instance path：

- parent 为空：`full_inst_path = name`
- parent 非空且 name 非空：`full_inst_path = {parent_inst_path,".",name}`
- name 为空：`full_inst_path = parent_inst_path`

这个路径用于 instance override 匹配。

### by name create

`create_object_by_name()`：

1. 构造 `inst_path`。
2. 清空 `m_override_info`。
3. 调用 `find_override_by_name(requested_type_name, inst_path)`。
4. 如果没有 override，则查 `m_type_names[requested_type_name]`。
5. 若未注册，报 `BDTYP` warning 并返回 null。
6. 调用 wrapper `create_object(name)`。

`create_component_by_name()` 类似，但调用 `create_component(name, parent)`。

### by type create

`create_object_by_type()`：

1. 构造 `full_inst_path`。
2. 清空 `m_override_info`。
3. `requested_type = find_override_by_type(requested_type, full_inst_path)`。
4. 调用 `requested_type.create_object(name)`。

`create_component_by_type()` 类似，但调用 `create_component(name, parent)`。

## Override 查找优先级

源码和注释都说明：

1. instance override 优先于 type override。
2. instance override queue 按注册顺序处理，第一条匹配 wins。
3. 因此更具体的 instance override 应先注册，更一般的后注册。
4. type override 在无 instance override 时应用。
5. override 会递归应用：如果 `bar -> foo`，`foo -> xyz`，请求 `bar` 会得到 `xyz`。
6. 如果递归中出现 loop，factory 报 `OVRDLOOP` error，并返回形成 loop 的类型。

## `find_override_by_type()` 算法

核心流程：

1. 先检查 `m_override_info` 中是否已经出现 `requested_type`，若出现说明 override 递归 loop，报 `OVRDLOOP`。
2. 查 `m_inst_override_queues[requested_type]`：
   - 要求 `full_inst_path != ""`。
   - 队列按注册顺序扫描。
   - 条件：original type 或 original type name 匹配 requested type，并且 `uvm_is_match(override.full_inst_path, full_inst_path)`。
   - 找到后递归查 override type，除非 override type 就是 requested type。
3. 查 `m_type_overrides`：
   - exact original type 或 original type name 匹配。
   - 找到后递归查 override type。
4. 无 override 返回 requested type。

## `find_override_by_name()` 算法

核心流程：

1. 尝试从 `m_type_names` 把 requested type name 转为 wrapper。
2. 如果有 full instance path，优先查 instance override：
   - 若 requested name 尚未注册，查 `m_inst_override_name_queues[requested_type_name]`。
   - 若已注册，查 `m_inst_override_queues[rtype]`。
   - 匹配条件包括 original type name 和 full instance path 都能 `uvm_is_match()`。
3. 若 rtype 已注册但还没有为它建立 queue，且存在 wildcard instance overrides，则补建 queue。
4. 查 type override：按 original type name 精确匹配。
5. 无 override 返回 null。

by name 查找没有找到 override 时，create API 会再尝试直接查 registered type name。

## Debug 与 print

### `debug_create_by_type/name`

这两个方法执行同 create 相同的 override 搜索，但不创建对象。它们设置 `m_debug_pass=1`，收集所有相关 override，并输出：

- 请求类型
- instance path
- 相关 override 列表
- 哪条被选中，哪些被忽略
- 最终 factory 会产生的类型

### `print(all_types)`

打印 factory 状态：

- instance overrides
- type overrides
- registered types

`all_types`：

- `0`：只显示 override。
- `1`：显示用户注册类型，过滤 `uvm_*`。
- `2`：包括 UVM 内部类型。

类型无关联 type name 时显示 `<unknown>`。

## Instance override 路径语义

registry proxy 的 `set_inst_override()` 支持 parent 参数：

- `parent == null`：`inst_path` 被解释为 absolute instance path。
- `parent != null`：`inst_path` 相对 parent，实际注册路径为：
  - `inst_path == ""`：`parent.get_full_name()`
  - 否则：`{parent.get_full_name(), ".", inst_path}`

`inst_path` 可以包含 wildcard，用于匹配多个 context。

## 与 `uvm_object_utils` 的完整链路

以 object 为例：

```systemverilog
class packet extends uvm_object;
  `uvm_object_utils(packet)
endclass
```

宏生成：

```systemverilog
typedef uvm_object_registry#(packet,"packet") type_id;
static function type_id get_type(); return type_id::get(); endfunction
virtual function uvm_object_wrapper get_object_type(); return type_id::get(); endfunction
virtual function uvm_object create(string name=""); ... endfunction
virtual function string get_type_name(); return "packet"; endfunction
```

用户调用：

```systemverilog
packet p = packet::type_id::create("p", this);
```

链路：

1. `packet::type_id::create("p", this)`
2. `uvm_object_registry#(packet,"packet")::create()`
3. 获取 core service factory。
4. `factory.create_object_by_type(packet::type_id::get(), parent_context, "p")`
5. factory 查 instance override。
6. 若无 instance override，查 type override。
7. 得到最终 wrapper。
8. wrapper `create_object("p")` 调用对应类型构造函数。
9. cast 回 `packet`。如果 override 类型不是 packet 兼容类型，会 fatal。

## 常见源码级约束与坑

- type-based 优于 name-based，尤其对 parameterized classes。
- parameterized `*_param_utils` 不自动提供 type name，因此 name-based lookup/debug print 可能显示 `<unknown>`。
- by name override 的 override type 必须已经注册，否则报错。
- by type override 会自动注册 original/override wrapper。
- instance override 的注册顺序重要：第一条匹配 wins，所以更具体路径应先注册。
- `full_inst_path="*"` 的 instance override 等效于 type override，但仍位于 instance override 机制中，优先级高于 type override。
- factory override 可递归；错误配置可能形成 override loop。
- 替换 factory 时，UVM 不自动保留旧 factory 状态，用户需要自己迁移或 delegate。

## 与其它主题的关系

- 依赖 [[knowledge_uvm_2_object_model]]：`uvm_object` 的 `get_object_type()`、`create()`、`get_type_name()` 由 utils/registry 提供。
- 后续学习 `uvm_component` 时要看 component convenience methods 如何包一层 factory API。
- 后续学习 `uvm_coreservice` 还要结合 report server/root/default transaction database。

## 原文位置

- `src/base/uvm_factory.svh`：factory 抽象接口、default factory 实现、override 查找和 create/debug/print。
- `src/base/uvm_registry.svh`：object/component registry proxy 和 `type_id::create()`。
- `src/base/uvm_coreservice.svh`：core service singleton 和 default factory 获取/替换。
- `src/macros/uvm_object_defines.svh`：utils 宏如何生成 registry typedef、type/get_object/create/type_name。

## 关联知识

- [[knowledge_uvm_index]]
- [[knowledge_uvm_1_source_architecture]]
- [[knowledge_uvm_2_object_model]]
- [[knowledge_uvm_5_component_phasing]]
