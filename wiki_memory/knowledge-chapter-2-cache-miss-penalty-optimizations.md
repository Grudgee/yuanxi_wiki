---
name: knowledge-chapter-2-cache-miss-penalty-optimizations
description: 第2章 §2.2 高级 cache 优化 6–7：critical word first、early restart 与 write-buffer merging。
metadata:
  node_type: memory
  type: reference
  modified: 2026-08-19
---

# 第2章：降低 Cache Miss Penalty 的高级优化

> 来源：`books/计算机体系结构量化研究方法.pdf`，第5版，第2章 §2.2。
> 核心范围：PDF p.114–115 / 书内 p.86–87 的优化 6–7；PDF p.116 / 书内 p.88 仅用于核验 write merging 图示和优化 8 的后续入口。
> OCR 页面源字符数约 7.3k（Tesseract `eng`、PSM 3，对既有约 140 dpi 页面图像提取；PDF p.114–116 以 `wc -m` 统计为 7,280）。低于 80,000 字符上限。

## 优化六：Critical Word First 与 Early Restart

- 处理器发生 block miss 后通常先需要其中一个 word，没有必要等待整个 block 填充完成才恢复执行。
- **Critical word first**：先向内存请求触发 miss 的 word；该 word 到达后立即交给处理器，其余 word 在后台继续填充。
- **Early restart**：仍按正常顺序获取 block，但只要请求的 word 到达就恢复处理器，不等待剩余 word。
- 两种方法主要适用于较大的 cache block；block 很小时，完整传输与关键 word 到达的时间差有限，收益较小。
- 空间局部性会削弱收益：恢复后的下一次访问可能立即需要尚未到达的相邻 word，再次造成等待。
- 因而有效 miss penalty 不是完整 block 传输时间，也不是固定的关键 word 延迟，而是处理器真正无法与执行重叠的等待时间；它取决于 block 大小和对未填充部分的后续访问概率（PDF p.114–115 / 书内 p.86–87）。

## 优化七：Write Buffer Merging

### 基本机制

- write-through cache 的所有 store 都进入下一层，因此依赖 write buffer；write-back cache 在替换 dirty block 时也需要缓冲。
- buffer 未满时，处理器把地址和数据放入 buffer 即可继续执行，由 buffer 在后台完成下层写入。
- 新 store 到来时，将其地址与有效 buffer entry 比较；若属于同一可合并块，就把数据并入已有 entry，而不另占一个 entry。这就是 write merging。
- 多 word 合并写通常比多个独立 word 写更高效，也降低 buffer 满而迫使 cache/处理器停顿的概率（PDF p.115 / 书内 p.87）。

### 容量示例

- 图 2.7 假设 write buffer 有 4 个 entry，每个可容纳 4 个 64-bit word。
- 若连续 4 个 store 不合并，它们各占一个 entry，buffer 被填满且每个 entry 的四分之三浪费。
- 使用 merging 后，4 个相邻 word 可放入同一个 entry，剩余 3 个 entry 继续接收其他写请求（PDF p.115–116 / 书内 p.87–88）。

### 限制

- 即使使用可合并的 4-entry write buffer，研究仍观察到 buffer stall 可造成约 5%–10% 性能损失，说明 merging 不能消除所有写带宽瓶颈。
- memory-mapped I/O 寄存器不能默认合并：相邻地址可能对应具有独立副作用的不同设备寄存器，设备可能要求逐地址、逐 word 写入。
- 系统通常通过页属性把 I/O 区域标记为 nonmerging write-through，保证访问次数、顺序和副作用语义不被合并改变（PDF p.115 / 书内 p.87）。

## 综合结论

- 优化 6 提前交付处理器当前需要的数据，以缩短读 miss 的可见等待时间；优化 7 压缩相邻写请求，以降低 write-buffer 满和下层写事务开销。
- 两者都降低可见 miss penalty，但必须保持正确性：部分 block 尚未到齐时要跟踪有效 word；I/O 和其他有副作用区域必须禁止 write merging。

## 下一断点

PDF p.115 / 书内 p.87 开始的优化 8：`Compiler Optimizations to Reduce Miss Rate`；具体第一项 loop interchange 从 PDF p.116 / 书内 p.88 开始。
