---
name: knowledge_pcie_chapter_6_flow_control_complete
description: PCIe第6章完整学习：Credit流控、VC Buffer、初始化、运行机制、示例、UpdateFC DLLP、更新频率和超时。
---

# 学习范围

- 文档：PCI Express Technology 3.0 中文版
- 位置：第6章§6.1–§6.7.3，PDF p.222–254
- 源文本约24,388字符；在80,000字符上限内完成整章，符合“优先学完完整章节”规则。

# 一、流量控制目标与分层职责

PCIe Flow Control 确保发送端在接收端拥有足够 Buffer 时才发送 TLP，避免接收 Buffer 溢出。与 PCI 的 Delayed Transaction/Retry 模型不同，PCIe 在发送前通过 Credit 判断可接收能力，减少断开、重试和等待态。

- 每条链路两端端口都必须实现流量控制；它是相邻链路、逐 VC 的本地机制，不是端到端带宽保证。
- 每个 VC 的事务流独立；一个 VC Buffer 满不会阻塞其他 VC，PCIe 最多支持 VC0–VC7。
- Transaction Layer 维护发送/接收 Credit Counter，Data Link Layer 发送/接收携带 Credit 的 DLLP，二者共同实现 Flow Control。

# 二、Credit 与 VC Buffer

每个启用 VC 按事务类别维护独立 Buffer：

- Posted：Memory Write、Message；
- Non-Posted：Memory Read、Configuration、IO；
- Completion：读/写 Completion。

每类又分 Header 和 Data，形成 PH、PD、NPH、NPD、CplH、CplD 六类 Credit。只有 Header 的读请求只消耗 Header Credit；写请求、带数据 Message 和带数据 Completion 必须同时拥有 Header 与 Data Credit。事务在 VC Buffer 内保持顺序，向软件或 Switch 出口转发时不能任意重排。

## Credit 单位

- Header Credit：按最大 Header + Digest 计；请求 Header 可按5DW，Completion Header 可按4DW。
- Data Credit：1 Credit=4DW=16 byte，对齐到16 byte。
- DLLP 自身不消耗 Transaction Layer Credit，因为 DLLP 在 Data Link Layer 生成/终止。

# 三、初始流控通告

链路初始化时，接收端把各 VC、各事务类别的 Buffer 大小通告给对端。发送端把通告写入本地 Credit Counter，只能发送不超过已知额度的 TLP。

## 最小/最大通告

- PH/NPH 最小通常为1个 Header Credit；PD 最小至少覆盖设备允许的最大 Payload；NPD 至少能容纳合法请求/AtomicOp需求。
- CplH/CplD 对支持 Peer-to-Peer 的 RC/Switch 有有限最小值；不发起事务的 RC/EP 可对 Completion 类通告 Infinite Credit。
- 书中给出的典型最大值：PH/NPH 128 Credits；PD/CplD 可到2048 Credits（对应多 Function、最大 Payload 的总和）；具体值受设备能力和规范字段宽度约束。

## Infinite Credit

Credit=00h 表示 Infinite Credit。通告后该类别不再通过普通更新维持有限计数，但发起事务的设备仍必须保留足够 Buffer 接收拆分事务返回的 Completion 数据/状态。只有实现 VC0 时，VC1–VC7 某些未使用 Data 类可以通告 Infinite，但仍需维护实际使用的 Header Credit。

# 四、流控初始化（FC_Init）

1. 链路进入可用状态后，双方交换 FC_Init1，建立各 VC 的初始 Header/Data Credit。
2. 双方解析并确认 FC_Init1 后交换 FC_Init2，完成初始化状态。
3. 初始化完成前不能按正常 TLP 流量使用尚未建立的 Credit；非法 Credit、顺序或字段会构成协议违例。
4. 接收端的 Credits Received（CR）不能超过其 Credits Allocated（CA）；CA/CR 关系用于检测 Buffer 溢出。

初始化不仅交换“Buffer 有多大”，还建立了之后 UpdateFC 的计数基准。Credit Counter 的回绕和字段宽度必须按 Header/Data 的不同规则处理。

# 五、运行期流控机制（§6.5）

发送端在 TLP 离开 Transaction Layer 前检查对应 VC 和事务类别的 Header/Data Credit。Credit 足够时扣减本地计数并发送；不足时只阻塞该类别/VC，其他有 Credit 的 VC 仍可发送。接收端消费 Buffer 后增加 CA，并通过 UpdateFC DLLP 把实际累计 Credit 值报告给对端。

- PH/PD、NPH/NPD、CplH/CplD 分别计数，不能用一种 Credit 代替另一种。
- 发送端必须同时满足 Header 和 Data Credit；如果其中任一不足，整个 TLP 不能发送。
- Credit 更新和 TLP 发送存在延迟；实现应预留保守余量，避免边界条件下超发。
- 流控只保护 Buffer 容量，不负责物理错误恢复；LCRC、Ack/Nak、Replay 负责链路错误。

# 六、流控示例与溢出检查（§6.6）

书中示例展示四个阶段：

1. 初始化后，发送端依据初始 Credit 发送，接收端消费并逐步增加 CA。
2. Buffer 接近/达到满状态，发送端因 Credit 不足停发对应 TLP，但其他 VC/类别仍可继续。
3. Credit Counter 采用有限位宽时可能回绕为0；发送端必须使用协议规定的模计数比较，不能把0简单理解为无 Credit（初始化通告中的0才表示 Infinite）。
4. 接收端比较 CA、CR 和字段宽度检测溢出；Header 字段和 Data 字段的 Field Size 不同，书中指出 PCIe 1.0a 曾把等号误写为溢出条件，正确判断应区分 CA=CR（未溢出）与真正超过可用范围。

溢出是严重协议错误：它意味着发送方获知的额度不足以解释已进入 Buffer 的事务，可能导致数据丢失或链路恢复。

# 七、UpdateFC DLLP（§6.7）

当接收端从 Buffer 移除 TLP、释放空间时，CA 增加。接收端在 UpdateFC DLLP 中报告新的实际 CA/CREDIT_LIMIT 值。书中特别强调报告累计实际值而不是增量：如果某个 DLLP 丢失，下一次累计值仍能重新同步；若只报告“+3”，丢包后双方会永久失步。

UpdateFC 携带 Header Credit 和 Data Credit 字段，可同时报告多类 VC/事务更新。发送端收到并校验 DLLP 后更新本地可用 Credit，解除因 Buffer 满造成的发送阻塞。

## 更新时机与频率

- 当 Buffer 满到连最大包都无法接收、随后释放至少一个必需 Credit 时，应立即调度 UpdateFC。
- 对非 Infinite Credit 类别，最大 UpdateFC 间隔通常为30 μs（-0%/+50%）；Control Link Register 的 Extended Sync 可能放宽到120 μs（-0%/+50%）。
- 更新只在链路 L0/L0s active 状态发送；更积极的低功耗状态有更长恢复延迟。
- 频率公式考虑 Max_Payload_Size、TLP Overhead、UpdateFactor、LinkWidth 和 InternalDelay。书中示例：Gen1、Max Payload=256、x2，计算约217.8 Symbol Time，对应表值217。Gen1/2/3 的表格值不能跨速率直接复用。
- 应用若发送大块连续数据，可能需要大于规范最小值的 Buffer 或更复杂更新策略，以平衡吞吐和功耗。

# 八、流控错误检测计时器（§6.7.3）

规范建议为每种有限 Credit 类型设置独立流控包超时计时器。两个流控包的最大间隔可按120 μs级别检查，超时界限约200 μs（-0%/+50%）；Infinite Credit 类别不能报告该超时错误。

- 计时器只在 L0/L0s active 状态工作。
- 收到 Init 或 UpdateFC 包时复位；实现也可选择收到任意 DLLP 时复位，但需避免掩盖真实流控失联。
- 超时意味着链路可能严重故障，物理层被通知进入 LTSSM Recovery 并重新训练。
- 流控超时不是普通 TLP UR；它表示 Credit 同步或链路维护失败，应按链路恢复/错误报告处理。

# 适用条件与例外

- Credit=00h 的 Infinite 语义只适用于初始化通告和允许的类别；计数器回绕过程中的0不能直接套用该语义。
- UpdateFC 是链路本地 DLLP，不由 Ack/Nak Replay；丢失依靠下一次累计 CA 重新同步。
- 流控规则与 QoS/VC/排序相互影响，但流控本身不提供确定性带宽保证。

# 关联章节

- 第5章TLP；第7章QoS与仲裁；第8章事务排序；第9章DLLP；第10章Ack/Nak。

# 待核验问题

- 无。本文件已覆盖第6章§6.1–§6.7.3，PDF p.222–254；下一章为第7章。
