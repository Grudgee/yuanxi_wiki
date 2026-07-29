# AXI Request Attributes 总结

> 本文整理自 Arm《AMBA AXI and ACE Protocol Specification》中的 **A4 Transaction Attributes**，主要介绍普通 AXI 请求的 `AxCACHE` 和 `AxPROT` 属性。
>
> 本地参考文档：[`amba_prot/AXIACE5.pdf`](amba_prot/AXIACE5.pdf)，Chapter A4，约 A4-61～A4-77。

## 1. 概述

AXI Request Attributes 用于描述一次读写请求的内存、缓存、权限和安全属性，核心信号包括：

| 信号 | 通道 | 作用 |
|---|---|---|
| `ARCACHE[3:0]` | AR 读地址通道 | 描述读事务的缓冲、修改、Cache lookup 和分配属性 |
| `AWCACHE[3:0]` | AW 写地址通道 | 描述写事务的缓冲、修改、Cache lookup 和分配属性 |
| `ARPROT[2:0]` | AR 读地址通道 | 描述读事务的权限、安全状态和访问类型 |
| `AWPROT[2:0]` | AW 写地址通道 | 描述写事务的权限、安全状态和访问类型 |

其中，`AxCACHE` 是 `ARCACHE` 和 `AWCACHE` 的统称，`AxPROT` 是 `ARPROT` 和 `AWPROT` 的统称。

这些属性并不只是性能提示，它们还会影响：

- 事务能否拆分或合并；
- 写响应能否在事务到达最终目标前返回；
- 读数据是否必须来自最终目标；
- 是否必须查询 Cache；
- 是否允许预取或写合并；
- Interconnect 是否可以改变 burst 形态；
- Subordinate 是否允许该权限和安全域的访问。

---

## 2. AxCACHE 的基本含义

### 2.1 AXI3 定义

AXI3 中，`AxCACHE[3:0]` 的含义如下：

| 位 | 名称 | `0` | `1` |
|---|---|---|---|
| `[0]` | Bufferable | Non-bufferable | Bufferable |
| `[1]` | Cacheable | Non-cacheable | Cacheable |
| `[2]` | Read-Allocate | No Read-Allocate | Read-Allocate |
| `[3]` | Write-Allocate | No Write-Allocate | Write-Allocate |

关键规则：

- `Cacheable=0` 时，Read-Allocate 和 Write-Allocate 不应置位；
- Bufferable 表示中间节点可以缓冲事务；
- Cacheable 表示允许缓存、预取和写合并等行为；
- Allocate 是缓存分配建议，不是强制命令。

参考：§A4.2，表 A4-1，p. A4-63。

### 2.2 AXI4 的变化

AXI4 将 `AxCACHE[1]` 从 **Cacheable** 更名为 **Modifiable**：

```text
AxCACHE[1] = 0  → Non-modifiable
AxCACHE[1] = 1  → Modifiable
```

这强调该位不仅关系到是否缓存，还决定 Interconnect 是否可以改变事务的传输形态。

---

## 3. Non-modifiable 与 Modifiable

### 3.1 Non-modifiable 事务

当 `AxCACHE[1]=0` 时，Interconnect 通常不得：

- 将一个事务拆成多个事务；
- 将多个事务合并；
- 修改 `AxADDR`；
- 修改 `AxSIZE`；
- 修改 `AxLEN`；
- 修改 `AxBURST`；
- 修改 `AxLOCK`；
- 修改 `AxPROT`。

允许的有限变化包括：

- `AxID` 和 `AxQOS` 可以改变；
- `AxCACHE` 可以从 Bufferable 改成 Non-bufferable；
- 长度超过 16 beats 的 burst 可以拆分，并相应调整地址和长度；
- Exclusive access 可以调整 `AxSIZE` 和 `AxLEN`，但总传输字节数必须保持不变。

因此，数据宽度转换器、burst splitter 等模块不能无条件修改 Non-modifiable 事务。

### 3.2 Modifiable 事务

当 `AxCACHE[1]=1` 时，Interconnect 可以：

- 拆分事务；
- 合并事务；
- 修改 `AxADDR`、`AxSIZE`、`AxLEN` 和 `AxBURST`；
- 为读请求获取比原请求更多的数据，例如预取；
- 扩大写事务的传输覆盖范围，但必须通过 `WSTRB` 保证仅修改原请求中的有效字节。

即使事务是 Modifiable，仍不得：

- 修改 `AxLOCK` 和 `AxPROT`；
- 将事务修改到另一个 4KB 地址空间；
- 破坏 single-copy atomicity；
- 通过修改属性降低事务的可见性要求；
- 取消原本必须进行的 Cache lookup 或最终目标传播。

参考：§A4.3.1，表 A4-2，pp. A4-64～A4-65。

---

## 4. Allocate 与 Other Allocate

AXI4 中，`AxCACHE[3:2]` 表示请求是否必须进行 Cache lookup，以及某地址是否可能已经因为某类访问而进入 Cache。

读写通道的位定义不同：

| 通道 | `bit[3]` | `bit[2]` |
|---|---|---|
| `AWCACHE` | Allocate | Other Allocate |
| `ARCACHE` | Other Allocate | Allocate |

共同规则：

```text
AxCACHE[3:2] != 2'b00  → 必须进行 Cache lookup
AxCACHE[3:2] == 2'b00  → 不要求 Cache lookup，事务应传播到最终目标
```

含义如下：

- **Allocate**：该类读或写事务可能使对应地址在 Cache 中得到分配，同时给出建议分配的性能提示；
- **Other Allocate**：该地址可能已经因为另一类事务或另一 Manager 的操作而存在于 Cache 中。

注意：

1. Allocate 只是建议，并不保证一定分配 Cache line；
2. No-Allocate 也不严格禁止实现进行分配；
3. `ARCACHE` 和 `AWCACHE` 不一定相同，不能机械地使用相同编码；
4. `AxCACHE[3:2]` 要求的 Cache lookup 不等同于 ACE snoop。

参考：§A4.3.2，表 A4-3、A4-4，pp. A4-65～A4-68。

---

## 5. 常见 Memory Type

### 5.1 编码速查

| Memory type | `ARCACHE` | `AWCACHE` | 主要特征 |
|---|---:|---:|---|
| Device Non-bufferable | `0000` | `0000` | 不可修改，读写必须到最终目标 |
| Device Bufferable | `0001` | `0001` | 不可修改，写响应可以提前返回 |
| Normal Non-cacheable Non-bufferable | `0010` | `0010` | 可修改，读写由最终目标完成 |
| Normal Non-cacheable Bufferable | `0011` | `0011` | 可修改，写可以缓冲并提前响应 |
| Write-Through No-Allocate | `1010` | `0010` | 必须查询 Cache，写及时传播到最终目标 |
| Write-Through Read-Allocate | `1110` | `0010` | 建议读分配 |
| Write-Through Write-Allocate | `1010` | `1110` | 建议写分配 |
| Write-Through Read/Write-Allocate | `1110` | `1110` | 建议读写均分配 |
| Write-Back No-Allocate | `1011` | `0011` | 写可以停留在中间 Cache |
| Write-Back Read-Allocate | `1111` | `0011` | 建议读分配 |
| Write-Back Write-Allocate | `1011` | `1111` | 建议写分配 |
| Write-Back Read/Write-Allocate | `1111` | `1111` | 建议读写均分配 |

未在规范表格中定义的组合属于保留编码，不应自行使用。

参考：§A4.4，表 A4-5，p. A4-69。

### 5.2 Device Non-bufferable

```text
ARCACHE = 4'b0000
AWCACHE = 4'b0000
```

特点：

- 读数据必须来自最终目标；
- 写响应必须由最终目标返回；
- 不允许读预取；
- 不允许写合并；
- 事务不可修改。

这类属性适合要求明确完成点和严格副作用语义的外设访问。

### 5.3 Device Bufferable

```text
ARCACHE = 4'b0001
AWCACHE = 4'b0001
```

特点：

- 写响应可以由中间缓冲位置返回；
- 写仍必须及时传播到最终目标；
- 读数据仍必须来自最终目标；
- 不允许读预取和写合并；
- 事务不可修改。

收到 Device Bufferable 写事务的 B 响应，并不表示写已经到达最终外设。

### 5.4 Normal Non-cacheable

典型编码：

```text
Non-bufferable: ARCACHE/AWCACHE = 4'b0010
Bufferable:     ARCACHE/AWCACHE = 4'b0011
```

特点：

- 事务可以被修改；
- 写事务可以合并；
- Bufferable 版本允许写响应提前返回；
- 读请求可以从正在向最终目标传播的写事务中取得最新数据；
- 不能建立一个长期服务后续读请求的 Cache 副本。

### 5.5 Write-Through

特点：

- 读写都必须进行 Cache lookup；
- 读可以从 Cache 副本返回；
- 写必须及时传播到最终目标；
- 写响应可以由中间点提前返回；
- 允许读预取和写合并。

### 5.6 Write-Back

特点：

- 读可以从中间 Cache 返回；
- 写响应可以由中间 Cache 返回；
- 写不要求在确定时间内到达最终目标；
- 允许读预取、写合并和缓存分配。

因此，Write-Back 写事务收到响应，不代表数据已经写入 DRAM 或其他最终存储位置。

参考：§A4.4.1，pp. A4-69～A4-72。

---

## 6. Bufferable 的完成和可见性

Bufferable 允许写响应在事务到达最终目标前返回，但不表示事务可以无限期停留在中间节点。

对于 Device Bufferable、Normal Non-cacheable Bufferable 和 Write-Through 写事务：

- 写响应可以提前返回；
- 写必须及时向最终目标推进；
- Interconnect 不能因为持续合并或新事务到来而使某笔事务无限期滞留。

规范没有给“及时”规定统一的周期数，因此实际延迟界限需要由具体系统架构确定。

参考：§A4.6，p. A4-74。

### 6.1 使用 Non-bufferable 写确认之前的写

一种常见外设访问流程是：

1. 发出一个或多个 Device Bufferable 写；
2. 向同一个 Subordinate 发出 Device Non-bufferable 写；
3. 等待后一个 Non-bufferable 写的响应。

当这些事务使用相同 AXI ID，并且访问同一个 Subordinate 时，后一个 Non-bufferable 写收到响应后，可以确认此前同 ID 的 Bufferable 写已经到达最终目标，并对其他 Manager 可见。

该保证不能推广到：

- 不同 AXI ID；
- 不同 Subordinate；
- 全系统范围的通用内存屏障。

参考：§A4.9.1，p. A4-77。

---

## 7. AxPROT：访问保护属性

`AxPROT[2:0]` 描述访问权限、安全状态和访问类型：

| 位 | `0` | `1` |
|---|---|---|
| `AxPROT[0]` | Unprivileged | Privileged |
| `AxPROT[1]` | Secure | Non-secure |
| `AxPROT[2]` | Data access | Instruction access |

对应关系：

```text
AxPROT[0] = 1  → Privileged
AxPROT[1] = 1  → Non-secure
AxPROT[2] = 1  → Instruction access
```

使用要点：

- AXI 协议只区分 Privileged 和 Unprivileged 两级；
- 处理器内部更细的权限级别如何映射到 `AxPROT`，由实现定义；
- `AxPROT[1]=1` 表示 **Non-secure**，这是常见易错点；
- `AxPROT[2]` 是提示属性；不能确认访问类型时，通常按 Data access 设置；
- Interconnect 或 Subordinate 可以依据 `AxPROT` 执行访问权限检查。

参考：§A4.7，表 A4-6，p. A4-75。

---

## 8. 多个 Manager 的属性一致性

多个 Manager 访问同一地址区域时，不能任意使用不兼容的 Cache 属性。

基本要求：

- 不可缓存区域：所有 Manager 都必须使用 `AxCACHE[3:2]==2'b00`；
- 可缓存区域：所有 Manager 都必须至少设置 `AxCACHE[3:2]` 中的一个位；
- Normal Non-cacheable 区域可以使用更严格的 Device 属性访问；
- 允许 Bufferable 的区域可以使用更严格的 Non-bufferable 属性访问。

如果需要把某个区域切换为不兼容的新属性，典型流程是：

1. 停止所有 Manager 对该区域的访问；
2. 由一个 Manager 执行必要的 Cache maintenance；
3. 所有 Manager 使用新属性恢复访问。

参考：§A4.5，p. A4-73。

---

## 9. 普通 AXI 与 ACE 的边界

本章属性属于普通 AXI 事务属性，主要包括：

- `ARCACHE` / `AWCACHE`；
- `ARPROT` / `AWPROT`；
- Memory type；
- Bufferable；
- Modifiable；
- Cache lookup 和 Allocate hint。

在 ACE 系统中，这些属性仍然存在，但完整的一致性行为还需要结合：

- `ARSNOOP` / `AWSNOOP`：一致性操作类型；
- `ARDOMAIN` / `AWDOMAIN`：一致性作用域；
- Barrier、DVM 和相关一致性机制。

因此：

- `AxCACHE[3:2]!=0` 表示必须 Cache lookup，但不等于执行 ACE snoop；
- 不能只依据 `AxCACHE` 判断一个 ACE 请求是否需要查询或失效其他 Cache；
- ACE 请求是否合法以及实际产生什么一致性效果，需要结合 `AxSNOOP` 和 `AxDOMAIN` 判断。

---

## 10. 易错点速查

1. **Bufferable 不等于写已到最终目标。** 它允许中间节点提前返回写响应。
2. **Allocate 不等于强制缓存分配。** 它只是性能建议。
3. **No-Allocate 不等于禁止分配。** 实现仍可能分配 Cache line。
4. **Non-modifiable 不等于任何信号都不能改变。** `AxID`、`AxQOS` 和少数特殊情况仍允许变化。
5. **`ARCACHE` 和 `AWCACHE` 不一定相同。** AXI4 中 Allocate 与 Other Allocate 的位含义在读写通道上不同。
6. **`AxCACHE[3:2]` 不只是“是否可缓存”。** 它还决定是否必须 Cache lookup。
7. **Write-Back 响应不表示数据已经到达 DRAM。** 数据可能仍停留在中间 Cache。
8. **`AxPROT[1]=1` 表示 Non-secure。** 不要反向理解。
9. **同地址的多个 Manager 必须保持兼容的内存属性视图。**
10. **普通 AXI 的 `AxCACHE` 不能替代 ACE 的一致性属性。**

---

## 11. 参考章节

| 内容 | 规范位置 |
|---|---|
| Transaction attributes 概述 | §A4.1，p. A4-62 |
| AXI3 `AxCACHE` 定义 | §A4.2，表 A4-1，p. A4-63 |
| AXI4 Modifiable 规则 | §A4.3.1，表 A4-2，pp. A4-64～A4-65 |
| Allocate / Other Allocate | §A4.3.2，表 A4-3、A4-4，pp. A4-65～A4-68 |
| Memory type 编码与行为 | §A4.4，表 A4-5，pp. A4-69～A4-72 |
| 多 Manager 属性要求 | §A4.5，p. A4-73 |
| Bufferable 传播要求 | §A4.6，p. A4-74 |
| `AxPROT` | §A4.7，表 A4-6，p. A4-75 |
| Device Bufferable 写完成确认 | §A4.9.1，p. A4-77 |

主要来源：Arm IHI 0022H.c，*AMBA AXI and ACE Protocol Specification*。
