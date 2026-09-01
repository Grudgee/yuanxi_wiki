---
name: knowledge-superscalar-processor-design-index
description: 《超标量处理器设计》本地学习索引、断点和后续学习入口。
---

# 《超标量处理器设计》学习索引

## 已完成周期

- [周期 01：概览与导论](knowledge-superscalar-processor-design-cycle-01-overview.md) — 前言、处理器分类、章节路线图与超标量 RISC/CISC 概览
- [周期 02：为什么需要超标量](knowledge-superscalar-processor-design-cycle-02-why-superscalar.md) — 性能需求、RISC/CISC路线和超标量设计动机
- [周期 03：普通流水线基础](knowledge-superscalar-processor-design-cycle-03-basic-pipeline.md) — 取指、译码、执行、访存、写回的基本流水组织
- [周期 04：流水线划分与相关性](knowledge-superscalar-processor-design-cycle-04-pipeline-dependences.md) — stage 划分、结构/数据/控制相关和流水线停顿来源
- [周期 05：超标量流水线入口](knowledge-superscalar-processor-design-cycle-05-superscalar-pipeline-entry.md) — 多条指令进入流水线、顺序执行与乱序执行的分叉
- [周期 06：顺序与乱序执行框架](knowledge-superscalar-processor-design-cycle-06-in-order-out-of-order.md) — 保持程序语义、乱序执行和 ROB/retire 的基本理由
- [周期 07：Cache 设计总览](knowledge-superscalar-processor-design-cycle-07-cache-overview.md) — Cache 层次、命中/缺失和超标量取数压力
- [周期 08：Cache 组成方式](knowledge-superscalar-processor-design-cycle-08-cache-organization.md) — direct mapped、set associative、tag/data/valid/dirty 结构
- [周期 09：Cache 写入与替换](knowledge-superscalar-processor-design-cycle-09-cache-write-replacement.md) — write through/write back、dirty 位和替换策略
- [周期 10：Cache 性能优化](knowledge-superscalar-processor-design-cycle-10-cache-performance.md) — 写缓存、流水化、多级 Cache 和 Victim Cache
- [周期 11：Cache 预取](knowledge-superscalar-processor-design-cycle-11-cache-prefetch.md) — 硬件预取、软件预取和时机约束
- [周期 12：多端口 Cache 基础](knowledge-superscalar-processor-design-cycle-12-multiport-cache.md) — true multi-port、复制和 multi-banking
- [周期 13：Opteron D Cache 与取指](knowledge-superscalar-processor-design-cycle-13-opteron-fetch.md) — AMD Opteron 双端口 D Cache 和超标量取指需求
- [周期 14：虚拟存储器与地址转换](knowledge-superscalar-processor-design-cycle-14-virtual-memory-translation.md) — VA/PA、页表、PTE 和 Page Fault
- [周期 15：程序保护](knowledge-superscalar-processor-design-cycle-15-protection.md) — 虚拟存储器的进程隔离、权限位和 cacheable 属性
- [周期 16：TLB 与 Cache 加入流水线](knowledge-superscalar-processor-design-cycle-16-tlb-cache-pipeline.md) — TLB 缓存 PTE、地址转换延迟和 Cache/TLB 协同
- [周期 17：TLB 设计](knowledge-superscalar-processor-design-cycle-17-tlb-design.md) — TLB 作为页表缓存的作用和目标
- [周期 18：MMU 转换流程](knowledge-superscalar-processor-design-cycle-18-mmu-translation-flow.md) — VA→PA、Page Fault 与异常入口
- [周期 19：页表状态位](knowledge-superscalar-processor-design-cycle-19-page-table-status-bits.md) — valid、dirty、use、cacheable 与 OS 协作
- [周期 20：TLB/Cache 协同](knowledge-superscalar-processor-design-cycle-20-tlb-cache-coordination.md) — 地址转换、Cache 命中和页表状态更新关系
- [周期 21：分支预测总览](knowledge-superscalar-processor-design-cycle-21-branch-prediction-overview.md) — 第4章概述与分支预测动机
- [周期 22：方向预测](knowledge-superscalar-processor-design-cycle-22-direction-prediction.md) — taken/not taken 与预测器折中
- [周期 23：目标地址预测](knowledge-superscalar-processor-design-cycle-23-target-prediction.md) — BTB 类思路与目标地址生成
- [周期 24：预测失败恢复](knowledge-superscalar-processor-design-cycle-24-branch-recovery.md) — 错误路径清空与重新取指
- [周期 25：超标量分支预测](knowledge-superscalar-processor-design-cycle-25-superscalar-branch-prediction.md) — 宽前端下的预测/恢复协同
- [周期 26：CISC 与 RISC 概述](knowledge-superscalar-processor-design-cycle-26-cisc-risc-overview.md) — 第5章导论、解码复杂度与 ISA 风格
- [周期 27：Load/Store 与计算指令](knowledge-superscalar-processor-design-cycle-27-load-store-compute.md) — 访存和 ALU 指令分类
- [周期 28：分支、杂项与异常](knowledge-superscalar-processor-design-cycle-28-branch-misc-exception.md) — ISA 级控制转移与异常清空
- [周期 29：一般解码](knowledge-superscalar-processor-design-cycle-29-instruction-decode-general.md) — 指令缓存、固定长度与宽解码
- [周期 30：解码特殊情况](knowledge-superscalar-processor-design-cycle-30-instruction-decode-special-cases.md) — 分支、乘累加、变址和多寄存器指令
- [周期 31：跨章衔接总结](knowledge-superscalar-processor-design-cycle-31-chapter-transitions-summary.md) — 从第3章到第6章的主线串联
- [周期 32：复杂解码与条件执行](knowledge-superscalar-processor-design-cycle-32-decode-complex-instructions.md) — 复杂指令拆分、条件执行和 CPSR 依赖
- [周期 33：重命名动机](knowledge-superscalar-processor-design-cycle-33-renaming-motivation.md) — RAW/WAR/WAW、memory/control/structure 相关
- [周期 34：重命名实现方式](knowledge-superscalar-processor-design-cycle-34-renaming-implementation-ways.md) — ARF 扩展、统一 PRF 和 ROB 三种实现
- [周期 35：RAT 与 Free List](knowledge-superscalar-processor-design-cycle-35-rat-free-list-flow.md) — mapping table、空闲列表与重命名流程
- [周期 36：超标量重命名](knowledge-superscalar-processor-design-cycle-36-superscalar-renaming.md) — 同周期多指令重命名和 RAW 处理
- [周期 37：Checkpoint 恢复](knowledge-superscalar-processor-design-cycle-37-checkpoint-recovery.md) — GC、MBC/CBC 和快速恢复
- [周期 38：WALK 与分发](knowledge-superscalar-processor-design-cycle-38-walk-and-dispatch.md) — ROB 逆向恢复 RAT 和 Dispatch 入口
- [周期 39：发射概述](knowledge-superscalar-processor-design-cycle-39-issue-overview.md) — Issue Queue、ready 状态和发射角色
- [周期 40：发射流水线](knowledge-superscalar-processor-design-cycle-40-issue-pipeline.md) — Wake-up/Select/RF read/Execute 分阶段
- [周期 41：发射分配](knowledge-superscalar-processor-design-cycle-41-issue-allocation.md) — 压缩队列、空位扫描和写入控制
- [周期 42：发射仲裁](knowledge-superscalar-processor-design-cycle-42-issue-arbitration.md) — 选择规则、FU 级仲裁和 tag 广播
- [周期 43：唤醒机制](knowledge-superscalar-processor-design-cycle-43-wakeup-single-multicycle.md) — 单周期与多周期指令唤醒、延迟广播
- [周期 44：load 推测唤醒](knowledge-superscalar-processor-design-cycle-44-load-speculative-wakeup.md) — D-Cache/TLB 不确定性、IW/SW 窗口
- [周期 45：执行与 FU](knowledge-superscalar-processor-design-cycle-45-execution-fu-conditional.md) — 第9章执行单元、条件执行和 select uOP
- [周期 46：旁路与 Cluster](knowledge-superscalar-processor-design-cycle-46-bypass-cluster-memory-acceleration.md) — 旁路网络、操作数选择、Cluster 和存储器指令加速

## 当前状态

- 书籍已定位为 `books/超标量处理器设计.pdf`。
- 当前已完成前 46 个学习周期，覆盖第 1 章到第 9 章的大部分正文。

## 推荐下一步

- 继续第 9 章存储器指令加速后半，或转入第 10 章提交。
