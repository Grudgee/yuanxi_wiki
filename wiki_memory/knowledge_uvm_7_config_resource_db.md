---
name: knowledge_uvm_7_config_resource_db
description: UVM 1.2 config/resource DB 源码机制：resource pool、scope/name/type/precedence、config_db、自动配置与命令行入口
metadata:
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T07:05:24.303Z
---

# UVM 1.2 Config / Resource DB 源码学习记忆

## 学习范围与入口

本节继续 [[knowledge_uvm_index]] 的第七阶段，聚焦 UVM 1.2 的 resource DB 与 config DB。源码入口在 `src/base/uvm_base.svh` 的 “Resources/configuration facility” 段，顺序是：

1. `base/uvm_spell_chkr.svh`
2. `base/uvm_resource.svh`
3. 可选 deprecated `uvm_resource_converter.svh`
4. `base/uvm_resource_specializations.svh`
5. `base/uvm_resource_db.svh`
6. `base/uvm_config_db.svh`

源码依据：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_base.svh:49-58`。

这个顺序很关键：`uvm_config_db#(T)` 是 `uvm_resource_db#(T)` 之上的 component 配置便利层；`uvm_resource_db#(T)` 又是 `uvm_resource#(T)` / `uvm_resource_pool` 之上的静态便利 API。底层真正全局存储在 `uvm_resource_pool`。

## 1. Resource facility 的设计模型

`uvm_resource.svh` 顶部注释把 resource 定义为“parameterized container”，可存任意 SV 类型，包括标量、对象句柄、队列、list、virtual interface 等；resource 可用于配置 component、给 sequence 提供数据、或跨 testbench 区域共享信息，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:24-39`。

底层模型：

- 每个 resource 有 `name`、`type handle`、`scope`、`precedence` 和 typed value。
- resource 同时进入全局 pool 的 name table 与 type table，可按 name 或 type 查找。
- scope 表示 resource 对哪些 hierarchical scopes 可见；resource 保存的是 scope 正则表达式。
- 多个同名或同类型 resource 存在 queue 中，搜索时结合 scope 匹配与 precedence 决策。

源码注释说明 resource queue 原始搜索是 front-to-back；还可以通过 `precedence` 或 set_priority 改变优先级。若多个匹配 resource 有相同最高 precedence，则队列中最早找到者获胜，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:52-67`。

## 2. `uvm_resource_base`：非参数化基类

`uvm_resource_base` 是所有 resource 的非参数化基类，继承 `uvm_object`：

- 成员：`scope`、`modified`、`read_only`、`access[string]`、`precedence`。
- 默认 precedence 是静态 `default_precedence = 1000`。
- 构造函数 `new(name, s)` 调 `set_scope(s)`，初始 `modified=0`、`read_only=0`、`precedence=default_precedence`。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:198-236`。

### scope 语义

resource 支持 glob 和正则：

- 正则用 `/.../` 包起来。
- 其他表达式按 glob 处理，内部用 `uvm_glob_to_re()` 转为正则。
- 匹配用 DPI `uvm_re_match()`。

说明与例子见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:299-375`。

实现上：

- `set_scope(s)`：`scope = uvm_glob_to_re(s)`。
- `match_scope(s)`：`uvm_re_match(scope, s) == 0`。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:377-405`。

### read-only、modified、audit

- `set_read_only()` 后，typed resource 的 `write()` 会报错并返回，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:250-278` 和 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:1604-1609`。
- `wait_modified()` 等待 `modified==1`，被释放后清零，可重复等待，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:281-296`。
- audit 默认开启，由 `uvm_resource_options::auditing` 控制；read/write 会记录 accessor 的 full name、时间和次数，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:136-188` 和 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:446-577`。

与 [[knowledge_uvm_6_reporting]] 的关联：resource debug/audit 输出通过 `` `uvm_info`` / `uvm_report_*` 发出，例如 `print_accessors()` 使用 `UVM/RESOURCE/ACCESSOR`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:540-565`。

## 3. `uvm_resource_pool`：全局单例数据库

`uvm_resource_pool` 是真正的全局资源池，文件末尾还给出 package-scope const handle：`const uvm_resource_pool uvm_resources = uvm_resource_pool::get();`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:1690-1693`。

核心成员：

- `rtab[string]`：name table，key 是 resource name，value 是 resource queue。
- `ttab[uvm_resource_base]`：type table，key 是 type handle，value 是 resource queue。
- `get_record[$]`：get history，记录成功/失败查找。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:595-664`。

单例：`static local uvm_resource_pool rp = get();`，`get()` 若空则 new，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:656-677`。

### set：同时进入 name map 与 type map

`uvm_resource_pool::set(rsrc, override)`：

- null resource 直接返回。
- 非匿名 name 进入 `rtab[name]`；name override 时 `push_front`，否则 `push_back`。
- 取 `rsrc.get_type_handle()` 进入 `ttab[type_handle]`；type override 时 `push_front`，否则 `push_back`。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:713-760`。

便捷 override：

- `set_override()`：同时 name/type override。
- `set_name_override()`：只 name override。
- `set_type_override()`：只 type override。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:762-790`。

### lookup/get：name、type、scope、precedence

按 name 查找：

- `lookup_name(scope, name, type_handle, rpterr)` 先查 `rtab[name]`；若不存在且 `rpterr` 为真，调用 spell checker。
- 逐个检查 type_handle（可为 null）和 `r.match_scope(scope)`。
- 返回所有匹配 resource queue。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:857-909`。

`get_by_name()` 在 `lookup_name()` 后调用 `get_highest_precedence(q)`，并记录 get 成功/失败，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:972-1000`。

`get_highest_precedence()` 从 queue 中选 `precedence` 最大的 resource；同 precedence 时保留先遇到的 resource，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:911-943`。

按 type 查找：

- `lookup_type(scope,type_handle)` 只查 type table，再过滤 scope。
- `get_by_type()` 返回 queue 中第一个匹配项，而不是再次按 precedence 选最高。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:1003-1054`。

`lookup_scope(scope)` 返回对某 scope 可见的所有 resource；它遍历 name table，且为了 array autoconfig 的特殊情况反向迭代 name，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:1100-1135`。

### priority 调整

`set_priority_queue()` 把某 resource 从 queue 中删除后，根据 `PRI_HIGH` push_front 或 `PRI_LOW` push_back。`set_priority_type/name` 分别改 type map 或 name map；`set_priority()` 两者都改，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:1137-1258`。

注意：priority 调整改变的是 queue 顺序，而 `get_by_name()` 还会比较 `precedence`。因此 queue 顺序只在 precedence 相同或 type lookup 的“第一个匹配”语义中特别关键。

## 4. `uvm_resource#(T)`：typed resource container

`uvm_resource#(T)` 继承 `uvm_resource_base`，保存 typed value：`protected T val;`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:1370-1379`。

类型安全依赖静态 type handle：

- `typedef uvm_resource#(T) this_type;`
- `static this_type my_type = get_type();`
- `get_type()` 返回每种 `T` 对应的 static singleton。
- `get_type_handle()` 多态返回 `get_type()`。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:1433-1463`。

set/get 便利：

- instance `set()`：把自己放入 global resource pool。
- instance `set_override()`：以 override 方式放入 pool。
- static `get_by_name(scope,name,rpterr)`：通过 pool 查找并 `$cast` 成当前 `T` 的 resource。
- static `get_by_type(scope,type_handle)`：按 type handle 查找并 `$cast`。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:1477-1567`。

read/write：

- `read(accessor)` 记录 read audit 后返回 `val`。
- `write(t, accessor)` 若 read-only 则 error；若 `val == t` 则直接返回，不记录 audit、不触发 modified；否则记录 write audit、更新 `val` 并设置 `modified=1`。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:1580-1621`。

typed `get_highest_precedence()` 会在 queue 中只考虑能 `$cast` 到 `uvm_resource#(T)` 的资源，然后选最高 precedence，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource.svh:1643-1686`。

## 5. Resource specializations：debug string 输出与 subtype get

`uvm_resource_specializations.svh` 定义若干常用 specialization，例如：

- `uvm_int_rsrc extends uvm_resource#(int)`，`convert2string()` 用十进制输出。
- `uvm_string_rsrc extends uvm_resource#(string)`，`convert2string()` 返回字符串。
- `uvm_obj_rsrc extends uvm_resource#(uvm_object)`。
- `uvm_bit_rsrc#(N)` / `uvm_byte_rsrc#(N)`。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource_specializations.svh:63-168`。

宏 `UVM_RESOURCE_GET_FCNS(base_type)` 生成 specialization 的 `get_by_name/get_by_type`，内部先调用 base `uvm_resource#(base_type)::get_by_*` 再 downcast；失败则 fatal，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource_specializations.svh:19-60`。

## 6. `uvm_resource_db#(T)`：resource 便利 API 层

`uvm_resource_db#(T)` 是全 static API：`typedef uvm_resource#(T) rsrc_t;`，用户以 `uvm_resource_db#(int)::set(...)` 形式调用，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource_db.svh:40-56`。

常用 API：

- `get_by_type(scope)` → `uvm_resource#(T)::get_by_type(scope, type_handle)`。
- `get_by_name(scope,name,rpterr)` → typed resource lookup。
- `set_default(scope,name)`：创建资源、set 到 DB，但不 write 值。
- `set(scope,name,val,accessor)`：new resource、write、set。
- `set_anonymous(scope,val,accessor)`：name 为空，不进 name map，只进 type map。
- `set_override/set_override_type/set_override_name()`：控制进入 name/type queue 头部。
- `read_by_name/read_by_type()`：查找后 read value。
- `write_by_name/write_by_type()`：先查找，找到后写；找不到返回 0，不自动创建。
- `dump()`：dump 全 resource pool。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource_db.svh:60-326`。

注意 `write_by_name/write_by_type()` 注释提醒：因为 resource 的 scope 可能是正则，写到“匹配到的 resource”可能影响其他 scope 中也能匹配该 resource 的对象；使用时要谨慎，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource_db.svh:259-314`。

resource DB tracing：`+UVM_RESOURCE_DB_TRACE` 或 `uvm_resource_db_options::turn_on_tracing()` 开启，read/write/set 时会通过 `` `uvm_info`` 打印访问，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource_db.svh:32-34` 和 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_resource_db.svh:331-398`。

## 7. `uvm_config_db#(T)`：component 配置便利层

`uvm_config_db#(T)` 继承 `uvm_resource_db#(T)`，所有 API 也都是 static。它的作用是把 component-relative `cntxt + inst_name + field_name` 转成 resource pool 的 scope/name/type lookup。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_config_db.svh:50-65`。

内部状态：

- `m_rsc[uvm_component]`：每个 context 有一个 pool，key 是 `{inst_name,"__M_UVM__",field_name}`，用于复用已经创建的 resource，避免同一个 context/scope/field 重复 new。
- `m_waiters[field_name]`：`wait_modified()` 的 waiter 队列。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_config_db.svh:66-72`。

### get：构造绝对 scope，再按 name/type 查 resource

`get(cntxt, inst_name, field_name, value)`：

1. 若 `cntxt == null`，改为 root。
2. 若 `inst_name == ""`，使用 `cntxt.get_full_name()`。
3. 否则若 `cntxt.get_full_name() != ""`，拼接 `{cntxt.get_full_name(), ".", inst_name}`。
4. 调 `rp.lookup_regex_names(inst_name, field_name, uvm_resource#(T)::get_type())`。
5. 从返回 queue 中选 typed `uvm_resource#(T)::get_highest_precedence(rq)`。
6. tracing 后若找到，`value = r.read(cntxt)`。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_config_db.svh:88-118`。

这意味着 config_db 的 `field_name` 实际进入 resource name map；`inst_name` 经过 context 拼接后作为 current scope 去匹配 resource 的 scope 正则。

### set：build-time 层级 precedence 与 run-time last-wins

`set(cntxt, inst_name, field_name, value)`：

1. 保存 process randstate，避免分配 resource 破坏随机稳定性。
2. 若 `cntxt == null`，用 root。
3. 用相同规则把 `cntxt + inst_name` 拼成目标 scope。
4. 在 `m_rsc[cntxt]` 中查 `{inst_name,"__M_UVM__",field_name}`，复用或创建 `uvm_resource#(T)(field_name, inst_name)`。
5. 若当前 phase 是 `build`，`r.precedence = default_precedence - cntxt.get_depth()`；否则用 default precedence。
6. `r.write(value,cntxt)`。
7. 如果 resource 已存在，调用 `rp.set_priority_name(r, PRI_HIGH)`；如果首次创建，`r.set_override()`，即 name/type 都进队头。
8. 触发 waiters：按 field_name 找 waiter，并用 `uvm_re_match(uvm_glob_to_re(inst_name), w.inst_name)` 判断是否命中。
9. 恢复 randstate，按需 tracing。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_config_db.svh:151-232`。

关键语义来自注释：

- build time：hierarchically higher 的设置 precedence 更高；root/top 使用 `default_precedence`，每往下一层减 1。
- 同一层级：last setting wins。
- build 之后：所有设置都用 default precedence，因此运行期低层 component 的晚设置可以覆盖早先 test-level 设置。

注释见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_config_db.svh:120-149`。

### exists / wait_modified / tracing / typedef

- `exists(cntxt, inst_name, field_name, spell_chk)` 用相同拼接规则后调用 `uvm_resource_db#(T)::get_by_name(...) != null`；可开 spell check，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_config_db.svh:235-259`。
- `wait_modified(cntxt, inst_name, field_name)` 创建 waiter 后阻塞在 event；set 时若 scope 匹配则触发，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_config_db.svh:262-303`。
- typedef：`uvm_config_int`、`uvm_config_string`、`uvm_config_object`、`uvm_config_wrapper`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_config_db.svh:307-340`。
- `+UVM_CONFIG_DB_TRACE` 或 options class 开启 tracing，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_config_db.svh:342-409`。

## 8. 与 `uvm_component` build/apply_config_settings 的关系

`uvm_component::build_phase()` 默认调用 backward-compatible `build()`；`build()` 调 `apply_config_settings(print_config_matches)`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_component.svh:2312-2325`。

`apply_config_settings()` 的文档说明：它会搜索所有匹配当前 component instance path 的 config settings，然后对每个匹配调用相应的 `set_*_local`。如果字段用 `` `uvm_field_*`` 注册，field macro 会提供 `set_*_local` 能力；否则用户需要 override `set_int_local/set_string_local/set_object_local` 等，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_component.svh:939-966`。

实现细节：

1. 调 `__m_uvm_field_automation(null, UVM_CHECK_FIELDS, "")` 填充 field_array。
2. 若 field_array 空则返回。
3. `rq = rp.lookup_scope(get_full_name())` 找当前 component 可见的所有 resource。
4. `rp.sort_by_precedence(rq)` 排序。
5. 反向遍历排序后的 queue，以便较高优先级最后 apply，从而覆盖较低优先级。
6. 对 resource name 做 `[]` 和 `.` 截断，用 base field name 查 field_array。
7. 依次尝试 cast 成 `uvm_resource#(uvm_integral_t)`、`uvm_bitstream_t`、`int`、`int unsigned`、`uvm_active_passive_enum`、`string`、`uvm_config_object_wrapper`、`uvm_object`，并调用对应 `set_*_local`。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_component.svh:3151-3262`。

这解释了 config_db 的两种使用模式：

- 显式 get：用户在 build_phase 中 `uvm_config_db#(T)::get(this,"","field",value)`。
- 自动 apply：通过 field automation 或 override `set_*_local`，由 `super.build_phase()` 自动把资源写入成员。

## 9. deprecated `set_config_* / get_config_*` 与 object clone 语义

UVM 1.2 仍保留 deprecated component API，但实现已转到 config_db：

- `set_config_int()` → `uvm_config_int::set(this, inst_name, field_name, value)`。
- `set_config_string()` → `uvm_config_string::set(...)`。
- `get_config_int/string()` → 对应 `uvm_config_*::get(this,"",field_name,value)`。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_component.svh:3006-3132`。

`set_config_object()` 特殊：

- 默认 `clone=1`，set 时先尝试 `value.clone()`。
- 若 clone 失败且 value 是 component，则 error，因为 component 不能 clone。
- 写入 `uvm_config_object::set(...)`。
- 同时创建 `uvm_config_object_wrapper`，保存 `obj` 与 `clone` bit，再通过 `uvm_config_db#(uvm_config_object_wrapper)::set(...)` 存入，用于自动 `apply_config_settings()` 时保留 clone 语义。

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_component.svh:2998-3081`。

注意源码注释特别说明 deprecated `get_config_object()` “does not honor the set_config_object clone bit”；它自己根据 get 端的 `clone` 参数决定是否 clone，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_component.svh:3110-3131`。

## 10. root build_phase 命令行入口

`uvm_root::build_phase()` 在 `super.build_phase(phase)` 后处理命令行：verbosity、timeout、factory、config、max quit、dump args，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_root.svh:637-653`。

Config 命令行入口：

- `+uvm_set_config_int=<component>,<field>,<value>`
- `+uvm_set_config_string=<component>,<field>,<value>`
- `+uvm_set_default_sequence=<sequencer>,<phase>,<type>`

`m_do_config_settings()` 抓取这些 plusargs 并调用 `m_process_config()` / `m_process_default_sequence()`，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_root.svh:897-915`。

`m_process_config()` 会解析逗号分隔的三段，int 支持 `'b`、`0b`、`'o`、`'d`、`'h`、`'x`、`0x` 等前缀，最后通过 `uvm_config_int::set(m_uvm_top, split_val[0], split_val[1], v)` 或 `uvm_config_string::set(...)` 写入，见 `/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_root.svh:793-851`。

`m_process_default_sequence()` 通过 factory 找 sequence type wrapper，然后写入：

```systemverilog
uvm_config_db#(uvm_object_wrapper)::set(this, {split_val[0], ".", split_val[1]}, "default_sequence", w);
```

源码：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/base/uvm_root.svh:853-894`。

与 [[knowledge_uvm_5_component_phasing]] 的关联：这些命令行 config 在 root 的 build_phase 中处理，普通 component 的 build/config get 也发生在 build top-down 阶段；因此 config 是否“来得及”取决于 set/get 所在的 build 层级与调用顺序。

## 一条 config_db 消息的完整路径

典型用法：

```systemverilog
uvm_config_db#(virtual if_t)::set(this, "env.agent*", "vif", vif);
uvm_config_db#(virtual if_t)::get(this, "", "vif", vif);
```

源码路径：

1. `set(cntxt, inst_name, field_name, value)` 把 context-relative path 转为 full scope。
2. 创建/复用 `uvm_resource#(T)(field_name, inst_name)`。
3. 根据 build/runtime 设 precedence。
4. `write()` typed value，放入 resource pool queue 头部或提升 name priority。
5. `get()` 构造当前 scope，按 `field_name` 查 name map，同时要求 type handle 为 `uvm_resource#(T)::get_type()`。
6. scope 正则匹配后，选最高 precedence resource。
7. `read(cntxt)` 返回 typed value 并记录 audit。

简图：

```text
uvm_config_db#(T)::set/get
        ↓ maps cntxt+inst_name, field_name
uvm_resource_db#(T) convenience layer
        ↓ creates/reads typed resource
uvm_resource#(T)
        ↓ stored in
uvm_resource_pool: rtab[name] + ttab[type_handle]
        ↓ lookup by field_name + scope regex + type_handle + precedence
T value
```

## 常见误区

1. **以为 config_db 是独立数据库。** `uvm_config_db#(T)` 继承 `uvm_resource_db#(T)`，底层就是 `uvm_resource_pool`；config setting 也是 resource。
2. **忽略 type 参数。** set/get 的 `T` 必须一致。`uvm_config_db#(int)::set` 与 `uvm_config_db#(uvm_bitstream_t)::get` 不是同一个 type handle，即使 field_name/scope 相同也可能查不到。
3. **误解 `inst_name`。** config_db 会把 `cntxt.get_full_name()` 与 `inst_name` 拼成完整 scope；`inst_name==""` 表示当前 context 自己。`cntxt==null` 表示从 root/global scope 开始。
4. **误解 build-time precedence。** build 期间高层 set 的 precedence 更高；build 后所有 set 使用 default precedence，运行期 late set 更容易覆盖早期设置。
5. **以为 wildcard 匹配 field_name 与 inst_name 同等方式。** resource 的 name table key 是 field_name；config get 最终通过 `lookup_regex_names(inst_name, field_name, type)` 查 exact name key，再用 resource scope 正则匹配 inst_name。field_name 可以是 glob/regex 写入，但底层 name map lookup 对具体 key 的行为要结合 resource name 存储理解。
6. **自动 apply_config_settings 不是魔法。** 它只处理 field macro 注册字段，或用户实现的 `set_*_local` 能识别字段；否则显式 `get()` 更直接。
7. **`write_by_name` 不会找不到就创建。** resource_db 注释虽说“write a val into database”，实现找不到直接返回 0；创建请用 `set()`。
8. **read-only resource 的相同值写入不会触发 modified。** `write()` 在 read-only 检查后，如果 `val == t` 就直接返回，不记录 write audit、不唤醒 `wait_modified()`。
9. **`wait_modified` 是 field-name 分桶，set 时再按 scope 匹配触发。** 不要把它理解成对某一个 resource handle 的 `wait_modified()`；config_db wait_modified 和 resource_base wait_modified 是两套机制。
10. **deprecated set_config_object clone bit 与 get_config_object 行为不完全一致。** 自动 apply 通过 wrapper 保存 clone bit；deprecated `get_config_object()` 注释明确不 honor set 端 clone bit，而由 get 端 clone 参数决定。

## 后续学习建议

下一主题建议学习 TLM1：`src/tlm1/uvm_tlm.svh` 及 ports/exports/imps/fifo/analysis port。原因：config/resource 已解释了 virtual interface、knob、default_sequence 等配置入口；下一步学习 TLM 能把 component 之间的数据通信、monitor → scoreboard 的 analysis 通道、driver/sequencer 交互逐步串起来。
