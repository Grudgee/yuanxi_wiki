---
name: knowledge-chapter-3-ilp-cycle-2-hazards
description: 第3章 §3.1 周期二，RAW/WAW/WAR、控制依赖与数据流正确性。
metadata:
  node_type: memory
  source: books/计算机体系结构量化研究方法.pdf
  modified: 2026-08-19
---

# 第3章周期二：Hazards 与控制依赖

> 范围：PDF p.181–183 / 书内 p.153–155。

- hazard 发生在指令存在名字/数据依赖且执行重叠可能改变相关操作顺序时；硬件和软件只需保留会影响结果的程序顺序。
- RAW 是 true dependence：消费者在生产者写回前读取旧值，必须保留顺序。
- WAW 是 output/name dependence：两个写入必须按原顺序完成；通常需要多写回阶段、乱序完成或更深流水线才会暴露。
- WAR 是 antidependence：后指令过早写入会破坏前指令尚未完成的读取；静态五级流水线通常因早读晚写而不产生 WAR，但乱序和重新排序会产生。
- RAR 不是 hazard，因为两个读取互不改变数据。
- control dependence 描述指令是否应在分支结果下执行；then 块不能被提前到 branch 前，branch 前的指令也不能被错误地移入受控区域。
- 为了提取 ILP，可以暂时违反 control dependence，但必须保持 exception behavior 与 data flow；仅保持静态 data dependence 不一定足够，因为分支决定哪个生产者实际提供值。

## 下一周期

PDF p.184–185 / 书内 p.156–157：software speculation、控制停顿，以及 §3.2 基础编译器 ILP 技术入口。
