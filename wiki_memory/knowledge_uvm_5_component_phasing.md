---
name: knowledge_uvm_5_component_phasing
description: UVM 1.2 uvm_component、uvm_root、uvm_coreservice、phasing 架构与 objection 机制
metadata:
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T06:18:47.443Z
---

# UVM 1.2 Component / Phasing / Objection 源码学习

## 知识点摘要

这一批学习把 UVM 从“对象模型”推进到“验证环境如何被组织并跑起来”。关键结论是：

- `uvm_component` 是 hierarchy 的基本单元，也是 phasing、reporting、config、recording 和 transaction 入口的载体。
- `uvm_root` 是隐式顶层与 phase controller，`uvm_top` 是其全局句柄。
- `uvm_coreservice_t` 把 factory、report server、default tr database、root、component visitor 集中到一个可替换入口里。
- `uvm_phase` / `uvm_domain` 定义了 phase 图、schedule、domain、jump/sync 关系。
- `uvm_topdown_phase` / `uvm_bottomup_phase` / `uvm_task_phase` 把 phase 执行模型分成 top-down、bottom-up、task fork 三类。
- `uvm_objection` 控制 task phase 生命周期，`run_phase` 何时结束由 objection / drain / timeout 决定。

## 这批学习的源码范围

- `src/base/uvm_component.svh`
- `src/base/uvm_root.svh`
- `src/base/uvm_coreservice.svh`
- `src/base/uvm_phase.svh`
- `src/base/uvm_domain.svh`
- `src/base/uvm_topdown_phase.svh`
- `src/base/uvm_bottomup_phase.svh`
- `src/base/uvm_task_phase.svh`
- `src/base/uvm_common_phases.svh`
- `src/base/uvm_runtime_phases.svh`
- `src/base/uvm_objection.svh`
- 回看 `src/base/uvm_base.svh` 的 include 顺序。

## `uvm_base.svh` 的 include 顺序意义

`src/base/uvm_base.svh:29-109` 体现了依赖顺序：

1. `uvm_coreservice.svh`
2. object/global/misc
3. `uvm_object.svh`
4. factory / registry
5. resource / config
6. policy classes
7. transaction recording
8. report classes
9. phase classes
10. `uvm_component.svh`
11. objection / heartbeat
12. globals / cmdline / traversal

这说明 component/phasing 不是独立孤岛，而是建立在前面所有基础设施之上。

## `uvm_coreservice_t`

源码位置：`src/base/uvm_coreservice.svh:37-220`

### 角色

`uvm_coreservice_t` 是 central service hub，负责：

- `get_factory()` / `set_factory()`
- `get_report_server()` / `set_report_server()`
- `get_default_tr_database()` / `set_default_tr_database()`
- `get_component_visitor()` / `set_component_visitor()`
- `get_root()`

`uvm_default_coreservice_t` 提供默认实现。

### 关键点

- `uvm_coreservice_t::get()` 返回 singleton service instance。
- `uvm_default_coreservice_t::get_factory()` 首次调用时创建 `uvm_default_factory`。
- `set_factory()` 允许替换 factory，但源码明确说明用户需要自己保留旧 factory 状态或 delegate。
- `get_default_tr_database()` 默认返回 `uvm_text_tr_database`。
- `get_component_visitor()` 默认返回 `uvm_component_name_check_visitor`。
- `get_root()` 返回 `uvm_root::m_uvm_get_root()`。

## `uvm_root`

源码位置：`src/base/uvm_root.svh:26-655`

### 角色

`uvm_root` 是：

- 隐式顶层 component。
- 全局 phase controller。
- component 搜索入口。
- 全局 report 配置入口。
- `uvm_top` 的实现类型。

### 关键接口

- `uvm_root::get()`：通过 core service 取 root。
- `run_test(string test_name="")`：创建 `uvm_test_top`，然后调用 `uvm_phase::m_run_phases()`。
- `find()` / `find_all()`：按路径或 wildcard 搜索 component。
- `print_topology()`：打印组件树。
- `set_timeout()`：设置全局 phase timeout。

### `run_test()` 的流程

`uvm_root::run_test` 的关键链路：

1. 调 `uvm_objection::m_init_objections()`。
2. 解析 `+UVM_TESTNAME=` 或参数 `test_name`。
3. 用 factory 创建 `uvm_test_top`。
4. 检查是否至少存在一个 component。
5. 打印 running test 信息。
6. fork 一个 phase runner：`uvm_phase::m_run_phases()`。
7. 等待 `m_phase_all_done`。
8. 关闭 runner，report summarize，必要时 `$finish`。

### `build_phase()` 中的命令行设置

`uvm_root::build_phase()` 会按顺序应用：

- verbosity
- timeout
- factory overrides
- config settings
- max_quit settings
- dump args

这让 root 成为“命令行参数落地”的集中入口。

## `uvm_component`

源码位置：`src/base/uvm_component.svh:29-3634`

### 角色

`uvm_component` 是 UVM 层次结构的基本节点，提供：

- hierarchy
- phasing
- reporting
- transaction recording
- factory convenience
- config application

它继承自 `uvm_report_object`，因此同时具备 report handler 能力。

### 构造与层次

`uvm_component::new(name, parent)` 的关键行为：

- `parent == null` 时默认挂到 `uvm_top`。
- 检查 build phase 是否已经结束，若是则 fatal，禁止在 build 后创建 component。
- 若名字为空，自动生成 `COMP_<id>`。
- 检查同名 child 冲突。
- `m_parent = parent`，`set_name(name)`，再 `m_add_child(this)`。
- 继承 parent 的 domain。
- 构造完成后 reseed。
- 从 `uvm_config_db` 读取 `recording_detail`。
- 把 report handler 名称设为 full name。
- 继承 parent 的 report verbosity。

### hierarchy API

重要方法：

- `get_parent()`
- `get_full_name()`：缓存式 full name 构造。
- `get_children()` / `get_first_child()` / `get_next_child()` / `get_child()` / `has_child()` / `get_num_children()`
- `lookup()`：支持相对/绝对路径查找。
- `set_name()`：component 名称创建后不可更改；会触发 full name 更新。

### phase hooks

`uvm_component` 提供一整套 phase hook：

- common phases：`build_phase`、`connect_phase`、`end_of_elaboration_phase`、`start_of_simulation_phase`、`run_phase`、`extract_phase`、`check_phase`、`report_phase`、`final_phase`
- runtime phases：`pre_reset_phase` 到 `post_shutdown_phase`
- extra callbacks：`phase_started`、`phase_ended`、`phase_ready_to_end`

旧风格方法如 `build()`、`run()` 仍存在，仅作为 backward compatibility。

### `build_phase()` 与 config

`uvm_component::build_phase()` 默认：

- 标记 `m_build_done`。
- 调 `build()`，而 `build()` 会执行 `apply_config_settings(print_config_matches)`。

这说明自动配置的入口其实挂在 build 期间。

### `apply_config_settings()`

源码位置：`src/base/uvm_component.svh:3154-3262`

它会：

1. 用 `__m_uvm_field_automation(null, UVM_CHECK_FIELDS, "")` 收集 field macros 声明字段。
2. 从 `uvm_resource_pool` 查找当前 scope 的资源。
3. 按 precedence 逆序应用资源。
4. 针对不同资源类型调用 `set_int_local()` / `set_string_local()` / `set_object_local()`。

这是旧 config API 与 field macro 的结合点。

### domain / phase mapping

重要方法：

- `define_domain(uvm_domain domain)`：为 domain 补 `uvm_sched`，并把 domain 加进 common domain 图。
- `set_domain(domain, hier=1)`：给 component 树设置 domain。
- `get_domain()`：返回当前 domain。
- `set_phase_imp(phase, imp, hier=1)`：为某 phase 设置实现替身。

### runtime control

- `kill()` / `do_kill_all()`：停止 phase process。
- `suspend()` / `resume()`：默认未实现，仅 warning。
- `stop()` / `stop_phase()`：兼容接口。

### transaction recording

`uvm_component` 也是 transaction recording 的入口：

- `accept_tr()`
- `begin_tr()` / `begin_child_tr()`
- `end_tr()`
- `get_tr_stream()` / `free_tr_stream()`
- `record_error_tr()` / `record_event_tr()`

这与 `uvm_recorder` / `uvm_tr_database` 配套。

### reporting hierarchy

`set_report_*_hier()` 会递归设置子树；`set_report_verbosity_level_hier()` 特别常用。

## `uvm_phase` / `uvm_domain`

### `uvm_phase`

源码位置：`src/base/uvm_phase.svh:785-2233`

`uvm_phase` 是 phase 图节点，拥有：

- `m_phase_type`：DOMAIN / SCHEDULE / NODE / IMP / TERMINAL
- 前驱 / 后继关系
- `find()` / `find_by_name()` / `is_before()` / `is_after()`
- `sync()` / `unsync()`
- `jump()` / `jump_all()`
- `raise_objection()` / `drop_objection()` / `get_objection_count()`
- `execute_phase()`
- `m_run_phases()`

### `uvm_phase::new()`

- 若是 common domain 的根，会初始化为 dormant。
- 读取 `+UVM_PHASE_TRACE`。
- 读取 `+UVM_USE_OVM_RUN_SEMANTIC`。
- schedule/domain 根节点会自动创建 `_end` terminal node。

### `add()`

`uvm_phase::add()` 支持：

- 插入单个 IMP 节点
- 插入一个 schedule/domain 子图
- `with_phase` 并行插入
- `before_phase` / `after_phase` 插入

插入后会把新节点状态设为 `UVM_PHASE_DORMANT`。

### `execute_phase()`

这是 phase 执行核心：

- 先等待前驱 DONE。
- 进入 `SYNCING`，处理 sync 关系。
- 进入 `STARTED` / `EXECUTING` / `READY_TO_END` / `ENDED` / `CLEANUP` / `DONE`。
- 对 task phase：fork component task，并在 objections、ready_to_end、timeout、jump 之间竞争结束条件。
- 对 function phase：直接同步遍历。

### `m_run_phases()`

`m_run_phases()` 是 top-level phase runner：

1. 从 common domain 开始。
2. 把起始 phase 放入 `m_phase_hopper`。
3. 循环取 phase，fork `execute_phase()`。

这就是 `run_test()` 启动后真正驱动 phase 图的地方。

### `uvm_domain`

源码位置：`src/base/uvm_domain.svh:56-218`

`uvm_domain` 是 phase schedule 的 domain node。

#### built-in domains

- `get_common_domain()`：build/connect/eoe/sos/run/extract/check/report/final
- `get_uvm_domain()`：runtime phases schedule

`get_common_domain()` 会把 common phases 挂到 common domain，并把 `uvm_run_phase` 与 runtime schedule 通过 `with_phase` 方式接起来。

#### 关键 API

- `get_domains()`：列出所有 domain。
- `add_uvm_phases(schedule)`：把 runtime phases 加进 schedule。
- `jump(phase)`：跳转当前 domain 中所有 active phase。
- `jump_all(phase)`：跳所有 domain。

## `uvm_topdown_phase` / `uvm_bottomup_phase` / `uvm_task_phase`

### `uvm_topdown_phase`

源码位置：`src/base/uvm_topdown_phase.svh:24-113`

- top-down function phase。
- `traverse()` 先处理当前节点，再递归 children。
- `EXECUTING` 状态下调用 `exec_func(comp, phase)`。
- `STARTED` / `READY_TO_END` / `ENDED` 时调用 component 对应 callback。
- 对 `build` phase，若 `comp.m_build_done` 已经置位，则跳过重复 build。

### `uvm_bottomup_phase`

源码位置：`src/base/uvm_bottomup_phase.svh:24-110`

- bottom-up function phase。
- `traverse()` 先递归 children，再处理当前节点。
- `EXECUTING` 时调用 `exec_func(comp, phase)`。
- `execute()` 会对当前 process 重新 seeding。

### `uvm_task_phase`

源码位置：`src/base/uvm_task_phase.svh:25-159`

- task phase 的核心区别是：`execute()` 会 fork 每个 component 的 `exec_task()`。
- `m_num_procs_not_yet_returned` 记录仍未返回的 task 进程数。
- `traverse()` 内部通过 `m_traverse()` 执行，并在 `STARTED` 时启动 sequencer 的 phase sequence。
- `ENDED` 时调用 `seqr.stop_phase_sequence()`。

## common phases / runtime phases

### common phases

源码位置：`src/base/uvm_common_phases.svh`

这些 phase 同步执行：

- `uvm_build_phase`：top-down，调用 `build_phase()`。
- `uvm_connect_phase`：bottom-up，调用 `connect_phase()`。
- `uvm_end_of_elaboration_phase`：bottom-up。
- `uvm_start_of_simulation_phase`：bottom-up。
- `uvm_run_phase`：task phase，跨 runtime phases 并行。
- `uvm_extract_phase` / `uvm_check_phase` / `uvm_report_phase` / `uvm_final_phase`。

### runtime phases

源码位置：`src/base/uvm_runtime_phases.svh`

runtime phases 包括：

- `pre_reset` / `reset` / `post_reset`
- `pre_configure` / `configure` / `post_configure`
- `pre_main` / `main` / `post_main`
- `pre_shutdown` / `shutdown` / `post_shutdown`

它们都是 task phase，默认由 `uvm_domain::get_uvm_domain()` 组织成 schedule。

## `uvm_objection`

源码位置：`src/base/uvm_objection.svh:62-1385`

### 角色

`uvm_objection` 是 phase 生命周期控制的核心。它决定 task phase 何时结束。

### 关键状态

- `m_trace_mode`
- `m_top_all_dropped`
- `m_total_count` / `m_source_count`
- `m_scheduled_list`
- `m_forked_list`
- `m_scheduled_contexts`
- `m_forked_contexts`
- `m_context_pool`
- `m_drain_proc`
- `m_cleared`

### 关键 API

- `trace_mode()`：开关 trace。
- `raise_objection(obj, description, count)`
- `drop_objection(obj, description, count)`
- `clear(obj)`
- `set_drain_time(obj, drain)`
- `wait_for(event, obj)`
- `wait_for_total_count(obj, count)`
- `get_objection_count()` / `get_objection_total()`
- `display_objections()`

### raise / drop 语义

#### raise

- 递增 source count 和 total count。
- 调用 `raised()` callback。
- 若之前有 drain 过程，尝试取消 scheduled/forked drain。
- 根据 propagation mode 决定是否向父级传播。

#### drop

- 递减 source count 和 total count。
- 调用 `dropped()` callback。
- 若 total 仍非 0，则继续向上传播。
- 若 total 变成 0，则把对象放入 `m_scheduled_list`，由后台进程处理 drain / all_dropped。

### drain/all_dropped

- `m_execute_scheduled_forks()` 是后台任务，负责把 scheduled context fork 出去。
- `m_forked_drain()` 负责 drain-time 等待和 `all_dropped` callback。
- 如果 drain 期间又有新的 raise，相关 fork 会被杀掉，旧的 drop 链被中止。

### `m_init_objections()`

`uvm_root::run_test()` 一开始调用它，目的是启动 objection 后台调度线程。

### `uvm_test_done_objection`

这是历史兼容类，用于 old stop request 语义，但现在更多是 `run_phase` 的 `phase_done` 机制。

## 整体执行链路

从 testbench 角度，真正的启动链路是：

1. 用户创建 top-level component。
2. `uvm_root::run_test()` 解析 test 名称并用 factory 创建 `uvm_test_top`。
3. `uvm_phase::m_run_phases()` 进入 common domain。
4. `build/connect/eoe/sos` 按 top-down/bottom-up 遍历 component tree。
5. `run_phase` 与 runtime phases 由 task phase + objection 控制结束。
6. `report_phase` / `final_phase` 做收尾。

## 关键源码约束与坑

- component 不能在 build 之后再创建；否则 `ILLCRT` fatal。
- component 名称创建后不可随意改；`set_name()` 会报错。
- `run_phase` 结束不等于 task 退出，真正结束由 objection / timeout 决定。
- `phase_started()` / `phase_ended()` / `phase_ready_to_end()` 是额外钩子，适合做跨 phase 的通用操作。
- custom domain 要通过 `define_domain()` / `set_domain()` 把 schedule 接入 common domain。
- `uvm_root::find()` / `find_all()` 依赖完整路径与 wildcard 匹配。
- `uvm_topdown_phase` 与 `uvm_bottomup_phase` 的遍历顺序不同，build 是 top-down，connect 和 eoe/sos 是 bottom-up。

## 与前后主题的关系

### 依赖前面

- `uvm_object` 提供命名、factory、clone、compare、print、pack、record 的基础。
- policy classes 支持 print/compare/pack/record。
- factory/registry 让 `run_test()` 能通过类型名创建 `uvm_test_top`。

### 为后面铺路

下一批最自然是：

- reporting：`uvm_report_object` / handler / server / message / catcher
- config/resource：`uvm_config_db` / `uvm_resource_db` / `uvm_resource`
- 再往后 TLM1 / components / sequences

## 原文位置

- `src/base/uvm_component.svh`
- `src/base/uvm_root.svh`
- `src/base/uvm_coreservice.svh`
- `src/base/uvm_phase.svh`
- `src/base/uvm_domain.svh`
- `src/base/uvm_topdown_phase.svh`
- `src/base/uvm_bottomup_phase.svh`
- `src/base/uvm_task_phase.svh`
- `src/base/uvm_common_phases.svh`
- `src/base/uvm_runtime_phases.svh`
- `src/base/uvm_objection.svh`

## 关联知识

- [[knowledge_uvm_index]]
- [[knowledge_uvm_1_source_architecture]]
- [[knowledge_uvm_2_object_model]]
- [[knowledge_uvm_3_factory_registry]]
- [[knowledge_uvm_4_policy_classes]]
- [[knowledge_uvm_6_reporting_config_resource]]
