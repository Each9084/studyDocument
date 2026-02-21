# Hadoop

## 一 Hadoop介绍

**HDFS** （**Hadoop Distributed File System**）是 Hadoop 下的分布式文件系统，具有高容错、高吞吐量等特性，可以部署在低成本的硬件上。

它的核心只有三块：**存数据的 (HDFS)**、**算数据的 (MapReduce)**、**管资源的 (YARN)**。





<img src="assets/dataDevAssets/1.HDFS_ARCHITECTURE.png" width="35%" style="display: block; margin: 0 auto;"><img src="assets/dataDevAssets/1.hdfs_component.png" width="65%" style="display: block; margin: 0 auto;">





在大数据面试和实战中，需要从**策略、过程、放置逻辑、以及自愈**四个维度来构建知识体系。

#### 1. HDFS 的副本放置策略 (Replica Placement Policy)

这是最常被问到的考点：HDFS 并不是随机乱放副本的，它有一套**兼顾“安全”与“效率”**的算法。

**默认副本数为 3 的放置逻辑：**

- **第 1 个副本**：放在上传数据的那个 DataNode 上（如果是集群外提交，则挑一台磁盘不太满、CPU 不太忙的节点）。
- **第 2 个副本**：放在与第 1 个副本**不同机架**的某个节点上（保证即使整个机架的交换机坏了，数据还在）。
- **第 3 个副本**：放在与第 2 个副本**相同机架**但不同的节点上。

> **逻辑核心**：这样配置既保证了**机架感知（Rack Awareness）**的安全性，又在第 2 和第 3 副本之间节省了跨机架的网络带宽。



#### 2. 写数据的“流水线”复制 (Replication Pipeline)

很多人误以为是 NameNode 负责把数据复制三份，**其实不是**。

当你要往 HDFS 写一个 128MB 的块时：

1. **建立管道**：客户端向 NameNode 申请写，NameNode 返回 3 台 DataNode 地址（A, B, C）。
2. **流水线传输**：客户端只把数据发给 **A**；A 收到一部分（Packet）后，立刻传给 **B**；B 再传给 **C**。
3. ** ACK 确认**：当 C 写完返回确认给 B，B 返回给 A，最后 A 告诉客户端：“写好了”。

这种**串行流水线**的方式比起“由客户端同时向三台机器发三份数据”，极大地节省了客户端的出口带宽。



#### 3. 副本自愈：如果数据坏了怎么办？

Hadoop 体系中有一套自动化的审计和修复机制：

- **数据校验和 (Checksum)**： DataNode 在存数据时会产生一个校验码。当你（或 Hive）读取数据时，客户端会重新计算校验码，如果对不上，说明数据损坏（由于磁盘磁道坏了等原因）。
- **定期心跳 (Heartbeat)**： DataNode 每 3 秒给 NameNode 发一次心跳。如果 NameNode 超过 10 分钟没收到某台机器的心跳，就会认为它“挂了”。
- **自动补全**： NameNode 发现某块数据的副本数从 3 掉到了 2，它会下令让剩下的 DataNode 把数据复制到另一台健康的机器上。



### 4. 体系扩展：不仅仅是 HDFS

在你的大数据体系中，复制不仅发生在 HDFS 层，还发生在以下地方：

| **层次**                 | **复制内容**   | **作用**                                                     |
| ------------------------ | -------------- | ------------------------------------------------------------ |
| **元数据层 (MySQL)**     | Hive 的元数据  | 生产环境必须做 **MySQL 主从复制**，否则数据库挂了，整个 Hive 就瘫痪了。 |
| **计算中间层 (Shuffle)** | Map 输出的数据 | 在 Shuffle 阶段，数据会从 Map 节点“复制”到 Reduce 节点。**数据倾斜**往往就是这里复制的数据量不均导致的。 |
| **高可用层 (ZooKeeper)** | ZK 的状态数据  | 采用 **Paxos/ZAB 协议**的强一致性复制，保证 HS2 高可用时的信息同步。 |



## 二 HDFS基本架构及设计原理

HDFS 解决了“一台机器装不下，几千台机器怎么存”的问题。

核心架构：Master-Slave (主从) 模式 (中心化模式)



<img src="assets/dataDevAssets/1.Master%20Slave.jpg" width="65%" style="display: block; margin: 0 auto;">

### 2.1 HDFS 架构

HDFS 遵循主/从架构，由单个 NameNode(NN) 和多个 DataNode(DN) 组成：

- **NameNode** : 负责执行有关 `文件系统命名空间` 的操作，例如打开，关闭、重命名文件和目录等。它同时还负责集群元数据的存储，记录着文件中各个数据块的位置信息。
- **DataNode**：负责提供来自文件系统客户端的读写请求，执行块的创建，删除等操作。



**NameNode (老大):**负责管理

- HDFS系统的主角色，是一个独立的进程
- 负责管理HDFS整个文件系统
- 负责管理DataNode



- 它不存真实数据，只存**元数据**（文件名、目录结构、文件被切成了几块、分别在哪台机器上）。

- <span style="color:green">*就像图书馆的索引卡。*</span>



**DataNode (小弟): **负责干活

- HDFS系统的从角色，是一个独立进程
- 主要负责数据的存储，即存入数据和取出
  数据



- 真实的数据块（Block）存在这里。
- <span style="color:green">*就像图书馆的书架。*</span>



**SecondaryNameNode:** 助教

- NameNode的辅助，是一个独立进程
- 主要帮助NameNode完成元数据整理工作（打杂）



- 帮老大分担压力，合并日志，防止老大挂了后启动太慢。



**核心机制：副本策略**

- **数据切块**：文件会被切成固定大小（默认 128MB）。
- **多副本**：默认每个块存 3 份。
  - *这就是为什么你在 HDFS 上删了文件，空间没立刻释放，或者挂了一台机器数据不丢失的原因。*



### 2.2 分布式系统中两种常见的工作模式

#### 2.2.1.分散汇总模式 (Scatter-Gather)

分散汇总模式（也称为 **分发-收集** 模式）是一种天然的并行计算模式，常用于需要并行处理大量数据的场景，例如 **MapReduce**。

| 步骤        | 名称                 | 执行动作                                                     | 关键特点                                                     |
| ----------- | -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 步骤 1      | 分散 (Scatter/Map)   | 主控节点（Master/Client）将一个大型任务或数据集 切分成若干独立的子任务或数据块，并将它们并行地发送给多个 工作节点（Worker/Slave）。 | 并行化：子任务之间相互独立，可以在不同的机器上同时运行。     |
| 步骤 步骤 2 | 执行 (Execute)       | 各个 工作节点 接收到自己的子任务后，独立地对分配到的数据块进行处理（例如，执行 Map 操作，数据清洗）。 | 无状态/本地计算：每个工作节点只关心自己的数据，通常不需要与其他节点通信。 |
| 步骤 3      | 汇总 (Gather/Reduce) | 各个 工作节点 完成计算后，将各自的 局部结果 返回给 主控节点 或一个指定的 汇集节点。 | 数据传输：涉及大量的网络 I/O，可能包含 Shuffle 过程（如果需要合并/分组）。 |
| 步骤 步骤 4 | 合并 (Merge)         | 主控节点/汇集节点 接收所有局部结果，执行最终的 合并、聚合或排序（例如，执行 Reduce 操作），得到最终的完整结果。 | 终态生成：生成用户所需的最终输出。                           |

**通俗总结：** 任务可以被完全切开，每个工人独立完成一部分，最后把各自的结果交上来，总指挥只负责最后相加。**速度快，但任务必须是可分割的。**



#### 2.2.2.🚦 中心调度模式 (Centralized Orchestration)

中心调度模式（也常被称为 **编排** 模式）强调由一个**中心协调者** 来定义、管理和推动整个工作流的顺序和依赖关系。它适用于涉及多个、有明确先后顺序和依赖关系的服务调用或任务。

| 步骤        | 名称                             | 执行动作                                                     | 关键特点                                                 |
| ----------- | -------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------- |
| 步骤 1      | 定义工作流 (Workflow Definition) | 调度中心（Orchestrator/Master）预先加载或定义一个 有向无环图 (DAG)，明确所有任务（Task A, B, C...）的执行顺序和依赖关系。 | 任务依赖：任务 B 只有在任务 A 成功完成后才能开始。       |
| 步骤 步骤 2 | 启动与监控 (Trigger & Monitor)   | 调度中心 启动第一个 无依赖 的任务（如 Task A）。在任务执行过程中，调度中心会持续监控 任务的状态（运行中、成功、失败）。 | 状态管理：调度中心持有整个工作流的全局状态。             |
| 步骤 3      | 顺序驱动 (Sequential Drive)      | 当 调度中心 确认一个任务（如 Task A）成功完成 后，它会根据 DAG 触发其所有 下游依赖 的任务（如 Task B）。 | 单一决策点：所有任务的执行或重试都由调度中心决定和驱动。 |
| 步骤 步骤 4 | 反馈与结束 (Feedback & Complete) | 整个工作流直到所有任务都按顺序成功执行完毕，调度中心 宣布工作流完成。如果任何任务失败，调度中心负责根据策略进行 重试 或 告警。 | 流程控制：确保复杂业务流程的完整性、顺序性和可控性。     |

**通俗总结：** 任务之间有严格的 **先后顺序** 和 **依赖关系**。有一个 **总指挥（调度中心）** 像交警一样，严格控制和检查每个步骤是否完成，确保整个流水线顺利、按部就班地走下去。



### 2.3 文件系统命名空间

HDFS 的 `文件系统命名空间` 的层次结构与大多数文件系统类似 (如 Linux)， 支持目录和文件的创建、移动、删除和重命名等操作，支持配置用户和访问权限，但不支持硬链接和软连接。`NameNode` 负责维护文件系统名称空间，记录对名称空间或其属性的任何更改。



### 2.4 数据复制

由于 Hadoop 被设计运行在廉价的机器上，这意味着硬件是不可靠的，为了保证容错性，HDFS 提供了数据复制机制。HDFS 将每一个文件存储为一系列**块**，每个块由多个副本来保证容错，块的大小和复制因子可以自行配置（默认情况下，块大小是 128M，默认复制因子是 3）。

<img src="assets/dataDevAssets/1.Duplicate.png" width="55%" style="display: block; margin: 0 auto;">



### 2.5 数据复制的实现原理

大型的 HDFS 实例在通常分布在多个机架的多台服务器上，不同机架上的两台服务器之间通过交换机进行通讯。在大多数情况下，同一机架中的服务器间的网络带宽大于不同机架中的服务器之间的带宽。因此 HDFS 采用机架感知副本放置策略，对于常见情况，当复制因子为 3 时，HDFS 的放置策略是：

在写入程序位于 `datanode` 上时，就优先将写入文件的一个副本放置在该 `datanode` 上，否则放在随机 `datanode` 上。之后在另一个远程机架上的任意一个节点上放置另一个副本，并在该机架上的另一个节点上放置最后一个副本。此策略可以减少机架间的写入流量，从而提高写入性能。

<img src="assets/dataDevAssets/1.Duplicate2.png" width="55%" style="display: block; margin: 0 auto;">

如果复制因子大于 3，则随机确定第 4 个和之后副本的放置位置，同时保持每个机架的副本数量低于上限，上限值通常为 `（复制系数 - 1）/机架数量 + 2`，需要注意的是不允许同一个 `dataNode` 上具有同一个块的多个副本。





对于以上内容 我们把这个过程拆解为三个核心逻辑：**就近原则**、**机架保命原则**和**冗余负载控制**,梳理一下:


**1. 基础逻辑：复制因子为 3 时（最经典情况）**

HDFS 就像一个聪明的快递员，他送货的逻辑是这样的：

- **第一步：就近（节省体力）**
  - 如果你就在仓库（DataNode）里操作，他就直接把第一个副本放在你脚下的机器上。
  - **目的**：本地写入速度最快，完全不占网络带宽。
- **第二步：跨机架保命（防灾）**
  - 他把第二个副本送到**另一个远程机架**。
  - **目的**：如果第一个机架的交换机坏了或停电了，第二个机架的数据还能用。这是为了**数据安全**。
- **第三步：同机架复制（省钱）**
  - 他在第二个机架里再找一台机器放下第三个副本。
  - **目的**：既然数据已经通过漫长的“跨机架”网络传到了机架 B，那么在机架 B 内部复制一次是非常快的，不需要再次跨越昂贵且拥挤的核心交换机。



**2.进阶逻辑：当复制因子 > 3 时（资源上限控制）**

这一部分也就是你提到的那个公式：`上限 = (复制系数 - 1) / 机架数量 + 2`。

你可以这样理解：**HDFS 严防“把鸡蛋放在同一个篮子里”。**

- **为什么要设上限？**

  如果你的副本数很多（比如 10 个），而你只有 3 个机架。如果 HDFS 偷懒把 8 个副本都塞进机架 A，那么机架 A 一旦断电，你瞬间就丢失了 80% 的备份，风险太高。

- **公式的意义**：它动态计算了每个机架能承受的“最大副本数”。

  - 比如：10 个副本，3 个机架。
  - 上限 $\approx (10 - 1) / 3 + 2 = 5$。
  - 这意味着没有任何一个机架可以存放超过 5 个副本。

- **禁忌**：最后那句“不允许同一个 DataNode 具有同一个块的多个副本”是死命令。哪怕你副本设为 100，只要你只有 10 台机器，你也只能存 10 份。





**3. 通俗总结：为什么要这么设计？**

这套体系的核心哲学是：**“大灾不灭，小灾不乱，平时飞快”。**

| **场景**     | **HDFS 的应对**                  | **结果**     |
| ------------ | -------------------------------- | ------------ |
| **平时写入** | 只需跨一次机架（1 次跨机架流量） | 写入性能极高 |
| **单机挂了** | 其他机器都有备份                 | 秒级恢复     |
| **整架挂了** | 另一个机架还有两份备份           | 业务不中断   |







<span style="color:red">**面试常考题：**</span>

> “如果用户在上传文件时，机架感知（Rack Awareness）没配置好，会发生什么？”
>
> **答案**：HDFS 会认为所有机器都在同一个机架上。这样它就会随机放副本。万一那个机架的交换机坏了，你的数据就彻底丢失了（Data Lost），即便你设了 3 副本也没用。







### 2.6 副本的选择

为了最大限度地减少带宽消耗和读取延迟，HDFS 在执行读取请求时，优先读取距离读取器最近的副本。如果在与读取器节点相同的机架上存在副本，则优先选择该副本。如果 HDFS 群集跨越多个数据中心，则优先选择本地数据中心上的副本。



这段话，揭示了大数据(分布式)处理中一个至高无上的准则：**移动计算，而不是移动数据（Move Code, Not Data）。**

在大数据环境下，由于数据量动辄几百 GB 甚至 TB，通过网络“搬运”数据是非常昂贵的。因此，HDFS 的读取策略本质上就是一个**“省体力”的选择过程**。

**1. 读取副本的“优先级（Distance）”算法**

HDFS 将节点间的距离量化，优先级由高到低排列如下：

1. **Node-Local（节点本地）**：
   - **场景**：计算任务（比如 Map 任务）恰好被分配到了存有该数据块的 DataNode 上。
   - **优势**：直接从本地磁盘读，**带宽消耗为 0**。
2. **Rack-Local（机架本地）**：
   - **场景**：同一个机架内，DataNode A 想读 DataNode B 的数据。
   - **优势**：数据只经过机架顶部的交换机，**延迟极低**。
3. **Off-Rack（跨机架）**：
   - **场景**：数据在机架 1，读取器在机架 2。
   - **优势**：没有优势，这是**最差情况**，会占用核心交换机带宽。
4. **Cross-Data-Center（跨数据中心）**：
   - **场景**：数据在北京机房，读取器在上海机房。
   - **优势**：极其罕见。除非本地机房副本全部损坏，否则 HDFS 不会这么干。



**2. 这个策略对你写 Hive SQL 有什么影响？**

虽然读取策略是自动的，但它解释了你之前遇到的一些疑惑：

- **为什么有时候 Hive 运行得飞快，有时候又很慢？** 如果你的 YARN 集群非常拥挤，它可能无法将任务分配到数据所在的节点（Node-Local 失败），甚至无法分配到同一个机架（Rack-Local 失败）。当大量的计算都变成了“跨机架读取”时，你的 SQL 就会变慢。
- **短路读取（Short-Circuit Local Reads）**： 在高级配置中，如果计算和数据在同一个节点，HDFS 甚至可以跳过 DataNode 的网络接口，直接由客户端去读磁盘文件。这比走网络协议栈还要快。



**3. 体系化串联：从“存”到“读”**

我们将你之前学到的知识点连起来看：

- **存的时候（Write）**：为了安全，HDFS 故意把副本分散在**不同机架**（机架感知策略）。
- **读的时候（Read）**：为了速度，HDFS 拼命寻找**最近**的那个副本（读取优先级策略）。

> **这就是分布式系统的艺术**：写的时候多花点网络代价去分散数据（保命），读的时候利用这种分散性，让全集群的机器都能就近拿到数据（提速）。



**4. 数据倾斜的“预警”**

读数据虽然有优先级，但如果**所有读取器**都发现：

> “哎呀，最近的副本都在 DataNode 1 号机上！”

于是几千个任务都涌向 1 号机去读数据，那么 1 号机的**网卡和磁盘 IO** 就会瞬间爆炸。这种现象被称为**“热点（Hotspot）”**，也是数据倾斜的一种表现形式。



### 2.7 架构的稳定性

#### 1. 心跳机制和重新复制

每个 DataNode 定期向 NameNode 发送心跳消息，如果超过指定时间没有收到心跳消息，则将 DataNode 标记为死亡。NameNode 不会将任何新的 IO 请求转发给标记为死亡的 DataNode，也不会再使用这些 DataNode 上的数据。 由于数据不再可用，可能会导致某些块的复制因子小于其指定值，NameNode 会跟踪这些块，并在必要的时候进行重新复制。



#### 2. 数据的完整性

由于存储设备故障等原因，存储在 DataNode 上的数据块也会发生损坏。为了避免读取到已经损坏的数据而导致错误，HDFS 提供了数据完整性校验机制来保证数据的完整性，具体操作如下：

当客户端创建 HDFS 文件时，它会计算文件的每个块的 `校验和`，并将 `校验和` 存储在同一 HDFS 命名空间下的单独的隐藏文件中。当客户端检索文件内容时，它会验证从每个 DataNode 接收的数据是否与存储在关联校验和文件中的 `校验和` 匹配。如果匹配失败，则证明数据已经损坏，此时客户端会选择从其他 DataNode 获取该块的其他可用副本。



#### 3.元数据的磁盘故障

`FsImage` 和 `EditLog` 是 HDFS 的核心数据，这些数据的意外丢失可能会导致整个 HDFS 服务不可用。为了避免这个问题，可以配置 NameNode 使其支持 `FsImage` 和 `EditLog` 多副本同步，这样 `FsImage` 或 `EditLog` 的任何改变都会引起每个副本 `FsImage` 和 `EditLog` 的同步更新。



#### 4.支持快照

快照支持在特定时刻存储数据副本，在数据意外损坏时，可以通过回滚操作恢复到健康的数据状态。



#### 5. 安全模式 (SafeMode) 

**—— “工厂启动的保护期”**

这是 NameNode 启动时的自我保护机制。

- **逻辑**：当 NameNode 刚启动时，它并不知道哪些 DataNode 还活着，也不知道数据块是否全。此时它进入 **SafeMode**。
- **操作**：在安全模式下，HDFS **只读不写**。NameNode 会等待 DataNode 上报块信息（Block Report），只有当副本数正常的块比例达到一个阈值（默认 **99.9%**）后，才会自动退出。
- **实战提示**：有时候你刚开机就去写数据会报错，就是因为还没过这个保护期。



#### 6. 脑裂预防与高可用 (HA & ZKFC) 

**—— “谁才是真老大”**

第 3 点（元数据磁盘故障）解决了“记录丢了”的问题，但没解决“老板跑了”的问题。

- **双 NameNode**：生产环境通常有 **Active**（干活）和 **Standby**（备份）两个老大。
- **共享存储 (QJM)**：它们通过一组 **JournalNodes** 共享 EditLog，保证元数据绝对一致。
- **ZKFC (ZooKeeper Failover Controller)**：监控老大的健康。如果 Active 挂了，ZooKeeper 自动把 Standby 顶上去。
- **隔离 (Fencing)**：防止出现两个 Active（脑裂），系统会强制关掉旧老大的权限。



#### 7. 数据平衡 (Balancer) 

**—— “别让有的机器累死”**

随着时间推移，你的 DataNode 可能会出现“贫富差距”：新加的机器空得很，旧机器快爆了。

- **问题**：数据分布不均会导致 IO 性能不均衡，引发你最担心的**数据倾斜**。
- **解决**：HDFS 提供了 `balancer` 工具。它会在后台默默地把数据块从忙的机器移动到闲的机器，且不影响业务运行。



#### 8. 集中式缓存管理 (Centralized Cache Management)

这是 Hadoop 2.x 引入的提速黑科技。

- **原理**：你可以指定某些高频访问的热点数据（比如 Hive 的维度表）**永久驻留在 DataNode 的内存中**，而不是磁盘上。
- **效果**：这样读取时可以实现“零拷贝”，极大地提升了稳定性查询的响应速度。







### 2.8其他





|      |      |      |
| ---- | ---- | ---- |
|      |      |      |
|      |      |      |
|      |      |      |





## 三、HDFS 的特点

### 3.1 高容错

由于 HDFS 采用数据的多副本方案，所以部分硬件的损坏不会导致全部数据的丢失。



但是高容错也会带来代价:需要考虑时间与空间的平衡

**深入理解**：高容错不代表“零风险”。副本策略是**以空间换安全**（3倍存储成本）。

**补充点——延迟感知**：当某个 DataNode 响应极慢（尚未完全损坏）时，HDFS 会启动**推测执行（Speculative Execution）**。它会在另一台节点上启动相同的读取任务，谁先返回用谁。



### 3.2 高吞吐量

HDFS 设计的重点是支持高吞吐量的数据访问，而不是低延迟的数据访问。



**深入理解**：为什么 HDFS 做不到低延迟？

- **寻道时间**：HDFS 的设计假设是“读取大块数据的时间远大于寻道时间”。
- **元数据开销**：每次请求都要先问 NameNode，这个往返过程对毫秒级的请求太慢了。

**补充点——适用场景**：HDFS 像一列**货运火车**（一次运几千吨，起步慢）；MySQL 像一辆**顺风车**（一次坐几个人，随叫随到）。



### 3.3 大文件支持

HDFS 适合于大文件的存储，文档的大小应该是是 GB 到 TB 级别的。



为什么讨厌“小文件”？

**深入理解**：这是你必须记住的**“150字节法则”**。

- 无论文件多小（哪怕只有 1KB），NameNode 都要在内存中用大约 **150 字节**来存它的元数据。
- **后果**：1 亿个 1KB 的文件会吃掉 NameNode 约 15GB 内存，而实际才存了 100GB 数据。这会让价值百万的集群因为 NameNode 内存溢出而瘫痪。

**补充点——解决方案**：如果你有大量小文件，通常需要用到 **Hadoop Archives (HAR)** 或序列化文件（**Sequence Files**）来合并它们。



### 3.4 简单一致性模型

HDFS 更适合于一次写入多次读取 (write-once-read-many) 的访问模型。支持将内容追加到文件末尾，但不支持数据的随机访问，不能从文件任意位置新增数据。



为什么不支持随机访问?

**深入理解**：

- HDFS 的数据是**分块（Block）** 存储且有副本的。如果你在文件中间插入 1 字节，会导致后面所有的字节位移，这意味着所有的副本、所有的块都要重新计算和重写。在分布式环境下，这会导致极严重的 **一致性冲突** 。

**补充点——追加（Append）的局限**：

- 虽然 2.x 版本支持了 `Append`，但也只能在文件末尾追加，且同一时间只能有一个写入者（单 Writer 模式）。



### 3.5 跨平台移植性

HDFS 具有良好的跨平台移植性，这使得其他大数据计算框架都将其作为数据持久化存储的首选方案。



**深入理解**：HDFS 是用 **Java** 编写的。

**补充点——生态护城河**：

- 因为 HDFS 稳定且开源，所以 Spark、Flink、HBase、Impala 等框架都优先适配它。学好 HDFS，就等于拿到了进入所有大数据框架的“入场券”。





## 附：图解HDFS存储原理

### 1. HDFS写数据原理



<img src="assets/dataDevAssets/1.principal%20manga1.jpg" width="55%" style="display: block; margin: 0 auto;">



<img src="assets/dataDevAssets/1.principal%20manga2.jpg" width="55%" style="display: block; margin: 0 auto;">



<img src="assets/dataDevAssets/1.principal%20manga3.jpg" width="55%" style="display: block; margin: 0 auto;">



### 2. HDFS读数据原理

<div align="center"> <img  src="https://gitee.com/heibaiying/BigData-Notes/raw/master/pictures/hdfs-read-1.jpg"/> </div>



### 3. HDFS故障类型和其检测方法

<div align="center"> <img  src="https://gitee.com/heibaiying/BigData-Notes/raw/master/pictures/hdfs-tolerance-1.jpg"/> </div>

<div align="center"> <img  src="https://gitee.com/heibaiying/BigData-Notes/raw/master/pictures/hdfs-tolerance-2.jpg"/> </div>



**第二部分：读写故障的处理**

<div align="center"> <img  src="https://gitee.com/heibaiying/BigData-Notes/raw/master/pictures/hdfs-tolerance-3.jpg"/> </div>



**第三部分：DataNode 故障处理**

<div align="center"> <img  src="https://gitee.com/heibaiying/BigData-Notes/raw/master/pictures/hdfs-tolerance-4.jpg"/> </div>



**副本布局策略**：

<div align="center"> <img  src="https://gitee.com/heibaiying/BigData-Notes/raw/master/pictures/hdfs-tolerance-5.jpg"/> </div>



# Yarn

我们始终需要思考:

1）如何管理集群资源？

2）如何给任务合理分配资源？

YARN是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而MapReduce等运算程序则相当于运行于操作系统之上的应用程序。所以YARN负责调度,MapReduce在此基础上才能进行运算.

YARN 作为一个资源管理、任务调度的框架，主要包含ResourceManager、NodeManager、ApplicationMaster和Container模块。



![1yarnStructure](../studyDoc/assets/dataDevAssets/1yarnStructure.png)



主角色:Resource Manager:整个集群资源的调度者,负责协调调度各个程序所需要的资源

从角色:NodeManager:单个服务器的资源调度者,负责调度单个服务器上的资源提供给应用程序使用



**1. ResourceManager (RM) —— 总经理 / CEO**

**角色**：整个集群的最高决策者。

**职责**：负责整个集群资源的**统一分配和调度**。它知道每一台机器还有多少内存和 CPU。

**面试重点**：RM 包含两个主要组件：**调度器 (Scheduler)**（负责分配资源，不负责监控）和 **应用程序管理器 (ApplicationsManager)**（负责接收任务提交）。



处理客户端请求,监控NodeManager,启动或监控ApplicationMaster,资源的分配与调度



**2.NodeManager (NM) —— 车间主任 / 门店经理**

**角色**：管理**单台机器**的资源。

**职责**：负责该节点上所有 Container 的生命周期管理，并向 RM 汇报自己家里的“余粮”（剩余资源）和运行情况。



管理单个节点上的资源

处理来自ResourceManager的命令

处理来自ApplicationMaster的命令



**3.ApplicationMaster (AM) —— 项目经理 (每个任务一个)**

**角色**：每个具体任务（如一个 Spark 作业）的负责人。

**职责**：

- 向 RM **申请资源**（要 Container）。
- 拿到资源后，告诉 NM 启动计算。
- **监控任务进度**。如果某个子任务挂了，AM 负责让它重启。

**注意**：AM 是运行在 Container 里的。



为应用程序申请资源并分配给内部的任务

任务的监督与容错



**4.Container —— 生产车间 / 资源箱**

**角色**：YARN 中的**资源抽象单元**。

**职责**：它封装了具体的资源（例如 **2核 CPU + 4GB 内存**）。所有的计算任务（包括 AM 和真正的计算逻辑）都必须运行在 Container 中。

Container是YARN中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等。



🛠️ 辅助组件：日志与安全

**5.代理服务器 (Web App Proxy)**

**作用**：为了安全。当你想通过浏览器查看运行中的任务 UI 时，Proxy 负责转发请求。它能防止 Web 攻击，并隐藏真实节点的 IP 端口。



**6.历史服务器 (JobHistoryServer)**

**作用**：**查账本**。

**重要性**：当任务运行结束，AM 就会消失。如果你想看昨天运行失败的那个任务报错日志，就必须去历史服务器里查。**数据开发调试离线任务（Hive/Spark）时离不开它。**



## MapReduce

 **Hive 就像是一个自动驾驶系统，而 MapReduce 就是这台车的引擎原理。**



<span style="color:red">面试官问 MapReduce，其实是想看你懂不懂分布式计算的底层逻辑。</span>

既然你已经搞定了 Hive，并且理解了它如何把 SQL 变成底层的任务，那么现在学习 **MapReduce** 正是时候。

因为 **Hive 就像是一个自动驾驶系统，而 MapReduce 就是这台车的引擎原理。** 面试官问 MapReduce，其实是想看你懂不懂分布式计算的底层逻辑。



### 1. 核心思想：分而治之 (Divide and Conquer)

想象你要数 10000 枚硬币，一个人数很慢。

- **Map（分）：** 你找 10 个朋友，每人发 1000 枚，让他们各自数出结果。
- **Reduce（合）：** 你把这 10 个朋友数出来的数字加在一起。

这就是 MapReduce 的全部真相。



### 2.<span style="color:red"> MapReduce 的三大阶段（面试必考）</span>

在 Hadoop 中，一个 MapReduce 任务必须经历三个核心阶段：

**① Map 阶段（切分与映射）**

- **输入：** 原始数据（比如几百 GB 的日志文件）。
- **动作：** 每一行数据被解析成 `key-value` 对。
- **例子：** 统计单词频率时，Map 会把 "Apple" 变成 `(Apple, 1)`。

**② Shuffle 阶段（洗牌/核心枢纽）**

这是 **MapReduce 的灵魂**，也是面试最爱问的地方。

- **动作：** 系统自动把所有相同的 `Key` 收集到一起。
- **例子：** 把所有 Map 出来的 `(Apple, 1)` 都送到同一个 Reduce 任务那里。
- **注意：** 这里涉及**排序 (Sort)**、**分组 (Group)** 和**网络传输**，是性能瓶颈所在。

**③ Reduce 阶段（规约/聚合）**

- **动作：** 接收 Shuffle 过来的数据，进行逻辑计算。
- **例子：** 收到一群 `(Apple, [1, 1, 1])`，相加得出 `(Apple, 3)`。

------

### **3. 一个完整的 WordCount 流程图**

假设我们有一行文本：`Hello Hive Hello Hadoop`

1. **InputSplit**: 系统切分数据。
2. **Map**: 产出 `(Hello, 1), (Hive, 1), (Hello, 1), (Hadoop, 1)`。
3. **Shuffle**: 排序并分发，变成 `(Hadoop, 1), (Hello, [1, 1]), (Hive, 1)`。
4. **Reduce**: 汇总，变成 `(Hadoop, 1), (Hello, 2), (Hive, 1)`。
5. **Output**: 写入 HDFS。

------

### **4. 为什么现在大家说 MapReduce“慢”？<span style="color:red">（面试深度题）</span>**

作为数据科学专业的学生，你要能从**架构设计**上回答这个问题：

1. **磁盘 I/O 频繁：** MapReduce 习惯把每一步的中间结果都写到硬盘（HDFS）上。硬盘读写比内存慢几千倍。
2. **启动开销大：** 每一个任务都要启动独立的 JVM 进程，启动就要好几秒。
3. **Shuffle 过程重：** 强制排序虽然保证了数据有序，但在不需要排序的场景下浪费了大量计算资源。

*所以：* **Spark** 出现了，它把中间结果留在**内存**里，极大地提升了速度。

------

### **5. MapReduce 的各个角色 (YARN 环境下)**

你在 `jps` 中看到的那些进程，在跑 MapReduce 时是这样分工的：

- **ResourceManager (RM)**：大 Boss，负责全集群资源的分配。
- **NodeManager (NM)**：小弟，每台机器上的资源监工。
- **ApplicationMaster (AM)**：一个任务的“包工头”，负责监督这一个 MapReduce 任务的生死。

------

### **6. 系统学习建议：你的第一个 MapReduce 程序**

虽然现在写 Hive SQL 多，但面试官可能会让你手写（或伪代码描述）一个 MapReduce 程序。

**你需要掌握的代码结构：**

1. **Mapper 类**：重写 `map` 方法。
2. **Reducer 类**：重写 `reduce` 方法。
3. **Driver 类**：配置 Job（如输入输出路径、关联 Map/Reduce 类等）。

------

### 💡 思考题（准备下一次交流）：

**如果在 Map 阶段数据不均匀，比如某一个单词出现了 1 亿次，而其他单词只出现了 1 次，会发生什么？**

这就是大数据里著名的 **“数据倾斜 (Data Skew)”**。它是所有大数据组件（Hive, Spark, Flink）共同的敌人。

**你想了解如何通过 MapReduce 的原理来解决数据倾斜吗？这直接决定了你面试能否拿到高薪。**



## MySql



普通用户密码:202428

远程密码:202428 / root(存疑 )







# HIVE

### 基础知识

Hive是单机工具，只需要部署在一台服务器即可。Hive虽然是单机的，但是它可以提交分布式运行的MapReduce程序运行。

我们知道Hive是单机工具后，就需要准备一台服务器供Hive使用即可。同时Hive需要使用元数据服务，即需要提供一个关系型数据库，我们也选择一台服务器安装关系型数据库即可。



| **服务**                                                | **机器**    |
| ------------------------------------------------------- | ----------- |
| Hive本体                                                | 部署在node1 |
| 元数据服务所需的关系型数据库（课程选择最为流行的MySQL） | 部署在node1 |

为了简单起见，都安装到node1服务器上。







#### 1. 什么是 Hive？（形象理解）

想象有一个巨大的仓库，里面堆满了成千上万个巨大的 Excel 表格（这就是 **HDFS**，分布式文件系统）。如果你想在这些文件里找数据，得写复杂的程序去遍历。

**Hive 就像是一个“翻译官”：**

- 它坐在这些文件上面。
- 只需要对它说标准的 **SQL**（Hive 里的 SQL 叫 HQL）。
- Hive 会自动把 SQL 语句翻译成复杂的底层计算程序（比如 MapReduce 或 Spark），然后去跑那些文件。

**核心结论：** Hive 的本质是**分布式 SQL 查询引擎**。它让开发者能用写 SQL 的方式，去处理存储在 Hadoop 上的海量数据。将SQL语句 翻译成MapReduce程序运行

![image-20260104174205821](../studyDoc/assets/dataDevAssets/2hadoopHiveIntro.png)

#### 2.基础知识

**① 元数据 (Metadata)**

- Hive 本身**不存储**数据。真正的数据文件（如 `.txt` 或 `.orc`）存在 HDFS 上。
- Hive 只存储“关于数据的数据”：比如表名是什么、有哪些列、每一列是什么类型、文件存在 HDFS 的哪个路径下。
- 这些元数据通常存在一个关系型数据库里（通常是 **MySQL**）。



#### 3.Hive组件

 Hive 的整体运作看作这样：

1. **用户接口 (UI/CLI/JDBC)：** 写代码的地方。
2. **Driver：** “大脑”，负责解析、编译、优化。
3. **Metastore：** “字典”，存表结构。
4. **Hadoop (HDFS/YARN)：** “体力劳动者”，负责存数据和跑计算。



**①Metastore**

通常是存储在关系数据库如 mysql/derby中。Hive 中的元数据包括表的名字，表的列和分区及其属性，表的属性（是否为外部表等），表的数据所在目录等。

-- Hive提供了 Metastore 服务进程提供元数据管理功能

![3hadoopHiveMetastore](../studyDoc/assets/dataDevAssets/3hadoopHiveMetastore.png)



**②Driver驱动程序**

包括语法解析器、计划编译器、优化器、执行器

完成 SQL 查询语句从词法分析、语法分析、编译、优化以及查询计划的生成。生成的查询计划存储在 HDFS 中，并在随后有执行引擎调用执行。这部分内容不是具体的服务进程，而是封装在Hive所依赖的Jar文件即Java代码中。

![3hadoopHiveDriver](F:\学习\studyDoc\assets\dataDevAssets\3hadoopHiveDriver.png)



##### 1.Hive Driver 的四个核心阶段

当在终端输入一条 SQL 并按下回车，Driver 会经历以下四个关键步骤：

**① 解析器 (Parser)**

- **作用：** 检查语法。
- **干什么：** 看看你的 SQL 有没有写错（比如 `SELECT` 写成了 `SELET`）。它会将 SQL 字符串转换成一棵 **抽象语法树 (AST)**。

**② 编译器 (Compiler)**

- **作用：** 语义分析。
- **干什么：** 它会去 **Metastore（元数据库）** 确认：你查询的表是否存在？字段名对不对？
- **产出：** 逻辑执行计划。

**③ 优化器 (Optimizer) —— 面试常考**

- **作用：** 寻找最优路径。
- **干什么：** * **列裁剪：** 只读取你 `SELECT` 的那一列，没用的列不读。
  - **谓词下推：** 把 `WHERE` 条件尽早执行，过滤掉没用的数据。
  - **Join 优化：** 决定先连哪张表更快。

**④ 执行器 (Executor)**

- **作用：** 翻译成计算任务。
- **干什么：** 将优化后的执行计划翻译成底层的分布式任务。在以前是 **MapReduce**，现在主流是 **Tez** 或 **Spark**。



##### 2. Driver 工作的核心：执行计划

在大数据面试中，有一个必考的命令：`EXPLAIN`。

如果想看 Driver 是怎么工作的，可以在 SQL 前面加个关键词。比如：

```SQL
EXPLAIN SELECT name, COUNT(*) FROM student GROUP BY name;
```

你会看到一个复杂的树状结构。作为开发者，我们要通过看这个执行计划来判断：

- 这个任务产生了多少个 Map 阶段？
- 是否发生了 **数据倾斜**？
- 优化器有没有按照我们预想的方式工作？



##### 3. *面试* 为什么面试官喜欢考 Driver？

面试官问 Driver，其实是在考你对**查询性能调优**的理解。

> **面试题模拟：** **问：** “你在做 Hive 优化时，是怎么看 SQL 执行过程的？” **答：** “我会利用 **EXPLAIN** 命令查看 **Driver** 生成的执行计划。我会重点关注物理执行阶段，看是否有不必要的全表扫描，或者是否存在可以利用谓词下推（Predicate Pushdown）来减少 IO 的地方。”



**③用户接口**

包括 CLI、JDBC/ODBC、WebGUI。其中，CLI(command line interface)为shell命令行；Hive中的Thrift服务器允许外部客户端通过网络与Hive进行交互，类似于JDBC或ODBC协议。WebGUI是通过浏览器访问Hive。-- Hive提供了 Hive Shell、 ThriftServer等服务进程向用户提供操作接口



需要了解以下 **3 种核心接口**，以及它们背后对应的**连接协议**:

**1. CLI (Command Line Interface) —— 命令行接口**

这是最原始、最简单的接口。

- **Hive CLI：** 早期直接输入 `hive` 进入的黑窗口。它直接和 Driver 交互，不经过远程服务器。
- **Beeline (推荐)：** 这是现在主流的命令行工具。它基于 **JDBC** 协议，通过连接 HiveServer2 来运行。
  - *面试点：* 为什么现在大家都用 Beeline 而不是传统的 Hive CLI？
  - *答案：* 因为 Hive CLI 比较“重”，它要求客户端机器也要安装完整的 Hive 环境；而 Beeline 很轻量，只要能通过 JDBC 连上服务器就能运行，更安全稳定。

**2. JDBC / ODBC 接口 —— 编程接口**

这是作为**开发人员**最常用的接口。

- **JDBC (Java Database Connectivity)：** 如果你用 Java 代码写一个程序去操作 Hive，或者用 IDEA 里的 Database 插件连接 Hive，用的就是 JDBC。
- **ODBC：** 主要是给 C++/C# 或者一些 BI 工具（如 Tableau, PowerBI）连接 Hive 用的。

**3. Web UI (HUE / Ambari) —— 可视化界面**

在企业生产环境下，很少有人天天对着黑窗口敲代码，通常会使用 **HUE (Hadoop User Experience)**。

- **功能：** 提供了一个像网页一样的编辑器。你可以直接在网页上写 SQL、查看结果图表、甚至直接浏览 HDFS 上的文件。
- **优势：** 门槛极低，适合数据分析师和产品经理。



**核心组件：HiveServer2 (HS2)**

提到接口，就必须提到 **HiveServer2**。它是所有接口（除了旧版 CLI）背后的“大管家”。

- **它的角色：** 所有的 JDBC 连接请求、Beeline 请求都会先发给 HiveServer2。
- **它的作用：** 1.  **多用户并发：** 允许多个人同时连上来写 SQL。 2.  **身份验证：** 检查你有没有权限操作这张表。 3.  **连接池管理：** 就像酒店前台，管理着所有进进出出的连接。





#### hive mysql和元数据管理的关系

##### 1. 角色扮演：图书馆模型

想象你要管理一个拥有 **10亿本书** 的超级大图书馆：

- **HDFS（仓库）：** 是巨大的**书架区域**。书就散乱地堆在那里。它只管存，不管书里写了什么，也不管书叫什么。
- **MySQL（档案卡）：** 是门口的一个**小抽屉**，里面放着几千张纸质卡片。每张卡片记录着：*“某本书叫《大数据》，在第 99 号书架的第三层”*。
- **Hive Metastore（管理员）：** 是图书馆的**专职管理员**。
- **Hive Driver（查询系统）：** 是读者使用的**电脑查询终端**。



##### 2. 它们是怎么配合的？

当你（用户）想看《大数据》这本书时：

1. **你：** 在电脑（**Hive CLI/Beeline**）输入：“我要看《大数据》”。
2. **电脑终端（Driver）：** 它不认识书，它跑去问**管理员（Metastore）**：“喂，有位读者要看《大数据》，它在哪？”
3. **管理员（Metastore）：** 管理员也记不住所有书，但他有钥匙。他打开**小抽屉（MySQL）**，翻出一张档案卡。
4. **MySQL（档案库）：** 档案卡告诉管理员：*“书在 A 区 5 号架”*。
5. **管理员（Metastore）：** 把这个位置告诉电脑终端（Driver）。
6. **电脑终端（Driver）：** 拿着地址，直接冲向**书架（HDFS）**，把那本书取出来展示给你。



##### 3. 为什么你觉得混乱？（理清 3 个核心痛点）

**Q1：既然元数据在 MySQL 里，我能直接改 MySQL 吗？**

- **原则：绝对不要手动改！** * 如果你偷偷在 MySQL 里把表名 `test` 改成了 `test2`，管理员（Metastore）再去找表的时候就会发现“对不上号”，导致 Hive 崩溃。**一定要通过 Hive 语句（如 `ALTER TABLE`）让 Hive 帮你去改。**

**Q2：Metastore 到底是一个软件，还是一个配置？**

- **它是一个“服务（Service）”**。
- 你之前启动的那个 `nohup hive --service metastore` 就是让这个“管理员”上线。如果不启动它，Hive 就像断了线的风筝，找不到 MySQL。
- 元数据库（MySQL）是“存钱的保险柜”，而 Metastore 是“银行柜员”。

**Q3：为什么说 MySQL 是“元数据库”，而 Hive 是“数据仓库”？**

- **MySQL（元数据库）：** 存的是**结构**（表名、列名、分区）。数据量很小，通常只有几十 MB。
- **Hive（数据仓库）：** 存的是**内容**（具体的业务数据）。数据量巨大，通常是几百 TB。



#### **元数据库（Metastore Database）** 和 **元数据服务（Metastore Service）**

**元数据库（MySQL）是“存钱的保险柜”，而 Metastore 是“银行柜员”。**

------

##### 1.深度辨析：元数据库 vs Metastore 服务

为了让你彻底不再混乱，我们把这两个东西剥离开来看：

**元数据库 (The Database)**

- **它是谁：** 通常就是你安装的 **MySQL**（或者是 Oracle、PostgreSQL）。
- **它的职责：** 它是**死**的。它只负责把数据持久化到硬盘上。如果你关掉 Hive，MySQL 里的那些表（TBLS, COLUMNS_V2 等）依然静静地躺在那。
- **面试术语：** 物理存储层。

**Metastore (The Service)**

- **它是谁：** 它是一个**活**的 Java 进程（即你启动的那个 `RunJar` 进程）。
- **它的职责：** 它是**中间人**。
  - 它负责连接 MySQL。
  - 它负责把 Hive 的请求翻译成 SQL 语句发给 MySQL。
  - 它通过 **Thrift 协议** 为外部（如 Spark, Presto, Beeline）提供元数据查询服务。
- **面试术语：** 逻辑服务层。

------

##### 2. 为什么非要搞个 Metastore 服务？直接让 Hive 连 MySQL 不行吗？

其实在 Hive 的早期架构（内嵌模式）里，确实是直接连的。但现在为什么要分开，这涉及大数据架构的两个核心原因：

1. **安全隔离：** 你总不希望几百个开发人员的电脑都能直接拿到 MySQL 的 root 密码吧？有了 Metastore 服务，大家只需要连接服务，由服务统一去读数据库。
2. **多组件共享：** 这是最重要的。在现代大数据生态里，**Spark**、**Flink**、**Presto** 都要读 Hive 的数据。它们不需要自己去翻 MySQL 的表结构，而是统一去问 Metastore 这个“管理员”。

------

##### **3. 一张图理清三者关系**

当你执行 `DESC test;`（查看表结构）时：

1. **Hive Client**: “告诉我 test 表的结构。”
2. **Metastore Service (活的进程)**: “好的，等我去查档。” -> *（执行一条类似 `SELECT \* FROM TBLS...` 的 SQL 发给 MySQL）*
3. **MySQL (元数据库)**: “查到了，给你。”
4. **Metastore Service**: “报告 Client，test 表有三列...”

------

4. 总结你的理解

- **元数据库 = MySQL**（存储元数据的**地方**）
- **Metastore = Hive 里的一个服务**（管理元数据的**程序**）

> **面试避坑指南：** 如果面试官问：“Hive 的元数据管理机制是怎么样的？” **你要答：** “Hive 采用 **Metastore 服务** 来解耦元数据操作。物理上，元数据存储在 **MySQL** 等关系型数据库中；逻辑上，由 **Metastore 守护进程** 通过 Thrift 协议对外提供访问接口。这样可以支持多客户端并发访问，并保证了数据库连接的安全。”



### HIVE客户端

Hive的客户端体系:

**HiveServer2 (HS2)** 和 **Beeline** 是考察“Hive 架构演进”和“企业级安全管理”的核心知识点。

过去我们常用 `hive` 命令行直接操作，但在生产环境中，这种方式已经逐渐被 **Beeline + HiveServer2** 的组合所取代。下面我为你系统性地梳理这部分内容。

<img src="assets/dataDevAssets/4hiveClientStructureSystem.png" alt="4hiveClientStructureSystem" style="zoom:55%;" />

**Beeline账户**
username:hadoop

password:无



**启动命令**

```bash
#注意需要在/export/server/hive目录下输入
bin/beeline

!connect jdbc:hive2://node1:10000
```

> **`!` (感叹号)** 在 Beeline（以及它底层的 SQLLine）中，所有**客户端控制命令**（如连接、退出、显示帮助）都必须以感叹号开头，以便与普通的 SQL 语句区分开。
>
> **`connect`** 这是告诉 Beeline：“我要开始建立一个新的数据库连接了”。
>
> **`jdbc:hive2://`** 这是 **JDBC 连接协议头**。
>
> - `jdbc`: 表示使用 Java 数据库连接驱动。
> - `hive2`: 明确指定连接的是 HiveServer2（而不是旧版的 HiveServer1）。
>
> **`node1:10000`**
>
> - `node1`: HiveServer2 所在服务器的主机名或 IP 地址。
> - `10000`: HiveServer2 的默认监听端口。



#### 1. 为什么需要 HiveServer2？（对比 Hive CLI）

在早期的 Hive 版本中，用户主要使用 **Hive CLI**。但它存在严重的缺陷，导致无法在大型多用户企业环境中使用：

- **胖客户端（Fat Client）：** Hive CLI 包含所有的查询编译和执行逻辑，用户必须在本地安装完整的 Hive 客户端。
- **权限隐患：** Hive CLI 直接以当前 OS 用户的身份访问 HDFS 数据，无法进行细粒度的权限控制（如使用 Ranger 或 Sentry）。
- **无并发管理：** 缺乏一个统一的服务端来协调多个用户的请求。

**HiveServer2 的出现解决了这些问题：** 它是一个基于 **Thrift** 协议的服务，允许远程客户端（如 JDBC/ODBC）并发地提交查询。它支持多用户并发、身份认证（Kerberos/LDAP）和更好的安全性。

#### 2. HiveServer2 核心架构

HiveServer2 是 Hive 的“大脑”，它负责接收请求并与 Hadoop 集群交互。

**核心组成部分：**

1. **Thrift 接口：** 提供跨语言的服务调用能力，支持 TCP 或 HTTP 模式通信。
2. **Session 管理：** 每个连接都会创建一个 Session，保存该用户的配置信息和临时状态。
3. **Operation 管理：** 负责 SQL 语句的编译（Compiler）、优化（Optimizer）和执行（Execution Engine）。
4. **安全层：** 负责认证（你是谁）和授权（你能查什么）。

#### 3. Beeline：现代化的 Hive 客户端

**Beeline** 是一个基于 **SQLLine** 的 JDBC 客户端。它的设计理念是“轻量化”，用户不需要在本地安装 Hive，只需要一个 JDBC 驱动即可连接。

**Beeline 的两种模式：**

1. **嵌入模式 (Embedded Mode)：** * 在本地启动一个隐藏的 HS2 实例。
   - 通常用于开发测试，生产环境**不推荐**。
2. **远程模式 (Remote Mode)：**
   - 通过 JDBC URL 连接远程 HS2 服务。
   - **生产环境唯一标准。**

```bash
# 基础连接方式 详细node1见上
beeline -u jdbc:hive2://<hs2-host>:10000 -n <username> -p <password>

# 生产中带 Kerberos 认证的连接
beeline -u "jdbc:hive2://<hs2-host>:10000/default;principal=hive/_HOST@REALM"
```

#### 4. <span style="color:red">关键面试考点：HS2 的高可用 (High Availability)</span>

在生产环境中，HS2 不能是单点故障。我们通常使用 **Zookeeper** 来实现高可用。

- **工作机制：** 多个 HS2 实例启动后，会在 Zookeeper 的指定目录下注册自己的节点信息（Ephemeral Nodes）。
- **客户端连接：** Beeline 连接时不再指向某个具体的 IP，而是指向 Zookeeper 地址。
- **连接字符串示例：** `jdbc:hive2://zk_host1:2181,zk_host2:2181/default;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2`





## 基本操作

### 1.数据库操作

#### ①创建数据库

```hive
CREATE DATABASE [IF NOT EXISTS] db_name [LOCATION position];
create database if not exists myhive;

use  myhive;
```



#### ②查看数据库详细信息

```hive
desc  database  myhive;
```

![2.hive database operation](assets/dataDevAssets/2.hive database operation.png)

数据库本质上就是在HDFS之上的文件夹。

默认数据库的存放路径是HDFS的：`/user/hive/warehouse`内

可以通过`LOCATION`关键字在创建的时候指定存储目录

![2.hive database operation2](assets/dataDevAssets/2.hive database operation2.png)



#### ③创建数据库并指定hdfs存储位置

```hive
create database myhive2 location '/myhive2';
```

使用**`location`**关键字，可以指定数据库在HDFS的存储路径。



#### ④删除数据库

删除一个空数据库，如果数据库下面有数据表，那么就会报错

```hive
drop  database  myhive;
```



**强制**删除数据库，包含数据库下面的表一起删除

```hive
CREATE DATABASE [IF NOT EXISTS] db_name [LOCATION position];

drop  database  myhive2  cascade; 
```



### 2数据表操作

#### 1.表操作语法和数据类型

##### ①创建/删除数据库表

创建表的语法还是比较复杂的

```hive
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
[ (col_name data_type [COMMENT col_comment], ...) ]
[COMMENT table_comment]
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
[CLUSTERED BY (col_name, col_name, ...)
 [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS
]
[ROW FORMAT row_format]
[STORED AS file_format]
[LOCATION hdfs_path];
```

| **子句**           | **说明**                                                     | **示例**                                       |
| ------------------ | ------------------------------------------------------------ | ---------------------------------------------- |
| **EXTERNAL**       | 创建外部表。删除表时**仅删除元数据**，不删除 HDFS 上的实际数据。 | `CREATE EXTERNAL TABLE ...`                    |
| **PARTITIONED BY** | 建立分区表，用于优化查询性能（物理上表现为子目录）。         | `PARTITIONED BY (dt STRING)`                   |
| **ROW FORMAT**     | 定义数据行的序列化与反序列化规则（常用分隔符）。             | `DELIMITED FIELDS TERMINATED BY ','`           |
| **STORED AS**      | 指定文件存储格式（如文本、列式存储等）。                     | `STORED AS ORC` 或 `PARQUET`                   |
| **LOCATION**       | 指定该表在 HDFS 上的具体存储路径。                           | `LOCATION '/user/hive/warehouse/my_db/my_tbl'` |



尽管建表语法比较复杂，目前我们暂时未接触到分区、分桶等概念。所以，创建一个简答的数据库表可以有如下SQL：

```hive
CREATE TABLE test(	
    id INT,    
    name STRING,    
    gender STRING
);
```



如果要删除表可以使用：

```hive
DROP TABLE table_name;
```





##### ②数据类型

| **分类** | **类型**                                    | **描述**                                       | **字面量示例**                                               |
| -------- | ------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| 原始类型 | BOOLEAN                                     | true/false                                     | TRUE                                                         |
|          | TINYINT                                     | 1字节的有符号整数 -128~127                     | 1Y                                                           |
|          | SMALLINT                                    | 2个字节的有符号整数，-32768~32767              | 1S                                                           |
|          | <span style="color:red">INT</span>          | 4个字节的带符号整数                            | 1                                                            |
|          | BIGINT                                      | 8字节带符号整数                                | 1L                                                           |
|          | FLOAT                                       | 4字节单精度浮点数1.0                           |                                                              |
|          | <span style="color:red">DOUBLE </span>   | 8字节双精度浮点数                              | 1.0                                                          |
|          | DEICIMAL                                    | 任意精度的带符号小数                           | 1.0                                                          |
|          | <span style="color:red">STRING</span>    | 字符串，变长                                   | “a”,’b’                                                      |
|          | <span style="color:red">VARCHAR</span>   | 变长字符串                                     | “a”,’b’                                                      |
|          | CHAR                                        | 固定长度字符串                                 | “a”,’b’                                                      |
|          | BINARY                                      | 字节数组                                       |                                                              |
|          | <span style="color:red">TIMESTAMP</span> | 时间戳，毫秒值精度                             | 122327493795                                                 |
|          | <span style="color:red">DATE</span>      | 日期                                           | ‘2016-03-29’                                                 |
|          |                                             | 时间频率间隔                                   |                                                              |
| 复杂类型 | ARRAY                                       | 有序的的同类型的集合                           | array(1,2)                                                   |
|          | MAP                                         | key-value,key必须为原始类型，value可以任意类型 | map(‘a’,1,’b’,2)                                             |
|          | STRUCT                                      | 字段集合,类型可以不同                          | struct(‘1’,1,1.0), named_stract(‘col1’,’1’,’col2’,1,’clo3’,1.0) |
|          | UNION                                       | 在有限取值范围内的一个值                       | create_union(1,’a’,63)                                       |



Hive中支持的数据类型还是比较多的

其中<span style="color:red">红色</span>的是使用比较多的类型(github直接浏览可能没有颜色,请下载md文档到本地查看)



#### 2.内/外部表介绍

> 开始之前,需要先介绍一下表分类
>
Hive中可以创建的表有好几种类型， 分别是：

<span style="color:red">内部表</span>

<span style="color:red">外部表</span>

分区表

分桶表

不同类型的表有各自的用途。

我们首先学习内部表和外部表的区别。



-----

**内部表**（CREATE TABLE table_name ......）

未被`external`关键字修饰的即是内部表， 即普通表。 
内部表又称管理表,内部表数据存储的位置由`hive.metastore.warehouse.dir`参数决定（默认：/user/hive/warehouse）,
删除内部表会直接<span style="color:red">删除元数据（metadata）及存储数据</span>，因此内部表不适合和其他工具共享数据。



**外部表**（CREATE EXTERNAL TABLE table_name ......LOCATION......）

被`external`关键字修饰的即是外部表， 即关联表。
外部表是指表数据可以在任何位置，通过`LOCATION`关键字指定。 数据存储的不同也代表了这个表在理念是并不是Hive内部管理的，而是可以随意临时链接到外部数据上的。
所以，在删除外部表的时候， 仅仅是删除元数据（表的信息），不会删除数据本身。



快速对比一下内部表和外部表:

|        | **创建**                     | **存储位置**                         | **删除数据**                     | **理念**           |
| ------ | ---------------------------- | ------------------------------------ | -------------------------------- | ------------------ |
| 内部表 | CREATE TABLE ......          | Hive管理，默认`/user/hive/warehouse` | 删除 元数据（表信息）删除 数据   | Hive管理表持久使用 |
| 外部表 | CREATE EXTERNAL TABLE ...... | 随意，`LOCATION`关键字指定           | 仅删除 元数据（表信息）保留 数据 | 临时链接外部数据用 |



#### 3.内部表操作

##### ①创建内部表

内部表的创建语法就是标准的：CREATE TABLE table_name......

创建一个基础的表:

```hive
create database if not exists myhive;

use myhive;

create table if not exists stu(id int,name string);

insert into stu values (1,"zhangsan"), (2, "wangwu");select * from stu;
```



查看表的数据存储在HDFS上，查看表的数据存储文件

![2.hive database operation3](assets/dataDevAssets/2.hive database operation3.png)



##### ②数据分隔符

可以看到，数据在HDFS上也是以明文文件存在的。

![2.hive database operation3](assets/dataDevAssets/2.hive database operation3.png)

奇怪的是， 列ID和列NAME，好像没有分隔符，而是挤在一起的。这是因为，默认的数据分隔符是：”\001”是一种特殊字符，是ASCII值，键盘是打不出来在某些文本编辑器中是显示为SOH的。

例如1zhangsan其实是:1`SOH`zhangsan

![2.hive database operation4](assets/dataDevAssets/2.hive database operation4.png)



##### ③自行指定分隔符

所以我们可以自行指定

在创建表的时候可以自己决定：

```hive
create table if not exists stu2(id int ,name string) row format delimited fields terminated by '\t';
```

`row format delimited fields terminated by '\t'`：表示以`\t`分隔 `\t`是我们常见的分隔符,这个大家就很熟悉了

这样中间就更友好的分隔显示出来了

![2.hive database operation5](assets/dataDevAssets/2.hive database operation5.png)



##### ④其它创建内部表的形式

除了标准的`CREATE TABLE table_name`的形式创建内部表外,我们还可以通过：

- `CREATE TABLE table_name as`，基于查询结果建表
  e.g. `create table stu3 as select * from stu2;`

- `CREATE TABLE table_name like`，基于已存在的表结构建表
  e.g.` create table stu4 like stu2;`

- 也可以使用`DESC FORMATTED table_name`，查看表类型和详情
  e.g. `DESC FORMATTED stu2;`



##### ⑥删除内部表

我们是内部表删除后，数据本身也不会保留。

- `DROP TABLE table_name`，删除表

  e.g. `drop table stu2`;

  ![2.hive database operation6](assets/dataDevAssets/2.hive database operation6.png)

  可以看到，stu2文件夹已经不存在了，数据被删除了。



#### 4.外部表操作

##### ①外部表创建

![2.hive database operation7](assets/dataDevAssets/2.hive database operation7.png)

外部表，创建表被`EXTERNAL`关键字修饰，从概念是被认为并非Hive拥有的表，只是临时关联数据去使用。

创建外部表也很简单，基于外部表的特性，可以总结出： 外部表 和 数据 是相互独立的， 即：

- <span style=color:red>可以先有表</span>，然后把数据移动到表指定的LOCATION中

- <span style="color:red">也可以先有数据</span>，然后创建表通过LOCATION指向数据

1.在Linux上创建新文件，test_external.txt，并填入如下内容：

```
1	itheima
2	itcast
3	hadoop
```

数据列用`\t`分隔,我们前文提到过



2.演示<span style="color:red">先创建外部表，然后移动数据</span>到LOCATION目录

- 首先检查：`hadoop fs -ls /tmp`，确认不存在`/tmp/test_ext1`目录

- 创建外部表：

  ```hive
  create external table test_ext1(id int, name string) row format delimited fields terminated by ‘\t’ location ‘/tmp/test_ext1’;
  ```

- 可以看到，目录`/tmp/test_ext1`被创建

- ```hive
  select * from test_ext1
  ```

  空结果，无数据

- 上传数据： `hadoop fs -put test_external.txt /tmp/test_ext1/` 

- ```hive
  select * from test_ext1
  ```

  即可看到数据结果



3.演示<span style="color:red">先存在数据，后创建外部表</span>

- `hadoop fs -mkdir /tmp/test_ext2`

- `hadoop fs -put test_external.txt /tmp/test_ext2/`

- ```hive
  create external table test_ext2(id int, name string) row format delimited fields terminated by ‘\t’ location ‘/tmp/test_ext2’;
  ```

- `select * from test_ext2;`



##### ②删除外部表

```hive
drop table test_ext1;

drop table test_ext2;
```

可以发现，在Hive中通过show table，表不存在了
但是在HDFS中，数据文件<span style="color:red">依旧保留</span>





##### ③内外部表转换

Hive可以很简单的通过SQL语句转换内外部表。

- 查看表类型：desc formatted stu;

<img src="assets/dataDevAssets/2.hive%20database%20operation8.png" width="60%" style="display: block; margin: 0 auto;">



转换

- 内部表转外部表

  ```hive
  alter table stu set tblproperties('EXTERNAL'='TRUE');
  ```

- 外部表转内部表

  ```hive
  alter table stu set tblproperties('EXTERNAL'='FALSE');
  ```

  通过stu set tblproperties来修改属性

  <span style="color:red">要注意：('EXTERNAL'='FALSE') 或 ('EXTERNAL'='TRUE')为固定写法，区分大小写！！！</span>





外部表和其数据，是相互独立的，即：

- 可以先有表后有数据
- 也可以先有数据，后有表
- 表和数据只是一个链接关系
- 所以删除表，表不存在了但数据保留。



#### 5.数据的加载和导出

##### ①数据加载 

**数据加载 -Load语法**

我们使用 LOAD 语法，从外部将数据加载到Hive内，语法如下：

<img src="assets/dataDevAssets/2.hive%20database%20operation9.png" width="70%" style="display: block; margin: 0 auto;">

建表：

```hive
CREATE TABLE myhive.test_load(
  dt string comment '时间（时分秒）', 
  user_id string comment '用户ID', 
  word string comment '搜索词',
  url string comment '用户访问网址'
) comment '搜索引擎日志表' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
```



数据为:search.txt

```
00:00:01        1233215666      传智播客        http://www.itcast.cn
00:50:00        2233216666      黑马程序员      http://www.itcast.cn
01:06:00        3233217666      大数据  http://www.itcast.cn
03:36:00        3233218666      hadoop  http://www.itcast.cn
19:39:00        2233219666      python  http://www.itcast.cn
21:26:00        1233211666      大数据开发      http://www.itcast.cn
23:22:00        6233212666      itheima http://www.itcast.cn
```



加载 还是一样`local`代表从linux中加载,不加`local`代表从HDFS中加载

```hive
load data local inpath '/home/hadoop/search_log.txt' into table myhive.test_load;
load data inpath '/tmp/search_log.txt' overwrite into table myhive.test_load;
```

> <span style="color:red">注意，基于HDFS进行load加载数据，源数据文件会消失（本质是被移动到表所在的目录中）</span>



---

**②数据加载 - INSERT SELECT 语法**

除了`load`加载外部数据外，我们也可以通过SQL语句，从其它表中加载数据。

语法：

```hive
INSERT [OVERWRITE | INTO] TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...) [IF NOT EXISTS]] select_statement1 FROM from_statement;
```

将`SELECT`查询语句的结果插入到其它表中，被SELECT查询的表可以是内部表或外部表。

示例：

```hive
INSERT INTO TABLE tbl1 SELECT * FROM tbl2;
INSERT OVERWRITE TABLE tbl1 SELECT * FROM tbl2;
```

>  `INTO`          是追加, 如果是插入相同的目标,则在末尾追加
>
> `OVERWRITE` 是覆写,如果插入相同的目标,会覆盖旧有内容 
>
> 其中 `INSERT INTO TABLE tbl1 SELECT * FROM tbl2;`是查询结果插入,把 `tb2` 表里的所有数据，“拷贝”一份放入 `tb1` 表中。



对于数据加载，我们学习了：LOAD和INSERT SELECT的方式，那么如何选择它们使用呢？

- 数据在本地推荐 :

  `load data local`加载

- 数据在HDFS

  - 如果不保留原始文件：	推荐使用LOAD方式直接加载

  - 如果保留原始文件：	    推荐使用外部表先关联数据，然后通过INSERT SELECT 外部表的形式加载数据

- 数据已经在表中只可以INSERT SELECT



在这里关于不保留原始文件和保留原始文件做一些解释:

**场景 A：使用 `LOAD DATA`（不保留原始文件）**

当你执行：`LOAD DATA INPATH '/tmp/my_data/words.txt' INTO TABLE stu;`

- **动作**：Hive 会直接调用 HDFS 的 `mv`（移动）命令。
- **结果**：文件从 `/tmp/my_data/` 消失了，跑到了 `/user/hive/warehouse/myhive.db/stu/` 下面。
- **代价**：原始路径空了，如果别的程序（比如 Spark 或 Flink）也要用这个文件，就找不到了。



**场景 B：使用“外部表关联 + INSERT SELECT”（保留原始文件）**

这是你问的重点，它分三步走：

**第一步：创建外部表“关联”原始路径**

```hive
CREATE EXTERNAL TABLE t_ext(line string) 
LOCATION '/tmp/my_data/'; -- 这里就是“关联”，只是登记了地址
```

- 此时，文件还在 `/tmp/my_data/`，没动。

**第二步：创建正式的内部表（存放到仓库）**

```hive
CREATE TABLE t_final(line string); 
```

**第三步：通过 INSERT SELECT 搬运数据**

```hive
INSERT INTO TABLE t_final SELECT * FROM t_ext;
```



**核心逻辑**：`LOCATION` 相当于“标记”了 `t_ext`，

- **对于外部表**：这个“标记”告诉 Hive：“这块地不是我的，我只是有个观察位（Metadata）。”
- **所以**：当你对这个“标记”的表做任何操作时，Hive 的底层逻辑是 **“只读不写”**。它绝对不会去移动或删除这个路径下的原始文件。

同时`INSERT SELECT` 这个动作本身就决定了它必须是**“复制”**。

> **`LOAD DATA` 命令**：它的本质是 **“文件管理命令”**。它像是在 Linux 里执行 `mv`。为了效率，它直接把文件挪走。

**`INSERT SELECT` 命令**：它的本质是 **“计算命令”**。

- 它会启动 **MapReduce**。
- **Map 阶段**：去 `t_ext` 标记的路径下**读取**（Read）每一行数据。
- **Reduce 阶段**：把读到的数据加工后，**写入**（Write）到 `t_final` 对应的仓库路径下。
- **结果**：读操作不会破坏原文件，写操作产生了新文件。所以，**客观上完成了“复制”**。



这里有个很有意思的问题:**如果 `t_ext` 是内部表，执行 `INSERT SELECT` 会怎样？也就是为什么我们要创立外部表**

假设 `t_ext` 也是一个**内部表**，你执行： `INSERT INTO TABLE t_final SELECT * FROM t_ext;`

- **结果依然是：** 两个表里都有数据（复制）。
- **差别在于：** 如果你接下来执行 `DROP TABLE t_ext`，因为它是内部表，它的那份数据就会被删掉。
- **而如果是外部表：** 你删掉 `t_ext`，原始文件依然在。



之所以“先建外部表关联”能保留原文件，最根本的原因在于：**外部表（External Table）在设计上放弃了对数据的“所有权（Ownership）”，它只保留了“使用权”。**



在 Hive 的逻辑里：

- **内部表（Managed Table）**：Hive 认为这块数据是它“家”里的。既然是家里的东西，它有权把文件从别处**搬进**自己家（`LOAD` 操作），或者在不需要时把东西**扔掉**（`DROP` 操作）。

- **外部表（External Table）**：Hive 认为这块数据是“邻居”家或者“公共仓库”里的。建表语句里的 `LOCATION` 实际上是在说：**“喂，Hive，那儿有一堆数据，你以后查这张表时，就去那个地址看一眼，别乱动人家的东西。”**

  

​      当你执行 `CREATE EXTERNAL TABLE ... LOCATION '/tmp/data'` 时：Hive 没有任何移动文件的动作。它只是在 MySQL 的元数据库里登记了一下：表名 t2 对应路径 `/tmp/data`。



**结果**：

1. 原始文件 `words.txt` 依然在 `/tmp/my_data/`（**保留了原始文件**）。
2. 仓库里多了一份经过处理的数据（**完成了加载**）。



##### **②数据导出**

**数据导出 - insert overwrite 方式**

将hive表中的数据导出到其他任意目录，例如linux本地磁盘，例如hdfs，例如mysql等等

语法：

```hive
insert overwrite [local] directory ‘path’ select_statement1 FROM from_statement;
```

将查询的结果导出到本地 - 使用默认列分隔符:

```hive
insert overwrite local directory '/home/hadoop/export1' select * from test_load ;
```

将查询的结果导出到本地 - 指定列分隔符

```hive
insert overwrite local directory '/home/hadoop/export2' row format delimited fields terminated by '\t' select * from test_load;
```

将查询的结果导出到HDFS上(不带local关键字)

```hive
insert overwrite directory '/tmp/export' row format delimited fields terminated by '\t' select * from test_load;
```



**hive表数据导出 - hive shell**

基本语法：（hive -f/-e 执行语句或者脚本 > file）

直接在命令行里写sql然后>导出

```shell
bin/hive -e "select * from myhive.test_load;" > /home/hadoop/export3/export4.txt
```

另一种是把sql命令vim到文件(export.sql)里,然后>导出,效果一样,只不过可预见的写入到文件里拓展性和上制更高

```shell
bin/hive -f export.sql > /home/hadoop/export4/export4.txt
```



##### ③特别内容:`>`

另外再总结一下导出`>`

在 Linux 中，每一个运行的程序默认都会打开三个“管道”。你提到的数字其实是**文件描述符 (File Descriptor)**：

| **符号** | **描述**     | **名称** | **作用**                                   |
| -------- | ------------ | -------- | ------------------------------------------ |
| **`0>`** | 标准输入     | `stdin`  | 比如你键盘输入的内容。                     |
| **`1>`** | **标准输出** | `stdout` | 程序正常运行产生的结果（默认显示在屏幕）。 |
| **`2>`** | **标准错误** | `stderr` | 程序报错时的信息（默认也显示在屏幕）。     |

常用组合技：

- **`>` (等同于 `1>`)**：**覆盖**重定向。把正常结果存入文件，报错信息依然吐在屏幕上。
- **`>>`**：**追加**重定向。不删除旧内容，在文件末尾续写。
- **`2>`**：**错误**重定向。只把报错信息存入文件。
- **`&>`**：**全都要**。把正常结果和报错信息通通存入同一个文件。
- **`1> a.txt 2> b.txt`**：**各回各家**。正常结果去 a，报错去 b。

> **关于 `3>`**：在 Linux 中，3-9 是预留给用户自定义的描述符。在大数据开发中几乎用不到，通常是写复杂脚本时用来做特殊的临时管道。



##### 总结

**为什么要加载数据而非使用外部表？**

- 外部表是临时使用的，重要数据建议存入内部表进行管理

**加载数据的语法**:

- `LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename;`
- `INSERT INTO|OVERWRITE SELECT...`

**注意事项**：
使用LOAD语句：

- 数据来源本地，本地数据文件会保留，本质是本地文件上传到
- HDFS据来自HDFS，加载后文件不存在，本质是在HDFS上进行文件移动



**Hive数据导出的方式:**

- `INSERT [LOCAL] SELECT`
- `bin/hive -e 'SQL' > file`
- `bin/hive -f ‘sql file’ > file`



### 3.分区表

在大数据中，最常用的一种思想就是分治，我们可以把大的文件切割划分成一个个的小的文件，这样每次操作一个小的文件就会很容易了

同样的道理，在hive当中也是支持这种思想的，就是我们可以把大的数据，按照每天，或者每小时进行切分成一个个的小的文件，这样去操作小的文件就会容易得多了。

**分区表 (Partitioned Table)** 也是 Hive 的“加速器”。



**为什么要分区？（解决“全表扫描”的痛苦）**

想象你有一个存了 10 年记录的表 `stu`，数据量有 100TB。

- **没有分区**：如果你只想看“昨天”的数据，Hive 必须把 10 年的数据**全部读一遍**，然后过滤出昨天的。这就像是在一个堆满 10 年报纸的房间里找昨天的头条，你会疯掉，集群也会被你跑死。
- **有了分区**：Hive 会把 10 年的数据分成 3650 个“小文件夹”。查询时，Hive 直接跳过其他文件夹，只读昨天的那个。这就是**分区裁剪 (Partition Pruning)**。

![2.hive database operation10](assets/dataDevAssets/2.hive database operation10.png)

**物理本质：分区 = HDFS 子目录**

这是理解分区最直接的方法。在 Hive 里，一个分区对应的就是一个 **HDFS 目录**。

假设你创建了一个按月分区的表：

```hive
CREATE TABLE order_monthly (
    order_id string,
    price double,
    user_id string
)
PARTITIONED BY (month string)  -- 分区字段
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
```

你在 HDFS 看到的目录结构长这样：


> `/user/hive/warehouse/score/month=2026-01/`
>
> `/user/hive/warehouse/score/month=2026-01/`

 

#### 1. 分区表的关键语法

##### A. 创建分区表

注意：分区字段（如 `dt`）**千万不要**出现在 `CREATE TABLE` 的普通字段列表里，它定义在 `PARTITIONED BY` 中。

```hive
CREATE TABLE stu_partition (id int, name string) 
PARTITIONED BY (month string);
```

##### B. 加载数据（静态分区）

你需要手动告诉 Hive，这批数据属于哪个“文件夹”。

```hive
LOAD DATA LOCAL INPATH '/home/hadoop/words.txt' 
INTO TABLE stu_partition PARTITION (month='202602');
```

##### C. 查询数据（性能飞跃的关键）

查询时必须带上分区字段作为过滤条件，否则依然会全表扫描。

```hive
SELECT * FROM stu_partition WHERE month = '202602';
```



静态分区 vs. 动态分区

**动态分区的“坑”：** 默认为了防止你一下子创建几万个文件夹把 NameNode 搞死，动态分区是关闭或受限的。开启它需要设置

```shell
SET hive.exec.dynamic.partition=true; -- 开启动态分区
SET hive.exec.dynamic.partition.mode=nonstrict; -- 允许所有分区都是动态的
```



#### 2.动态分区

##### 介绍

**静态分区**（手动指定 `month='202006'`），**动态分区 (Dynamic Partitioning)** 会更加便利,类似于自动分拣机。

| **特性**     | **静态分区 (Static)**                             | **动态分区 (Dynamic)**                               |
| ------------ | ------------------------------------------------- | ---------------------------------------------------- |
| **决定权**   | **你(开发者)**（在 SQL 里写死：`month='202006'`） | **数据本身**（根据某列的值自动分目录）               |
| **操作命令** | `LOAD DATA` 或 `INSERT`                           | **只能用 `INSERT SELECT`**                           |
| **适用场景** | 每天处理一份确定的数据（如：昨天的日志）          | 历史数据全量入库（比如把 10 年的数据一次性按月分好） |



为了防止新手不小心创建了几万个小文件夹导致 NameNode 崩溃，Hive 默认把动态分区管得很严。你想用，必须先执行这两行“口令”：

```shell
-- 1. 开启总开关
SET hive.exec.dynamic.partition=true;

-- 2. 设置为非严格模式 (nonstrict)
-- 默认是 strict（严格），要求至少有一个静态分区。
-- 设置为 nonstrict，表示允许所有分区字段都是动态的。
SET hive.exec.dynamic.partition.mode=nonstrict;
```



##### 实战演示

假设你有一个**临时表** `score_temp`，里面存了 3 个月的数据，长这样：

| **id** | **name** | **score** | **dt_col** |
| ------ | -------- | --------- | ---------- |
| 1      | 周杰伦   | 99        | 202005     |
| 2      | 林俊杰   | 88        | 202006     |
| 3      | 王力宏   | 77        | 202007     |

你想把这些数据插到你的**分区表** `score` 里，让它自动进对应的文件夹。

**错误的直觉：** 你可能会想用 `LOAD DATA`。

- **真相**：`LOAD DATA` 只是搬运文件，它不会去读文件内容，所以**无法**实现动态分区。

**正确的姿势：使用 `INSERT SELECT`**

```hive
INSERT OVERWRITE TABLE score PARTITION (month) -- 这里只写字段名，不写具体的值
SELECT id, name, score, dt_col -- 重点：分区字段必须放在 SELECT 的最后一列！
FROM score_temp;
```



##### 规则

**同时有一些规则也是需要注意的:**

**位置决定论**： Hive 是根据 `SELECT` 语句中**最后一列**的值来决定分区的，而不是根据列名。

- 哪怕你的最后一列叫 `abc`，只要它的值是 `202005`，Hive 就会把它丢进 `month=202005` 文件夹。

**自动创建目录**： 如果 `SELECT` 出来的月份在 HDFS 里还没目录，Hive 会自动帮你 `mkdir`。

**性能压力**： 如果你的数据里有 1000 个不同的月份，这个任务会同时打开 1000 个文件写入流，非常消耗内存。



##### **使用场景**

在标准的生产环境中，静态分区和动态分区并不是非黑即白的选择，它们通常是“组合拳”,

**1. 静态分区 (Static)：**

**核心场景：每日增量离线加工。** 这是大数据开发中最常见的场景。比如每天凌晨 1 点，跑昨天的经营数据。

- **为什么用静态？**
  - **可控性极高**：你在脚本里明确知道今天要处理的是 `2026-02-20` 的数据。
  - **效率最高**：不需要 Hive 去扫描整份数据来判断该往哪投递，直接对准那个“坑”就灌进去。
- **生产实战：** 通常配合 **Shell 脚本** 或 **调度工具（如 Airflow, DolphinScheduler）** 使用。

```shell
# 脚本中的伪代码
today=$(date -d "yesterday" +%Y%m%d)
hive -e "LOAD DATA LOCAL INPATH '...' INTO TABLE score PARTITION (dt='$today')"
```



------

**2.动态分区 (Dynamic)：**

**核心场景 A：历史数据一次性补录 (Backfill)。** 老板突然说：“把过去 3 年的 Excel 成绩单全部导进 Hive。”

- **痛点**：如果你用静态，你要写 1000 多个 `LOAD` 语句（每天一个）。
- **解法**：先把 3 年数据传到一个临时表，然后一个**动态分区 `INSERT`**，Hive 自动按数据里的日期分好 1000 个目录。

**核心场景 B：数据流中包含多个分区的数据。** 比如一个埋点日志流，虽然是今天收到的，但里面可能混杂了前天因为网络延迟刚传回来的数据。

- **解法**：通过动态分区，让数据根据自己的时间戳“各回各家”，避免前天的数据被误存在今天的目录下。





#### **3. 分区表的“坑”与避坑指南**

在生产中，随意使用动态分区可能会导致集群崩溃（尤其是 OOM 或 NameNode 压力过大）。以下是职业选手的操作规范：

1. **限制分区总数**： 设置 `hive.exec.max.dynamic.partitions=1000;`。如果你的 SQL 尝试一次性创建 1 万个分区，Hive 会直接“自杀”报错，保护集群不被你搞死。

2. **强制开启 `DISTRIBUTE BY`**： 在执行动态分区 `INSERT` 时，最好在 SQL 最后加上 `DISTRIBUTE BY 分区字段`。

   - **作用**：让相同的分区数据流向同一个 Reducer，这样每个 Reducer 只会打开一个文件句柄。否则，如果每个 Reducer 都写 1000 个分区，文件句柄数会瞬间爆炸。

3. **严格模式 (Strict Mode)**： 虽然我们实验时用了 `nonstrict`，但生产环境有些公司强行要求 `strict`。即：**必须指定一个静态分区，剩下的再动态。**

   ```hive
   -- 比如：年份是定的（静态），月份是变的（动态）
   INSERT INTO table_name PARTITION (year='2026', month) 
   SELECT ..., month_col FROM tmp;
   ```
