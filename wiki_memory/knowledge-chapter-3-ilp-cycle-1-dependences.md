---
name: knowledge-chapter-3-ilp-cycle-1-dependences
description: 第3章 §3.1 周期一，ILP、依赖与程序顺序基础。
metadata:
  node_type: memory
  source: books/计算机体系结构量化研究方法.pdf
  modified: 2026-08-19
---

# 第3章周期一：ILP 与依赖

> 范围：PDF p.176–180 / 书内 p.148–152。

- ILP 是指同时评估多条相互独立指令的潜在并行性；流水线通过重叠执行暴露 ILP。
- 流水线 CPI 可分解为 `Ideal pipeline CPI + structural stalls + data hazard stalls + control stalls`。
- 两条主路线是静态路线（编译器发现/调度并行性）和动态路线（硬件运行时发现/调度）；桌面/服务器常用动态路线，能效敏感的移动设备更常采用静态路线。
- 依赖分为 data dependence、name dependence 和 control dependence。真正的数据依赖表示值从生产者流向消费者；name dependence 只是共享名字。
- 寄存器/内存数据流较易定义，但内存依赖更难判断，因为不同地址表达式可能别名，同一表达式在不同迭代也可能指向不同位置。
- name dependence 包括 antidependence（WAR 对应的读后写名字依赖）和 output dependence（WAW 对应的写后写名字依赖）；寄存器重命名可以消除这两类伪依赖。

## 下一周期

PDF p.181 / 书内 p.153：进入 RAW/WAW/WAR data hazards 与 control dependence。
