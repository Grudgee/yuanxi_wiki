---
name: knowledge_uvm_index
description: UVM 1.2 源码深度学习索引、源码范围、学习路线与主题入口
metadata:
  node_type: memory
  type: reference
  originSessionId: 8f0655e3-e328-4930-855a-ff19c6c920b2
  modified: 2026-07-23T08:54:41.618Z
---

# UVM 1.2 源码知识库索引

## 资料范围

- 本地目录：`file:///home/yuanxi/yuanxi_cc/wiki_files/UVM/uvm-1.2`
- 版本：Accellera Universal Verification Methodology version 1.2
- release notes 日期：Fri Jun 13 11:13:37 IDT 2014
- repository state：`UVM_1_2_RELEASE-1-gd6e87e2`
- 资料类型：SystemVerilog class library reference implementation、examples、HTML reference、User Guide/Reference 入口、DPI C/C++ 源码、转换脚本。
- license：Apache-2.0，完整文本在 `LICENSE.txt`。

## 当前学习目标

用户要求“吃透 UVM 源码”。因此学习方式不是只摘要 API，而是按源码依赖顺序理解：入口文件、include 组织、核心基类、factory、phasing、reporting、config/resource DB、TLM、seq、reg 等子系统，并记录类之间的继承、调用关系、生命周期和常见坑。

## 已学习进度

- 第一阶段已完成：源码目录盘点、顶层入口、包入口、宏入口和学习路线。见 [[knowledge_uvm_1_source_architecture]]。
- 第二阶段已完成：`uvm_object` 对象模型、object globals、misc 工具、object/field automation 宏。见 [[knowledge_uvm_2_object_model]]。
- 第三阶段已完成：factory、registry proxy、type/instance override、create 路径和 core service 获取/替换 factory。见 [[knowledge_uvm_3_factory_registry]]。
- 第四阶段已完成：policy classes，即 `uvm_printer`、`uvm_comparer`、`uvm_packer`、`uvm_recorder`，以及它们与 `uvm_object` 固定入口和 field automation 的配合。见 [[knowledge_uvm_4_policy_classes]]。
- 第五阶段已完成：`uvm_component`、`uvm_root`、`uvm_coreservice`、`uvm_phase` / `uvm_domain`、`uvm_objection`，以及 common/runtime phases 的执行链路。见 [[knowledge_uvm_5_component_phasing]]。
- 第六阶段已完成：reporting，即 `uvm_report_object`、`uvm_report_handler`、`uvm_report_server`、`uvm_report_message`、`uvm_report_catcher`、report macros、verbosity/action/file/override/catcher/exit 链路。见 [[knowledge_uvm_6_reporting]]。
- 第七阶段已完成：config/resource DB，即 `uvm_resource_base`、`uvm_resource#(T)`、`uvm_resource_pool`、`uvm_resource_db#(T)`、`uvm_config_db#(T)`、scope/name/type/precedence、自动配置和命令行配置入口。见 [[knowledge_uvm_7_config_resource_db]]。
- 第八阶段已完成：TLM1，即 ports/exports/imps、analysis port、FIFO、req/rsp channel。见 [[knowledge_uvm_8_tlm1]]。
- 第九阶段已完成：TLM2，即 transport interface、sockets、generic payload、time、extensions。见 [[knowledge_uvm_9_tlm2]]。
- 第十阶段已完成：sequencer/sequence，即 sequence_item、start/start_item/finish_item、driver pull interface、arbitration、response routing、default sequence。见 [[knowledge_uvm_10_sequences_sequencer]]。
- 第十一阶段已完成：标准 components，即 driver/monitor/agent/env/test/subscriber/scoreboard/comparator。见 [[knowledge_uvm_11_standard_components]]。
- 第十二阶段已完成：Register Model 总览；当前进入深入阶段，已补充 `uvm_reg_block` / `uvm_reg_map` / `uvm_reg` / `uvm_mem` / `uvm_reg_field` 访问语义、`uvm_reg_predictor` 多拍拼装逻辑与 `src/reg/sequences/` 内建序列行为。见 [[knowledge_uvm_12_register_model]]。
- 第十三阶段已完成：DPI 层，包括 SV/C 入口、命令行参数扫描、regex/glob、HDL backdoor API、vendor backend（Questa/VCS/INCA）与 register model backdoor 的关系。见 [[knowledge_uvm_13_dpi]]。
- 第十四阶段已完成：examples 第一轮，包括 simple/hello_world、simple/factory、simple/objections、integrated/apb；重点串联 component/phasing、factory、config DB、TLM1、sequence-driver、callbacks、analysis port、reg adapter。见 [[knowledge_uvm_14_examples]]。
- 第十五阶段已完成：integrated/codec 端到端示例，包括 tb_top/test/tb_env、reg_model、APB/VIP agent 互联、scoreboard、default sequence、runtime phases、phase jump、ISR register sequence。见 [[knowledge_uvm_15_codec_example]]。
- 第十六阶段已完成：callbacks、transaction recording/database/stream/recorder、command-line plusargs 在 root/component/factory/config/report 中的处理路径。见 [[knowledge_uvm_16_callbacks_recording_cmdline]]。
- 第十七阶段已完成：最后一轮源码查漏，覆盖 event/barrier/pool/queue、DAP、links/traversal、misc/global helpers、object globals 与 spell checker。见 [[knowledge_uvm_17_final_infrastructure]]。

## 源码入口

- `src/uvm.sv`：顶层编译入口，仅 include `uvm_pkg.sv`。
- `src/uvm_pkg.sv`：定义 `package uvm_pkg;`，按顺序 include：
  1. `uvm_macros.svh`
  2. `dpi/uvm_dpi.svh`
  3. `base/uvm_base.svh`
  4. `dap/uvm_dap.svh`
  5. `tlm1/uvm_tlm.svh`
  6. `comps/uvm_comps.svh`
  7. `seq/uvm_seq.svh`
  8. `tlm2/uvm_tlm2.svh`
  9. `reg/uvm_reg_model.svh`
- `src/uvm_macros.svh`：宏总入口，包含 version/global/message/phase/object/printer/tlm/sequence/callback/reg/deprecated 等宏文件。

## 主题索引

- 源码架构与入口：[[knowledge_uvm_1_source_architecture]]
- Object 模型与 field automation：[[knowledge_uvm_2_object_model]]
- Factory/registry：[[knowledge_uvm_3_factory_registry]]
- Base 核心类族：已完成。覆盖 `uvm_object`、factory/registry、policy classes、`uvm_component`、`uvm_root`、`uvm_coreservice`、phasing/objection、reporting、config/resource DB、event/barrier、pool/queue、DAP、links/traversal 与 misc/global helpers。见 [[knowledge_uvm_2_object_model]]、[[knowledge_uvm_17_final_infrastructure]]。
- Policy classes：已完成。重点：`uvm_printer`、`uvm_comparer`、`uvm_packer`、`uvm_recorder`，以及 object 固定入口/field automation 与 policy 的配合。见 [[knowledge_uvm_4_policy_classes]]。
- Phasing/objection/domain：已完成。重点：`uvm_phase`、topdown/bottomup/task phases、runtime/common phases、`uvm_objection`。见 [[knowledge_uvm_5_component_phasing]]。
- Reporting：已完成。重点：`uvm_report_object`、handler/server/message/catcher、macros、verbosity/action/file/severity override、catcher 和 `UVM_EXIT`/summary 链路。见 [[knowledge_uvm_6_reporting]]。
- Config/resource DB：已完成。重点：`uvm_config_db`、`uvm_resource_db`、`uvm_resource`、scope/name/type/precedence、自动配置和命令行配置入口。见 [[knowledge_uvm_7_config_resource_db]]。
- TLM1/TLM2：已完成。重点：ports/exports/imps/fifo/analysis port/sockets/generic payload。见 [[knowledge_uvm_8_tlm1]]、[[knowledge_uvm_9_tlm2]]。
- Components：已完成常用组件 agent/env/driver/monitor/scoreboard/subscriber/test/comparator。见 [[knowledge_uvm_11_standard_components]]。
- Sequences/sequencer：已完成。重点：sequence_item、sequence_base、sequence、sequencer_base、arbitration、default sequence。见 [[knowledge_uvm_10_sequences_sequencer]]。
- Register model：深入阶段已完成一轮。重点已扩展为 reg/block/field/map/adapter/predictor/sequence/backdoor/mem/vreg 的访问语义、预测链路与内建测试序列。见 [[knowledge_uvm_12_register_model]]。
- DPI：已完成。重点：regex、HDL backdoor、command line processor support、vendor backend。见 [[knowledge_uvm_13_dpi]]。
- Examples：已完成 simple/hello_world、factory、objections、integrated/apb 与 integrated/codec，用作行为验证材料。见 [[knowledge_uvm_14_examples]]、[[knowledge_uvm_15_codec_example]]。

## 建议学习顺序

1. 顶层入口与 include 依赖：`uvm.sv`、`uvm_pkg.sv`、`uvm_macros.svh`、各聚合 include 文件。已完成。
2. `src/base/uvm_object.svh` 与 object 宏：建立 UVM 数据对象、copy/compare/print/pack/record/clone/create 的基础。已完成。
3. Factory/registry：理解类型注册、type override、instance override、UVM 1.2 factory service 变化。已完成。
4. Policy classes：理解 object 数据方法背后的 printer/comparer/packer/recorder。已完成。
5. `uvm_component`/`uvm_root`/phasing/objection：理解 testbench 树、build/connect/run 等 phase 的调度。已完成。
6. Reporting：理解消息、verbosity/action/file 决策、catcher、summary 和退出链路。已完成。
7. Config/resource DB：理解配置传播、资源查找、scope/name/type/precedence 和 build/run-time 优先级。已完成。
8. TLM1/TLM2：理解 ports/exports/imps/fifo/analysis port 和组件间通信。已完成。
9. Register model：因其依赖前面大部分机制，已完成总览与一轮深入阶段。
10. DPI 与 examples：DPI、callbacks/recording/cmdline、examples 与最后一轮基础设施查漏均已完成。

## 关联知识

- [[knowledge_uvm_1_source_architecture]]
- [[knowledge_uvm_2_object_model]]
- [[knowledge_uvm_3_factory_registry]]
- [[knowledge_uvm_4_policy_classes]]
- [[knowledge_uvm_5_component_phasing]]
- [[knowledge_uvm_6_reporting]]
- [[knowledge_uvm_7_config_resource_db]]
- [[knowledge_uvm_8_tlm1]]
- [[knowledge_uvm_9_tlm2]]
- [[knowledge_uvm_10_sequences_sequencer]]
- [[knowledge_uvm_11_standard_components]]
- [[knowledge_uvm_12_register_model]]
- [[knowledge_uvm_13_dpi]]
- [[knowledge_uvm_14_examples]]
- [[knowledge_uvm_15_codec_example]]
- [[knowledge_uvm_16_callbacks_recording_cmdline]]
- [[knowledge_uvm_17_final_infrastructure]]
