---
name: knowledge-chapter-1-quantitative-design-principles
description: 第1章 §1.9 Quantitative Principles of Computer Design：并行性、局部性、常见情况与 Amdahl 定律。
metadata: 
  node_type: memory
  type: reference
  originSessionId: 31ab2ef3-e850-4e9a-bd7b-0500dcf7c5cf
  modified: 2026-08-12T07:43:29.226Z
---

# 第1章：定量化计算机设计原则

> 来源：计算机体系结构量化研究方法（第5版）第1章 §1.9 Quantitative Principles of Computer Design。
> 本批学习范围：PDF p.73–p.75；书内 p.45–p.47。下一断点：PDF p.76 / 书内 p.48 的 The Processor Performance Equation。

## 知识点摘要

本单元的核心是：利用并行性，利用程序局部性，优先优化常见情况，并用 Amdahl 定律把局部优化换算为整体收益。局部加速比不能直接当作系统加速比，未被优化的执行时间决定上限。

## 关键细节

### 利用并行性

流水线通过重叠指令执行来减少指令序列的总完成时间；由于并非每条指令都依赖其直接前驱，因此可形成指令级并行。数字设计中，组相联 cache 的多个存储体可并行搜索；ALU 的超前进位也利用并行性缩短加法时间。这些是数据级并行的例子（PDF p.73 / 书内 p.45；承接前页 PDF p.72 / 书内 p.44）。

### Principle of Locality

程序倾向于重复使用最近使用过的数据和指令；书中的经验法则是约 90% 的执行时间耗费在约 10% 的代码上。时间局部性表示最近访问的项目近期可能再次访问；空间局部性表示地址相近的项目往往在时间上接近被引用。局部性可根据最近访问预测近期访问，缓存层次结构正利用此性质（PDF p.73 / 书内 p.45）。

### Focus on the Common Case

设计权衡应优先处理高频情况，因为频率越高，优化对性能、功耗和资源分配的影响越大。取指/译码可能比乘法器更常用；数据库服务器若每处理器有 50 个磁盘，存储可靠性可能支配系统可靠性；加法溢出罕见时，应优化无溢出路径，而让罕见路径更慢（PDF p.73–p.74 / 书内 p.45–p.46）。

### Amdahl’s Law

定义 `Fraction_enhanced` 为原执行时间中可使用增强的比例（`0≤f≤1`），`Speedup_enhanced` 为若整个任务都使用增强时的加速比（`s>1`）。

```text
Speedup = Performance_entire_task_using_enhancement_when_possible
          / Performance_entire_task_without_using_enhancement
        = Execution_time_old / Execution_time_new

Execution_time_new = Execution_time_old × ((1 − f) + f/s)
Speedup_overall = 1 / ((1 − f) + f/s)
```

即使增强部分无限快，整体加速也不超过 `1/(1−f)`；这体现递减收益。`f` 必须是增强前原任务时间中可转换的比例，不能误用增强后的比例。Amdahl 定律也适用于可靠性和功耗（PDF p.74–p.75 / 书内 p.46–p.47）。

#### 例题

1. **Web 服务**：计算占 40%，I/O 等待占 60%，计算快 10 倍：
   `S=1/(0.6+0.4/10)=1/0.64≈1.56`（PDF p.75 / 书内 p.47）。
2. **FPSQR 与全部 FP**：FPSQR 占 20%，加速 10 倍，`S=1/(0.8+0.2/10)=1/0.82≈1.22`；全部 FP 占 50%，加速 1.6 倍，`S=1/(0.5+0.5/1.6)=1/0.8125≈1.23`。优化全部 FP 略好，因为覆盖的常见部分更多（PDF p.75 / 书内 p.47）。
3. **磁盘可靠性**：电源冗余模块局部可靠性提高 4150 倍，但可改善失效率仅 `5/23≈0.22`，系统改善为 `1/(0.78+0.22/4150)=1/0.78≈1.28`（PDF p.75 / 书内 p.47）。

## 原文引用

- “Pipelining is the best-known example of instruction-level parallelism.”（PDF p.73 / 书内 p.45）
- “Programs tend to reuse data and instructions they have used recently.”（PDF p.73 / 书内 p.45）
- “Temporal locality states that recently accessed items are likely to be accessed again in the near future.”（PDF p.73 / 书内 p.45）
- “Spatial locality says that items whose addresses are near one another tend to be referenced close together in time.”（PDF p.73 / 书内 p.45）
- “Perhaps the most important and pervasive principle of computer design is to focus on the common case.”（PDF p.73 / 书内 p.45）
- “A fundamental law, called Amdahl’s law, can be used to quantify this principle.”（PDF p.74 / 书内 p.46）
- “Amdahl’s law expresses the law of diminishing returns.”（PDF p.75 / 书内 p.47）

## 适用条件与关联

Amdahl 分析需明确增强覆盖范围，并计入同步、通信、I/O 等未增强部分；并行化还需计入通信与负载不均衡开销。局部性是经验性质，访问模式不同则强弱不同。关联 §1.7 dependability、§1.8 benchmarks；第2章继续应用局部性。

## 下一断点

从 PDF p.76 / 书内 p.48 的 **The Processor Performance Equation** 开始。

## 后续学习：The Processor Performance Equation

### 基本定义与公式（PDF p.76–78 / 书内 p.48–50）

- 时钟以固定速率运行；时钟周期（clock cycle / cycle time）是一个周期的持续时间，时钟速率（clock rate）是其频率，二者互为倒数。
- CPU 时间的两种等价表达：

```text
CPU time = CPU clock cycles × clock cycle time
CPU time = CPU clock cycles / clock rate
```

- 指令数（instruction count, IC）与每条指令的平均周期数（CPI）给出：

```text
CPI = CPU clock cycles / instruction count
CPU time = instruction count × CPI × clock cycle time
```

- 单位核对：`Instructions/program × Clock cycles/instruction × Seconds/clock cycle = Seconds/program`。
- 处理器性能依赖三个因素：clock cycle time（或 clock rate）、CPI、instruction count；任何一个因素提升 10%，若其他因素保持不变，CPU time 约降低相应比例，但实际改动通常会联动影响另两个因素。
- 影响来源大致为：clock cycle time 受硬件技术与组织影响；CPI 受组织与 ISA 影响；instruction count 受 ISA 与编译器技术影响（PDF p.77 / 书内 p.49）。

### 按指令类型展开（PDF p.78 / 书内 p.50）

对共有 n 类指令的程序：

```text
CPU clock cycles = Σ(IC_i × CPI_i)
CPU time = (Σ(IC_i × CPI_i)) × clock cycle time
CPI = Σ((IC_i / instruction count) × CPI_i)
```

其中 `IC_i` 是第 i 类指令执行次数，`CPI_i` 是该类指令平均占用的时钟周期。实际 CPI 应通过测量或模拟获得，因为它要包含流水线、cache miss 和其他存储系统影响，不能只从参考手册表格直接假定。

### 例题：FPSQR 优化与全部 FP 优化（PDF p.78–79 / 书内 p.50–51）

已测得：FP 指令占 25%，平均 CPI 为 4.0；其他指令平均 CPI 为 1.33；FPSQR 占 2%，其 CPI 为 20。比较“将 FPSQR CPI 降至 2”和“将全部 FP 指令 CPI 降至 2.5”：

- 原始总体 CPI：`(4 × 25%) + (1.33 × 75%) = 2.0`。
- 仅优化 FPSQR：`2.0 − 2% × (20 − 2) = 1.64`。
- 优化全部 FP：`(75% × 1.33) + (25% × 2.5) = 1.625`。
- 若 clock rate 与 instruction count 不变，CPU time 与 CPI 成正比，因此全部 FP 优化略优；整体 speedup 为 `2.00 / 1.625 ≈ 1.23`，与 Amdahl 定律的结果一致（PDF p.79 / 书内 p.51）。

### 设计使用限制

- 处理器性能方程适合把可测量的组成部分联系起来；实际设计中需要测量 instruction count、clock cycles、clock rate，或通过硬件计数器/模拟估计。
- DVFS 和 overclocking 会使 clock speed 在程序运行期间变化，影响直接使用固定 clock rate 的比较；为获得可复现结果，需关闭相关动态特性或明确记录其状态。
- 更快完成程序通常也更节能，但性能、功耗与能量仍需分别测量。

## 下一断点

从 PDF p.80 / 书内 p.52 的 **§1.10 Putting It All Together: Performance, Price, and Power** 开始。
