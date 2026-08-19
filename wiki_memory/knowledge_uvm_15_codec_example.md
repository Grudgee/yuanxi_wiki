---
name: knowledge_uvm_15_codec_example
description: UVM 1.2 integrated/codec 端到端示例学习记忆：reg model、APB/VIP agent、scoreboard、phase jump 与 ISR 流程
metadata:
  type: reference
  node_type: memory
  originSessionId: manual-2026-07-23
  modified: 2026-07-23T07:46:32.478Z
---

# UVM 1.2 integrated/codec：端到端 phasing + reg model + agent 互联

## 源码范围

本轮阅读：

- `examples/integrated/codec/README.txt`
- `examples/integrated/codec/tb_top.sv`
- `examples/integrated/codec/test.sv`
- `examples/integrated/codec/tb_env.svh`
- `examples/integrated/codec/reg_model.svh`
- `examples/integrated/codec/apb2txrx.svh`
- `examples/integrated/codec/sym_sb.svh`
- `examples/integrated/codec/testlib.svh`
- `examples/integrated/codec/vip/vip.sv`
- `examples/integrated/codec/vip/vip_agent.svh`
- `examples/integrated/codec/vip/vip_driver.svh`
- `examples/integrated/codec/vip/vip_monitor.svh`
- `examples/integrated/codec/vip/vip_seqlib.svh`
- `examples/integrated/codec/vip/vip_tr.svh`

这是 UVM 1.2 examples 中较完整的端到端示例：一个 full-duplex parallel-to-serial codec DUT，通过 APB register frontdoor 控制 Tx/Rx FIFO；通过串行 VIP 产生/监视符号流；scoreboard 在 ingress/egress 两个方向比较 expected/observed 数据；env 使用 UVM runtime phases（reset/configure/main/shutdown）协调生命周期。

## DUT/寄存器规格概览

`README.txt` 描述 DUT：

- 全双工 parallel-to-serial codec。
- 写入 TxFIFO 的 byte 会按 MSB-first 串行发送。
- Rx 方向接收串行 byte，过滤 SYNC/IDLE/ESC 等控制字符后进入 RxFIFO。
- 每 7 个 byte 插入 SYNC `0xB2`。
- IDLE `0x81` 表示无效数据，ESC `0xE7` 用于转义 IDLE/ESC 作为有效数据。

host-visible register：

- `0x0000 IntSrc`：interrupt source，包含 TxEmpty/TxLow/TxFull/RxEmpty/RxHigh/RxFull/SA；SA 是 W1C。
- `0x0004 IntMask`：interrupt mask。
- `0x0010 TxStatus`：TxEn。
- `0x0014 TxLWM`：Tx FIFO low-water mark。
- `0x0020 RxStatus`：RxEn + Align。
- `0x0024 RxHWM`：Rx FIFO high-water mark。
- `0x0100 TxRx`：写为 TxData，读为 RxData。

## tb_top：module 世界和 program test 世界的接口边界

`tb_top.sv` 定义硬件世界：

- `tb_ctl_if ctl(clk, sclk, rst, intr)`：环境控制 reset / 观察 interrupt。
- `apb_if apb0(clk)`：APB bus interface。
- `vip_if vip0(sclk, rx, tx)`：串行 VIP interface。
- `dut dut0(tx, rx, sclk, apb0, intr, clk, rst)`。

initial 中：

1. 先让 reset resolve：`#100 ctl.rst = 1'b1`。
2. 再启动 `clk` 与 `sclk`。
3. 注释强调：clocks eventually toggle，但释放 reset 由 environment 负责。

`test.sv` 使用 `program test`：

- include `apb.sv` 与 `vip.sv`。
- import `apb_pkg::*` 与 `vip_pkg::*`。
- include `sym_sb.svh`、`apb2txrx.svh`、`tb_env.svh`、`testlib.svh`。
- 手工创建 `tb_env env = new("env")`。
- 用 config DB 设置三个 virtual interface：
  - `uvm_config_db#(tb_ctl_vif)::set(null, "env", "vif", tb_top.ctl)`
  - `uvm_config_db#(apb_vif)::set(null, "env.apb", "vif", tb_top.apb0)`
  - `uvm_config_db#(vip_vif)::set(null, "env.vip", "vif", tb_top.vip0)`
- `run_test()` 启动 UVM。

这个例子与 `hello_world` 类似，env 是手工 new 的，而不是通过 factory 由 `run_test("...")` 创建；但 test class 仍通过 factory 注册，`hw_reset_test extends test` 可通过 `+UVM_TESTNAME=hw_reset_test` 或 `run_test` 机制运行。

`test extends uvm_test` 的 `start_of_simulation_phase` 通过 core service 找 root，再 `top.find("env")` 把手工创建的 env 句柄取回：

```systemverilog
uvm_root top = cs_.get_root();
$cast(env, top.find("env"));
```

这展示了 component tree 全局查找的用法，也说明手工 new 的 `env` 只要 parent 为 null，仍会挂到 `uvm_top` 下。

## reg_model：寄存器对象、field、map 与访问策略

`reg_model.svh` 定义多个 `uvm_reg` 派生类：

- `reg_IntSrc`
- `reg_IntMask`
- `reg_TxStatus`
- `reg_TxLWM`
- `reg_RxStatus`
- `reg_RxHWM`
- `reg_TxRx`

每个 register 的 build pattern：

1. `uvm_reg_field::type_id::create(field_name,,get_full_name())` 创建 field。
2. `field.configure(this, size, lsb_pos, access, volatile, reset, has_reset, is_rand, individually_accessible)`。
3. class 用 `uvm_object_utils` 注册。

例子：

- `IntSrc` 总宽 9 bit，多个 RO flag，SA field 为 `W1C`。
- `IntMask` 总宽 9 bit，全部 RW，reset 0。
- `TxStatus.TxEn` RW。
- `RxStatus.RxEn` RW，`Align` RO。
- `TxRx.TxRx` 8 bit，access 写成 `RW`，volatile=1，地址层面表现为写 Tx FIFO / 读 Rx FIFO。

`reg_dut extends uvm_reg_block`：

- build 中 create/configure/build 每个 register。
- 创建 default map：

```systemverilog
default_map = create_map("default_map", 'h0, 4, UVM_LITTLE_ENDIAN, 1);
default_map.add_reg(IntSrc,  'h0000, "RW");
default_map.add_reg(IntMask, 'h0004, "RW");
default_map.add_reg(TxStatus,'h0010, "RW");
default_map.add_reg(TxLWM,   'h0014, "RW");
default_map.add_reg(RxStatus,'h0020, "RW");
default_map.add_reg(RxHWM,   'h0024, "RW");
default_map.add_reg(TxRx,    'h0100, "RW");
```

要点：

- `create_map(..., n_bytes=4, UVM_LITTLE_ENDIAN, byte_addressing=1)` 表示 APB data bus 以 4-byte granularity 接入。
- field-level access 和 map-level access 都参与最终访问语义；此例 map 统一写 `RW`，field 的 RO/W1C/RW 仍体现具体语义。
- `volatile=1` 用于状态/FIFO 类寄存器，提醒 mirror/check 行为不能按普通稳定寄存器理解。

## tb_env build/connect：把 APB、VIP、regmodel 和 scoreboard 拼起来

`tb_env extends uvm_env` 的成员：

- `tb_ctl_vif vif`
- `apb_agent apb`
- `reg_dut regmodel`
- `vip_sequencer tx_src`
- `uvm_seq_item_pull_port#(vip_tr) tx_src_seq_port`
- `vip_agent vip`
- `sym_sb ingress`：VIP -> DUT 方向 scoreboard。
- `sym_sb egress`：DUT -> VIP 方向 scoreboard。
- `apb2txrx adapt`：把 APB TxRx register transaction 转换成 VIP symbol transaction。

### build_phase

- 从 config DB 获取 `tb_ctl_vif`，失败 fatal。
- factory create `apb`。
- 创建 regmodel：
  - `reg_dut::type_id::create("regmodel",,get_full_name())`
  - `regmodel.build()`
  - `regmodel.lock_model()`
- factory create `tx_src`、`vip`、`ingress`、`egress`、`adapt`。
- 给两个 sequencer 设置 default sequence：

```systemverilog
uvm_config_db #(uvm_object_wrapper)::set(this, "vip.sqr.main_phase",
                                         "default_sequence",
                                         vip_sentence_seq::type_id::get());
uvm_config_db #(uvm_object_wrapper)::set(this, "tx_src.main_phase",
                                         "default_sequence",
                                         vip_sentence_seq::type_id::get());
```

这正对应 sequence/sequencer 学习中的 default_sequence 机制：phase-specific path + `uvm_object_wrapper`。

### connect_phase

```systemverilog
reg2apb_adapter reg2apb = new;
regmodel.default_map.set_sequencer(apb.sqr, reg2apb);
regmodel.default_map.set_auto_predict(1);
```

含义：register frontdoor 访问会经 default_map 发到 APB sequencer，adapter 在 `uvm_reg_bus_op` 与 `apb_rw` 之间转换；`set_auto_predict(1)` 表示每次 frontdoor read/write 后由 map 自动更新 mirror，而不需要单独显式 predictor。

其他连接：

- `tx_src_seq_port.connect(tx_src.seq_item_export)`：env 自己从一个 standalone sequencer 拉 stimulus item，再写 DUT Tx FIFO。
- `apb.mon.ap.connect(adapt.apb)`：APB monitor 观察到 TxRx register 读/写后进入 adapter component。
- `vip.tx_mon.ap.connect(ingress.expected)`：VIP 发往 DUT 的串行流，作为 ingress expected。
- `vip.rx_mon.ap.connect(egress.observed)`：从 DUT 发到 VIP 的串行流，作为 egress observed。
- `adapt.tx_ap.connect(egress.expected)`：通过 APB 写入 DUT Tx FIFO 的 byte，期望最终从 DUT 串行输出。
- `adapt.rx_ap.connect(ingress.observed)`：通过 APB 从 DUT Rx FIFO 读出的 byte，应该匹配 VIP 输入给 DUT 的 byte。

于是形成两个闭环：

```text
Ingress: VIP driver -> DUT RX serial -> DUT RxFIFO -> APB read TxRx -> adapt.rx_ap -> ingress.observed
         VIP tx_monitor -------------------------------------------> ingress.expected

Egress:  tx_src sequencer -> env main_phase -> APB write TxRx -> DUT TxFIFO -> DUT TX serial -> VIP rx_monitor
         APB monitor -> adapt.tx_ap -------------------------------> egress.expected
```

## apb2txrx：analysis imp 到 analysis ports 的协议转换

`apb2txrx extends uvm_component`：

- `uvm_analysis_imp#(apb_rw, apb2txrx) apb`
- `uvm_analysis_port#(vip_tr) tx_ap`
- `uvm_analysis_port#(vip_tr) rx_ap`
- `radd/wadd` 默认都是 `0x0100`，可通过 config DB 覆盖。

`write(apb_rw rw)`：

- 如果 `rw.kind == READ && rw.addr == radd`：创建 `vip_tr("rx")`，`tr.chr=rw.data`，写 `rx_ap`。
- 如果 `rw.kind == WRITE && rw.addr == wadd`：创建 `vip_tr("tx")`，`tr.chr=rw.data`，写 `tx_ap`。

要点：analysis imp 的 `write()` 是被 APB monitor 调用的“入口”；两个 analysis port 是转换后的“出口”。这个 component 是 scoreboard glue，不参与驱动。

## sym_sb：双 analysis imp scoreboard

`sym_sb` 使用两个 `uvm_analysis_imp_decl` 生成不同后缀的 imp 类型：

```systemverilog
`uvm_analysis_imp_decl(_sym_sb_expected)
`uvm_analysis_imp_decl(_sym_sb_observed)
```

成员：

- `expected` 接收 expected stream。
- `observed` 接收 observed stream。
- `m_sb[$]` 保存 expected byte queue。
- `n_obs_thresh` 默认 10，可从 config DB 读取。

`write_sym_sb_expected(vip_tr tr)`：push expected。

`write_sym_sb_observed(vip_tr tr)`：pop expected 并比较，不同则 `uvm_error("SB/MISMTCH",...)`，同时 `m_n_obs++`。

`reset_phase` 清空 scoreboard。

`main_phase` raise objection，直到 `m_n_obs > n_obs_thresh` 才 drop，作为“测试完成条件”之一。

易错点：这个 scoreboard 假设 observed 不会早于 expected，否则 `pop_front()` 会在空 queue 上出错/返回默认；真实工程需处理 underflow、乱序、latency window。

## VIP package：串行协议 agent

`vip.sv` 定义 `vip_pkg`：

- virtual interface typedef：`vip_vif`、`vip_tx_vif`、`vip_rx_vif`
- `vip_tr extends uvm_sequence_item`：一个 `rand bit[7:0] chr`
- `typedef uvm_sequencer#(vip_tr) vip_sequencer`
- include driver/monitor/agent/sequence library

### vip_sequence 与 default sequence

`vip_sequence extends uvm_sequence#(vip_tr)` 的 constructor 调用：

```systemverilog
set_automatic_phase_objection(1);
```

这表示 sequence 作为 phase default sequence 启动时，会自动为对应 phase raise/drop objection。其派生类：

- `vip_one_char_seq`：发一个 item。
- `vip_sentence_seq`：repeat 128 次 `uvm_do(req)`。
- `vip_idle_esc_seq`：repeat 128 次，约束 `chr inside {8'hE7, 8'h81}`，专门测试 ESC/IDLE 转义。

### vip_agent

`vip_agent extends uvm_agent`：

- `sqr`、`drv`、`tx_mon`、`rx_mon`。
- config DB 获取 `vip_vif`。
- 将 interface modport 分发给 monitor：
  - `uvm_config_db#(vip_rx_vif)::set(this, "tx_mon", "vif", vif.tx_mon)`
  - `uvm_config_db#(vip_rx_vif)::set(this, "rx_mon", "vif", vif.rx)`
- connect：`drv.seq_item_port.connect(sqr.seq_item_export)`。
- 提供 `reset_and_suspend()` / `suspend()` / `resume()`，协调 driver 和 monitors。

### vip_driver

`vip_driver extends uvm_driver#(vip_tr)`：

- 内部 suspend/resume state：`m_suspend`、`m_suspended`、`m_proc`、`m_interrupted`。
- reset/pre_reset 会 kill 正在发送的进程并置 Tx。
- `run_phase` 主循环在 resume 后 fork 一个发送进程：
  - 每 6 个 data slot 后插入 SYNC `0xB2`。
  - `seq_item_port.try_next_item(tr)` 非阻塞取 item。
  - 无 item 时发送 IDLE `0x81`。
  - item 为 IDLE/ESC 时先发 ESC `0xE7`，必要时穿插 SYNC。
  - 对真实 item 调 `seq_item_port.item_done()`。
- `send(data)` 按 MSB-first 在 negedge `vif.clk` 驱动 8 bit。
- callback：`vip_driver_cbs.pre_tx/post_tx`。

这展示了比 APB master 更复杂的 driver：它不是简单 blocking `get_next_item`，而是持续发送 line encoding，需要用 `try_next_item` 在无 stimulus 时仍发送 IDLE。

### vip_monitor

`vip_monitor extends uvm_monitor`：

- `uvm_analysis_port#(vip_tr) ap`
- build 从 config DB 获取 `vip_rx_vif`。
- reset/suspend/resume 管理内部 process。
- run flow：
  1. 等待 resume。
  2. shift bits，寻找第一个 SYNC `0xB2`。
  3. 再检查后续 3 个 SYNC 是否每隔 7 symbols 出现，确认 sync acquired。
  4. 在 sync 状态下，每 6 个 data symbols 解析一次，跳过 IDLE/ESC 控制字符，处理 escape。
  5. 有有效 byte 时创建 `vip_tr` 并 `ap.write(tr)`。
  6. 每组后检查下一 symbol 是否为 SYNC，否则 `m_in_sync=0` 并 warning。

`is_in_sync()` / `wait_for_sync_change()` 被 env 的 `pre_main_phase` 用于等待 VIP/DUT 进入同步状态。

## tb_env runtime phases：端到端验证流程

### phase_started / phase_ended

`phase_started`：

- reset 开始时，如果 `pull_from_RxFIFO_thread` 存在则 kill，支持 phase jump 回 reset。
- main 开始时 fork `pull_from_RxFIFO(phase)` 后台线程。
- shutdown 开始时设置 `m_in_shutdown=1`，使后台线程不再等待 interrupt。

`phase_ended`：

- 若存在 phase jump target，只允许跳回 `reset`，否则 fatal。
- main 结束时清 `m_isr[TX_ISR]`。
- shutdown 结束时清线程状态。

这展示了 UVM phase callback 可以集中管理跨 phase thread，而不只是在 `*_phase` task 中写行为。

### pre_reset_phase / reset_phase

- `pre_reset_phase` 等待 reset 不再是 X。
- `reset_phase`：
  - assert reset。
  - `regmodel.reset()` 同步 mirror/reset values。
  - `vip.reset_and_suspend()`。
  - 等 10 个 clk。
  - deassert reset。
  - `vip.resume()`。
  - `tx_src.stop_sequences()`。

要点：hardware reset 必须同时作用 DUT、VIP、sequencer 与 register mirror。只 reset DUT 而不 reset regmodel mirror，会导致后续 mirror/update 语义错乱。

### configure_phase

配置 DUT：

- `regmodel.IntMask.SA.set(1)`
- `regmodel.TxStatus.TxEn.set(1)`
- `regmodel.RxStatus.RxEn.set(1)`
- 对 `IntSrc/IntMask/TxStatus/RxStatus` 分别 `update(status)`。

注释特意避免 `regmodel.update(status)`，因为不想写 `TxRx` data register。要点：block-level update 会遍历所有需要更新的 register；含 FIFO/data port 的寄存器不一定适合全 block update。

### pre_main_phase

等待系统同步：

- fork timeout：`repeat (100*8) @(posedge vif.sclk)` 后 fatal。
- 等 VIP 两个 monitor 都 `is_in_sync()`。
- 通过 `regmodel.RxStatus.mirror(status)` 检查 DUT symbol align。
- 若未 align：打开 SA interrupt mask，等待 `vif.intr`。
- 最后清 mask 与 interrupt source：`IntMask.write(status,'h000)`、`IntSrc.write(status,-1)`。

这里把 monitor state 与 register state 一起作为进入 main stimulus 的条件。

### pull_from_RxFIFO：跨 main/shutdown 的 Rx ISR 线程

`pull_from_RxFIFO(phase)` 在 main phase started 时启动，但它对 shutdown phase raise objection：

```systemverilog
uvm_phase shutdown_ph = phase.find_by_name("shutdown");
shutdown_ph.raise_objection(this, "Pulling data from RxFIFO");
```

循环：

1. 等 interrupt 或 shutdown。
2. `IntSrc.mirror(status)`。
3. 若 `SA` 置位，fatal sync loss。
4. 如果不是 RxHigh 且不在 shutdown，则等待其他 ISR 完成后继续。
5. 创建 `rx_isr_seq`，设置 `rx_seq.model = regmodel`，`rx_seq.start(null)`。
6. `rx_isr_seq` 循环 mirror `TxRx` 和 `IntSrc`，直到 `RxEmpty`。
7. shutdown 时 drop shutdown objection 并 break。

这是一个很有价值的 register sequence 用法：sequence 不一定挂在 sequencer 上；`uvm_reg_sequence` 可直接以 `model` 为入口调用 register API。

### main_phase：Tx ISR 和 APB 写 Tx FIFO

main_phase fork 两个线程：

1. watchdog：2,000,000 time 后显示 objections 并 `$finish`。
2. 主 ISR/stimulus loop：
   - raise objection 配置 interrupt mask。
   - forever：
     - drop objection，表示“当前无活跃工作，只等 interrupt”。
     - 等 `vif.intr`。
     - raise objection，开始 DUT-> stimulus。
     - mirror `IntSrc`，若不是 TxLow 则继续。
     - mask TxLow，避免重复 interrupt。
     - while Tx FIFO 未 full：
       - drop objection 等 `tx_src_seq_port.get_next_item(tr)`。
       - item_done 后 raise objection。
       - `regmodel.TxRx.write(status, tr.chr)` 把 byte 写入 DUT TxFIFO。
       - mirror `IntSrc` 更新 TxFull。
     - 重新 unmask TxLow。

要点：

- env 自己像 driver 一样从 `tx_src` sequencer 拉 item，但实际“驱动”是 register write。
- objection 在 wait stimulus / apply stimulus 之间反复 drop/raise，避免 main phase 被永久占住。
- `tx_src` 的 default sequence 自动产生 `vip_tr`，但 env 决定何时把这些 byte 经 APB 写入 DUT。

### shutdown_phase / report_phase

shutdown：

- 若 TxFIFO 非空，打开相应 interrupt mask 并等待 interrupt。
- 等 16 个 serial clock，保证最后 symbol 发出。
- drop objection。

report：

- 从 core service 取 `uvm_report_server`。
- 如果 fatal/error count 为 0，打印 `** UVM TEST PASSED **`；否则打印 fail。

## hw_reset_test：phase jump 回 reset

`testlib.svh` 定义 `hw_reset_test extends test`：

```systemverilog
task main_phase(uvm_phase phase);
  if (once) begin
    once = 0;
    phase.raise_objection(this);
    repeat (100 * 8) @(posedge env.vif.sclk);
    `uvm_info("TEST", "Jumping back to reset phase", UVM_NONE);
    phase.jump(uvm_reset_phase::get());
  end
endtask
```

目的：验证 env 是否支持 runtime phase jump 回 reset。

与 tb_env 的 `phase_started/phase_ended` 配合：

- phase_ended 检查只允许 jump target 为 reset。
- reset started 时 kill RxFIFO 后台线程。
- reset_phase 再次 reset DUT/VIP/regmodel/sequencer。

这展示了 UVM phasing 中 phase jump 的实际约束：不是所有 env 都自然支持任意 phase jump；需要显式清理跨 phase thread、sequence、mirror 和 interface 状态。

## 这例子串起来的 UVM 源码机制

- [[knowledge_uvm_5_component_phasing]]：
  - pre_reset/reset/pre_configure/configure/pre_main/main/shutdown/report。
  - phase_started/phase_ended。
  - phase.jump 与 jump target 检查。
  - objection 控制 phase 生命周期。
- [[knowledge_uvm_7_config_resource_db]]：
  - module/program 将 virtual interface 下发给 env/agent。
  - env 设置 sequencer default_sequence。
  - scoreboard/adaptor 可通过 config DB 调参。
- [[knowledge_uvm_8_tlm1]]：
  - seq_item_pull_port/export。
  - analysis port/imp 多路连接。
  - env 作为“register-driver”从 sequencer 拉 item。
- [[knowledge_uvm_10_sequences_sequencer]]：
  - phase default sequence。
  - `set_automatic_phase_objection(1)`。
  - `try_next_item` vs `get_next_item`。
  - `stop_sequences()` 在 reset 时清理。
- [[knowledge_uvm_11_standard_components]]：
  - agent/driver/monitor/env/scoreboard 的工程组织方式。
- [[knowledge_uvm_12_register_model]]：
  - reg_block/reg/field/map。
  - `set_sequencer(apb.sqr, reg2apb)`。
  - `set_auto_predict(1)`。
  - `set/update/write/mirror/reset`。
  - `uvm_reg_sequence` 以 model 直接访问 register。
- [[knowledge_uvm_14_examples]]：
  - APB agent 的 driver/monitor/reg adapter 在 codec 中被真正接入完整 env。

## 易错点总结

1. 手工 new 的 env 也会进入 UVM component tree，但 test class 若要访问它，需要 `uvm_top.find("env")` 或其他句柄传递方式。
2. virtual interface 必须在 env/agent build 前通过 config DB 设置到正确路径，否则 agent fatal。
3. block-level `regmodel.update()` 可能误写 FIFO/data-port 类型寄存器；本例手工列出要 update 的 registers。
4. reset 不只是 DUT reset，还要 reset register mirror、suspend/resume VIP、stop sequencer sequences、清 scoreboard。
5. phase jump 回 reset 需要 kill 跨 phase thread；否则旧 ISR 线程可能继续访问旧状态。
6. analysis scoreboard 简洁但不健壮：未显式处理 expected queue underflow、乱序、延迟窗口。
7. driver 使用 `try_next_item` 时必须在拿到 item 后准确 `item_done()`；reset/kill 路径也要避免遗漏或重复 item_done。
8. `set_auto_predict(1)` 适合简单 frontdoor；如果有 independent monitor/predictor 或 out-of-order bus，需要更谨慎地使用 explicit predictor。
