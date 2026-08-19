---
name: knowledge_uvm_14_examples
description: UVM 1.2 examples 学习记忆：hello_world、factory、objections、integrated APB 如何串联源码机制
metadata:
  type: reference
  node_type: memory
  originSessionId: manual-2026-07-23
  modified: 2026-07-23T07:27:29.610Z
---

# UVM 1.2 examples：从最小例子到 APB agent

## 源码范围

本轮阅读了以下例子：

- `examples/simple/hello_world/`
  - `hello_world.sv`
  - `top.sv`
  - `packet.sv`
  - `producer.sv`
  - `consumer.sv`
- `examples/simple/factory/`
  - `test.sv`
  - `env_pkg.sv`
  - `gen_pkg.sv`
  - `packet_pkg.sv`
- `examples/simple/objections/simple.sv`
- `examples/integrated/apb/`
  - `apb.sv`
  - `apb_agent.sv`
  - `apb_master.sv`
  - `apb_monitor.sv`
  - `apb_rw.sv`
  - `apb_config.sv`
  - `apb_sequencer.sv`

本轮目标不是学习 APB 协议本身，而是观察 UVM 源码机制在例子中的“落地点”：component tree、config DB、factory、phasing/objection、TLM1、sequence/sequencer、callback、analysis port、register adapter。

## hello_world：最小 component + TLM1 + config + recording

### testbench 入口

`hello_world.sv` 是 module 级入口：

```systemverilog
module hello_world;
  import uvm_pkg::*;
  `include "uvm_macros.svh"
  `include "packet.sv"
  `include "producer.sv"
  `include "consumer.sv"
  `include "top.sv"

  top mytop;

  initial begin
    uvm_config_int::set(null, "top.producer1", "num_packets", 2);
    uvm_config_int::set(null, "top.producer2", "num_packets", 4);
    uvm_config_int::set(null, "*", "recording_detail", UVM_LOW);
    mytop = new("top");
    run_test();
  end
endmodule
```

关键点：

- 这个例子没有通过 `run_test("test_name")` 创建 `uvm_test`；而是手工 `mytop = new("top")` 创建一个 `uvm_component` 顶层，再调用 `run_test()` 进入 UVM phase 调度。
- `uvm_config_int::set(null, "top.producer1", "num_packets", 2)` 在 component 构造前设置资源，随后 producer 构造时读取。
- `recording_detail` 用通配符 `*` 作用到所有匹配组件，用于 transaction recording。
- 直接调 `uvm_default_table_printer.knobs.*` 展示 policy class 如何影响 print/sprint 输出。

### top component 的结构

`top extends uvm_component`：

- 子组件：
  - `producer #(packet) p1`
  - `producer #(packet) p2`
  - `uvm_tlm_fifo #(packet) f`
  - `consumer #(packet) c`
- 在构造函数中直接 `new` 子组件，并立即连接 TLM：
  - `p1.out.connect(c.in)`
  - `p2.out.connect(f.blocking_put_export)`
  - `c.out.connect(f.get_export)`

这个例子刻意把创建/连接放在 constructor，而不是更规范的 `build_phase/connect_phase`。教学意义是简单展示 component hierarchy 与 TLM 连接；真实工程更常在 `build_phase` factory create，`connect_phase` 连接。

`run_phase` 中：

```systemverilog
phase.raise_objection(this);
uvm_top.print_topology();
#1us;
phase.drop_objection(this);
```

说明 runtime phase 是否结束由 objection 控制；如果没有 raise objection，run phase 中的 producer/consumer 活动可能没有足够仿真时间完成。

### packet：transaction/object automation

`packet extends uvm_transaction`：

- `rand int addr`
- `constraint c { addr >= 0 && addr < 'h100; }`
- `uvm_object_utils_begin/end` + `uvm_field_int(addr, UVM_ALL_ON)`

落到源码机制：

- object registry/factory：宏生成 `type_id`、`get_type_name`、`create` 等。
- policy class：field macro 自动参与 print/compare/copy/pack/record。
- `uvm_transaction` 提供 transaction recording 所需接口；producer/consumer 调用了 `enable_recording`、`begin_tr`、`end_tr`、`accept_tr`。

### producer：config DB + TLM blocking put + clone/randomize

`producer #(T=packet) extends uvm_component`：

- 端口：`uvm_blocking_put_port #(T) out`
- 构造中 `out = new("out", this)`
- 构造中读取配置：

```systemverilog
void'(uvm_config_int::get(this, "", "num_packets", this.num_packets));
```

run flow：

1. `p = proto.clone()`：通过 `uvm_object::clone` + field automation 复制原型。
2. `p.set_name({get_name(), "-", num})`
3. `p.set_initiator(this)`：建立 transaction 与发起组件关系。
4. 若 `recording_detail != UVM_NONE`，`p.enable_recording(get_tr_stream("packet_stream"))`。
5. `p.randomize()`。
6. `out.put(p)`：通过 TLM1 blocking put port 发送。

关键点：producer 不知道下游是 consumer 的 imp，还是 FIFO 的 export；它只依赖 `uvm_blocking_put_port` 接口，这是 TLM 解耦的核心。

### consumer：put imp + get port + semaphore

`consumer #(T=packet) extends uvm_component`：

- `uvm_blocking_put_imp #(T, consumer #(T)) in`：把外部 `put` 调用分发到 consumer 的 `task put(T p)`。
- `uvm_get_port #(T) out`：主动从 FIFO 获取 transaction。
- `local semaphore lock = new(1)`：串行化多个 put/get 来源调用 `put` 的处理。

`run_phase`：

```systemverilog
while(out.size()) begin
  out.get(p);
  put(p);
end
```

注意：这里的 `while(out.size())` 只在当前时刻 FIFO 非空时循环；它不是 forever consumer。这个例子主要靠 `p1` 直接 put 到 `c.in`，以及 `p2` 写 FIFO 后由 consumer 在合适时机处理，展示 TLM 连接方式，而非严谨的生产者消费者模板。

`put(p)` 中调用：

- `accept_tr(p)`
- `begin_tr(p)`
- `end_tr(p)`

说明 transaction recording 是 component 与 transaction 协作的行为；`recording_detail` 配置会影响是否记录。

## factory：type override + instance override + string run_test

### 结构

`factory/test.sv` 中 module `top` 导入三个 package：

- `packet_pkg`：定义 `packet extends uvm_object`
- `gen_pkg`：定义 `gen extends uvm_component`
- `env_pkg`：定义 `env extends uvm_env`

并在 module 内定义两个派生类：

- `mygen extends gen`
- `mypacket extends packet`

### env/gen/packet 的 factory 使用

`env` 构造中：

```systemverilog
gen1 = gen::type_id::create("gen1", this);
```

`gen.get_packet()` 中：

```systemverilog
p = packet::type_id::create("p", this);
void'(p.randomize());
return p;
```

这两个 create 都通过 factory proxy，因此可以被 override 改写。

### override 设置

module initial：

```systemverilog
uvm_coreservice_t cs_ = uvm_coreservice_t::get();
uvm_factory factory = cs_.get_factory();
gen::type_id::set_inst_override(mygen::get_type(), "uvm_test_top.gen1");
packet::type_id::set_type_override(mypacket::get_type());
factory.print(1);
run_test("env");
```

关键点：

- `run_test("env")` 使用字符串类型名，通过 factory 创建 `uvm_test_top` 下的 env instance。
- `gen::type_id::set_inst_override(mygen::get_type(), "uvm_test_top.gen1")` 只替换实例路径 `uvm_test_top.gen1`，不会替换其他 gen。
- `packet::type_id::set_type_override(mypacket::get_type())` 替换所有后续 `packet::type_id::create`。
- `mypacket` 添加 `addr <= 10` 约束，展示 object type override 不需要修改 generator 代码。
- `mygen.get_packet()` 覆盖方法并调用 `super.get_packet()`，展示 component instance override 不只改变 type name，也改变行为。

易错点：

- 注释 `env e; // you need to use something from the package to have the factory registration occur` 提醒：package 中的类注册静态初始化可能受编译/elaboration/use 影响。示例通过声明一个 `env` 变量确保 package 内容被引用。
- instance override 路径是 UVM component full name。`run_test("env")` 创建出的顶层实例名默认为 `uvm_test_top`，所以路径是 `uvm_test_top.gen1`，不是 module 名 `top.gen1`。

## objections：end-of-test 与 drain time

`examples/simple/objections/simple.sv` 定义 `simple_test extends uvm_test`，通过：

```systemverilog
initial run_test("simple_test");
```

进入标准 test 创建流程。

### run_phase 与并发任务

`run_phase` 中设置 drain time：

```systemverilog
uvm_test_done.set_drain_time(this, 10);
```

然后 fork 四个任务：

```systemverilog
fork
  doit(35);
  doit(25);
  doit(50);
  doit(15);
join
```

每个 `doit(delay)`：

1. `uvm_test_done.raise_objection(this)`
2. 等待 `#delay`
3. `uvm_test_done.drop_objection(this)`

最后一个任务在 time 50 drop objection；drain time 10 生效后，test 在 time 60 完成。

### callback hook

`simple_test` 覆盖：

```systemverilog
virtual function void dropped(uvm_objection objection,
                              uvm_object source_obj,
                              string description,
                              int count);
```

在每次 drop 时打印当前 total count：

```systemverilog
objection.get_objection_total(this)
```

这展示了 objection 不是只有 raise/drop 两个 API，还有 raised/dropped/all_dropped 等 callback 钩子。源码层面对应 `uvm_objection` 维护 source/total count、drain time、propagation 与 callback 调用链。

易错点：

- 示例用的是 `uvm_test_done` 全局 objection；UVM 1.2 中更现代/规范的写法通常是 `phase.raise_objection(this)` / `phase.drop_objection(this)`。但这个例子展示的是历史 API 与 objection callback。
- drain time 不是每次 drop 都延迟结束，而是在 objection total 归零时延迟 all_dropped 完成。

## integrated APB：标准 agent/driver/monitor/sequencer 骨架

`examples/integrated/apb/apb.sv` 定义 package `apb_pkg`：

```systemverilog
`include "apb_if.sv"
`include "uvm_macros.svh"
package apb_pkg;
  import uvm_pkg::*;
  typedef virtual apb_if apb_vif;
  typedef class apb_agent;
  `include "apb_rw.sv"
  `include "apb_config.sv"
  `include "apb_master.sv"
  `include "apb_monitor.sv"
  `include "apb_sequencer.sv"
  `include "apb_agent.sv"
endpackage
```

关键点：package 聚合接口 typedef、transaction、config、driver、monitor、sequencer、agent，是一个可复用 VIP 的常见组织方式。

### apb_rw：sequence item + register adapter

`apb_rw extends uvm_sequence_item`：

- `rand bit [31:0] addr`
- `rand logic [31:0] data`
- `rand kind_e kind`，`kind_e={READ,WRITE}`
- field automation 使用 `UVM_ALL_ON | UVM_NOPACK`
- `convert2string()` 输出 transaction 摘要

`reg2apb_adapter extends uvm_reg_adapter`：

- `reg2bus(const ref uvm_reg_bus_op rw)`：创建 `apb_rw`，把 `rw.kind/rw.addr/rw.data` 转成 bus transaction。
- `bus2reg(uvm_sequence_item bus_item, ref uvm_reg_bus_op rw)`：把 monitor/driver 返回的 `apb_rw` 转回 `uvm_reg_bus_op`，并设置 `rw.status = UVM_IS_OK`。

这正是 [[knowledge_uvm_12_register_model]] 中 frontdoor register access 的 adapter 落点：register layer 不认识 APB transaction，只认识 `uvm_reg_bus_op`；adapter 负责桥接。

### apb_agent：build_phase 创建，connect_phase 连接

`apb_agent extends uvm_agent`，包含：

- `apb_sequencer sqr`
- `apb_master drv`
- `apb_monitor mon`
- `apb_vif vif`

`build_phase`：

```systemverilog
sqr = apb_sequencer::type_id::create("sqr", this);
drv = apb_master::type_id::create("drv", this);
mon = apb_monitor::type_id::create("mon", this);
if (!uvm_config_db#(apb_vif)::get(this, "", "vif", vif))
  `uvm_fatal("APB/AGT/NOVIF", ...)
```

`connect_phase`：

```systemverilog
drv.seq_item_port.connect(sqr.seq_item_export);
```

这比 hello_world 更接近真实工程模式：build 负责 factory create 与 config get，connect 负责 TLM 连接。

易错点：此 agent 未根据 `is_active` 条件创建 driver/sequencer；它总是创建 active agent 三件套。真实 VIP 通常根据 `get_is_active()` 决定是否创建 sqr/drv。

### apb_master：driver pull protocol + callbacks + APB pin driving

`apb_master extends uvm_driver#(apb_rw)`：

- `apb_vif sigs`
- build 时优先从 parent agent 取 `agent.vif`；若 parent 不是 agent，则从 config DB 获取。
- run_phase 里 forever：
  1. 等待 clocking block event。
  2. `seq_item_port.get_next_item(tr)` 从 sequencer 拉取 item。
  3. 执行 callback `trans_received`。
  4. 根据 `tr.kind` 调 `read()` 或 `write()` 驱动 APB SETUP/ENABLE。
  5. 执行 callback `trans_executed`。
  6. `seq_item_port.item_done()` 完成 item。

源码机制对应：

- `uvm_driver` 内建 `seq_item_port` 是 `uvm_seq_item_pull_port`。
- `uvm_sequencer` 暴露 `seq_item_export`。
- `get_next_item/item_done` 是典型 pull 型 sequence-driver 握手。
- `uvm_do_callbacks(apb_master, apb_master_cbs, ...)` 展示 callbacks 子系统如何把用户扩展点插入 driver 行为。

APB 行为：

- write：设置 `paddr/pwdata/pwrite/psel`，下一拍 `penable=1`，再清 `psel/penable`。
- read：设置 `paddr/pwrite=0/psel`，下一拍 `penable=1`，再读取 `prdata`，清控制信号。

易错点：driver 的 forever loop 本身不 raise objection；test/sequence/environment 必须负责仿真生命周期。

### apb_monitor：passive sampling + analysis port

`apb_monitor extends uvm_monitor`：

- `virtual apb_if.passive sigs`
- `uvm_analysis_port#(apb_rw) ap`
- build 时从 parent agent 或 config DB 获取 vif。

run loop：

1. 等待 SETUP cycle：`psel==1 && penable==0`。
2. `apb_rw::type_id::create("tr", this)` 创建 transaction。
3. 采样 `kind/addr`。
4. 下一拍检查 `penable==1`，否则 `uvm_error("APB", "APB protocol violation...")`。
5. 根据 READ/WRITE 采样 `prdata` 或 `pwdata`。
6. 调 callback `trans_observed`。
7. `ap.write(tr)` 广播给 scoreboard/predictor/subscriber。

源码机制对应：analysis port 是 one-to-many broadcast，不阻塞 monitor；它适合把 observed transaction 同时送到 scoreboard、coverage subscriber、reg predictor。

### apb_sequencer

`apb_sequencer extends uvm_sequencer#(apb_rw)`，没有额外逻辑。这说明许多简单总线 VIP 的 sequencer 只是类型化仲裁器；复杂功能通常在 sequence 或 virtual sequencer 中实现。

## 从 examples 回看源码机制

- Component/phasing：
  - hello_world 手动 `new("top")` + `run_test()`；factory 例子 `run_test("env")`；objections/APB 使用标准 phase callbacks。
  - build/connect/run 的职责在 APB 例子中最清楚。
- Factory：
  - object/component macros 产生 registry proxy。
  - `type_id::create` 是 override 生效点。
  - `run_test("env")` 是 string-based factory create 的入口。
- Config/resource DB：
  - hello_world 用 `uvm_config_int::set/get` 配 `num_packets`。
  - APB agent/driver/monitor 用 `uvm_config_db#(apb_vif)::get` 配 virtual interface。
- TLM1：
  - hello_world 展示 blocking put/get + FIFO。
  - APB driver/sequencer 展示 seq_item_pull port/export。
  - APB monitor 展示 analysis port。
- Sequences：
  - APB transaction 是 `uvm_sequence_item`，driver 用 pull protocol 消费。
- Reporting：
  - `uvm_info/uvm_error/uvm_fatal` 在所有例子中出现，展示 report id、verbosity、fatal stop 行为。
- Callbacks：
  - APB master/monitor 用 `uvm_do_callbacks` 提供 transaction hook。
- Register model：
  - `reg2apb_adapter` 是 register frontdoor 接入真实 bus 的最小例子。

## 学习建议

下一步若继续 examples，建议进入 `examples/integrated/codec/`：

- `reg_model.svh`：完整 register model 定义。
- `tb_env.svh`：env 如何集成 APB agent、scoreboard/coverage/predictor。
- `apb2txrx.svh`：APB 到 codec transaction 的桥接。
- `testlib.svh`：sequence/test 如何驱动 reg model 和 bus。

这样可以把本轮 APB agent 与前面 register model 深入学习串起来，形成一个完整 UVM verification environment 的端到端路径。

## 关联知识

- [[knowledge_uvm_2_object_model]]
- [[knowledge_uvm_3_factory_registry]]
- [[knowledge_uvm_5_component_phasing]]
- [[knowledge_uvm_6_reporting]]
- [[knowledge_uvm_7_config_resource_db]]
- [[knowledge_uvm_8_tlm1]]
- [[knowledge_uvm_10_sequences_sequencer]]
- [[knowledge_uvm_11_standard_components]]
- [[knowledge_uvm_12_register_model]]
- [[knowledge_uvm_13_dpi]]
