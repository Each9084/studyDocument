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



## UPDATE

`UPDATE` 语句是用于修改数据库表中**已存在记录**的指令。如果说 `INSERT` 是“增加”，`DELETE` 是“移除”，那么 `UPDATE` 就是“整容”和“修正”。

为了让你系统性地掌握它，我们将从**基础语法、条件控制、多表联动、以及实战安全规则**四个维度展开讲解。

### 1. 基础语法：精准修改

`UPDATE` 的基本逻辑是：**定位行 → 修改列 → 写入新值**。

```sql
UPDATE 表名
SET 列1 = 值1, 列2 = 值2, ...
WHERE 条件;
```

- **`SET`**：指定要更新的列名。你可以同时修改多列，用逗号隔开。
- **`WHERE`**：极其重要！它决定了哪些行会被修改。**如果不写 `WHERE`，全表的所有数据都会被修改。**

**示例：** 将工号为 `1001` 的员工薪水调为 `5000`，职位改为 `Manager`。

```SQL
UPDATE employees
SET salary = 5000, job_title = 'Manager'
WHERE emp_id = 1001;
```



### 2. 进阶用法：动态计算

`UPDATE` 不仅可以填入死数据，还可以利用原有的值进行动态计算。

- **原值累加**：

  ```SQL
  -- 给研发部所有员工涨薪 10%
  UPDATE employees 
  SET salary = salary * 1.1 
  WHERE dept_id = 'RD';
  ```

- **配合函数**：

  ```SQL
  -- 将所有演员的名字转换为大写
  UPDATE actor 
  SET first_name = UPPER(first_name);
  ```

------

### 3. 多表联动更新（Update Join）

当你需要根据**另一张表**的信息来更新当前表时，单表 `UPDATE` 就不够用了。这时需要结合 `JOIN`。

**场景：** 根据“折扣表（discounts）”里的比例，更新“产品表（products）”的现价。

```SQL
UPDATE products p
INNER JOIN discounts d ON p.category_id = d.category_id
SET p.price = p.price * d.discount_rate
WHERE d.status = 'active';
```

- **逻辑**：先通过 `JOIN` 将两张表关联，然后根据关联后的结果集进行针对性修改。

------

### 4. 嵌套子查询更新

有时候我们需要根据子查询的结果来定位更新目标。

**注意（MySQL 限制）**：MySQL 不允许在 `UPDATE` 的子查询中直接引用目标表。

- **❌ 报错写法**：`UPDATE A SET col = 1 WHERE id = (SELECT MAX(id) FROM A);`

- **✅ 修正写法（套娃法）**：

  ```SQL
  UPDATE titles_test 
  SET title = 'Senior' 
  WHERE id = (
      SELECT max_id FROM (
          SELECT MAX(id) AS max_id FROM titles_test
      ) AS tmp
  );
  ```

------

### 5. 执行原理与性能建议

1. **行级锁（Row Locking）**：

   - 在 InnoDB 引擎中，如果你通过**索引列**（如主键）进行 `UPDATE`，数据库只会锁定那几行（行锁）。
   - 如果你没有用到索引，数据库可能会升级为**表锁**，导致整个表在更新期间无法被其他人访问。

2. **事务回滚**： `UPDATE` 操作通常放在事务中，防止操作中途出错无法恢复：

   ```SQL
   BEGIN; -- 开始
   UPDATE ...;
   ROLLBACK; -- 发现改错了，撤销
   COMMIT; -- 确认无误，提交
   ```

3. **大表分批处理**： 如果要更新 1000 万行数据，一次性执行会导致长时间锁表。**建议分批更新**：

   ```SQL
   -- 配合 LIMIT 每次更新 5000 条
   UPDATE orders SET status = 'closed' WHERE order_date < '2020-01-01' LIMIT 5000;
   ```

------

### 6. 职业级安全规则（保命准则）

在真实的生产环境中，直接写 `UPDATE` 是非常危险的。请遵循以下流程：

1. **先改 SELECT**：在执行 `UPDATE` 前，先把 `UPDATE` 换成 `SELECT *` 运行一遍，看看查出来的行是不是你想改的那几行。

2. **检查 Affected Rows**：执行后，查看数据库返回的“受影响行数”。如果只想改 1 行却返回了 10000 行，说明 `WHERE` 条件写错了，赶紧检查！

3. **开启安全模式**：

   ```SQL
   SET sql_safe_updates = 1; 
   ```

   开启后，如果你写的 `UPDATE` 语句没有带 `WHERE` 条件或者没有用到主键索引，MySQL 会拒绝执行，防止误删/误改全表。



## ALTER

**`ALTER TABLE`** 是一个功能极其丰富的指令，它允许开发者在不删除表的情况下，动态地调整表的结构。我们可以从 **列操作、约束操作、表级别操作** 三个维度来系统梳理。



### **1.对“列”（Column）**

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



## REPLACE

`REPLACE` 是一个非常有意思的关键字，它其实有**两种完全不同**的用法：一种是**修改字符串的内容**，另一种是**“替换式”插入数据**。

### 维度一：字符串替换函数 `REPLACE()`

当你需要批量修改字段中的某些字符（比如把手机号中间四位换成 `*`，或者纠正错别字）时，使用这个函数。

**1. 语法**

```sql
REPLACE(str, from_str, to_str)
```

- **str**：原始字符串（字段名）。
- **from_str**：需要被替换的子串。
- **to_str**：替换后的新子串。

**2. 实战示例**

假设你要把 `address` 字段里所有的“街道”改为“路”：

```sql
UPDATE actor 
SET address = REPLACE(address, '街道', '路')
WHERE address LIKE '%街道%';
```



### 维度二：替换式插入语句 `REPLACE INTO`

这是 MySQL 的一个**增强型插入指令**。它的逻辑逻辑非常霸道：**“如果不存在就插入；如果已经存在，就先删掉，再插入新的。”**

**1. 为什么需要它？**

普通的 `INSERT` 遇到主键（Primary Key）或唯一索引（Unique Index）冲突时会直接报错报错中止。而 `REPLACE INTO` 会自动处理这种冲突。

**2. 执行逻辑**

1. **尝试插入**新记录。
2. **发生冲突**：如果发现主键或唯一索引重复。
3. **删除**：把表里那行旧数据杀掉。
4. **再插入**：把当前的新数据存进去。

**3. 语法示例**

SQL

```SQL
REPLACE INTO actor (actor_id, first_name, last_name) 
VALUES (1, 'PENELOPE', 'NEW_LAST_NAME');
```

------

### **3. 两种对比**

这是很多面试官喜欢考的细节。虽然它们都能解决冲突，但底层动作完全不同：

| **特性**       | **REPLACE INTO**                                         | **INSERT ... ON DUPLICATE KEY UPDATE** |
| -------------- | -------------------------------------------------------- | -------------------------------------- |
| **底层动作**   | **DELETE + INSERT**                                      | **UPDATE**                             |
| **自增 ID**    | **会变**（旧行删了，新行会产生新 ID，除非你手动指定 ID） | **不变**（只是在原行修改数据）         |
| **性能**       | 较慢（涉及两次 IO：删除和插入）                          | 较快（只是一次更新动作）               |
| **受影响行数** | 如果发生了替换，返回 `2`（删 1 行，插 1 行）             | 如果发生了更新，返回 `2`               |



**4. 重点避坑指南**

1. **触发器失效**：由于 `REPLACE INTO` 的本质包含一个 `DELETE` 动作，如果你表上挂了 `DELETE` 触发器，它会被意外触发。
2. **外键级联**：如果你的表被其他表外键关联，`REPLACE INTO` 删掉旧行时可能会导致关联表的数据被级联删除或报错。
3. **默认值丢失**：由于它是先删再插，如果你在 `REPLACE` 语句中没有给某些字段赋值，那些字段会变回**默认值**（或者是 NULL），而不会保留旧数据的值。







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



**详细讲解:**

要实现这种“折叠”效果，你需要使用一个专门处理字符串聚合的函数：**`GROUP_CONCAT`**（这是 MySQL 和 SQLite 的标准写法）。

| dept_no | employees         |
| ------- | ----------------- |
| d001    | 10001,10002       |
| d002    | 10006             |
| d003    | 10005             |
| d004    | 10003,10004       |
| d005    | 10007,10008,10010 |
| d006    | 10009,10010       |

```mysql
SELECT 
    dept_no, 
    GROUP_CONCAT(emp_no) AS employees
FROM dept_emp
GROUP BY dept_no;
```

这个函数就像一个“胶水”，它会把属于同一个分组（GROUP BY）的多行数据，按照你指定的符号连接成一个长字符串。

**它的强大之处：**

- **默认分隔符**：如果你不指定，它默认用逗号 `,` 连接（正如图片所示）。
- **自定义分隔符**：如果你想用分号，可以写 `GROUP_CONCAT(emp_no SEPARATOR ';')`。
- **排序**：你甚至可以让格子里的员工编号按顺序排列：`GROUP_CONCAT(emp_no ORDER BY emp_no DESC)`。

**完整语法**

```sql
GROUP_CONCAT([DISTINCT] 要连接的字段 [ORDER BY 排序字段 ASC/DESC] [SEPARATOR '分隔符'])
```

#### <span style="color:red">避坑 关键限制：长度溢出</span>

这是很多新手会踩的坑。`GROUP_CONCAT` 返回的结果是有最大长度限制的（默认通常是 **1024 字节**）。

- **后果**：如果一个部门有上千人，合并后的字符串会被无情截断。
- **对策**：需要临时调大服务器配置： `SET SESSION group_concat_max_len = 10240;`



## LEAD

在 SQL 中，`LEAD()` 属于**偏移窗口函数**。它的核心能力是：**在处理当前行时，直接“偷看”后面第 N 行的数据。**

### 1. `LEAD()` 的标准语法

```SQL
LEAD(column_name, offset, default_value) OVER (
    PARTITION BY group_column 
    ORDER BY sort_column
)
```

**`column_name`**：你想看的那个“未来”的列。

**`offset`**（可选）：往后看几行？默认是 **1**（即看下一行）。

**`default_value`**（可选）：如果没有下一行了（比如到了最后一行），显示什么？默认是 **`NULL`**。

**`PARTITION BY`**：分组。比如按 `user_id` 分组，LEAD就不会看到别人的数据去。

**`ORDER BY`**：排序。决定了谁是“下一行”。



### 2.与普通CASE WHEN的优势(结合[牛客SQL280](https://www.nowcoder.com/practice/348afda488554ceb922efd2f3effc427?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1&sourceUrl=%2Fexam%2Foj))

**普通思路（Pivot）**：

1. 算出每一行的 `rn`。
2. 用 `CASE WHEN` 分类。
3. 用 `GROUP BY` 和 `MIN` 强行把多行合并。

**`LEAD` 思路（时光机）**：

1. 对每个订单，用 `LEAD(date)` 直接把“下一单的日期”抓到当前行。
2. 过滤出 `rn = 1`（第一单）的记录。
3. **结果自然就出来了：** 这一行包含了“今天”和“下次”的日期。



### 3. 实战模拟：数据是怎么流动的？

假设用户 `557336` 有 3 个订单：

| **user_id** | **date (当前行)** | **LEAD(date) OVER(...)** | **说明**                   |
| ----------- | ----------------- | ------------------------ | -------------------------- |
| 557336      | 2025-10-23        | **2025-10-25**           | 望远镜看到了下一行的日期   |
| 557336      | 2025-10-25        | **2025-10-26**           | 望远镜看到了再下一行的日期 |
| 557336      | 2025-10-26        | *NULL*                   | 后面没了，显示 NULL        |

**你会发现：** 如果你只要“第一次”和“第二次”的日期，你只需要取出 `rn = 1` 的那一整行，你就同时拥有了 `date`（第一次）和 `LEAD(date)`（第二次）。**一行代码搞定，连 `GROUP BY` 都不用写！**



### 4. `LEAD` vs `LAG`：一对双胞胎

这两个函数逻辑完全一样，只是方向相反：

- **`LEAD()`**：向**后**看（看未来，看下一行）。
- **`LAG()`**：向**前**看（看过去，看上一行）。

> **趣闻**：在英文里，Lead 意味着“领先”，Lag 意味着“滞后”。



### 5.其他使用场景 

1. **计算时间差**：比如计算“两次购买之间隔了几天”：`DATEDIFF(LEAD(date), date)`。
2. **状态变更**：看一个用户这次登录后，下次登录是不是换了设备。
3. **连续性分析**：判断用户的分数是不是一直在连续上涨。







## SUBSTR

`SUBSTR`（或 `SUBSTRING`）是处理文本的核心工具。要系统性掌握它，需要记住三个核心参数：**数据源、起始位置、截取长度**。

**基本语法:**

$$SUBSTR(\text{string}, \text{start}, [\text{length}])$$



**起始位置（start）的正负规律**

这是最容易混淆的地方：

- **正数索引 (1, 2, 3...)**：从**左**往右数。
  - `SUBSTR('ABCDE', 2)` $\rightarrow$ 'BCDE'（从第 2 个字母 B 开始）
- **负数索引 (-1, -2, -3...)**：从**右**往左数。
  - `SUBSTR('ABCDE', -2)` $\rightarrow$ 'DE'（从倒数第 2 个字母 D 开始）



**长度参数（length）**

- 如果省略：截取到字符串的末尾。
- 如果指定：只截取指定的字符个数。
  - `SUBSTR('ABCDE', 2, 2)` $\rightarrow$ 'BC'





## DATE_ADD

在 SQL 中，处理日期加减最核心的函数就是 `DATE_ADD`。它就像是日期维度的“加法器”，让你能够轻松实现“昨天”、“下周”、“去年”等时间偏移计算。

我们将从 **基础语法、时间单位、不同数据库的差异、以及实战场景** 四个维度来系统性拆解。

------

### 1. 语法结构

在标准的 MySQL 语法中，`DATE_ADD` 的结构非常直观：

$$DATE\_ADD(start\_date, INTERVAL\ expr\ unit)$$

- **`start_date`**：起始日期。可以是 `DATE`（如 '2024-01-01'）或 `DATETIME`。
- **`INTERVAL`**：关键字，告知数据库后面要进行时间间隔计算。
- **`expr`**：数值。可以是正数（向后加），也可以是**负数**（向前减）。
- **`unit`**：时间单位。

------

### 2. 时间单位 (Unit)

`DATE_ADD` 强大之处在于它支持非常细致的单位，满足各种业务需求：

| **单位 (Unit)** | **说明** | **示例 (加 1 单位)**                  |
| --------------- | -------- | ------------------------------------- |
| **SECOND**      | 秒       | `DATE_ADD(now(), INTERVAL 1 SECOND)`  |
| **MINUTE**      | 分钟     | `DATE_ADD(now(), INTERVAL 1 MINUTE)`  |
| **HOUR**        | 小时     | `DATE_ADD(now(), INTERVAL 1 HOUR)`    |
| **DAY**         | 天       | `DATE_ADD(now(), INTERVAL 1 DAY)`     |
| **WEEK**        | 周       | `DATE_ADD(now(), INTERVAL 1 WEEK)`    |
| **MONTH**       | 月       | `DATE_ADD(now(), INTERVAL 1 MONTH)`   |
| **QUARTER**     | 季度     | `DATE_ADD(now(), INTERVAL 1 QUARTER)` |
| **YEAR**        | 年       | `DATE_ADD(now(), INTERVAL 1 YEAR)`    |

复合单位（高级用法）：

你甚至可以一次性加“1小时30分钟”：

DATE_ADD('2024-01-01 00:00:00', INTERVAL '1:30' HOUR_MINUTE)

------

### 3. 变体

虽然逻辑一致，但不同数据库的“方言”略有不同，这是最容易报错的地方：

- **MySQL**: `DATE_ADD(date, INTERVAL 1 DAY)` 或简写为 `date + INTERVAL 1 DAY`。
- **PostgreSQL**: `date + INTERVAL '1 day'`。
- **SQL Server**: 使用 `DATEADD(day, 1, date)`（注意参数顺序不同）。
- **SQLite**: 使用 `date(date, '+1 day')`。

------

### 4. 实战场景：它能解决什么问题？

#### ① 计算“次日留存”或“昨天”

在留存分析中，我们需要匹配“第一天”和“第二天”。

```sql
WHERE login_date_tomorrow = DATE_ADD(login_date_today, INTERVAL 1 DAY)
```

#### ② 自动过期判断

找出 30 天内没有登录过的活跃用户：

```sql
SELECT user_id 
FROM users 
WHERE last_login < DATE_ADD(CURDATE(), INTERVAL -30 DAY); -- 用负数实现 DATE_SUB 的效果
```

#### ③ 财务报表：上个季度末

```sql
SELECT DATE_ADD('2024-04-01', INTERVAL -1 QUARTER); -- 得到 2024-01-01
```

------

### 5. 常见坑点（避坑指南）

1. 月末对齐问题：

   如果你在 1 月 31 日加 1 个月，结果会变成 2 月 29 日（如果是闰年）或 2 月 28 日。它会自动处理月份天数的不一致，不会报错。

2. 类型隐式转换：

   如果 start_date 是字符串且格式不标准（如 '2024.01.01'），DATE_ADD 可能会返回 NULL 或报错。建议永远使用标准的 'YYYY-MM-DD' 格式。

3. DATE_ADD vs DATE_SUB：

   DATE_SUB(date, INTERVAL 1 DAY) 等同于 DATE_ADD(date, INTERVAL -1 DAY)。为了代码简洁，建议统一使用 DATE_ADD 配合正负数即可。





## UNION

### UNION vs. JOIN：维度的博弈

这两个操作最本质的区别在于：**JOIN 是水平连接（增加列），UNION 是垂直堆叠（增加行）。**

**UNION（垂直合并）**

`UNION` 将两个或多个 `SELECT` 语句的结果集组合成一个结果集。

- **逻辑**：就像把两张名片盒里的卡片倒在一起，摞成一叠。
- **规则**：
  1. 两边的**列数**必须相同。
  2. 对应列的**数据类型**必须兼容。
  3. 默认会**去重**（如果不想去重，使用 `UNION ALL`）。

**LEFT JOIN（水平关联）**

`LEFT JOIN` 根据两个表之间的关联键（ON 条件），将另一张表的字段拼接到当前表。

- **逻辑**：就像在已有的表格右边粘上新的纸条，增加信息的宽度。

| **维度** | **UNION**                      | **LEFT JOIN**                    |
| -------- | ------------------------------ | -------------------------------- |
| **方向** | **垂直**（增加行数）           | **水平**（增加列数）             |
| **关系** | 两个结果集地位平等，是“加法”。 | 区分左表和右表，是“关联”。       |
| **要求** | 列数和类型必须匹配。           | 必须有共同的关联键（Join Key）。 |



## EXISTS

`EXISTS` 是一个专门用来执行**“存在性检查”**的布尔运算符。它不关心具体找回了什么数据，只关心**“能不能找到”**。

### 1. 底层逻辑：真与假的博弈

`EXISTS` 后面通常跟着一个**相关子查询（Correlated Subquery）**。其工作原理如下：

1. **遍历外表**：数据库先从外层查询（Main Query）中取出一行数据。
2. **执行子查询**：将这行数据的相关字段传入子查询中运行。
3. **判断结果**：
   - 如果子查询返回了**至少一行**数据，`EXISTS` 结果为 **TRUE**，外层查询保留这一行。
   - 如果子查询**一条数据都没找到**，`EXISTS` 结果为 **FALSE**，外层查询丢弃这一行。

> **核心秘密**：因为 `EXISTS` 只看“有没有”，所以子查询里的 `SELECT *` 或 `SELECT 1` 没有任何性能区别，数据库会自动优化。



#### 2.语法 实战场景：查询“拿过奖金”的员工

假设我们有 `employees` 表和 `emp_bonus` 表：

```sql
SELECT first_name, last_name
FROM employees e
WHERE EXISTS (
    SELECT 1 
    FROM emp_bonus b 
    WHERE b.emp_no = e.emp_no
);
```

**运行流程分解：**

1. **取行**：数据库引擎首先从外表 `employees` (别名为 `e`) 中取出**第一行**记录。
2. **传递参数**：将这一行的 `e.emp_no` 传入子查询。
3. **内层探测**：子查询拿着这个具体的 `emp_no` 去 `emp_bonus` (别名为 `b`) 表里查找。
4. **返回判定**：
   - **找到了**：只要在 `b` 表中发现 **1 条** 匹配记录，子查询就立刻停止扫描（这就是**短路机制**），并向外层返回 `TRUE`。
   - **没找到**：如果搜遍 `b` 表都没找到，返回 `FALSE`。
5. **输出结果**：外层查询根据 `TRUE` 或 `FALSE` 决定是否显示这一行员工的名字。
6. **循环**：接着取 `employees` 表的**第二行**，重复上述步骤。



### 3. `EXISTS` vs `IN`：深度对比

这是面试中最常被问到的问题。虽然它们有时能实现相同的功能，但执行机制完全不同。



#### 1.执行机制：谁在迁就谁？

这是 `EXISTS` 和 `IN` 最本质的区别 ：

**IN：先算内表（由内而外）**

1. 数据库先执行子查询，算出所有的值，并存进一个**内存集合**中。
2. 然后遍历外表，看外表的值在不在这个集合里。

- **缺点**：如果子查询查出的结果非常多（比如几百万个 ID），内存压力会巨大，且 `IN` 的匹配效率会随着集合变大而下降。

**EXISTS：先算外表（由外而内）**

1. 数据库先从外表拿出**第一行**。
2. 带着这行的数据，去内表里探测（Probe）一下有没有匹配的。
3. 一旦在内表里找到**任何一个**匹配项，就立刻停止内表的扫描（这叫**短路效应**），直接判定这一行过关。
4. 接着拿外表的第二行重复上述动作。



#### 2.EXISTS 相对于 IN 的三大核心优势

**① 性能上的“短路”优势**

如果你的内表非常庞大（比如千万级），但外表很小：

- **`IN`** 会尝试把千万级的 ID 都捞出来塞进内存。
- **`EXISTS`** 只要在内表里利用索引找到 **1 条** 匹配数据就收手。

> **总结**：当子查询的数据量远大于外查询时，`EXISTS` 配合索引的效率近乎无敌。

**② 处理 NULL 值的“安全性”优势**

这是 `EXISTS` 最大的逻辑优势。在 SQL 中，`NULL` 是一个极其头疼的“未知数”。

- **`IN` 的雷区**：如果子查询结果里包含一个 `NULL`，那么 `NOT IN` 将永远返回空结果（因为无法确定某值是否不等于一个“未知值”）。
- **`EXISTS` 的稳健**：它只看能不能匹配上。即便内表有 `NULL`，它也只会认为“没匹配上”，从而返回 `FALSE`，不会让整个查询逻辑崩溃。

**③ 灵活的“多列关联”**

`IN` 通常只能比较一列（除非写成复杂的 `(a, b) IN ...`），而 `EXISTS` 可以在子查询的 `WHERE` 里写非常复杂的关联逻辑，甚至关联三四张表，这在 `IN` 里面很难实现。



#### 4. 什么时候用谁？（黄金法则）

| **场景**               | **推荐**         | **理由**                                         |
| ---------------------- | ---------------- | ------------------------------------------------ |
| **子查询结果集很小**   | **`IN`**         | 内存集合小，匹配速度极快。                       |
| **子查询结果集很大**   | **`EXISTS`**     | 利用“短路效应”和索引，避免大数据量加载到内存。   |
| **排除数据（求差集）** | **`NOT EXISTS`** | 比 `NOT IN` 更安全，能正确处理 `NULL` 值。       |
| **外表非常小**         | **`EXISTS`**     | 哪怕内表再大，外表只需查几次，每次内表都能秒回。 |



| **特性**      | **IN**                                             | **EXISTS**                                                   |
| ------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| **执行顺序**  | 先运行子查询，生成一个集合，再遍历外表匹配。       | 先遍历外表，每一行都去跑一遍子查询。                         |
| **NULL 处理** | **非常危险**。如果子查询有 NULL，`NOT IN` 会失效。 | **非常安全**。`NULL` 对 `EXISTS` 来说只是“没找到”，不会报错。 |
| **性能优势**  | 当**子查询结果集很小**，而外表很大时，`IN` 更快。  | 当**外表很小**，而子查询表很大且有索引时，`EXISTS` 更快。    |





### 4. `NOT EXISTS`：处理“差集”的利器

`NOT EXISTS` 经常被用来寻找“在 A 表中存在，但在 B 表中不存在”的数据。它是替代 `NOT IN` 的最佳安全方案，因为它不受 `NULL` 的干扰。

**场景：查询“从来没拿过奖金”的员工**

```sql
SELECT first_name
FROM employees e
WHERE NOT EXISTS (
    SELECT 1 
    FROM emp_bonus b 
    WHERE b.emp_no = e.emp_no
);
```



### 5. 系统性性能优化（避坑指南）

1. **索引是关键**：子查询中被关联的列（如 `b.emp_no`）**必须建立索引**。如果没有索引，外表每一行都要全表扫描一遍子查询表，速度会瞬间崩溃。
2. **避免全表扫描**：`EXISTS` 的优势在于**“短路效应”**。一旦子查询找到第一行匹配的数据，它就会立刻停止扫描并返回 TRUE，这比 `COUNT(*)` 要高效得多。
3. **逻辑转换**：在现代数据库（如 MySQL 8.0+）中，优化器非常聪明，有时会自动把 `IN` 转换成 `EXISTS`。但在处理涉及 `NULL` 的列时，建议手动选择 `EXISTS` 以确保逻辑正确。



## 开窗函数

开窗函数（Window Functions）**既能保留原始明细行，又能同时计算聚合数据**,开窗函数必须写 OVER。

### 1. 概念

在传统的 `GROUP BY` 中，数据会被“折叠”。比如计算部门平均工资，一个部门最后只剩一行。 而**开窗函数**就像是在每一行数据旁边打开了一扇“观察窗”，窗口里透出的数据是根据某种规则计算出来的，但**原始行一行都不会少**。



### 2. 标准语法公式

```sql
<函数名>(列名) OVER (
    [PARTITION BY 字段名]  -- 分组（相当于分组的墙）
    [ORDER BY 字段名]      -- 排序（相当于计算的顺序）
    [ROWS/RANGE ...]      -- 窗口子句（精确定义计算哪几行）
)
```

- **`PARTITION BY`**：将数据切片。比如按“部门”切片，计算就不再跨部门，而是每个部门内部独立计算。

- **`ORDER BY`**：决定了窗口内数据的排列方式。**在聚合函数中，它直接触发了“从起点到当前行”的动态累加逻辑。**



### 3. 三大类开窗函数

**① 聚合类（Aggregate Window Functions）**

- **函数**：`SUM()`, `AVG()`, `COUNT()`, `MAX()`, `MIN()`
- **场景**：计算累计工资、部门总薪水、所有员工的平均分对比。

**② 排序类（Ranking Window Functions）**

这是面试和实战中最常用的：

- **`ROW_NUMBER()`**：唯一连续排名（1, 2, 3, 4），即便数值一样也排先后。
- **`RANK()`**：跳跃排名（1, 2, 2, 4），数值一样则并列，占掉后面的位置。
- **`DENSE_RANK()`**：连续排名（1, 2, 2, 3），数值一样则并列，但不占后面的位置。

**③ 值偏移类（Value Window Functions）**

- **`LAG(col, n)`**：获取当前行向上数第 $n$ 行的值（常用于计算“环比”）。
- **`LEAD(col, n)`**：获取当前行向下数第 $n$ 行的值。



### **4. 进阶：ROWS 窗口子句（实现“滑动窗口”）**

除了“从开头到当前行”，你还可以精确控制窗口的“尺寸”。这就是实现 **“移动平均”** 的关键。

语法示例：

ROWS BETWEEN <起点> AND <终点>

- **`UNBOUNDED PRECEDING`**：起点是第一行。
- **`CURRENT ROW`**：当前行。
- **`n PRECEDING`**：向上数 $n$ 行。
- **`n FOLLOWING`**：向下数 $n$ 行。



实战案例：计算当前及前两行的“三月移动平均薪水”

```sql
SELECT emp_no, salary,
       AVG(salary) OVER (
           ORDER BY from_date 
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS moving_avg
FROM salaries;
```



### 5. 系统化避坑指南

1. **执行顺序**：开窗函数是在 `WHERE`, `GROUP BY`, `HAVING` 之后执行的，但在 `ORDER BY`（全表排序）之前。**这意味着你不能在 `WHERE` 子句里直接使用开窗函数产生的新列。**（需要嵌套子查询）
2. **性能成本**：开窗函数涉及大量排序操作，在海量数据上使用时，务必确保 `PARTITION BY` 和 `ORDER BY` 的字段上有索引。
3. **空窗口**：如果 `OVER()` 括号里什么都不写，意味着窗口覆盖全表，计算全表的总和。



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





## 外键约束

外键约束（Foreign Key Constraint）是关系型数据库的灵魂。它不仅仅是一根连接表的“线”，更是一套复杂的**数据准入与联动规则**。

以下拆解为：**底层逻辑、级联策略、操作实务、以及企业级权衡**四个部分。

### 1. 核心定义：父与子的契约

外键建立了两个表之间的“父子关系”：

- **父表（Parent Table）**：被引用的表，通常是基础信息表（如“部门表”）。它必须拥有主键（Primary Key）或唯一索引。
- **子表（Child Table）**：引用的表，通常是业务表（如“员工表”）。它通过外键列指向父表。



### 2. 四大级联策略（On Delete / On Update）

这是外键最精华的部分。当父表的数据被删除或修改时，子表该如何反应？

| **策略名称**             | **行为描述**                                                 | **适用场景**                             |
| ------------------------ | ------------------------------------------------------------ | ---------------------------------------- |
| **RESTRICT / NO ACTION** | **严禁乱动**。只要子表还有人在引用，就不准删除/修改父表记录。 | **最常用**。防止误删还在使用的基础数据。 |
| **CASCADE**              | **生死相依**。父表删了，子表关联行自动跟着删；父表 ID 改了，子表自动同步更新。 | 级联删除：订单详情、账号配置。           |
| **SET NULL**             | **断开连接**。父表删了，子表那一行保留，但外键字段设为 NULL。 | 员工所在部门撤销了，员工依然留职。       |
| **SET DEFAULT**          | **回归默认**。父表删了，子表外键设为一个预设的默认值。       | MySQL 存储引擎（InnoDB）暂不支持此项。   |



### 3. 操作实务：如何增删改查？

#### ① 创建表时定义

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    user_id INT,
    -- 建立关联
    CONSTRAINT fk_user 
    FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE CASCADE -- 删用户时，订单跟着删
);
```

#### ② 已有表追加（最常用）

```sql
ALTER TABLE orders 
ADD CONSTRAINT fk_user 
FOREIGN KEY (user_id) REFERENCES users(id);
```

#### ③ 查看外键

```sql
-- 查看表结构详情，可以看到约束名称和定义
SHOW CREATE TABLE orders;
```

#### ④ 删除外键

```sql
ALTER TABLE orders DROP FOREIGN KEY fk_user;
```



### 4. 约束背后的“潜规则”

要成功创建外键，必须满足这 4 个硬性条件：

1. **引擎支持**：MySQL 中只有 **InnoDB** 支持物理外键（MyISAM 不支持）。
2. **类型绝对一致**：父子表字段的类型、长度、是否有符号（Unsigned）必须**完全一模一样**。
3. **索引要求**：父表被引用的列必须是主键或唯一索引；子表的外键列也必须有索引（MySQL 会自动在子表创建索引）。
4. **数据洁癖**：添加外键时，子表不能存在任何父表里没有的“孤儿数据”。



### 5. 物理外键 vs 逻辑外键

在企业级开发中，这是一个充满争议的话题。

- **物理外键（数据库强制）**：
  - **优点**：绝对安全，数据永远一致，不会出现脏数据。
  - **缺点**：性能损耗（每次写都要查另一张表）、分库分表时无法使用、并发高时容易死锁。
- **逻辑外键（代码维护）**：
  - **做法**：表结构上不建外键，但在代码逻辑（Java/Python）里保证先删子表再删父表。
  - **优点**：高性能、高扩展性。
  - **缺点**：对程序员要求高，一旦代码写错，数据库里就会出现“幽灵数据”。

> **大厂共识**：互联网大厂（如阿里《Java开发手册》）明确建议**禁止使用物理外键**，所有约束在应用层完成。但对于中小型、安全性要求极高的系统，外键依然是神器。







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



## 数值精度(ROUND,TRUNCATE,CAST CONVERT)

### 1. ROUND(x, d)：四舍五入的“美容师”

`ROUND` 是最符合人类数学直觉的函数。它会观察你指定位数后的那一位，决定是“进”还是“舍”。

- **语法**：`ROUND(数值, 保留位数)`
- **特性**：
  - 如果 `d` 是正数，保留到小数点后 `d` 位。
  - 如果 `d` 是 0，保留到整数部分。
  - 如果 `d` 是负数，保留到小数点左侧（十位、百位等）。

**示例：**

SQL

```
SELECT ROUND(123.456, 2);  -- 123.46 (五入)
SELECT ROUND(123.454, 2);  -- 123.45 (四舍)
SELECT ROUND(123.456, -1); -- 120    (对个位四舍五入)
```

------

### 2. TRUNCATE(x, d)：硬核截断的“裁剪师”

`TRUNCATE` 非常“冷酷”，它完全不看后面的数字大小，只要超过了指定位数，一律**直接扔掉**。

- **语法**：`TRUNCATE(数值, 保留位数)`
- **关键区别**：它永远不会进位。
- **适用场景**：在金融或虚拟货币领域，为了防止“进位带来的微量超支”，通常要求只舍不进。

**示例对比：**

SQL

```
SELECT TRUNCATE(123.459, 2); -- 123.45 (即使后面是 9 也不进位)
SELECT ROUND(123.459, 2);    -- 123.46 (ROUND 会进位)
```

------

### 3. CAST 与 CONVERT：类型转换的“变身专家”

这两个函数不只是为了保留小数，它们的核心任务是**改变数据的“身份证”（数据类型）**。在保留小数时，它们会将数值转为 `DECIMAL` 类型。

#### **CAST(x AS type)**

这是 **SQL 标准语法**，兼容性极强。

SQL

```
SELECT CAST(123.4567 AS DECIMAL(10, 2)); -- 123.46
```

- `DECIMAL(10, 2)`：意思是总共最多存 10 位数，其中 2 位是小数。**它默认会进行四舍五入。**

#### **CONVERT(x, type)**

这是 **MySQL/SQL Server 特有语法**（两者的参数顺序略有不同）。

- **MySQL**: `CONVERT(123.456, DECIMAL(10, 2))`
- **功能**：与 `CAST` 几乎一致，但在处理字符集转换（如把 UTF8 转为 GBK）时比 `CAST` 更强大。

## LIMIT 与 OFFSET

`LIMIT` 和 `OFFSET` 是处理大规模数据集时最常用的两个关键字。它们的主要功能是**数据切片**，最典型的应用场景就是网页上的**分页显示**。

### 1. 基础定义

- **`LIMIT`**：限制返回结果的最大行数（“我要多少条”）。
- **`OFFSET`**：跳过指定数量的行（“从哪里开始”）。

#### 语法结构：

```SQL
SELECT 列名 FROM 表名
ORDER BY 某列 -- 强烈建议配合排序使用，否则结果顺序是不可控的
LIMIT 数量 OFFSET 偏移量;
```

------

### 2. 常见用法与写法

不同的数据库（如 MySQL 和 PostgreSQL）对语法的支持略有不同，但逻辑是一致的。

#### ① 标准写法

```SQL
-- 获取第 6 到第 15 条记录（跳过前 5 条，取 10 条）
SELECT * FROM actor ORDER BY actor_id LIMIT 10 OFFSET 5;
```

#### ② MySQL 的简写（逗号法）

MySQL 支持一种更简洁但容易混淆的写法：`LIMIT 偏移量, 数量`。

```SQL
-- 注意：这里第一个数字是 OFFSET，第二个是 LIMIT
SELECT * FROM actor LIMIT 5, 10; -- 同样是跳过 5 条，取 10 条
```

------

### 3. 分页逻辑的计算公式

如果你在开发一个网页，用户点击“第 `n` 页”，每页显示 `pageSize` 条数据，你的 SQL 应该这样写：

- **LIMIT** = `pageSize`
- **OFFSET** = `(n - 1) * pageSize`

| **页码 (n)** | **每页数量 (pageSize)** | **LIMIT** | **OFFSET** |
| ------------ | ----------------------- | --------- | ---------- |
| 第 1 页      | 10                      | 10        | 0          |
| 第 2 页      | 10                      | 10        | 10         |
| 第 3 页      | 10                      | 10        | 20         |

------

### 4. 深度原理与性能陷阱（面试常考）

这是 `OFFSET` 最坑的地方：**深分页问题**。

当你执行 `LIMIT 10 OFFSET 1000000` 时，你可能以为数据库会直接跳到第 100 万行。**错！**

底层逻辑：

数据库必须先把前 100 万条数据都读出来，然后扔掉，最后只给你剩下的 10 条。

- **后果**：随着页码越来越往后，查询速度会越来越慢，IO 压力剧增。

#### 🚀 优化方案：

1. **子查询优化**：先通过覆盖索引（只查 ID）快速定位偏移后的 ID，再回表查详情。

2. **“上一页最后 ID”法**：不使用 `OFFSET`，而是记录上一页最后一条记录的 ID（假设 ID 是连续递增的）。

   ```sql
   -- 查找 ID 大于 1000000 的前 10 条
   SELECT * FROM actor WHERE actor_id > 1000000 LIMIT 10;
   ```

   *这种方式无论翻到多少页，速度都极快（走索引扫描）。*

------

### 5. 注意事项

1. **必须配合 `ORDER BY`**：如果没有排序，数据库返回的“前 5 条”可能是随机的（取决于物理存储顺序），这会导致分页数据重复或遗漏。
2. **执行顺序**：在 SQL 执行生命周期中，`LIMIT` 和 `OFFSET` 是**最后一步**执行的。它发生在 `WHERE`、`GROUP BY`、`HAVING` 和 `ORDER BY` 之后。
3. **不同数据库差异**：
   - **Oracle**：旧版使用 `ROWNUM` 伪列，新版支持 `OFFSET n ROWS FETCH NEXT m ROWS ONLY`。
   - **SQL Server**：使用 `TOP` 或 `OFFSET...FETCH`。





## != 与 NOT IN

`!=` (或 `<>`) vs. `NOT IN` :条件过滤的区别

虽然它们都表示“不等于”，但在处理**多个值**和 **NULL 值**时，表现完全不同。

#### **(1) 处理数量的能力**

- **`!=`**：只能一次比较**一个值**。
  - 例子：`salary != 5000`
- **`NOT IN`**：可以一次比较**一个列表/集合**。
  - 例子：`salary NOT IN (5000, 6000, 7000)`

#### **(2) 致命的 NULL 值陷阱（系统性重点！）**

这是 SQL 中最容易导致 Bug 的地方：

- **`!=` 的表现**：如果 `salary` 是 `NULL`，`salary != 5000` 的结果是 `Unknown`（既不是真也不是假），这条记录会被过滤掉。
- **`NOT IN` 的表现**：如果子查询或列表里包含一个 `NULL` 值，整个 `NOT IN` 表达式会**永远返回空结果**！
  - 例子：`WHERE salary NOT IN (5000, NULL)` $\rightarrow$ 这条 SQL 永远查不到任何数据。
  - **原因**：因为 `NOT IN` 逻辑上等同于 `salary != 5000 AND salary != NULL`。在 SQL 中，任何与 `NULL` 的相等/不等比较都是 `Unknown`。

#### **(3) 什么时候用哪个？**

| **场景**               | **推荐工具**                                  | **理由**                                             |
| ---------------------- | --------------------------------------------- | ---------------------------------------------------- |
| **排除单个具体值**     | `!=`                                          | 语法简洁，执行速度快。                               |
| **排除一组已知常量**   | `NOT IN (...)`                                | 代码可读性强。                                       |
| **排除另一个表的内容** | `NOT EXISTS` 或 `LEFT JOIN ... WHERE IS NULL` | **比 `NOT IN` 安全**，能完美处理 NULL 值且性能更好。 |





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

