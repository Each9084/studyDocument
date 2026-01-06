## CREATE

在 SQL 的世界里，`CREATE` 属于 **DDL（数据定义语言）**。它不仅仅是用来建表，更是整个数据库系统的“架构师”。

要把 `CREATE` 讲透，我们需要从**数据库、表、索引、视图**这四个核心维度来系统梳理。

------

### 1. 顶层设计：CREATE DATABASE (建库)

在建表之前，得先有个“仓库”。

- **标准语法**：

  ```SQL
  CREATE DATABASE IF NOT EXISTS school_db 
  CHARACTER SET utf8mb4 
  COLLATE utf8mb4_general_ci;
  ```

- **关键细节**：

  - **`IF NOT EXISTS`**：职业习惯。防止脚本因为库已存在而报错中断。
  - **`CHARACTER SET utf8mb4`**：**必选**。这是目前最通用的编码，支持表情符号（Emoji）和各种生僻字。

------

### 2. 核心工程：CREATE TABLE (建表)

这是用得最频繁的部分。一个完美的 `CREATE TABLE` 语句由三部分组成：**字段定义、约束、引擎配置**。

#### ① 字段定义（数据类型选择）

- **整数**：`TINYINT` (开关), `INT` (普通), `BIGINT` (海量数据/ID)。
- **字符串**：`VARCHAR(n)` (变长，省空间), `CHAR(n)` (定长，查询快)。
- **时间**：`DATETIME` (绝对时间), `TIMESTAMP` (带时区，自动更新)。

#### ② 约束（数据规则）

- **`PRIMARY KEY`**：唯一标识，不可重复。
- **`NOT NULL`**：强制必填，避免后续复杂的 `NULL` 逻辑判断。
- **`UNIQUE`**：保证某列唯一（如手机号、身份证）。
- **`DEFAULT`**：设定默认值（如刚才聊到的 `(CURRENT_DATE)`）。

#### ③ 存储引擎（物理特性）

```SQL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

- **InnoDB**：现代数据库 99% 的选择，支持事务（数据不丢失）和外键。

------

### 3. CREATE INDEX (建索引 ) 

> 索引内容详见: **索引INDEX板块**

表建好了，但查询慢？你需要给搜索频率高的列建索引。

- **普通索引**：`CREATE INDEX idx_lastname ON actor(last_name);`
- **唯一索引**：`CREATE UNIQUE INDEX uk_email ON users(email);`
- **联合索引**：`CREATE INDEX idx_name ON actor(first_name, last_name);`
- **主键索引**
- **全文索引**

> **比喻**：表是书的内容，索引就是书开头的“目录”。没有目录你得翻整本书（全表扫描），有了目录你直接跳到指定页。

------

### 4. 逻辑视图：CREATE VIEW (建视图)

视图是一张“虚拟表”。它不存数据，只存一段 SQL 逻辑。

- **场景**：你写了一个超复杂的 5 表连接查询。

- **做法**：把这个查询封装成视图。

  ```sql
  CREATE VIEW view_actor_info AS 
  SELECT a.first_name, a.last_name, s.salary 
  FROM actor a JOIN salaries s ON a.actor_id = s.emp_no;
  ```

- **好处**：以后直接 `SELECT * FROM view_actor_info` 就像查单表一样简单，保护了原始表结构。

------

### 5. 职业级建表“三要素”规范

在正规公司里，建表脚本通常必须包含以下内容：

1. **自增主键**：一定要有 `id`，且建议使用 `BIGINT AUTO_INCREMENT`。
2. **审计字段**：每个表都该有 `create_time` 和 `update_time`。
3. **注释说明**：每个字段都必须写 `COMMENT`，否则三个月后你自己也看不懂。



### 一些细节补充

#### **①.更高级的“克隆”方式**

如果想创建一个和原表**结构完全一致**（包括索引、注释、字符集）的新表，而不仅仅是选几个字段，可以使用 `LIKE` 语法：



**方式 A：只克隆结构（不带数据）**

这是职业开发最常用的备份表结构的方法。

```sql
CREATE TABLE actor_name LIKE actor;
```

- **优点**：它会把 `actor` 表的所有细节（主键、自增、索引）原封不动地复制过来。



**方式 B：结构 + 数据 一步到位**

如果你想连数据带结构一起复制，可以使用 `AS SELECT`。

```sql
CREATE TABLE actor_name AS 
SELECT first_name, last_name FROM actor;
```





#### ②系统性理解：CREATE 的“派生”操作

当使用 `CREATE ... SELECT` 或 `INSERT ... SELECT` 时，实际上是在做**数据的流动**。

- **`INSERT INTO ... SELECT`**：要求目标表**必须先存在**。它像是一台往空盒子里装东西的机器。
- **`CREATE TABLE ... AS SELECT`**：它会**一边建盒子一边装东西**。



## ALTER

**`ALTER TABLE`** 是一个功能极其丰富的指令，它允许开发者在不删除表的情况下，动态地调整表的结构。我们可以从 **列操作、约束操作、表级别操作** 三个维度来系统梳理。



### 1.对“列”（Column）

这是最常用的场景，就像给房子加个插座或者换个窗户。

**增加列 (ADD)**：

```sql
ALTER TABLE actor ADD COLUMN phone VARCHAR(20) NOT NULL;
```

**修改列属性 (MODIFY)**： 改变字段的长度、数据类型或是否允许为空（不改名）。

```sql
ALTER TABLE actor MODIFY COLUMN first_name VARCHAR(100) NOT NULL;
```

**重命名列并修改属性 (CHANGE)**： MySQL 特有，可以同时改名和改属性。

```sql
ALTER TABLE actor CHANGE COLUMN first_name name_title VARCHAR(50);
```

**删除列 (DROP)**： **警告**：这是一个危险动作，删除后数据不可找回。

```sql
ALTER TABLE actor DROP COLUMN phone;
```



### 2. 对“约束/索引”（Constraints/Index）

用于调整表的数据规则，比如主键、外键和索引。

**添加/删除主键**：

```sql
ALTER TABLE actor ADD PRIMARY KEY (actor_id);
ALTER TABLE actor DROP PRIMARY KEY;
```

**添加唯一约束/普通索引**：

```sql
ALTER TABLE actor ADD UNIQUE (email);
ALTER TABLE actor ADD INDEX idx_last_name (last_name);
```

**删除索引**：

```sql
ALTER TABLE actor DROP INDEX idx_last_name;
```



### 3.对“表本身”的操作

**重命名表 (RENAME)**：

```sql
ALTER TABLE actor RENAME TO top_actors;
```

**修改表选项（如字符集、注释、自增起始值）**：

```sql
ALTER TABLE actor COMMENT = '演员基本信息表';
ALTER TABLE actor AUTO_INCREMENT = 1000; -- 下一个 ID 从 1000 开始
```





## INSERT

### 1. 基础插入：三种标准做法

假设我们的表结构包含：`actor_id` (自增), `first_name`, `last_name`, `last_update` (默认值)。

#### ① 全字段插入（最死板）

这种方式要求你必须按照表结构的顺序，把**所有**列的值都写出来。

SQL

```sql
INSERT INTO actor VALUES (1, 'PENELOPE', 'GUINESS', '2026-01-04');
```

- **缺点**：如果以后表加了一个字段，这段代码就会报错。**不推荐在生产环境使用。**

#### ② 指定字段插入（最推荐 ✨）

只给需要的列传值，顺序可以自己定。没写的列会自动使用**默认值**或 **NULL**。

SQL

```sql
INSERT INTO actor (first_name, last_name) 
VALUES ('NICK', 'WAHLBERG');
```

- **优点**：代码健壮性强。即使表增加了新列，这段 SQL 依然有效。

#### ③ 批量插入（最高效）

用一个 `VALUES` 后面跟多个括号，一次性插入多条。

SQL

```sql
INSERT INTO actor (first_name, last_name) VALUES 
('ED', 'CHASE'),
('JENNIFER', 'DAVIS'),
('JOHNNY', 'LOLLOBRIGIDA');
```

- **性能提升**：比执行三次单条插入快得多，因为减少了与数据库的连接握手次数。



### 2. 进阶技巧：处理“冲突”

当尝试插入一个主键 ID 已经存在的记录时，普通的 `INSERT` 会报错。MySQL 提供了如下几种方案：

#### ① `INSERT IGNORE`

如果数据冲突（主键重复），它不报错，而是直接跳过这一行，继续插后面的。

```sql
INSERT IGNORE INTO actor (actor_id, first_name, last_name) 
VALUES (1, 'NEW', 'NAME'); -- 如果 ID=1 已存在，这行就被无视了
```

#### ② `ON DUPLICATE KEY UPDATE`（存在即更新）

这是最强大的功能：**“如果没有就插入，如果有就更新它。”**

```sql
INSERT INTO actor (actor_id, first_name, last_name) 
VALUES (1, 'PENELOPE', 'NEW_LAST_NAME')
ON DUPLICATE KEY UPDATE last_name = 'NEW_LAST_NAME';
```

- **形象理解**：这就像是在同步数据，确保数据库里的信息始终是最新的。



### 3. 从另一张表“搬运”数据：`INSERT INTO ... SELECT`

有时候需要把 `table_A` 的数据导入 `table_B`。

```sql
INSERT INTO actor_archive (actor_id, first_name, last_name)
SELECT actor_id, first_name, last_name 
FROM actor 
WHERE last_update < '2025-01-01';
```

- **场景**：数据迁移、备份旧数据。

------

### 4. 执行原理

当执行一次 `INSERT` 时，数据库后台发生了这些事：

1. **语法校验**：检查 SQL 写对没，表存不存在。
2. **约束检查**：检查 `NOT NULL` 是不是填了？主键是不是重复了？
3. **索引更新**：主键和索引都需要更新 B+ 树结构。
4. **写入 Redo Log**：为了防止宕机，先在日志里记一笔。
5. **提交写入**：最后才真正写进磁盘。





## COUNT

### 1.`COUNT(DISTINCT ...)`,过滤计数

这是面试和报表中最常考的。它不是数“有多少行”，而是数“**有多少种**”。

- **场景**：统计每个部门有多少个**不同的**职称（不管这些职称下有多少人）。
- **写法**：

```SQL
SELECT dept_no, COUNT(DISTINCT title) 
FROM titles 
GROUP BY dept_no;
```



### 2. `COUNT(IF ...)` 或 `COUNT(CASE ...)` ,条件计数

这是 `COUNT` 最强大的进阶用法。它能让你在**同一行结果**里，统计出不同分类的数量。

- **场景**：你想统计每个部门的总人数，以及其中“高级工程师（Senior Engineer）”的人数。

- **逻辑**：如果符合条件，就给 `COUNT` 一个值；如果不符合，就给一个 `NULL`（记得吗？`COUNT(列名)` 会无视 NULL）。

- **写法**：

  ```SQL
  SELECT 
      dept_no, 
      COUNT(*) AS 总人数,
      COUNT(CASE WHEN title = 'Senior Engineer' THEN 1 END) AS 高工人数
  FROM titles
  GROUP BY dept_no;
  ```

  

### 3. `COUNT(1)` 实现全表扫描检查

在性能优化时，资深开发会用 `COUNT(1)`。

- 其实 `COUNT(1)`、`COUNT(*)`、`COUNT(主键)` 在现代数据库（如 MySQL）里速度几乎一样。
- 但如果你写 `COUNT(具体列名)`，数据库必须去检查那一列是不是 `NULL`。如果你只是想快速点名，用 `COUNT(*)` 是最专业的做法，因为它会利用索引（Index）直接给出答案，而不需要去翻看每一行的具体内容。



## CONCAT

**有时我们会遇到新的需求:**

请将employees表的所有员工的last_name和first_name字段拼接起来作为Name，中间以一个空格区分。



这时候就需要用到`CONCAT`

在 MySQL 中，`CONCAT` 系列函数是处理字符串拼接的核心工具。虽然看起来简单，但在处理**空值（NULL）**、**分隔符**以及**复杂报表**时，有非常具体的逻辑。





在 MySQL 中，`CONCAT` 系列函数是处理字符串拼接的核心工具。虽然看起来简单，但在处理**空值（NULL）**、**分隔符**以及**复杂报表**时，有非常具体的逻辑。

我们可以把 MySQL 的拼接功能系统性地拆解为三个部分：



### 1. 基础拼接：`CONCAT(s1, s2, ...)`

这是最常用的基本函数，可以将任意数量的字符串连接在一起。

- **语法**：`CONCAT(string1, string2, ...)`
- **特性**：
  - 它可以接受多个参数（不仅仅是两个）。
  - **致命弱点**：只要参数中有一个是 `NULL`，整个函数的结果就会变成 `NULL`。

**❌ 潜在坑点演示：** 假设某员工没有中间名（Middle Name 为 NULL）：

```sql
SELECT CONCAT('张', NULL, '三'); 
-- 结果是 NULL，而不是 '张三'。这就是为什么很多初学者发现数据“无故消失”的原因。
```



### 2. 智能分隔拼接：`CONCAT_WS(separator, s1, s2, ...)`

所以为了解决上面可能为null的问题们提出了`CONCAT_WS`

`WS` 是 **"With Separator"**（带分隔符）的缩写。这是处理格式化输出（如姓名、地址）的最佳选择。

- **语法**：`CONCAT_WS(分隔符, 字符串1, 字符串2, ...)`
- **三大优势**：
  1. **统一分隔**：你只需要在第一个参数写一次空格（或逗号），后面所有的间隙都会自动填充该符号。
  2. **自动处理 NULL**：它会**自动忽略** NULL 值，而不会让结果变成 NULL。
  3. **简洁**：不需要像 `CONCAT` 那样反复写空格。

**✅ 实战演示：**

```sql
-- 即使 last_name 是 NULL，它也会正常显示 first_name，不会报错
SELECT CONCAT_WS(' ', last_name, first_name) AS Name FROM employees;
```

在 MySQL 中，`CONCAT` 系列函数是处理字符串拼接的核心工具。虽然看起来简单，但在处理**空值（NULL）**、**分隔符**以及**复杂报表**时，有非常具体的逻辑。

我们可以把 MySQL 的拼接功能系统性地拆解为三个部分：

------

### 1. 基础拼接：`CONCAT(s1, s2, ...)`

这是最常用的基本函数，可以将任意数量的字符串连接在一起。

- **语法**：`CONCAT(string1, string2, ...)`
- **特性**：
  - 它可以接受多个参数（不仅仅是两个）。
  - **致命弱点**：只要参数中有一个是 `NULL`，整个函数的结果就会变成 `NULL`。

**❌ 潜在坑点演示：** 假设某员工没有中间名（Middle Name 为 NULL）：

SQL

```
SELECT CONCAT('张', NULL, '三'); 
-- 结果是 NULL，而不是 '张三'。这就是为什么很多初学者发现数据“无故消失”的原因。
```

------

### 2. 智能分隔拼接：`CONCAT_WS(separator, s1, s2, ...)`

`WS` 是 **"With Separator"**（带分隔符）的缩写。这是处理格式化输出（如姓名、地址）的最佳选择。

- **语法**：`CONCAT_WS(分隔符, 字符串1, 字符串2, ...)`
- **三大优势**：
  1. **统一分隔**：你只需要在第一个参数写一次空格（或逗号），后面所有的间隙都会自动填充该符号。
  2. **自动处理 NULL**：它会**自动忽略** NULL 值，而不会让结果变成 NULL。
  3. **简洁**：不需要像 `CONCAT` 那样反复写空格。

**✅ 实战演示：**

SQL

```
-- 即使 last_name 是 NULL，它也会正常显示 first_name，不会报错
SELECT CONCAT_WS(' ', last_name, first_name) AS Name FROM employees;
```





### 3. 分组聚合拼接：`GROUP_CONCAT()`

这是 `CONCAT` 的**高级形态**，通常配合 `GROUP_BY` 使用。它能把**多行**的数据拼接成**一行**里的一个字符串。

- **场景**：你想知道每个部门有哪些员工，且想把名字列在一起。
- **语法**：`GROUP_CONCAT(要拼接的列 [ORDER BY 排序] [SEPARATOR '分隔符'])`

**📊 形象图解：**

**代码演示：**

```sql
SELECT dept_no, GROUP_CONCAT(last_name ORDER BY last_name SEPARATOR '; ') AS members
FROM dept_emp
GROUP BY dept_no;

-- 结果显示：
-- d001 | 张三; 李四; 王五
-- d002 | 赵六; 孙七
```



## 索引Index

索引（Index）是数据库中**提升查询性能最核心的工具**。如果把数据库看作一本厚厚的字典，索引就是字典前面的“音序表”或“部首检字表”。

要系统性地掌握索引，需要从**底层原理、分类、设计原则**三个维度来拆解。

### 1. 底层原理：为什么索引快？

数据库的数据是存在磁盘上的。如果没有索引，数据库为了找一条记录，必须进行**全表扫描（Full Table Scan）**，即把所有数据页从磁盘读入内存，挨个比对。

**B+ 树（B+ Tree）：MySQL 索引的灵魂** MySQL（InnoDB引擎）主要使用 **B+ 树** 结构。

- **非叶子节点**：只存储索引字段的值（类似书的目录页，只写着标题）。
- **叶子节点**：存储实际的数据或主键（类似书的正文页）。
- **优势**：树的高度通常只有 3-4 层。这意味着即使在千万级的数据中找一条记录，也只需要 **3 到 4 次磁盘 IO** 就能定位到。



索引（Index）是数据库中**提升查询性能最核心的工具**。如果把数据库看作一本厚厚的字典，索引就是字典前面的“音序表”或“部首检字表”。

要系统性地掌握索引，我们需要从**底层原理、分类、设计原则**三个维度来拆解。



### 2. 索引的分类

索引根据不同的维度有不同的叫法，容易搞混。我们分类梳理：

#### ① 按功能逻辑分类

- **普通索引 (Normal Index)**：只能通过**一个列**创建,是最基本的索引，没有任何限制,只是为了加速查询。。

```sql
-- 方式一：直接创建
CREATE INDEX idx_last_name ON actor(last_name);

-- 方式二：修改表结构
ALTER TABLE actor ADD INDEX idx_last_name (last_name);
```



- **唯一索引 (Unique Index)**：列中的值必须唯一，但允许为 NULL。常用于手机号、身份证号。

```sql
-- 方式一：直接创建
CREATE UNIQUE INDEX uk_first_name ON actor(first_name);

-- 方式二：修改表结构
ALTER TABLE actor ADD UNIQUE INDEX uk_first_name (first_name);
```



- **主键索引 (Primary Key)**：一种特殊的唯一索引，不允许有 NULL 值。通常在建表时指定，每张表只能有一个。

```sql
-- 通常在建表时直接指定
ALTER TABLE actor ADD PRIMARY KEY (actor_id);
```



- **全文索引 (Full-text Index)**：用于在长文本（如 `TEXT` 类型）中搜索关键词，类似于搜索引擎(百度搜索）。

```sql
-- 仅支持 CHAR, VARCHAR, TEXT 类型
ALTER TABLE actor ADD FULLTEXT INDEX ft_info (first_name, last_name);

-- 查询时使用特殊语法：MATCH...AGAINST
SELECT * FROM actor WHERE MATCH(first_name, last_name) AGAINST('NICK');
```



- **复合索引（Composite Index）**:在创建索引时，同时关联了**两个或两个以上**的列。

```sql
-- 顺序非常重要：先按 last_name 排，再按 first_name 排
CREATE INDEX idx_full_name ON actor(last_name, first_name);
```



#### ② 按物理存储分类（重点）

- **聚簇索引 (Clustered Index)**：
  - **定义**：索引的叶子节点里直接存的就是**整行数据**。
  - **特点**：一张表只能有一个（通常是主键）。
- **二级索引 / 非聚簇索引 (Secondary Index)**：
  - **定义**：叶子节点存的是索引字段的值 + **对应的主键 ID**。
  - **回表查询**：如果你通过二级索引找数据，SQL 会先找到主键 ID，再回主键索引里找整行记录。这个过程叫“回表”。



### 3.索引的使用

**查看表中有哪些索引：**

```sql
SHOW INDEX FROM Table;
```

**删除索引：**

```sql
DROP INDEX idx_last_name ON Table;
-- 或者
ALTER TABLE actor DROP INDEX idx_last_name;
```

#### 强制索引:

**查询优化器（Optimizer）** 会自动根据统计数据（数据量、区分度、IO 成本等）来决定使用哪个索引，或者干脆不走索引。但有的时候可能用户想要自行指定规则,那么就可以使用**` FORCE INDEX`**

**用法:**

强制索引紧跟在 `FROM` 子句中的表名之后：

```sql
SELECT * FROM actor 
FORCE INDEX (idx_lastname) -- 强制告诉 MySQL 必须用这个索引
WHERE last_name = 'GUINESS' AND first_name = 'PENELOPE';
```



**三种干预手段的对比**

MySQL 提供了三个类似的指令，力度由轻到重：

| **指令**         | **含义**                                                     | **力度**   |
| ---------------- | ------------------------------------------------------------ | ---------- |
| **USE INDEX**    | 建议 MySQL 使用这个索引，但它如果觉得全表扫描更快，可以不理你。 | 建议级     |
| **IGNORE INDEX** | 强制 MySQL **禁止**使用这个索引。                            | 禁止级     |
| **FORCE INDEX**  | 强制 MySQL 使用这个索引，除非该查询物理上根本没法用这个索引（比如没这列）。 | **强制级** |



**一般场景使用建议：先诊断，再强制**

如果发现查询慢，不要上来就 `FORCE INDEX`。标准的“手术流程”应该是：

1. **执行 `EXPLAIN`**：看看 `type` 是不是 `ALL`，`possible_keys` 里有没有你预想的索引。
2. **执行 `ANALYZE TABLE 表名`**：这会更新表的统计信息。很多时候，运行一下这个，优化器就自己变聪明了，不需要写强制索引。
3. **检查索引区分度**：如果索引列本身重复值太多，强制索引也没意义。



### 4. 索引的设计原则：哪些列该建索引？

索引不是越多越好，因为索引本身占空间，且每次 `INSERT/UPDATE` 都要维护索引树，会变慢。

**✅ 适合建索引的情况：**

1. **经常出现在 `WHERE` 条件中的列**。
2. **用于 `JOIN` 连接的列（外键）**。
3. **经常需要 `ORDER BY`、`GROUP BY` 或 `DISTINCT` 的列**（因为索引本身就是排好序的）。
4. **区分度高的列**（例如：身份证号适合，性别只有男女，就不太适合）。

**❌ 不适合建索引的情况：**

1. **数据量很小的表**（全表扫描可能比查索引还快）。
2. **经常更新的列**。
3. **查询中很少使用的列**。



### 5. 复合索引与“最左匹配原则”

如果你给 `(first_name, last_name)` 建了一个复合索引，它就像是一个**二级目录**。

- **规则**：必须先查 `first_name`，才能用到索引。
- **例子**：
  - `WHERE first_name = 'Nick'` (✅ 走索引)
  - `WHERE first_name = 'Nick' AND last_name = 'Wahlberg'` (✅ 走索引)
  - `WHERE last_name = 'Wahlberg'` (❌ **不走索引**，因为你跳过了第一级目录)



### 6. 性能调试神器：`EXPLAIN`

想知道你的索引有没有生效？在 SQL 语句前加上 `EXPLAIN`。

```sql
EXPLAIN SELECT * FROM actor WHERE last_name = 'GUINESS';
```

- **type**: `ref` 或 `eq_ref` 说明索引生效了；`ALL` 说明在全表扫描。
- **key**: 实际使用的索引名称。
- **rows**: 预计扫描的行数，越小越好。



## 视图（View）

视图（View）是数据库设计中非常有用的“高级工具”。简单来说，**视图就是一张“虚拟表”**。它在数据库里不存储实际的数据，只存储一段 **SQL 查询逻辑**。

当开发者查询视图时，数据库后台会自动跑一遍这段 SQL，然后把结果像表格一样呈现给开发者。



### 1. 为什么要用视图？

如果你发现自己每天都在写同一个复杂的 `JOIN` 查询，或者为了安全不想让新人看到员工的薪资细节，视图就是最佳解决方案。

#### ① 简化复杂的查询（封装）

如果你有一个 5 表关联的统计查询，代码长达 50 行。你可以把它存为视图。

- **之前**：每次都要写那 50 行 SQL。
- **之后**：`SELECT * FROM my_view;`（一行搞定）。

#### ② 数据安全性（脱敏）

你可以只给用户访问视图的权限，而不给原始表的权限。

- **原始表**：有 `emp_no`, `name`, `salary`, `home_address`。
- **视图**：只选 `name` 和 `dept_no`。这样敏感信息（薪资、地址）就被隔离了。

#### ③ 逻辑独立性

如果数据库后台改了表结构（比如把一个大表拆成了两个小表），你只需要修改视图的 SQL 定义，而前端程序的代码完全不需要动。



### 2. 视图的创建与管理

#### ① 创建视图

语法非常直观：`CREATE VIEW 视图名 AS SELECT ...`

```SQL
CREATE VIEW v_actor_info AS
SELECT a.actor_id, CONCAT(a.first_name, ' ', a.last_name) AS full_name, f.title
FROM actor a
JOIN film_actor fa ON a.actor_id = fa.actor_id
JOIN film f ON fa.film_id = f.film_id;
```

#### ② 使用视图

像查普通表一样查它：

```SQL
SELECT * FROM v_actor_info WHERE full_name LIKE 'A%';
```

#### ③ 修改和删除

- **修改**：`CREATE OR REPLACE VIEW 视图名 AS ...`
- **删除**：`DROP VIEW 视图名;`



### 3. 视图的底层原理：它是如何工作的？

当你执行 `SELECT * FROM v_actor_info` 时，数据库并不是去读一个叫 `v_actor_info` 的文件，而是执行了以下步骤：

1. **查询改写**：数据库发现这是一个视图，于是把视图的定义拿出来。
2. **合并 SQL**：把你当前的查询条件（比如 `WHERE id=1`）和视图原本的 SQL 合并在一起。
3. **执行**：最终去原始表（物理表）里拿数据。

### 4. 视图的“禁区”：能修改数据吗？

这是一个常见的面试题。**答案是：视情况而定，但通常不建议。**

- **可更新视图**：如果视图是简单的 `SELECT * FROM table`，你可以通过 `UPDATE` 视图来改原始表。
- **不可更新视图**：如果视图包含以下任何内容，就**不能**修改数据：
  - 聚合函数（`SUM`, `COUNT`, `AVG` 等）
  - `DISTINCT`
  - `GROUP BY`
  - `UNION`
  - 复杂的 `JOIN`

> **最佳实践**：始终把视图当作**“只读”**的工具，数据的增删改（CUD）应该直接对原始表操作。



### 5. 视图 vs. 物化视图 (Materialized View)

在一些高级数据库（如 Oracle, PostgreSQL）中，还有一种东西叫**物化视图**。

| **特性**   | **普通视图 (View)**            | **物化视图 (Materialized View)** |
| ---------- | ------------------------------ | -------------------------------- |
| **存储**   | 只存 SQL 语句                  | **存真实的计算结果**             |
| **性能**   | 每次查询都要重新计算           | 查询极快（直接读结果）           |
| **实时性** | 实时（原始表变了，视图立刻变） | 有延迟（需要手动或定时刷新）     |

*注：MySQL 官方原生不支持物化视图，通常通过临时表或触发器手动模拟。*





## 触发器（Trigger）

在数据库的世界里，**触发器（Trigger）** 就像是安装在表上的“自动感应报警器”或“自动化脚本”。它是一种特殊的存储过程，但它不需要你手动调用，而是由特定的**数据库事件**自动触发执行。



### 1. 定义

触发器是绑定在某张表上的程序。当该表发生 **INSERT（插入）**、**UPDATE（更新）** 或 **DELETE（删除）** 操作时，数据库会自动执行触发器中定义的 SQL 逻辑。



### 2. 四要素

要定义一个触发器，必须明确四个核心点：

1. **监视表（Table）**：触发器安装在哪张表上？
2. **监视事件（Event）**：是 `INSERT`、`UPDATE` 还是 `DELETE`？
3. **触发时间（Timing）**：是在动作发生**之前（BEFORE）还是之后（AFTER）**？
4. **触发逻辑（Action）**：触发后要执行什么 SQL 代码？



### 3. 语法展示

以 MySQL 为例，创建一个触发器的标准模板如下：

```sql
CREATE TRIGGER 触发器名称
{BEFORE | AFTER} {INSERT | UPDATE | DELETE}
ON 表名 FOR EACH ROW
BEGIN
    -- 这里写你的逻辑代码
END;
```

#### 关键关键字：`NEW` 与 `OLD`

在触发器内部，有两个非常重要的虚拟表：

- **`NEW`**：代表即将插入或已经更新后的“新数据”。（适用于 INSERT 和 UPDATE）
- **`OLD`**：代表即将删除或更新前的“老数据”。（适用于 UPDATE 和 DELETE）



### 4. 应用场景

#### ① 数据审计（操作日志）

这是最常见的用法。当有人修改了 `salary` 表时，自动在 `audit_log` 表里记录谁在什么时候改了多少钱。

#### ② 业务逻辑自动补全（数据同步）

例如：每当 `orders` 表插入一笔新订单，触发器自动去 `inventory` 表里把对应商品的库存减掉。

#### ③ 数据校验与强制约束

虽然我们可以用 `CHECK` 约束，但触发器可以实现更复杂的逻辑。比如：如果发现更新后的工资涨幅超过 50%，直接报错拦截。

```sql
-- 逻辑示例
IF (NEW.salary > OLD.salary * 1.5) THEN
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = '涨薪幅度过大，拒绝更新';
END IF;
```

------

### 5. 触发器的分类对比

| **触发时间** | **典型用途**                                                 |
| ------------ | ------------------------------------------------------------ |
| **BEFORE**   | **检查或修改数据**。在数据进入表之前拦截，比如检查格式或给空字段填默认值。 |
| **AFTER**    | **后续连锁反应**。数据已经成功入库，去更新其他表（如增加积分、记录日志）。 |

------

### 6. 优缺点分析（避坑指南）

#### ✅ 优点：

- **自动化**：保证了业务逻辑的强制执行，不会因为程序员忘了写日志代码而漏记。
- **数据完整性**：可以跨表保证数据的一致性。

#### ❌ 缺点（职业警告）：

- **性能开销**：每一行数据的变动都要运行触发器，在大批量导入数据时会显著拖慢速度。
- **隐蔽性强**：触发器是“悄悄”运行的。新来的同事可能查遍了代码都没发现为什么库存会自动变，增加了调试难度。
- **逻辑死循环**：如果不小心写了“表 A 触发改表 B，表 B 触发改表 A”，可能会导致服务器崩溃。

------

###  总结：什么时候用？

- 如果是简单的默认值，用 **`DEFAULT`**。
- 如果是简单的唯一性，用 **`UNIQUE`**。
- 如果是复杂的**跨表联动**或**强制审计**，才考虑用 **`TRIGGER`**。





## SQL的书写顺序（Syntax Order）

1. **SELECT**：指定要返回的列
2. **FROM**：指定要查询的表
3. **WHERE**：对表中的记录进行过滤
4. **GROUP BY**：将数据分组
5. **HAVING**：对分组后的结果进行过滤
6. **ORDER BY**：对结果集进行排序
7. **LIMIT** / **OFFSET**：限制返回的行数

## SQL 的真实执行顺序：

SQL 不是从上到下执行的，而是按照这个顺序：

1. **FROM** 和 **JOIN**（确定数据来源）
2. **WHERE**（过滤行）
3. **GROUP BY**（分组）
4. **HAVING**（过滤分组）
5. **SELECT**（选择列+计算表达式+定义别名）
6. **ORDER BY**（排序）
7. **LIMIT**（限制结果）



值得注意的是:**在大多数 SQL 数据库中，GROUP BY 确实在 SELECT 之前执行，但 GROUP BY 子句中可以使用 SELECT 中定义的别名，这是一个语法糖。**

例如:

```mysql
SELECT 
    CASE 
        WHEN age >= 25 THEN "25岁及以上"
        WHEN age < 25 OR age IS NULL THEN "25岁以下"
    END AS age_cut,
    COUNT(id) as number 
FROM user_profile 
GROUP BY age_cut;
```



哪些地方可以使用 SELECT 别名？

✅ 可以使用的：

- **GROUP BY**
- **ORDER BY**
- **HAVING**（在某些数据库中）





## SQL JOIN

![sqlJoin](../studyDoc/assets/javaAsstes/sqlJoin.jpg)





## SQL的SELECT和GROUP BY中的内容

> 问: 为什么要GROUP BY d.dept_no, d.dept_name而不是d.dept_no一个就能满足?
>
> 遇到这种情况 我要怎么知道要写哪些上去?



```
SELECT d.dept_no,d.dept_name,COUNT(s.emp_no) as sum 
FROM departments d JOIN dept_emp de 
ON d.dept_no = de.dept_no
JOIN salaries s 
ON de.emp_no = s.emp_no
GROUP BY d.dept_no, d.dept_name
ORDER BY d.dept_no;
```



这是一个非常棒的问题！很多初学者甚至是已经工作的开发者，都会对这一点感到疑惑：**“既然一个 `dept_no` 就能唯一确定一个部门了，为什么非要把 `dept_name` 也塞进 `GROUP BY` 里？”**

答案分为两个层面：一个是**为了通过“SQL 警察”的检查**（语法规范），另一个是**为了防止“数据分裂”**（逻辑严谨）。

------

### 1. 语法金律：非聚合列必须进 GROUP BY

在标准 SQL（SQL-92 标准及以后）中，有一个铁律：

> **如果在 `SELECT` 中出现了一个字段，且它没有被包裹在聚合函数（如 `SUM`, `COUNT`, `MAX`）里，那么它必须出现在 `GROUP BY` 子句中。**

为什么会有这个规定？

想象一下，如果一个 dept_no 对应了两个不同的 dept_name（虽然在数据库设计上这不合理，但在 SQL 引擎看来是有可能的）。

如果你只按 dept_no 分组，SQL 引擎在处理 SELECT dept_name 时就会陷入混乱：“这一组里有俩名字，你到底要我显示哪一个？”

为了消除这种“不确定性”，标准 SQL 要求你把所有要显示的非聚合字段都写进 `GROUP BY`。

------

### 2. 我怎么知道要写哪些字段上去？

你不需要去背，只需要遵循一个简单的**“过滤检查法”**：

#### 第一步：看你的 `SELECT`

列出你 SELECT 后面所有的字段。

例如：SELECT d.dept_no, d.dept_name, COUNT(s.salary)

#### 第二步：剔除“带函数”的

把带 SUM()、COUNT()、AVG() 等函数的字段划掉。

剩下：d.dept_no, d.dept_name

#### 第三步：无脑复制

剩下的这些，**全部、一个不漏地**写到 `GROUP BY` 后面。

------

### 3. 特殊情况：MySQL 的“宽容” (MySQL 5.7 以前)

你可能会发现，在某些 MySQL 环境下，只写 `GROUP BY d.dept_no` 竟然**不报错**且能跑出正确结果。

这是因为 MySQL 有一个开关叫 `ONLY_FULL_GROUP_BY`。

- **如果关掉它**：MySQL 会“自作聪明”地从每个组里随便挑一个 `dept_name` 显示出来。
- **如果开启它（现代数据库默认开启）**：它会严格遵循标准 SQL，强制要求你写全。

职业建议：

永远按照“写全”的标准来要求自己。因为这样你的代码在 Oracle、PostgreSQL、SQL Server 等所有数据库之间都是通用的。

------

### 4. 逻辑上的“保底”

有时候，两个不同的部门可能叫同一个名字（比如两个不同的校区都有“后勤部”）。

- 如果你只 `GROUP BY dept_name`：这两个部门会被**强行合并**成一行，数据就错了。
- 如果你写 `GROUP BY dept_no, dept_name`：由于 ID 不同，它们会被正确地分为两行显示。

------

###  总结

> **“选什么，分什么；算什么，留什么。”**

- **选什么（SELECT）**：选出的普通字段，都要去 `GROUP BY` 报道。
- **分什么（GROUP BY）**：分类的依据。
- **算什么（聚合函数）**：不需要去 `GROUP BY`。

