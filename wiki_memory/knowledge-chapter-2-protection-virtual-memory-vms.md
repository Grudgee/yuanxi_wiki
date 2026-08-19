---
name: knowledge-chapter-2-protection-virtual-memory-vms
description: 第2章 §2.4–§2.5 的虚拟内存、虚拟机、Xen 与缓存一致性导论。
metadata:
  node_type: memory
  source: books/计算机体系结构量化研究方法.pdf
  modified: 2026-08-19
---

# 第2章：Protection、Virtual Memory and Virtual Machines

> 来源：`books/计算机体系结构量化研究方法.pdf`，第5版，第2章 §2.4–§2.5。
> 本批学习范围：PDF p.133–140；书内 p.105–112。PDF p.141 / 书内 p.113 开始 §2.6。

## 1. 虚拟内存提供进程隔离

- 进程切换要求多个程序共享硬件，但不能互相读写状态。体系结构至少需要 user/supervisor 两种模式、受保护的处理器状态、系统调用/返回机制，以及限制内存访问的机制。
- 分页把虚拟地址空间划分为固定大小的 page，并通过 page table 映射到物理页；每个页表项可携带 user read/write/execute 等保护位。只有操作系统能修改页表，因此用户进程不能任意改变访问权限。
- 直接查页表会让一次内存访问至少增加一次地址转换访问。TLB 是地址转换的 cache：tag 保存虚拟页号，data 保存物理页号、保护位、valid/use/dirty 等状态。
- 修改页表中的权限或映射后，操作系统必须使对应 TLB 项失效，否则处理器可能继续使用旧的转换与保护信息。

## 2. 为什么需要虚拟机

- 虚拟机把同一硬件抽象成多个相互隔离的完整系统环境；VMM/hypervisor 控制 host 的物理资源，并把它们映射给 guest VM。
- VMM 既提供接近原生 ISA 的环境，又必须隔离 guest、保护自身，并尽量让普通指令直接在硬件上执行。
- 处理器密集型、很少进入操作系统的程序可能几乎没有虚拟化开销；I/O 密集型程序因频繁系统调用、特权指令和设备访问，通常开销更大。
- VM 的商业价值不仅是安全隔离，还包括运行旧软件、合并原本需要多台服务器的软件栈，以及在负载均衡或硬件故障时迁移运行中的 VM。这也是云服务器大量使用 VM 的原因。

## 3. VMM 的体系结构要求

- guest 软件应表现得像运行在真实硬件上，除性能和共享资源限制外不应感知差异；guest 不能直接改变真实资源分配。
- VMM 必须控制特权状态、地址转换、I/O、异常和中断。比如 timer interrupt 到达时，VMM 保存当前 guest 状态、调度下一个 guest，并向 guest 提供虚拟定时器与模拟中断。
- ISA 至少需要 user/system 模式和只在 system 模式可执行的特权指令；guest 执行敏感指令时应 trap 到 VMM。
- 若 ISA 在设计时就考虑虚拟化，敏感指令可直接 trap 且高效模拟，称为 virtualizable。早期 80x86 和多数 RISC ISA 缺少充分支持，因此需要额外软件技巧或硬件扩展。

## 4. 虚拟化地址转换、TLB 与 I/O

- 传统 VM 中，guest OS 维护 guest virtual→guest physical 映射，而 VMM 还需维护 guest physical→machine physical 映射，形成额外开销。
- 一种办法是 shadow page table：VMM 根据 guest 页表构造直接映射到真实物理页的影子页表，并通过写保护/捕获页表修改来保持一致。
- 另一种思路是在硬件中加入额外地址转换层，让 guest 页表保持原样，由 VMM 管理第二级映射；这可减少 shadow page table 的维护。
- TLB 虚拟化可由 VMM 管理每个 VM 的 TLB 状态；带 Process ID/ASID 标签的 TLB 能混合保存多个 VM 与 VMM 的项，减少 VM 切换时的整体 flush。
- I/O 是最难虚拟化的部分：设备种类多、多个 VM 需要共享同一设备、不同 guest OS 还可能需要不同驱动。物理磁盘可被切分成 virtual disks，网络接口则按虚拟地址和时间片共享。

## 5. Xen 与 paravirtualization

- 当 guest OS 允许少量修改、主动避开难以虚拟化的 ISA 行为时，称为 paravirtualization。Xen 是典型例子。
- Xen 把自身映射到每个 VM 地址空间的高端区域，以减少 TLB flush；在 80x86 上利用多个 privilege level，让 Xen、guest OS、应用处于不同权限层。
- Xen 通过 driver domains 管理 I/O：特权 VM 运行真实设备驱动，普通 guest domain 使用虚拟驱动，通过通道和 page remapping 访问设备。
- 这种方法牺牲了 guest OS 完全不修改的要求，但能减少 VMM 模拟路径和虚拟化开销；应用二进制接口不必改变。

## 6. 缓存一致性导论

- 同一数据可能同时存在于内存和 cache；单处理器且只有处理器访问时，cache 通常能维持正确副本。
- 多处理器会让同一数据出现在多个 cache 中，I/O 设备也可能直接更新主存，从而产生 stale data。多处理器共享数据的一致性会直接影响程序性能。
- 对 I/O，常见策略是让设备直接访问主存，把主存作为 I/O buffer；软件可将输入 buffer 标为 noncacheable 或在 I/O 前 flush cache，硬件也可检测输入地址并使匹配 cache 项失效。
- write-through 能让主存保持最新副本，但现代系统通常只在 L1 D-cache 使用 write-through，后级 cache 多采用 write-back，因此输入/输出仍需额外一致性处理。

## 设计要点

- 虚拟内存的核心是“页级权限 + 地址重映射 + TLB 加速”；虚拟机进一步把 guest 的物理资源也变成可控的虚拟资源。
- VM 的性能关键不只是执行普通指令的速度，更取决于特权指令、地址转换、TLB、异常、中断和 I/O 的 trap/模拟成本。
- 保护和性能是一组共同设计问题：更严格的隔离需要更多转换与检查，但 TLB、硬件虚拟化扩展、页表层次和 paravirtualization 可以把成本压低。

## 下一断点

PDF p.141 / 书内 p.113：继续 §2.6 `Putting It All Together: Memory Hierarchies in the ARM Cortex-A8 and Intel Core i7`，先完成 Cortex-A8 的层次结构，再分析 Intel Core i7。
