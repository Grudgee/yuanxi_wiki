---
name: knowledge_uvm_13_dpi
description: UVM 1.2 DPI 层、命令行参数、regex/glob、HDL backdoor 与 vendor backend 学习记忆
metadata:
  type: reference
  node_type: memory
  originSessionId: manual-2026-07-23
  modified: 2026-07-23T06:27:46.320Z
---

# UVM 1.2 DPI 层：cmdline / regex / HDL backdoor

## 源码范围

- SystemVerilog DPI 入口：`/home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2/src/dpi/uvm_dpi.svh`
- HDL backdoor SV 声明：`src/dpi/uvm_hdl.svh`
- regex/glob SV 声明：`src/dpi/uvm_regex.svh`
- 命令行 DPI SV 声明：`src/dpi/uvm_svcmd_dpi.svh`
- 命令行处理 class：`src/base/uvm_cmdline_processor.svh`
- C/C++ 汇总入口：`src/dpi/uvm_dpi.cc`
- C/C++ 公共头：`src/dpi/uvm_dpi.h`
- DPI report glue：`src/dpi/uvm_common.c`
- regex/glob C++：`src/dpi/uvm_regex.cc`
- 命令行/VPI 参数 C：`src/dpi/uvm_svcmd_dpi.c`
- HDL vendor 分发：`src/dpi/uvm_hdl.c`
- HDL vendor backend：`uvm_hdl_questa.c`、`uvm_hdl_vcs.c`、`uvm_hdl_inca.c`

## 入口与编译开关

`uvm_pkg.sv` 在 base 子系统之前 include `dpi/uvm_dpi.svh`，所以 DPI 提供的函数对后续 base/config/report/reg 等子系统可见。`uvm_dpi.svh` 是 SV 侧 DPI 总入口：

- `UVM_NO_DPI` 会同时定义 `UVM_HDL_NO_DPI`、`UVM_REGEX_NO_DPI`、`UVM_CMDLINE_NO_DPI`。
- 然后 include：
  1. `dpi/uvm_hdl.svh`
  2. `dpi/uvm_svcmd_dpi.svh`
  3. `dpi/uvm_regex.svh`

含义：UVM 把 DPI 分成三块能力：HDL backdoor、命令行参数、regex/glob。用户可整体关闭 DPI，也可按子能力关闭。

C/C++ 侧的总入口是 `uvm_dpi.cc`，它在 `extern "C"` 中 include：

1. `uvm_common.c`
2. `uvm_regex.cc`
3. `uvm_hdl.c`
4. `uvm_svcmd_dpi.c`

所以某些 simulator makefile 可以只编译 `uvm_dpi.cc`，也可以分别编译这些 C/C++ 文件后链接。

## DPI report glue

`uvm_common.c` 是 C/C++ DPI 层向 SV reporting 系统回报错误的桥：

- 声明外部 SV 函数 `m__uvm_report_dpi(...)`。
- `m_uvm_report_dpi(...)` 先用 `svSetScope(svGetScopeFromName(...))` 切到 `uvm_pkg` scope，再调用 `m__uvm_report_dpi`，最后恢复旧 scope。
- Cadence/INCA 使用 package scope 字符串 `uvm_pkg::`，其他后端使用 `uvm_pkg`。

意义：DPI C 层不能直接调用普通 SV report API；它通过一个 wrapper 进入正确 SV scope，再统一接入 UVM report server/handler 链路。regex 编译错误、HDL path 找不到、宽度超限等都经由这里报 `UVM/DPI/...` ID。

## 命令行参数：`uvm_cmdline_processor` 与 `uvm_svcmd_dpi`

### SV 侧 class

`uvm_cmdline_processor` 继承 `uvm_report_object`，并以 singleton 使用：

- `static local uvm_cmdline_processor m_inst`
- `get_inst()` 懒创建 `new("uvm_cmdline_proc")`
- 文件底部创建常量句柄：`const uvm_cmdline_processor uvm_cmdline_proc = uvm_cmdline_processor::get_inst();`

构造函数中循环调用 `uvm_dpi_get_next_arg(doInit)`：

1. 第一次 `doInit=1` 触发 C 侧初始化。
2. 每次返回一个参数字符串，直到空字符串结束。
3. 填充：
   - `m_argv`：全部命令行参数。
   - `m_plus_argv`：以 `+` 开头的 plusargs。
   - `m_uvm_argv`：以 `+` 或 `-` 开头，且后面前三个字符大小写无关为 `UVM` 的参数。

常用 API：

- `get_args(output string args[$])`
- `get_plusargs(output string args[$])`
- `get_uvm_args(output string args[$])`
- `get_arg_matches(string match, ref string args[$])`
- `get_arg_value(string match, ref string value)`
- `get_arg_values(string match, ref string values[$])`
- `get_tool_name()` / `get_tool_version()`

### `get_arg_matches` 的两种匹配语义

`get_arg_matches(match,args)` 的语义很关键：

- 如果 `match` 被 `/.../` 包住，按 extended regular expression 编译；底层调用 `uvm_dpi_regcomp` / `uvm_dpi_regexec` / `uvm_dpi_regfree`。
- 否则按“前缀匹配”：`m_argv[i].substr(0,len-1) == match`。

这与 `uvm_re_match()` 的 glob/regex 体系不同：`get_arg_matches` 对 `/.../` 直接使用 POSIX extended regex，对非 `/.../` 字符串只做前缀匹配。

### C 侧参数获取

`uvm_svcmd_dpi.c` 用 VPI `vpi_get_vlog_info(&info)` 获取 simulator 启动参数。

它有一个重要处理：递归展开 `-f` / `-F` 参数文件。实现分两遍：

1. `walk_level(..., cmd=0)` 统计参数数量，跳过 `-f/-F` 本身和文件名，并递归进入文件内容。
2. 分配 `argv_stack`。
3. `walk_level(..., cmd=1)` 填入扁平化后的参数指针。

`uvm_dpi_get_next_arg_c(init)` 用静态 `idx` 和 `argv_ptr` 逐个返回参数。`uvm_dpi_get_tool_name_c()` / `uvm_dpi_get_tool_version_c()` 直接返回 VPI info 中的 `product` / `version`。

易错点：

- 参数扫描发生在 `uvm_cmdline_processor` 构造时，后续不能动态添加命令行。
- `-f/-F` 展开依赖 simulator 提供的 `argv` 数据结构格式；这是 DPI/VPI 绑定行为。
- 若定义 `UVM_CMDLINE_NO_DPI`，`uvm_dpi_get_next_arg()` 返回空字符串，命令行 processor 将看不到真实参数；`get_tool_name/version` 返回 `?`。

## Regex / glob：两套接口

### `uvm_regex.svh`：UVM 通用匹配

默认 DPI 打开时导入：

- `uvm_re_match(string re, string str)`
- `uvm_dump_re_cache()`
- `uvm_glob_to_re(string glob)`

如果 `UVM_REGEX_NO_DPI` 打开，则使用纯 SV fallback：

- 只能做 glob 风格匹配，不支持完整 regex。
- `*` 匹配任意长度，`?` 匹配单字符。
- 返回值约定：匹配成功返回 `0`，失败返回 `1`。这与 POSIX `regexec` 一致，但与很多 SV API “1 表示 true” 的直觉相反。

### `uvm_regex.cc`：POSIX extended regex

`uvm_re_match(re,str)`：

- 对空指针返回失败 `1`。
- 最大 regex 长度 `UVM_REGEX_MAX_LENGTH=2048`。
- 如果输入被 `/.../` 包住，会去掉外层 `/` 后编译。
- 使用 `regcomp(..., REG_EXTENDED)` 与 `regexec`。
- 每次调用都会 malloc/compile/free；注释提到 cache，但实际 `uvm_dump_re_cache()` 明确报告 “cache not implemented”。

`uvm_glob_to_re(glob)`：

- 空 glob 或单独 `/` 转成空字符串。
- 若输入已经是 `/.../`，原样返回。
- 否则把 glob 转成 `/^...$/` 包裹的 regex：
  - `*` → `.*`
  - `+` → `.+`
  - `.` → `\.`
  - `?` → `.`
  - `[`、`]`、`(`、`)` 转义
- 如果 glob 开头已经是 `^`，不再补 `^`；如果末尾不是 `$`，补 `$`。

易错点：

- `uvm_re_match` / `regexec` 返回 0 才是匹配成功。
- `uvm_dump_re_cache()` 名字暗示有 cache，但 UVM 1.2 源码中 cache 未实现。
- fallback 纯 SV 版本不支持完整正则，只能 glob；关闭 DPI 后高级 regex 行为会退化。

## HDL backdoor SV API

`uvm_hdl.svh` 定义：

- 默认最大 backdoor vector 宽度宏 `UVM_HDL_MAX_WIDTH=1024`。
- 参数 `parameter int UVM_HDL_MAX_WIDTH = \`UVM_HDL_MAX_WIDTH;`，C 侧通过 `vpi_handle_by_name("uvm_pkg::UVM_HDL_MAX_WIDTH",0)` 查询。
- 类型 `typedef logic [UVM_HDL_MAX_WIDTH-1:0] uvm_hdl_data_t;`

DPI 打开时导入：

- `uvm_hdl_check_path(string path)`：路径存在返回 1，否则 0。
- `uvm_hdl_deposit(string path, uvm_hdl_data_t value)`：deposit/no-delay put。
- `uvm_hdl_force(string path, uvm_hdl_data_t value)`：force。
- `uvm_hdl_release_and_read(string path, inout uvm_hdl_data_t value)`：release 后读回。
- `uvm_hdl_release(string path)`：release。
- `uvm_hdl_read(string path, output uvm_hdl_data_t value)`：读。
- task `uvm_hdl_force_time(path,value,force_time)`：若 `force_time==0`，退化为 deposit；否则 force，等待指定时间，再 `release_and_read`。

DPI 关闭时，这些函数会 `uvm_report_fatal` 提示 “DPI routines are compiled off”。因此 register model 的 backdoor path 依赖 DPI/PLI 可见性；编译关闭后不是静默失败，而是 fatal。

## HDL C backend 分发

`uvm_hdl.c` 根据编译宏选择 vendor backend：

- `VCS` 或 `VCSMX` → `uvm_hdl_vcs.c`
- `QUESTA` → `uvm_hdl_questa.c`
- `INCA` 或 `NCSC` → `uvm_hdl_inca.c`
- 否则 `#error "hdl vendor backend is missing"`

这说明 UVM 1.2 的 HDL backdoor C 层不是完全 simulator-neutral 的，必须由 makefile/工具 overlay 定义正确 vendor 宏并提供 VPI/VHPI/FLI 可见性。

## HDL backend 共通调用链

典型调用链：

```text
SV/reg backdoor code
  -> uvm_hdl_read/deposit/force/release(path,value)
  -> DPI-C imported function
  -> vendor backend uvm_hdl_* implementation
  -> vpi_handle_by_name / vhpi_handle_by_name
  -> vpi_get_value / vpi_put_value 或 VHPI/MHPI 等 vendor API
  -> 失败时 m_uvm_report_dpi -> m__uvm_report_dpi -> UVM reporting
```

Verilog/VPI 后端的核心逻辑：

- `uvm_hdl_check_path(path)`：`vpi_handle_by_name(path,0)` 是否非空。
- `uvm_hdl_read(path,value)`：
  1. 查找 handle。
  2. `vpi_get(vpiSize,r)` 检查宽度是否超过 `UVM_HDL_MAX_WIDTH`。
  3. `vpi_get_value(..., vpiVectorVal)`。
  4. 拷贝 `aval/bval` chunks 到 DPI vector。
- `uvm_hdl_deposit(path,value)`：`vpi_put_value(..., vpiNoDelay)`。
- `uvm_hdl_force(path,value)`：`vpi_put_value(..., vpiForceFlag)`。
- `uvm_hdl_release` / `release_and_read`：使用 `vpiReleaseFlag`；不同 backend 对 release 后读回行为略有差异。

## Vendor 差异

### Questa backend

`uvm_hdl_questa.c` 支持一些 Questa 特有处理：

- 如果 path 以 `$root.` 开头，调用 `vpi_handle_by_name(path+6,0)`，即去掉 `$root.` 前缀。
- 支持 part-select workaround：
  - `uvm_hdl_set_vlog_partsel` / `uvm_hdl_get_vlog_partsel` 解析形如 `[lhs:rhs]` 的路径。
  - 对 part-select 拆成单 bit 路径逐位 set/get。
- read 时会清零 vector chunks，避免旧值残留。

### VCS backend

`uvm_hdl_vcs.c` 的基本 Verilog/VPI 路径类似：`vpi_handle_by_name`、`vpi_get_value`、`vpi_put_value`。

额外支持 `VCSMX` mixed-language：

- 使用 `mhpi` / `vhpi` 处理 VHDL 路径。
- `uvm_hdl_get_mhdl` 把 VHDL string value 转成二进制/整数后填入 `aval/bval`。
- `uvm_hdl_set_mhdl` 把 `aval` 转成二进制字符串后调用 `mhpi_force_value`。

局限：VCSMX 代码对宽向量/4-state/VHDL 编码的处理相对简化，学习 register backdoor 调试时要优先怀疑工具/可见性/语言边界。

### INCA/NCSC backend

`uvm_hdl_inca.c` 同时处理 Verilog 与 VHDL：

- 使用 `vhpi_handle_by_name` 获取对象并判断 `vhpiLanguageP`。
- Verilog 路径走 VPI read/write。
- VHDL 路径走 `uvm_hdl_get_vhdl` / `uvm_hdl_set_vhdl`，把 VHPI enum/enum vector 与 VPI `aval/bval` 表示互转。
- 默认对 part-select 手工拆 bit；若定义 `INCA_EXTENDED_PARTSEL_SUPPORT`，可依赖更强的 `vpi_handle_by_name` 切片能力。

## 与 register model 的关系

Register model 的 backdoor 访问、HDL path 检查、内建 `uvm_reg_mem_hdl_paths_seq` 都依赖 `uvm_hdl_*` API。其关键约束来自 DPI 层：

- HDL path 必须能被 simulator 的 VPI/VHPI/MHPI 按名字找到。
- 编译/仿真必须打开足够的 signal visibility / PLI/ACC 权限，否则 path 存在于设计概念中但 C 层找不到。
- path 宽度不能超过 `UVM_HDL_MAX_WIDTH`，默认 1024 bit；大寄存器/大 memory slice 需要编译期提高宏。
- 4-state 值通过 `aval/bval` chunks 表示，跨 VHDL/Verilog 或 mixed-language 时 backend 可能做额外转换。
- release/force 对 `reg` 与 `wire` 的语义不同：SV 注释指出 release 后，`reg` 可能仍保持被 force 的值直到过程赋值，`wire` 通常立即回到 continuous drivers 的解析值。

## 常见调试入口

- 命令行参数不生效：检查 `UVM_CMDLINE_NO_DPI` / `UVM_NO_DPI` 是否定义；确认参数是否被 `uvm_cmdline_processor` 构造时扫描到；注意 `get_arg_matches` 非 regex 模式只是前缀匹配。
- regex/glob 不匹配：记住返回 0 表示匹配；确认输入是否 `/.../`；关闭 DPI 后只剩 glob fallback。
- backdoor path 找不到：先调用/查看 `uvm_hdl_check_path`；确认 simulator signal visibility、层次路径、`$root.` 前缀、语言边界和 vendor backend。
- backdoor 宽度失败：提高 `+define+UVM_HDL_MAX_WIDTH=<value>`，并确认 SV package parameter 能被 C 侧通过 VPI 读到。
- release 后值“不变”：区分 `reg` 与 `wire` release 语义，必要时用 `release_and_read` 观察释放后的真实值。

## 与已有知识的连接

- 与 [[knowledge_uvm_6_reporting]]：DPI C 层错误通过 `m_uvm_report_dpi` 回到 UVM reporting。
- 与 [[knowledge_uvm_7_config_resource_db]]：命令行 `+uvm_set_config_*`、trace 开关、factory override 等由 `uvm_cmdline_processor` 收集参数，再在 root/config/factory 等子系统中解释执行。
- 与 [[knowledge_uvm_10_sequences_sequencer]]：`+uvm_set_default_sequence` 的命令行说明位于 cmdline processor，实际通过 config DB 设置 sequencer phase default sequence。
- 与 [[knowledge_uvm_12_register_model]]：register/memory backdoor、HDL path sequence、predict/mirror/check 的后门访问最终落到 `uvm_hdl_*`。
