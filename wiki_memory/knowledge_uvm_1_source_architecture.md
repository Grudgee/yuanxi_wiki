---
name: knowledge_uvm_1_source_architecture
description: UVM 1.2 源码目录结构、顶层入口、include 组织和深度学习路线
metadata: 
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-22T03:33:52.813Z
---

# UVM 1.2 源码架构与入口

## 知识点摘要

UVM 1.2 是 Accellera 的 Universal Verification Methodology reference implementation。源码主体是 SystemVerilog class library，顶层编译入口是 `src/uvm.sv`，实际 package 定义在 `src/uvm_pkg.sv`。宏系统由 `src/uvm_macros.svh` 统一入口。

本目录不应作为一个整体一次性加载。适合按 include 依赖和功能子系统分批学习：先入口和 base，再 factory/phasing/reporting/config，再 TLM/components/sequences，最后 register model、DPI 和 examples。

## 顶层资料

- `README.txt`：安装、编译和使用说明。
- `release-notes.txt`：UVM 1.2 release notes，列出 1.2 相对早期版本的重要变更和 Mantis issue。
- `LICENSE.txt`：Apache-2.0 license。
- `NOTICE.txt`：版权/通知。
- `UVM_Reference.html` 与 `docs/html/`：HTML reference 文档入口。
- `examples/`：simple/integrated 示例和不同 simulator 的 Makefile。
- `bin/`：辅助脚本，包括 UVM 版本迁移脚本和环境识别脚本。

## 使用和编译入口

`README.txt` 的关键使用流程：

1. 设置 `UVM_HOME` 为 UVM kit 的绝对路径。
2. 编译 `$UVM_HOME/src/uvm.sv`。
3. 在用户 SystemVerilog 代码中 `import uvm_pkg::*;`。
4. 若使用 UVM shorthand macros，需要 include：
   ```systemverilog
   `include "uvm_macros.svh"
   ```
5. 需要编译 `src/dpi/uvm_dpi.cc` 或使用 simulator 提供的 UVM DPI shared library；具体方式依赖仿真器。

## Package/include 结构

### `src/uvm.sv`

源码内容极简：

```systemverilog
`include "uvm_pkg.sv"
```

因此 `uvm.sv` 是编译入口，不承载实际类定义。

### `src/uvm_pkg.sv`

`uvm_pkg.sv` 先 include `uvm_macros.svh`，然后定义 `package uvm_pkg;`，并在 package 内按顺序 include 各子系统：

1. `dpi/uvm_dpi.svh`
2. `base/uvm_base.svh`
3. `dap/uvm_dap.svh`
4. `tlm1/uvm_tlm.svh`
5. `comps/uvm_comps.svh`
6. `seq/uvm_seq.svh`
7. `tlm2/uvm_tlm2.svh`
8. `reg/uvm_reg_model.svh`

这个顺序体现了依赖方向：base 是核心；DAP、TLM、components、sequences、TLM2、register model 在后面逐步叠加。

### `src/uvm_macros.svh`

`uvm_macros.svh` 是宏总入口，负责：

- vendor-specific define 兼容，例如 `MODEL_TECH` 到 `QUESTA`。
- string queue pack、`uvm_typename` 等工具宏。
- deprecated feature control。
- include 各宏文件：
  - `macros/uvm_version_defines.svh`
  - `macros/uvm_global_defines.svh`
  - `macros/uvm_message_defines.svh`
  - `macros/uvm_phase_defines.svh`
  - `macros/uvm_object_defines.svh`
  - `macros/uvm_printer_defines.svh`
  - `macros/uvm_tlm_defines.svh`
  - `macros/uvm_sequence_defines.svh`
  - `macros/uvm_callback_defines.svh`
  - `macros/uvm_reg_defines.svh`
  - `macros/uvm_deprecated_defines.svh`

## 源码目录分组

### `src/base/`

核心基础设施，包含：

- object/component/root：`uvm_object.svh`、`uvm_component.svh`、`uvm_root.svh`。
- factory/registry/core service：`uvm_factory.svh`、`uvm_registry.svh`、`uvm_coreservice.svh`。
- phasing：`uvm_phase.svh`、`uvm_domain.svh`、common/runtime/topdown/bottomup/task phases。
- objection/heartbeat/barrier/event：同步和 phase 控制。
- reporting：report object、handler、server、message、catcher。
- config/resource DB：`uvm_config_db.svh`、`uvm_resource_db.svh`、`uvm_resource.svh`。
- print/compare/pack/record/tr database：对象操作与 transaction recording。
- containers/utilities：pool、queue、misc、globals、links、traversal、spell checker。

### `src/macros/`

UVM 宏定义，覆盖对象注册、field automation、reporting、phase callbacks、TLM 端口、sequence action、callback、register model 等。

### `src/dap/`

Data Access Policy 相关类，用于控制 set/get、lock、set-before-get 等访问策略。

### `src/tlm1/`

TLM1 基础：ports、exports、imps、interfaces、fifo、analysis port、req/rsp、sequencer connection。

### `src/tlm2/`

TLM2 基础：generic payload、time、interfaces、ports/exports/imps、socket base/socket。

### `src/comps/`

常用验证组件基类：agent、driver、push_driver、monitor、env、scoreboard、subscriber、test、comparator、random stimulus 等。

### `src/seq/`

sequence/sequencer 体系：sequence_item、sequence_base、sequence、sequencer_base、param sequencer、sequencer、push sequencer、builtin sequence、sequence library、analysis FIFO。

### `src/reg/`

UVM register abstraction layer：register、field、block、map、file、mem、vreg、adapter、predictor、backdoor、sequences、callbacks、MAM 等。

### `src/dpi/`

DPI C/C++ 和 SV 声明，提供 command line、regex、HDL backdoor access 等底层能力。不同 simulator 有专门文件，如 `uvm_hdl_inca.c`、`uvm_hdl_questa.c`、`uvm_hdl_vcs.c`。

### `src/deprecated/`

兼容旧版本的废弃内容，例如 resource converter。

## UVM 1.2 release notes 初步重点

从 release-notes 可见，UVM 1.2 是 reference implementation，包含 SystemVerilog class library、examples、标准参考文档和用户指南。重要变更包括：

- `uvm_event` 变为 parameterized class。
- 旧的 `set/get_config_int/string/object` deprecated。
- 增加 enum by string 配置支持。
- 增加 `+uvm_set_default_sequence` 命令行功能。
- messaging 全部经由 `uvm_report_server`。
- factory 可替换，用于 trace create/override 和识别未使用 overrides；代码应通过 `uvm_factory::get` 获取 factory。
- 支持 undo factory override。
- 增加 objection propagation mode。
- `uvm_sequence::starting_phase` 访问变更，用户应使用 `get_starting_phase` / `set_starting_phase`。

这些变更提示后续学习要特别关注 factory service、report server、objection、sequence starting phase、config DB 迁移。

## 后续深度学习路线

为“吃透源码”，后续每批应围绕一个类族展开，记录：

- 文件和类列表。
- 继承关系。
- 关键字段。
- 构造/create 流程。
- 核心方法调用链。
- 与其它子系统交互。
- 用户代码常见调用入口。
- 源码中的边界条件和隐含约束。

建议下一批从 `src/base/uvm_object.svh`、`src/base/uvm_object_globals.svh`、`src/base/uvm_misc.svh` 和 `macros/uvm_object_defines.svh` 开始，建立 UVM object 模型和 field automation 的基础。

## 原文位置

- `README.txt`：安装、编译、import package、include macros、DPI 编译说明。
- `release-notes.txt`：UVM 1.2 release notes 和 Mantis 变更列表。
- `src/uvm.sv`：顶层 include 入口。
- `src/uvm_pkg.sv`：package 与子系统 include 顺序。
- `src/uvm_macros.svh`：宏入口和宏 include 顺序。

## 关联知识

- [[knowledge_uvm_index]]
