---
name: knowledge-chapter-2-memory-technology-optimizations
description: 第2章 §2.3 的 DRAM、SDRAM/DDR、GDDR、Flash 与内存可靠性基础。
metadata:
  node_type: memory
  source: books/计算机体系结构量化研究方法.pdf
  modified: 2026-08-19
---

# 第2章：Memory Technology and Optimizations

> 来源：`books/计算机体系结构量化研究方法.pdf`，第5版，第2章 §2.3。
> 本批学习范围：PDF p.124–132；书内 p.96–104。PDF p.124 下半页进入 §2.3；正文在 PDF p.132 继续。

## 1. 主存技术的核心矛盾

- Cache 主要关心 miss penalty；主存则必须在容量、延迟、带宽、功耗、成本和可靠性之间折中。
- SRAM 速度快但面积和成本高，适合 cache；DRAM 密度高、成本低，适合主存。
- DRAM 的带宽提升快于延迟改善，因此现代内存优化更偏向提高连续传输能力，而不是消除首次访问延迟。

## 2. DRAM 的组织与刷新

- DRAM 用一个晶体管保存一个 bit；读出会破坏内容，因此读后必须恢复。
- 地址分两次发送：RAS 提供 row address，CAS 提供 column address。行被激活后进入 row buffer，同一打开行的后续列访问可以更快。
- 多个 bank 可独立工作；地址因此包含 bank、row、column。切换到新 bank/新 row 要付出打开延迟，同一 bank 的 open-row 命中则只需列访问。
- DRAM 必须周期性 refresh；整行可同时刷新，但 refresh 会暂时占用内存系统，使访问延迟具有波动，并扩大 cache miss penalty 的可变性。
- DIMM 通常由多个 DRAM 芯片组成，桌面/服务器系统常组织成 8-byte 数据宽度并额外提供 ECC 位。

## 3. SDRAM、DDR 与带宽

- SDRAM 用时钟同步接口，并允许一次请求连续发送多个传输；burst mode 让地址开销被一串数据摊薄。
- 更宽的数据接口和 bank interleaving 进一步提高吞吐；DDR 在时钟上升沿和下降沿都传输数据，使每个时钟周期传两次数据。
- 典型峰值带宽关系：`DIMM bandwidth = transfers/second × 8 bytes`。因此 DDR-266（133 MHz 时钟、266 MT/s）对应约 2128 MB/s 的 PC2100 DIMM。
- DDR 的名称主要表示数据传输速率；DIMM 名称通常表示峰值带宽，二者容易混淆。峰值带宽不等于随机访问的有效带宽，因为首次 RAS/CAS、row conflict 和 precharge 仍需付出代价。

## 4. GDDR 与功耗

- GDDR 面向 GPU 的高带宽需求，通常使用更宽接口、更高数据引脚速率，并直接焊接在 GPU 板上。
- GPU 访问局部性较弱，单纯 burst 的收益可能不如 CPU；保持多个 bank 打开并合理调度 bank 更重要。
- 降低 SDRAM 电压可同时降低动态和待机功耗；bank 化还允许只激活需要的部分。
- power-down mode 可停止对时钟的响应，但仍需自动 refresh，否则超过 refresh 窗口会丢失数据；退出低功耗状态还会引入恢复延迟。

## 5. Flash 在层次结构中的位置

- Flash 是非易失 EEPROM，速度慢于 DRAM 但远快于磁盘；移动设备中既可作磁盘替代，也可作为 DRAM 与磁盘之间的中间层。
- NAND Flash 必须先按 block 擦除，再写入；小写入往往需要重组整个 block，不能像 DRAM 一样直接覆盖单个字节。
- Flash 有有限写入寿命，系统需要 wear leveling，使写入均匀分布到各 block。
- Flash 的成本、功耗和速度均处于 SDRAM 与磁盘之间；读写性能不对称，写入通常比读取更慢。

## 6. 内存可靠性：parity、ECC、Chipkill

- 制造缺陷可通过 spare rows 替换；运行中的永久性故障称 hard error，宇宙射线等导致的瞬态内容变化称 soft error。
- parity 能检测有限范围内的单 bit 错误，但不能纠正，也可能漏检偶数个错误；指令 cache 只读，通常 parity 已足够。
- ECC 可检测两个错误并纠正一个错误；书中典型开销为每 64 个数据 bit 增加 8 个校验 bit。数据 cache 和主存通常采用 ECC。
- 超大系统还需考虑整颗 DRAM 芯片失效。Chipkill 类似磁盘 RAID，把数据和 ECC 分散到多个芯片，从剩余芯片重建失效芯片的数据，可靠性显著高于 parity-only 或 ECC-only。

## 设计要点

- `latency` 与 `bandwidth` 必须分开分析：DDR/SDRAM 主要改善连续传输带宽，不代表首次访问延迟同比改善。
- 内存控制器通过 row-buffer、bank、burst、refresh 调度把器件级约束转化为系统级性能；因此同一 DRAM 的访问时间不是固定常数。
- 从 cache 到主存再到 Flash，每一级都在用更低成本/更高密度交换更大延迟；可靠性机制又会增加容量、功耗或访问开销。

## 下一断点

PDF p.133 / 书内 p.105：继续 `Enhancing Dependability in Memory Systems`，核对 Chipkill 数值案例后进入后续 §2.3 内容。
