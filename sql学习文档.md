

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

