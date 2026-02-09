## Hadoop



## Hadoop架构

![image-20251025104815212](C:\Users\EACH\AppData\Roaming\Typora\typora-user-images\image-20251025104815212.png)



### HDFS基本架构

HDFS是主从模式(中心化模式)

![1](F:\学习\数据开发\asset\img\1.jpg)

**NameNode:**

- HDFS系统的主角色，是一个独立的进程
- 负责管理HDFS整个文件系统
- 负责管理DataNode



**DataNode:**

- HDFS系统的从角色，是一个独立进程
- 主要负责数据的存储，即存入数据和取出
  数据



**SecondaryNameNode:**

- NameNode的辅助，是一个独立进程
- 主要帮助NameNode完成元数据整理工作（打杂）





#### 分布式系统中两种常见的工作模式

1.分散汇总模式 (Scatter-Gather)

分散汇总模式（也称为 **分发-收集** 模式）是一种天然的并行计算模式，常用于需要并行处理大量数据的场景，例如 **MapReduce**。

| 步骤        | 名称                 | 执行动作                                                     | 关键特点                                                     |
| ----------- | -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 步骤 1      | 分散 (Scatter/Map)   | 主控节点（Master/Client）将一个大型任务或数据集 切分成若干独立的子任务或数据块，并将它们并行地发送给多个 工作节点（Worker/Slave）。 | 并行化：子任务之间相互独立，可以在不同的机器上同时运行。     |
| 步骤 步骤 2 | 执行 (Execute)       | 各个 工作节点 接收到自己的子任务后，独立地对分配到的数据块进行处理（例如，执行 Map 操作，数据清洗）。 | 无状态/本地计算：每个工作节点只关心自己的数据，通常不需要与其他节点通信。 |
| 步骤 3      | 汇总 (Gather/Reduce) | 各个 工作节点 完成计算后，将各自的 局部结果 返回给 主控节点 或一个指定的 汇集节点。 | 数据传输：涉及大量的网络 I/O，可能包含 Shuffle 过程（如果需要合并/分组）。 |
| 步骤 步骤 4 | 合并 (Merge)         | 主控节点/汇集节点 接收所有局部结果，执行最终的 合并、聚合或排序（例如，执行 Reduce 操作），得到最终的完整结果。 | 终态生成：生成用户所需的最终输出。                           |

**通俗总结：** 任务可以被完全切开，每个工人独立完成一部分，最后把各自的结果交上来，总指挥只负责最后相加。**速度快，但任务必须是可分割的。**



2.🚦 中心调度模式 (Centralized Orchestration)

中心调度模式（也常被称为 **编排** 模式）强调由一个**中心协调者** 来定义、管理和推动整个工作流的顺序和依赖关系。它适用于涉及多个、有明确先后顺序和依赖关系的服务调用或任务。

| 步骤        | 名称                             | 执行动作                                                     | 关键特点                                                 |
| ----------- | -------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------- |
| 步骤 1      | 定义工作流 (Workflow Definition) | 调度中心（Orchestrator/Master）预先加载或定义一个 有向无环图 (DAG)，明确所有任务（Task A, B, C...）的执行顺序和依赖关系。 | 任务依赖：任务 B 只有在任务 A 成功完成后才能开始。       |
| 步骤 步骤 2 | 启动与监控 (Trigger & Monitor)   | 调度中心 启动第一个 无依赖 的任务（如 Task A）。在任务执行过程中，调度中心会持续监控 任务的状态（运行中、成功、失败）。 | 状态管理：调度中心持有整个工作流的全局状态。             |
| 步骤 3      | 顺序驱动 (Sequential Drive)      | 当 调度中心 确认一个任务（如 Task A）成功完成 后，它会根据 DAG 触发其所有 下游依赖 的任务（如 Task B）。 | 单一决策点：所有任务的执行或重试都由调度中心决定和驱动。 |
| 步骤 步骤 4 | 反馈与结束 (Feedback & Complete) | 整个工作流直到所有任务都按顺序成功执行完毕，调度中心 宣布工作流完成。如果任何任务失败，调度中心负责根据策略进行 重试 或 告警。 | 流程控制：确保复杂业务流程的完整性、顺序性和可控性。     |

**通俗总结：** 任务之间有严格的 **先后顺序** 和 **依赖关系**。有一个 **总指挥（调度中心）** 像交警一样，严格控制和检查每个步骤是否完成，确保整个流水线顺利、按部就班地走下去。





## Yarn

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







## HIVE

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
