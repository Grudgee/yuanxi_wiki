---
name: knowledge_uvm_12_register_model
description: "UVM 1.2 register model core classes, access semantics, prediction/update flow, predictor, backdoor, mem, vreg, and built-in test sequences"
metadata: 
  node_type: memory
  originSessionId: bf334e1e-ece6-4e83-801e-0e36bcc6d613
  modified: 2026-07-23T01:52:53.668Z
---

# 知识点摘要

- UVM Register Model 的公共入口是 `src/reg/uvm_reg_model.svh`：它先声明全局 typedef、枚举和辅助类型，再由 `uvm_pkg.sv` 统一 include 进 `uvm_pkg`。
- Register Model 的核心对象链路是 `uvm_reg_block` → `uvm_reg_map` → `uvm_reg` / `uvm_mem` / `uvm_reg_file` / `uvm_vreg`，其中 `uvm_reg_field` 和 `uvm_vreg_field` 是更细粒度的字段抽象。
- `uvm_reg_item` 是寄存器层的通用事务描述，`uvm_reg_adapter` 负责在 `uvm_reg_bus_op` 与具体总线 `sequence_item` 之间转换，`uvm_reg_predictor` 负责把观测到的总线事务回灌到 mirror。
- UVM 1.2 的 register layer 支持 frontdoor、backdoor、predict 三类路径；`uvm_reg_backdoor` 与 callback 机制配合，用于用户自定义 HDL 访问和自动更新。
- `uvm_reg_sequence` 既可作为用户 test sequence 的基类，也可作为 translation sequence 在 bus sequencer 上执行 register transaction。
- `uvm_reg_mem_tests_e` 选择一组内建自检序列，覆盖 reset、bit-bash、access、memory access、shared access 和 memory walk。
- Memory 记忆：`uvm_mem` 不做像 register 那样的镜像维护，文档明确建议更依赖 backdoor；`uvm_mem_mam` 用于内存区间分配，也服务于 virtual register 的实现。
- Virtual register（`uvm_vreg` / `uvm_vreg_field`）是软件语义上的字段/寄存器视图，最终落到 memory 访问上，可静态实现也可动态重新实现或重新分配。

# 关键细节

- `uvm_reg_model.svh` 中定义的关键类型：`uvm_reg_data_t`、`uvm_reg_addr_t`、`uvm_reg_byte_en_t`、`uvm_reg_cvr_t`、`uvm_hdl_path_slice`。
- `uvm_reg_model.svh` 中定义的关键枚举：`uvm_status_e`、`uvm_path_e`、`uvm_check_e`、`uvm_endianness_e`、`uvm_elem_kind_e`、`uvm_access_e`、`uvm_hier_e`、`uvm_predict_e`、`uvm_coverage_model_e`、`uvm_reg_mem_tests_e`。
- `uvm_reg_block`：`configure()` 连接 parent block 和 HDL path；`create_map()` 构造物理接口映射；`set_default_map()` 决定多映射 register/memory 的默认访问 map；`lock_model()` 会递归锁定 block/reg/mem 并初始化 address map，之后不能再增删结构。
- `uvm_reg_block` 的层次 API：`get_blocks()` / `get_maps()` / `get_registers()` / `get_fields()` / `get_memories()` / `get_virtual_registers()` / `get_virtual_fields()` 可按 `UVM_HIER` 递归展开；`get_full_hdl_path()` 会把 block 的 incremental path 与父路径拼接。
- `uvm_reg_map`：`configure()` 保存 base_addr、bus width、endianness、byte addressing；`add_reg()` / `add_mem()` / `add_submap()` 都要求同一个 block 层次约束，并会把 offset/rights/frontdoor 缓存在 `uvm_reg_map_info`。
- `uvm_reg_map_info` 关键字段是 `offset`、`rights`、`unmapped`、`addr[]`、`frontdoor`、`mem_range`、`is_initialized`。
- `uvm_reg_map::backdoor()` 是一个伪 map 单例，用于把 backdoor 当成特殊访问路径传递给 `get_access()` / `predict()` / `read()` / `write()`。
- `uvm_reg_map::get_physical_addresses()` 会按本地 bus width、endianness 和 byte_addressing 拆分地址，再递归映射到父 map；返回的是 top-level granularity 的物理地址序列。
- `uvm_reg_map::get_reg_by_offset(offset, read)` 需要 block 已锁定；当读写同址冲突时，`m_regs_by_offset_wo` 用于 WO/RO 共址的额外查找。
- `uvm_reg`：`configure()` 绑定 parent block/regfile 并可设置单一 HDL path；`add_field()` 按 LSB 排序并检查总位宽、重叠和锁定状态；`get_fields()` 返回按 LSB→MSB 的字段数组。
- `uvm_reg_field::configure()` 的参数依次是 parent、size、lsb_pos、access、volatile、reset、has_reset、is_rand、individually_accessible；访问策略支持 `RO/RW/RC/RS/WRC/WRS/WC/WS/WSRC/WCRS/W1C/W1S/W1T/W0C/W0S/W0T/W1SRC/W1CRS/W0SRC/W0CRS/WO/WOC/WOS/W1/WO1/NOACCESS`。
- `uvm_reg_field::get_access(map)` 会先看字段自身 access，再叠加 register 在该 map 下的 rights；例如 map 为 RO 时，RW/WC/WS/W1C/W1S/W1T/W0C/W0S/W0T/W1 会被压成 RO，RC/WRC/... 会压成 RC，WO/WOC/WOS/WO1 会变成 NOACCESS。
- `uvm_reg_field::set(value)` 更新的是 desired 值，不是 DUT；`predict()` 更新 mirrored/desired；`update()` 会把各 field 的 `XupdateX()` 拼成写值后调用 `uvm_reg::write()`。
- `uvm_reg_field::XpredictX(cur_val, wr_val, map)` 和 `set()` / `XupdateX()` 一一对应，分别刻画“观察到一次真实访问后镜像怎么变”和“为了达到 desired，需要写什么值”。
- `uvm_reg::write()` / `read()` / `update()` / `mirror()` 都通过 `XatomicX()` + busy 标志保护并发；`do_write()` / `do_read()` 负责 callback、frontdoor/backdoor 分发、auto_predict、check_on_read、sample 和最终 `do_predict()`。
- `uvm_reg::Xcheck_accessX()` 会把 `UVM_DEFAULT_PATH` 解析成 block 默认 path；若 `UVM_BACKDOOR` 没有 backdoor/HDL path，会回退 frontdoor；若 map 不存在或 unmapped 但又没自定义 frontdoor，会报错并拒绝操作。
- `uvm_reg::do_write()` 的 backdoor 分支会先用 backdoor 读出当前值，按字段 access 做 peek-modify-poke 的最终写值，再 `do_predict(UVM_PREDICT_WRITE)`；frontdoor 分支调用 map 的 `do_write()` 或 user frontdoor，若 `auto_predict` 打开则操作后立即 `predict()`。
- `uvm_reg::do_read()` 的 backdoor 分支会对 RC/RS/WO 类字段做“读后清零/置位/屏蔽”处理，并在需要时回写修正值；frontdoor 分支在 `auto_predict` 下会先 `sample()`，再 `do_predict(UVM_PREDICT_READ)`。
- `uvm_reg::do_predict()` 会逐 field 调用 `uvm_reg_field::do_predict()`；`kind=UVM_PREDICT_DIRECT` 时若 register 正在 busy，会告警并拒绝，防止 testbench race。
- `uvm_reg_field::do_predict()` 在 `UVM_PREDICT_READ` / `UVM_PREDICT_WRITE` 下会调用 `post_predict()` 回调并按字段 access 修正读写后的镜像；`UVM_PREDICT_DIRECT` 只做直接赋值，不走 access 语义。
- `uvm_mem` 明确不维护类似寄存器的 mirror；其访问以 offset 为中心，支持 `write/read/burst_write/burst_read/poke/peek`，其中 `poke/peek` 是纯 backdoor。
- `uvm_mem::get_access(map)` 会在多 map 场景下叠加 shared memory 的 rights；`get_rights()` 默认返回 RW，只有 shared memory 才可能被限制为 RO/WO。
- `uvm_mem::Xcheck_accessX()` 与 register 类似处理默认 path、backdoor 回退、map 解析和 unmapped/frontdoor 检查；burst 访问还会检查宽度与范围是否越界。
- `uvm_mem::do_write()` / `do_read()` 在 frontdoor 下调用 map/frontdoor 后会 sample block 和 memory；backdoor 下若 access 不是 RW，则模拟 frontdoor 行为不写入只读 memory。
- `uvm_reg_predictor` 使用 `m_pending[uvm_reg]` 按 register 聚合多拍总线事务；当 register 宽于 bus 宽时，它按 `map_info.addr[]` 收集每个 beat，直到地址数齐全才触发 predict。
- `uvm_reg_predictor` 在完成整笔事务后会：可选 `do_check()`（check_on_read）→ `pre_predict()` → `XsampleX()`（register 与 block coverage）→ `rg.do_predict()` → `reg_ap.write(reg_item)`；memory prediction 在该文件里明确未实现。
- `uvm_reg_map::set_auto_predict()` 决定 map 级别的自动镜像更新；`set_check_on_read()` 会级联到所有子 map。
- `uvm_reg_transaction_order_policy::order()` 允许在一次 reg transaction 拆成多个 bus transaction 后重排顺序，适合宽寄存器/特殊总线顺序。
- `uvm_reg_sequence`：`body()` 作为 translation sequence 时会持续从 `reg_seqr.peek()` 获取 `uvm_reg_item`，调用 `do_reg_item()`，再 `reg_seqr.get()` 消费；`do_reg_item()` 根据 `rw.kind` 调 `rw.local_map.do_bus_write()` / `do_bus_read()`。
- `uvm_reg_mem_built_in_seq` 是总入口，按 `tests` 位掩码依次调度 `uvm_reg_hw_reset_seq`、`uvm_reg_bit_bash_seq`、`uvm_reg_access_seq`、`uvm_mem_access_seq`、`uvm_reg_mem_shared_access_seq`、`uvm_mem_walk_seq`。
- `uvm_reg_hw_reset_seq`：先 `reset_blk()` 再 `model.reset()`，然后对每个 leaf map 的 register 调 `mirror(UVM_CHECK, UVM_FRONTDOOR, map)`；它跳过 `NO_REG_TESTS` / `NO_REG_HW_RESET_TEST` 资源标记的 block/reg。
- `uvm_reg_bit_bash_seq`：按每个 map、每个 bit 翻转后写回，再读回，忽略 WO/NOACCESS/不可预测位；它依赖字段 access 生成 bit mode，并用 `dc_mask` 跳过不该测试的位。
- `uvm_reg_access_seq`：对每个 register、每个 map，先 frontdoor 写补值，再 backdoor mirror 校验，再 backdoor 写回原值，再 frontdoor mirror 校验；前提是 register 有 backdoor 或 HDL path，且所有字段在该 map 下的 access 都是已知类型。
- `uvm_reg_shared_access_seq`：只测试映射到至少 2 个 map 的 register；写入时保留不可预测位，读取时对 WO 位用 `wo_mask` 修正期望值，按每个 map 交叉读写验证共享一致性。
- `uvm_mem_access_seq`：对 memory 的每个 map、每个 location 做 frontdoor 写、backdoor 读、backdoor poke、frontdoor 读的往返验证；RO memory 会在最开始先用 backdoor 读初值作期望。
- `uvm_mem_walk_seq`：只测试 RW memory；按 walking-ones 逻辑在地址 k 写 `~k`，检查 k-1，修正 k-1，再检查最后一个地址。
- `uvm_reg_mem_shared_access_seq`、`uvm_reg_mem_access_seq`、`uvm_reg_mem_built_in_seq` 是 block 级组合入口，逐 block 递归向下执行对应的 register/memory 子序列。
- `uvm_reg_mem_hdl_paths_seq` 是零时间 HDL path 自检序列，会对每个 register/memory 的 `get_full_hdl_path()` 逐条路径执行 `uvm_hdl_read()` / `uvm_hdl_check_path()`；它不检查读写权限，只检查路径可达性。

# 原文引用

- 文档：`src/reg/uvm_reg_model.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：文件开头到类型与枚举定义区（约 lines 23-399）
- 依据：`typedef ... uvm_reg_data_t`、`uvm_path_e`、`uvm_reg_mem_tests_e`、`uvm_hdl_path_concat` 等全局声明。
- 文档：`src/reg/uvm_reg_item.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_item` 类定义（约 lines 23-220）
- 依据：`element_kind`、`value[]`、`local_map`、`map`、`path`、`parent`、`status` 等事务字段。
- 文档：`src/reg/uvm_reg_adapter.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_adapter` 与 `uvm_reg_tlm_adapter` 定义（约 lines 23-219）
- 依据：`reg2bus()`、`bus2reg()`、`supports_byte_enable`、`provides_responses`、`get_item()`。
- 文档：`src/reg/uvm_reg_backdoor.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_backdoor` 类定义（约 lines 27-218）
- 依据：pre/post read/write callback 链、`wait_for_change()`、`is_auto_updated()`。
- 文档：`src/reg/uvm_reg_cbs.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：callback 基类定义（约 lines 37-219）
- 依据：callback 顺序说明和对 `rw.value` / `rw.status` / `rw.offset` / `rw.path` 的可修改性说明。
- 文档：`src/reg/uvm_reg_field.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_field` 类定义与实现（约 lines 28-1325）
- 依据：field configure 参数、访问策略表、`get_access()` 权限折叠、`set()` / `predict()` / `XpredictX()` / `XupdateX()`。
- 文档：`src/reg/uvm_reg.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg` 类头部、`do_predict`、`write/read/update/mirror`、`Xcheck_accessX`、`do_write/do_read`、HDL/backdoor 实现（约 lines 27-3100）
- 依据：register 的 parent/maps/fields/backdoor/locks、frontdoor/backdoor 分发、auto_predict、check_on_read、callback 与 sample 链。
- 文档：`src/reg/uvm_reg_block.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_block` 类头部、`lock_model()`、层次 get API、HDL path API（约 lines 26-2180）
- 依据：block 树、`create_map()`、`set_default_map()`、`lock_model()`、`get_*()`、`get_full_hdl_path()`。
- 文档：`src/reg/uvm_reg_map.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_map_info`、`uvm_reg_transaction_order_policy`、`uvm_reg_map` 类定义和实现（约 lines 23-1661）
- 依据：map info、adapter/sequencer/auto_predict/check_on_read、`add_reg()`、`add_mem()`、`add_submap()`、`get_physical_addresses()`、`get_reg_by_offset()`。
- 文档：`src/reg/uvm_mem.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_mem` 类定义和实现（约 lines 24-2309）
- 依据：memory 不做 mirror、MAM 句柄、burst access、backdoor/frontdoor、HDL path、shared access / rights。
- 文档：`src/reg/uvm_reg_file.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_file` 类定义（约 lines 25-218）
- 依据：register file 的层次化组合与 HDL path API。
- 文档：`src/reg/uvm_vreg.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_vreg` 类定义（约 lines 41-199）
- 依据：virtual register 通过 memory 实现、静态/动态 implement/allocate。
- 文档：`src/reg/uvm_vreg_field.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_vreg_field` 类定义（约 lines 39-199）
- 依据：virtual field 的 configure/read/write/access 接口。
- 文档：`src/reg/uvm_reg_predictor.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_predictor` 类定义与 `write()` 实现（约 lines 40-260）
- 依据：bus_in/reg_ap/map/adapter、pending beat 聚合、check_on_read、predict、reg_ap 发布。
- 文档：`src/reg/uvm_reg_sequence.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_sequence` 类定义（约 lines 34-260）
- 依据：translation sequence body/do_reg_item 与 model/adapter/reg_seqr 关系。
- 文档：`src/reg/sequences/uvm_reg_hw_reset_seq.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_hw_reset_seq`（约 lines 24-169）
- 依据：reset 后按 map 逐 reg mirror 校验。
- 文档：`src/reg/sequences/uvm_reg_bit_bash_seq.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_single_bit_bash_seq` / `uvm_reg_bit_bash_seq`（约 lines 31-302）
- 依据：逐 bit 翻转、dc_mask、按 map 测试。
- 文档：`src/reg/sequences/uvm_reg_access_seq.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_single_access_seq` / `uvm_reg_access_seq`（约 lines 37-300）
- 依据：frontdoor/backdoor 交叉写读验证。
- 文档：`src/reg/sequences/uvm_reg_mem_shared_access_seq.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_shared_access_seq` / `uvm_mem_shared_access_seq` / `uvm_reg_mem_shared_access_seq`（约 lines 33-484）
- 依据：shared access 交叉写读、wo_mask、跨 map 验证。
- 文档：`src/reg/sequences/uvm_mem_access_seq.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_mem_single_access_seq` / `uvm_mem_access_seq`（约 lines 28-365）
- 依据：memory frontdoor/backdoor 往返验证。
- 文档：`src/reg/sequences/uvm_mem_walk_seq.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_mem_single_walk_seq` / `uvm_mem_walk_seq`（约 lines 33-299）
- 依据：walking-ones memory 测试。
- 文档：`src/reg/sequences/uvm_reg_mem_built_in_seq.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_mem_built_in_seq`（约 lines 24-137）
- 依据：按 bitmask 调度内建序列。
- 文档：`src/reg/sequences/uvm_reg_mem_hdl_paths_seq.svh`
- 版本/日期：Accellera UVM 1.2 / 2014
- 位置：`uvm_reg_mem_hdl_paths_seq`（约 lines 28-174）
- 依据：HDL path 可达性检查与 zero-time 验证。

# 适用条件与例外

- 以上结论适用于 Accellera UVM 1.2 源码树 `UVM_1_2_RELEASE-1-gd6e87e2`。
- `uvm_reg_predictor` 文档中明确写出 memory prediction 不处理；不要把 register predictor 的行为直接推广到所有 memory 场景。
- `uvm_mem` 的“通常不 mirror”是源码注释与设计建议，不等于任何场景都绝对不能 mirror；实现上仍需结合具体 backdoor/frontdoor 使用方式理解。
- `uvm_reg_sequence` 的 convenience API 在本文件注释中提到“not yet implemented”时，指的是注释所述上下文；具体子类可能自行封装更高层 API。
- `uvm_reg_field` 的预定义 access policy 里，只有部分 policy 会允许 `is_rand` 生效；非可写策略通常会关闭该字段的 rand_mode。
- `uvm_mem::burst_read/burst_write` 的实现要求 map 宽度、memory 位宽和 offset 范围同时满足约束，否则会报错或返回不成功状态。

# 关联章节

- `src/uvm_pkg.sv` 中 `uvm_reg_model.svh` 的 include 位置
- `src/reg/` 下所有 register layer 子文件
- `knowledge_uvm_7_config_resource_db.md`（config/resource 与 reg 层常配合）
- `knowledge_uvm_8_tlm1.md`、`knowledge_uvm_9_tlm2.md`（adapter / predictor 常与 TLM/sequence 结合）
- `knowledge_uvm_10_sequences_sequencer.md`（register sequence 与普通 sequence 的关系）
- `knowledge_uvm_11_standard_components.md`（predictor/monitor/scoreboard 在 testbench 中常见搭配）

# 待核验问题

- 无
