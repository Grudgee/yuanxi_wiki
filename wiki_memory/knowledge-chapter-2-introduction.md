---
name: knowledge-chapter-2-introduction
description: 第2章 §2.1 Memory Hierarchy Design 导论：层次结构、处理器-存储差距、cache 基础、3C miss 模型、AMAT 与六项基础优化。
metadata:
  node_type: memory
  type: reference
  modified: 2026-08-18
---

# 第2章：存储层次结构设计导论

> 来源：`books/计算机体系结构量化研究方法.pdf`，第5版，第2章 §2.1。
> 本批学习范围：PDF p.100–106 / 书内 p.72–78。§2.1 在 PDF p.106 / 书内 p.78 结束；同页开始 §2.2。
> OCR 源字符数：19,160，低于单批 80,000 字符上限。

## 层次结构的目标与包含关系

- 存储层次利用时间局部性、空间局部性，以及不同存储技术在成本、容量和速度上的权衡，为程序提供接近最快层的访问速度和接近最便宜层的每字节成本。
- 越靠近处理器的层级通常越小、越快、每字节越贵；越远的层级越大、越慢、每字节越便宜。
- 多数层级满足 inclusion property：低层包含高层数据的超集。对 cache 的最低层主存、虚拟存储的最低层磁盘/后备存储，这一包含关系是必需的。
- 服务器层次通常落到磁盘，PMD 通常落到 Flash；两者还在时钟、cache 容量、主存容量和功耗预算上显著不同（PDF p.100 / 书内 p.72）。

## 处理器与 DRAM 差距

- 处理器请求速率长期增长快于 DRAM 访问延迟改善，形成 processor-memory performance gap。多核使聚合带宽需求随核心数继续增长。
- 书中 Intel Core i7 示例：四核、3.2 GHz，每核每周期最多发出两次数据存储访问，数据峰值为 25.6G 次 64-bit 引用/秒；另有约 12.8G 次 128-bit 指令引用/秒，总峰值带宽约 409.6 GB/s，而 DRAM 峰值约 25 GB/s，仅为其约 6%。
- 这一差距通过 cache 多端口、流水化、多级 cache、每核私有 cache，以及 L1 指令/数据分离等方式缓解（PDF p.101 / 书内 p.73）。

## 性能与功耗共同约束

- 传统目标是优化平均存储访问时间，核心变量是 hit time、miss rate 与 miss penalty；现代设计还必须考虑 cache bandwidth 和 power。
- 高端处理器可能有 10 MB 以上片上 cache，大型 L2/L3 同时消耗静态泄漏功耗与读写时的动态功耗。
- PMD 的功耗预算可能比高性能处理器小 20–50 倍，cache 可能占总功耗的 25%–50%，因此不能只按性能选择层次结构（PDF p.102 / 书内 p.74）。

## Cache 基础组织

- cache miss 时，从下一层取回包含多个字的 block/line；一次搬运多个字利用空间局部性。每个 cache block 用 tag 标识其对应内存地址。
- n-way set associative cache 先以 `(Block address) MOD (Number of sets)` 选择集合，再在集合内并行查找。direct-mapped 是每组 1 块；fully associative 是全 cache 只有 1 组。
- write-through 同时更新 cache 和下层存储；write-back 只更新 cache，在块被替换时写回。两者都可用 write buffer 隐藏写入延迟（PDF p.102–103 / 书内 p.74–75）。

## Miss 分类与评价指标

### 3C 模型

- **Compulsory miss**：块首次访问，即使 cache 无限大也会发生。
- **Capacity miss**：cache 无法容纳执行期间所需的全部块，块被逐出后又被访问。
- **Conflict miss**：非全相联映射中，多个块竞争同一组导致逐出和重新取回。
- 多核一致性还会引入第四类 coherency miss，本书第5章处理（PDF p.103 / 书内 p.75）。

### 公式

```text
Misses per instruction
  = Miss rate × Memory accesses per instruction

Average memory access time (AMAT)
  = Hit time + Miss rate × Miss penalty
```

- miss rate 不包含每次 miss 的代价，可能误导；misses per instruction 更接近程序粒度，但同样不包含代价。
- AMAT 比 miss rate 更完整，但仍是执行时间的间接指标。乱序/推测执行和多线程可能用其他工作隐藏 miss latency，所以 AMAT 不能替代真实 execution time（PDF p.103–104 / 书内 p.75–76）。

## 六项基础 Cache 优化

1. **增大 block**：利用空间局部性减少 compulsory miss，也减少 tag 数和部分静态功耗；代价是 miss penalty 变大，并可能增加小 cache 的 capacity/conflict miss。
2. **增大 cache**：减少 capacity miss；代价是 hit time、成本、静态功耗和动态功耗增加。
3. **提高 associativity**：减少 conflict miss；代价是 hit time 和功耗可能增加。
4. **多级 cache**：小而快的 L1 保持周期速度，大而慢的 L2/L3 截获本会进入主存的访问。多级结构通常比同容量单级 cache 更节能。

```text
AMAT = Hit time_L1
     + Miss rate_L1 × (Hit time_L2 + Miss rate_L2 × Miss penalty_L2)
```

5. **读 miss 优先于写**：read miss 时检查 write buffer；若无地址冲突且下层可用，则让读越过写，降低 miss penalty。必须处理 read-after-write hazard。
6. **cache 索引与地址翻译并行**：使用虚拟地址和物理地址相同的 page offset 做 cache index，再用 physical tag 校验，即 virtually indexed, physically tagged。这样可把 TLB 访问移出关键路径，但会限制 L1 的大小/组织并增加系统复杂度。

这些优化都可能在特定条件下反而提高 AMAT，必须联合技术参数、工作负载和功耗模型评估。CACTI 之类模型适合缩小设计搜索空间，不应被当作脱离实现假设的精确预测器（PDF p.104–106 / 书内 p.76–78）。

## 工作负载差异

- 服务器可能同时服务数百用户和数十应用，除平均延迟外还重视 memory bandwidth。
- 桌面系统通常更关注平均 memory latency；高端桌面可能接近低端服务器结构。
- PMD 通常单用户、OS 和应用更轻、并发较少，底层使用 Flash，并把能耗/电池寿命作为一等目标（PDF p.106 / 书内 p.78）。

## 限制与下一入口

- 数值来自第5版当时的技术条件，用于理解量化方法，不代表现代处理器参数。
- 图表和公式由扫描页 OCR 后人工按上下文核对；若用于精确计算，应再次对照原始页面图像。
- 下一入口：PDF p.106 / 书内 p.78 的 §2.2 `Ten Advanced Optimizations of Cache Performance`；下一页 PDF p.107 / 书内 p.79 继续其五类分类和第一项高级优化。
