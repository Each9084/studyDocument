### **SQL201 **并列基础DENSE_RANK问题

**查找入职员工时间升序排名的情况下的倒数第三的员工所有信息**

**描述**

有一个员工employees表简况如下:

| emp_no | birth_date | first_name | last_name | gender | hire_date  |
| ------ | ---------- | ---------- | --------- | ------ | ---------- |
| 10001  | 1953-09-02 | Georgi     | Facello   | M      | 1986-06-26 |
| 10002  | 1964-06-02 | Bezalel    | Simmel    | F      | 1985-11-21 |
| 10003  | 1959-12-03 | Parto      | Bamford   | M      | 1986-08-28 |
| 10004  | 1954-05-01 | Christian  | Koblick   | M      | 1986-12-01 |

请你查找employees里入职员工时间升序排名的情况下倒数第三的员工所有信息，以上例子输出如下:

| emp_no | birth_date | first_name | last_name | gender | hire_date  |
| ------ | ---------- | ---------- | --------- | ------ | ---------- |
| 10001  | 1953-09-02 | Georgi     | Facello   | M      | 1986-06-26 |

注意：可能会存在同一个日期入职的员工，所以入职员工时间排名倒数第三的员工可能不止一个,存在多个员工的情况按照emp_no升序排列。



<span style="color:red;">可能被忽略的点:</span>

如果两人是并列,那么使用传统的`LEFT OFFSET`就只能取出一个

所以关键是使用`DENSE_RANK() OVER()`这样就可以取出并列的多个单位



答案:

```sql
 SELECT emp_no,birth_date,first_name,last_name,gender,hire_date 
FROM (
SELECT *,DENSE_RANK() OVER (ORDER BY hire_date desc) AS RK )e
WHERE RK = 3 ORDER BY hire_date ASC;
```



### **SQL217** 不适用ORDER BY

**获取当前薪水第二多的员工的emp_no以及其对应的薪水salary**



**描述**

有一个员工表employees简况如下:

| emp_no | birth_date | first_name | last_name | gender | hire_date  |
| ------ | ---------- | ---------- | --------- | ------ | ---------- |
| 10001  | 1953-09-02 | Georgi     | Facello   | M      | 1986-06-26 |
| 10002  | 1964-06-02 | Bezalel    | Simmel    | F      | 1985-11-21 |
| 10003  | 1959-12-03 | Parto      | Bamford   | M      | 1986-08-28 |
| 10004  | 1954-05-01 | Chirstian  | Koblick   | M      | 1986-12-01 |

有一个薪水表salaries简况如下:

| emp_no | salary | from_date  | to_date    |
| ------ | ------ | ---------- | ---------- |
| 10001  | 88958  | 2002-06-26 | 9999-01-01 |
| 10002  | 72527  | 2001-08-02 | 9999-01-01 |
| 10003  | 43311  | 2001-12-01 | 9999-01-01 |
| 10004  | 74057  | 2001-11-27 | 9999-01-01 |

请你查找薪水排名第二多的员工编号emp_no、薪水salary、last_name以及first_name，**不能使用order by完成**，以上例子输出为:

**（温馨提示:sqlite通过的代码不一定能通过mysql，因为SQL语法规定，使用聚合函数时，select子句中一般只能存在以下三种元素：常数、聚合函数，group by 指定的列名。如果使用非group by的列名，sqlite的结果和mysql 可能不一样)**

| emp_no | salary | last_name | first_name |
| ------ | ------ | --------- | ---------- |
| 10004  | 74057  | Koblick   | Chirstian  |



<span style = "color:red">思路就是先找出房间里最高的人 然后把他请出去,再找到的最高的就是排名第二的</span>

```SQL
SELECT e.emp_no,s.salary,e.from_date,e.to_date
FROM emplyees e join salaries s 
ON e.emp_no = s.emp_no
WHERE s.salary = (SELECT MAX(salary) 
FROM salaries 
WHERE salary < (SLECT MAX(salary) FROM salaries))
```





### **SQL220** 多个SELECT

**查找在职员工自入职以来的薪水涨幅情况**



**描述**

有一个员工表employees简况如下:

| emp_no | birth_date | first_name | last_name | gender | hire_date  |
| ------ | ---------- | ---------- | --------- | ------ | ---------- |
| 10001  | 1953-09-02 | Georgi     | Facello   | M      | 2001-06-22 |
| 10002  | 1964-06-02 | Bezalel    | Simmel    | F      | 1999-08-03 |

有一个薪水表salaries简况如下:

| emp_no | salary | from_date  | to_date    |
| ------ | ------ | ---------- | ---------- |
| 10001  | 85097  | 2001-06-22 | 2002-06-22 |
| 10001  | 88958  | 2002-06-22 | 9999-01-01 |
| 10002  | 72527  | 1999-08-03 | 2000-08-02 |
| 10002  | 72527  | 2000-08-02 | 2001-08-02 |

请你查找在职员工自入职以来的薪水涨幅情况(注意这里强调的是在职员工)，给出在职员工编号emp_no以及其对应的薪水涨幅growth，并按照growth进行升序，以上例子输出为

（注: to_date为薪资调整某个结束日期，或者为离职日期，to_date='9999-01-01'时，表示依然在职，无后续调整记录）

| emp_no | growth |
| ------ | ------ |
| 10001  | 3861   |



**<span style ="color:red">思路:</span>**

<span style ="color:teal">首先就是需要找到`from_date`=`hire_date`的最初信息 然后再找到 ` to_date`为`9999-01-01`的最新信息,将最新-最初即为最终的增幅,并且可以过滤掉离职的情况</span>



```mysql
SELECT old.emp_no,(new.salary-old.salary) AS growth 
FROM
(SELECT e.emp_no,s.salary
FROM employees e
JOIN salaries s
ON e.hire_date = s.from_date
)old 
JOIN 
(
SELECT e.emp_no,s.salary
FROM employees e 
    LEFT JOIN salaries s
ON e.emp_no = s.emp_no
WHERE s.to_date = '9999-01-01')new
ON old.emp_no = new.emp_no
ORDER BY growth asc;

```



### **SQL224**

**获取员工其当前的薪水比其manager当前薪水还高的相关信息**

描述

有一个，部门关系表dept_emp简况如下:

| emp_no | dept_no | from_date  | to_date    |
| ------ | ------- | ---------- | ---------- |
| 10001  | d001    | 1986-06-26 | 9999-01-01 |
| 10002  | d001    | 1996-08-03 | 9999-01-01 |

有一个部门经理表dept_manager简况如下:

| dept_no | emp_no | from_date  | to_date    |
| ------- | ------ | ---------- | ---------- |
| d001    | 10002  | 1996-08-03 | 9999-01-01 |

有一个薪水表salaries简况如下:

| emp_no | salary | from_date  | to_date    |
| ------ | ------ | ---------- | ---------- |
| 10001  | 88958  | 2002-06-22 | 9999-01-01 |
| 10002  | 72527  | 1996-08-03 | 9999-01-01 |

获取员工其当前的薪水比其manager当前薪水还高的相关信息，

第一列给出员工的emp_no，
第二列给出其manager的manager_no，
第三列给出该员工当前的薪水emp_salary,
第四列给该员工对应的manager当前的薪水manager_salary

以上例子输出如下:

| emp_no | manager_no | emp_salary | manager_salary |
| ------ | ---------- | ---------- | -------------- |
| 10001  | 10002      | 88958      | 72527          |



```sql
--方法一笛卡尔积

SELECT de.emp_no,dm.emp_no AS manager_no,s1.salary AS emp_salary,s2.salary AS manager_salary 
FROM dept_emp de JOIN dept_manager dm ON dm.dept_no = de.dept_no
JOIN salaries s1 ON s1.emp_no = de.emp_no
JOIN salaries s2 ON s2.emp_no = dm.emp_no
WHERE s1.salary > s2.salary 
--
SELECT emp.emp_no,manager.manager_no,emp.emp_salary,manager.manager_salary
FROM
(SELECT de.emp_no,de.dept_no,s.salary AS emp_salary
FROM dept_emp de JOIN salaries s ON de.emp_no = s.emp_no
WHERE de.emp_no NOT IN (SELECT emp_no FROM dept_manager) AND de.to_date = '9999-01-01' AND s.to_date = '9999-01-01'
)emp JOIN 
(
    SELECT dm.emp_no AS manager_no,dm.dept_no,s.salary AS manager_salary
    FROM dept_manager dm JOIN salaries s On dm.emp_no = s.emp_no
    WHERE dm.to_date = '9999-01-01' AND s.to_date = '9999-01-01'
)manager

ON emp.dept_no = manager.dept_no
WHERE emp.emp_salary > manager.manager_salary;
```



**SQL241** **删除嵌套表**

**删除emp_no重复的记录，只保留最小的id对应的记录。**

删除emp_no重复的记录，只保留最小的id对应的记录。

```sql
CREATE TABLE IF NOT EXISTS titles_test (
id int(11) not null primary key,
emp_no int(11) NOT NULL,
title varchar(50) NOT NULL,
from_date date NOT NULL,
to_date date DEFAULT NULL);

insert into titles_test values ('1', '10001', 'Senior Engineer', '1986-06-26', '9999-01-01'),
('2', '10002', 'Staff', '1996-08-03', '9999-01-01'),
('3', '10003', 'Senior Engineer', '1995-12-03', '9999-01-01'),
('4', '10004', 'Senior Engineer', '1995-12-03', '9999-01-01'),
('5', '10001', 'Senior Engineer', '1986-06-26', '9999-01-01'),
('6', '10002', 'Staff', '1996-08-03', '9999-01-01'),
('7', '10003', 'Senior Engineer', '1995-12-03', '9999-01-01');
```

删除后titles_test表为(注：最后会select * from titles_test表来对比结果)

| id   | emp_no | title           | from_date  | to_date    |
| ---- | ------ | --------------- | ---------- | ---------- |
| 1    | 10001  | Senior Engineer | 1986-06-26 | 9999-01-01 |
| 2    | 10002  | Staff           | 1996-08-03 | 9999-01-01 |
| 3    | 10003  | Senior Engineer | 1995-12-03 | 9999-01-01 |
| 4    | 10004  | Senior Engineer | 1995-12-03 | 9999-01-01 |



<span style="color:red">需要注意的点就是不能直接删除</span>

❌错误示范

```SQL
DELETE FROM titles_test
FROM WHERE id NOT IN (
SELECT MIN(id) AS min_id FROM titles_test GROUP BY emp_no);
```



✅而是要加一个嵌套

```SQL
DELETE FROM titles_test WHERE id NOT IN(
SELECT min_id FROM (
SELECT MIN(id) AS min_id 
FROM titles_test
GROUP BY emp_no
)t
);
```



<span style="color:red">为了防止MYSQL死循环,你不能“一边删这棵树，一边又拿这棵树当梯子”。</span>

**在子查询外面再套一层查询，并给它起个别名（Alias）。**

这样 MySQL 就会先执行最里面的查询，把结果存入一个**临时表**（Temporary Table）。此时，外层的删除动作引用的就是这个临时表，而不是原表，限制就解除了。

**为什么“套一层”就能解决？（物化临时表）**

当把子查询改写成 `SELECT ... FROM (SELECT ... FROM A) AS tmp` 时，情况发生了本质变化：

1. **强制物化（Materialization）**：MySQL 优化器在看到“嵌套子查询”加“别名”时，会认为这是一个独立的**临时表**。
2. **先运行内层**：数据库会先完整地运行最里面的那条 `SELECT` 语句。
3. **结果落地**：它把查询到的结果（比如那些 `min_id`）存放在内存的一张**临时表（Temporary Table）**里。
4. **断开连接**：此时，原始表 `A` 的读取操作已经结束了。
5. **执行删除**：外层的 `DELETE` 动作开始执行。它现在参考的是那张**已经算好的临时表**，而不是正在被删除的原始表。



<span style="color:green">当然更加推荐的写法: DBA 更推荐使用 `JOIN` 语法，因为它能让优化器更清晰地处理多表逻辑，通常不需要创建完整的临时表：</span>

```SQL
DELETE t1 FROM titles_test t1
JOIN (
SELECT MIN(id) AS min_id,emp_no FROM titles_test
GROUP BY emp_no
)t2 ON t1.emp_no = t2.emp_no AND t1.id>t2.min_id;
```



这个就相当于左边有一个册子t1包含所有的原版titles_test 然后右边有一个找到最小id并进对emp_no进行分类的册子

然后根据左侧册子一条一条逐行比对,如过左边的id大于右边,那么处决

如果不是(t1.id=t2.min_id),那么保留.

而如果想找最大也很简单:<span style="color:red">只需要改为小于号即可</span>

```sql
DELETE t1 FROM titles_test t1
JOIN (
SELECT MAX(id) AS max_id,emp_no FROM titles_test
GROUP BY emp_no
)t2 ON t1.emp_no = t2.emp_no AND t1.id < t2.max_id;
```





### **SQL245** 外键约束

**在audit表上创建外键约束，其emp_no对应employees_test表的主键id**

**描述**

在audit表上创建外键约束，其emp_no对应employees_test表的主键id。

(以下2个表已经创建了)

> ```sql
> CREATE TABLE employees_test(
> ID INT PRIMARY KEY NOT NULL,
> NAME TEXT NOT NULL,
> AGE INT NOT NULL,
> ADDRESS CHAR(50),
> SALARY REAL
> );
> 
> CREATE TABLE audit(
> EMP_no INT NOT NULL,
> create_date datetime NOT NULL
> );
> ```

后台会判断是否创建外键约束，创建输出1，没创建输出0



<span style="color:red">这个知识点关于 **外键约束**,由于这个表已经创建了 所以我们不能在创建情况下下去指定,所以要使用`ALTER`</span>

```SQL
ALTER TABLE audit
ADD CONSTRAINT fk_audit
FOREIGN KEY (EMP_no) REFERENCES employees_test(ID)
ON DELETE CASCADE
ON UPDATE CASCADE;
```



### **SQL250** LENGTH REPLACE

**查找字符串中逗号出现的次数**

中等 通过率：60.96% 时间限制：1秒 空间限制：32M

知识点[SQL](https://www.nowcoder.com/exam/oj?tag=3427)

**描述**

现有strings表如下：

- id指序列号；
- string列中存放的是字符串，且字符串中仅包含数字、字母和逗号类型的字符。

| id   | string         |
| ---- | -------------- |
| 1    | 10,A,B,C,D     |
| 2    | A,B,C,D,E,F    |
| 3    | A,11,B,C,D,E,G |

请你统计每个字符串中逗号出现的次数cnt。

以上例子的输出结果如下：

| id   | cnt  |
| ---- | ---- |
| 1    | 4    |
| 2    | 5    |
| 3    | 6    |

<span style="color:teal">这道题的思路就是 我们要统计逗号的次数 = 总长度 - 没有逗号的次数,通过`LENGTH`可以统计长度,但是对于符号却没有相关的方法,那么我们通过`REPLACE`将`,`替换为空 即是没有逗号的长度</span>

```sql
SELECT id,(LENGTH(string)-LENGTH(REPLACE(string,",",""))) AS cnt
FROM strings;
```



<span style="color:blue">拓展:如果统计的是“单词”个数怎么办？</span>

比如 `10,A,B` 有 3 个元素。

- **思维推演**：单词数 = 逗号数 + 1。
- **代码**：`LENGTH(string) - LENGTH(REPLACE(string, ',', '')) + 1`。

<span style="color:blue"> 如果字符串里有中文怎么办？</span>

- **思维推演**：`LENGTH` 算的是字节，`CHAR_LENGTH` 算的是字符。在 UTF-8 下，一个中文占 3 字节。
- **系统经验**：处理文本内容统计时，优先使用 `CHAR_LENGTH` 以增强代码的健壮性。

<span style="color:blue">如果需要统计的是多个字符（比如统计 "AB" 出现了几次）？</span>

**思维推演**：差值不能直接等于次数，因为 "AB" 长度是 2。

**公式**：`(原长 - 替换后长度) / 目标子串的长度`。



### **SQL251 SUBSTR**

**获取employees中的first_name**

中等 通过率：63.88% 时间限制：1秒 空间限制：32M

知识点[SQL](https://www.nowcoder.com/exam/oj?tag=3427)

**描述**

现有employees表如下：

| emp_no | birth_date | first_name | last_name | gender | hire_date  |
| ------ | ---------- | ---------- | --------- | ------ | ---------- |
| 10001  | 1953-09-02 | Georgi     | Facello   | M      | 1986-06-26 |
| 10002  | 1964-06-02 | Bezalel    | Simmel    | F      | 1985-11-21 |
| 10003  | 1959-12-03 | Parto      | Bamford   | M      | 1986-08-28 |
| 10004  | 1954-05-01 | Christian  | Koblick   | M      | 1986-12-01 |
| 10005  | 1955-01-21 | Kyoichi    | Maliniak  | M      | 1989-09-12 |
| 10006  | 1953-04-20 | Anneke     | Preusig   | F      | 1989-06-02 |
| 10007  | 1957-05-23 | Tzvetan    | Zielinski | F      | 1989-02-10 |
| 10008  | 1958-02-19 | Saniya     | Kalloufi  | M      | 1994-09-15 |
| 10009  | 1952-04-19 | Sumant     | Peac      | F      | 1985-02-18 |
| 10010  | 1963-06-01 | Duangkaew  | Piveteau  | F      | 1989-08-24 |
| 10011  | 1953-11-07 | Mary       | Sluis     | F      | 1990-01-22 |

请你输出employees中的first_name，按照first_name最后两个字符升序输出。

以上示例数据的输出如下：

| first_name |
| ---------- |
| Christian  |
| Tzvetan    |
| Bezalel    |
| Duangkaew  |
| Georgi     |
| Kyoichi    |
| Anneke     |
| Sumant     |
| Mary       |
| Parto      |
| Saniya     |



<SPAN STYLE="COLOR:RED">这道题唯一的考点在于如何处理最后两个字符 答案很简单 `substr`,`substr`可以截取对应的内容</SPAN>

```sql
SELECT first_name FROM employees ORDER BY SUBSTR(first_name,-2);

--这种也没有区别,因为-2本身就是从右往左起数2位,然后处理倒数第二位到末尾,所以本身就是已经处理了2的内容
SELECT first_name FROM employees ORDER BY SUBSTR(first_name,-2,2);
```



### **SQL253** **平均工资**

中等 通过率：30.48% 时间限制：1秒 空间限制：32M

知识点[SQL](https://www.nowcoder.com/exam/oj?tag=3427)

**描述**

查找排除在职(to_date = '9999-01-01' )员工的最大、最小salary之后，其他的9999-01-01在职员工的平均工资avg_salary。

```sql
CREATE TABLE `salaries` ( `emp_no` int(11) NOT NULL,
   `salary` int(11) NOT NULL,
   `from_date` date NOT NULL,
   `to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

如：

```sql
INSERT INTO salaries VALUES(10001,85097,'2001-06-22','2002-06-22');
INSERT INTO salaries VALUES(10001,88958,'2002-06-22','9999-01-01');
INSERT INTO salaries VALUES(10002,72527,'2001-08-02','9999-01-01');
INSERT INTO salaries VALUES(10003,43699,'2000-12-01','2001-12-01');
INSERT INTO salaries VALUES(10003,43311,'2001-12-01','9999-01-01');
INSERT INTO salaries VALUES(10004,70698,'2000-11-27','2001-11-27');
INSERT INTO salaries VALUES(10004,74057,'2001-11-27','9999-01-01');
```

输出格式:

| avg_salary |
| :--------- |
| 73292      |



<span style="color:teal">这里一共有三种思路可以参考:</span>

①通过UNION将MAX()和MIN()的值转成列

②直接通过`AND `进行`WHERE salary != (SELECT MAX(salary) FROM salaries)  AND salary != (SELECT MIN(salary) FROM salaries);`

③<span style="color:blue">数学思路: </span>

​	$$AVG = \frac{总和 - 最大值 - 最小值}{总人数 - 2}$$

```sql
SELECT (SUM(salary) - MAX(salary) - MIN(salary)) / (COUNT(*) - 2)
FROM salaries;
```

即可



着重展示第一个思路:

```sql
SELECT AVG(salary) FROM salaries
WHERE salary NOT IN (
	SELECT MAX(salary) FROM salaries WHERE to_date = '9999-01-01'
	UNION
	SELECT MIN(salary) FROM salaries WHERE to_date = '9999-01-01'
) AND to_date = '9999-01-01'
```



### **SQL259** 开窗函数的动态累加

**统计salary的累计和running_total**

中等 通过率：42.60% 时间限制：1秒 空间限制：32M

知识点[SQL](https://www.nowcoder.com/exam/oj?tag=3427)

**描述**

统计salaries表中to_date = "9999-01-01"的员工累计工资和running_total，举例：第三个员工的running_total为前两个员工的salary累计和，其他以此类推。

注意：running_total要的是对在职员工的salary进行cumulative sum，在职员工的筛选条件为to_date = "9999-01-01"

CREATE TABLE `salaries` ( `emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
输出格式:

| emp_no | salary | running_total |
| :----- | :----- | :------------ |
| 10001  | 88958  | 88958         |
| 10002  | 72527  | 161485        |
| 10003  | 43311  | 204796        |
| 10004  | 74057  | 278853        |
| 10005  | 94692  | 373545        |
| 10006  | 43311  | 416856        |
| 10007  | 88070  | 504926        |
| 10009  | 95409  | 600335        |
| 10010  | 94409  | 694744        |
| 10011  | 25828  | 720572        |



<span style="color:red">这道题的思路就是,我们的目标是将salary累加到对应的一个列中,那么能实现这个步骤的就是开窗函数,因为只有开窗函数可以使不同行的值放在同一个列对应的位置上</span>

```sql
SELECT emp_no,salary,SUM(salary) OVER (ORDER BY emp_no) AS running_total  
FROM salaries 
WHERE to_date='9999-01-01';
```

如果不加`ORDER BY `就变成了全表的总和默认**“窗口 = 整个结果集”

**结果**：每一行都会显示**全表所有人的工资总和**。



写了 `ORDER BY`（累加模式）**行为**：

`ORDER BY` 像是一个**信号**，告诉数据库：“请开启**逐行动态增长**模式”。

**逻辑**：它会默认执行 `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`（从第一行到当前行）。

**结果**：实现你想要的 **Running Total（累计和）**。



### **(重点)SQL267 连续登录问题**

 **牛客每个人最近的登录日期(三)**

较难 通过率：28.44% 时间限制：1秒 空间限制：256M

知识点[数据分析师](https://www.nowcoder.com/exam/oj?tag=894)[牛客](https://www.nowcoder.com/exam/oj?tag=1727)[2021](https://www.nowcoder.com/exam/oj?tag=10059)[SQL](https://www.nowcoder.com/exam/oj?tag=3427)

**描述**

牛客每天有很多人登录，请你统计一下牛客新登录用户的次日成功的留存率，
有一个登录(login)记录表，简况如下:

| id   | user_id | client_id | date       |
| ---- | ------- | --------- | ---------- |
| 1    | 2       | 1         | 2020-10-12 |
| 2    | 3       | 2         | 2020-10-12 |
| 3    | 1       | 2         | 2020-10-12 |
| 4    | 2       | 2         | 2020-10-13 |
| 5    | 4       | 1         | 2020-10-13 |
| 6    | 1       | 2         | 2020-10-13 |
| 7    | 1       | 2         | 2020-10-14 |

第1行表示user_id为2的用户在2020-10-12使用了客户端id为1的设备第一次新登录了牛客网
。。。

第4行表示user_id为2的用户在2020-10-13使用了客户端id为2的设备登录了牛客网

。。。

最后1行表示user_id为1的用户在2020-10-14使用了客户端id为2的设备登录了牛客网



请你写出一个sql语句查询新登录用户次日成功的留存率，即第1天登陆之后，第2天再次登陆的概率,保存小数点后面3位(3位之后的四舍五入)，上面的例子查询结果如下:

| p     |
| ----- |
| 0.500 |

查询结果表明:

user_id为1的用户在2020-10-12第一次新登录了，在2020-10-13又登录了，算是成功的留存

user_id为2的用户在2020-10-12第一次新登录了，在2020-10-13又登录了，算是成功的留存

user_id为3的用户在2020-10-12第一次新登录了，在2020-10-13没登录了，算是失败的留存

user_id为4的用户在2020-10-13第一次新登录了，在2020-10-14没登录了，算是失败的留存

故次日成功的留存率为 2/4=0.5

(sqlite里查找某一天的后一天的用法是:date(yyyy-mm-dd, '+1 day')，四舍五入的函数为round，sqlite 1/2得到的不是0.5，得到的是0，只有1*1.0/2才会得到0.5

mysql里查找某一天的后一天的用法是:DATE_ADD(yyyy-mm-dd,INTERVAL 1 DAY)，四舍五入的函数为round)



#### 解答

<span style="color:red">思路: 对于连续登录问题,可以通过`DATE_ADD([date],INTERVAL [number] DAY/WEEK/MONTH....)`来解决</span>

那么对于这道题:

```MYSQL
SELECT ROUND(COUNT(l2.date) * 1.0/COUNT(l1.base_date),3)AS p FROM
(SELECT user_id,
		MIN(date) AS base_date
		FROM login 
		GROUP BY user_id
) AS l1
LEFT JOIN login l2 
ON l1.user_id = l2.user_id 
AND l2.date = DATE_ADD(l1.base_date,INTERVAL 1 DAY)
```



对于以上有两个点需要关注

**1.Q :为什么l2当作分子来计算,<span style="color:red">l2不应该是原始的login表吗?</span>**

A:当写 `COUNT(l2.user_id)` 时，数据库并不是跑去后台翻看那张原始的、静止的 `login` 表，而是在数此时此刻**已经被 `ON` 条件处理过、正在内存中生成的这堆数据。**

我们可以用**“投影仪”**来打比方：

- **原始表 `l2`**：是一个放在桌子上的实体物品（包含所有登录记录）。
- **`ON` 条件**：是一个带有特定形状的**遮板（滤镜）**。
- **结果集**：是投影在墙上的影子。

当执行 `SELECT COUNT(l2.user_id)` 时，你是在**数墙上的影子**，而不是在数桌子上的实物。

因为遮板（`ON l2.date = l1.date + 1`）的存在，只有符合条件的记录能穿过遮板在墙上投下影子；不符合条件的记录被挡住了，在墙上留下的只是**一片空白（NULL）**。



**2.连续登录的思路**

连续登录”问题是 SQL 面试中的顶级高频考点。这类问题的核心难点在于：**SQL 本质上是处理“集合”的，而“连续”是一个“顺序”概念。**

要用处理集合的工具去解决顺序的问题，最系统、最万能的思路叫做 **“等差数列转化法”**。

------

**1. 核心思想：等差数列的性质**

想象一下，如果一个人是连续登录的，那么他的 **“登录日期”** 和 **“行号（排名）”** 都是两个递增的等差数列。

- **规律**：如果两个等差数列的步长相同（都是 1），那么它们**对应相减的结果（差值）必然是一个常数**。

------

**2. 系统化解题三部曲**

无论题目要求“连续登录 3 天”还是“连续登录 N 天”，都可以套用这个模版：

**第一步：去重并编号 (Row Number)**

同一天可能登录多次，先按“人+天”去重，然后用 `ROW_NUMBER()` 为每个人的登录日期排个序。

**第二步：计算“偏移基准日期” (Group Key)**

用 **登录日期 - 它的行号**。如果日期是连续的，减出来的那个日期（我们管它叫 `base_date`）就会一模一样。

**第三步：按基准日期分组统计**

按照 `user_id` 和 `base_date` 进行 `GROUP BY`。如果某个组的 `COUNT` 大于等于 N，说明该用户连续登录了 N 天。

------

**3. 实战模拟**

假设用户 A 在 10-01, 10-02, 10-03 连续登录。

| **登录日期 (date)** | **行号 (rn)** | **计算：date - rn** | **结果 (base_date)** |
| ------------------- | ------------- | ------------------- | -------------------- |
| 2020-10-01          | 1             | 2020-10-01 - 1      | **2020-09-30**       |
| 2020-10-02          | 2             | 2020-10-02 - 2      | **2020-09-30**       |
| 2020-10-03          | 3             | 2020-10-03 - 3      | **2020-09-30**       |

**你看，只要是连续的，减出来的日期永远是 `2020-09-30`。这就是这组连续日期的“身份证”。**

------

**4. 标准代码模版**

以“找出连续登录 3 天及以上的用户”为例：

```sql
SELECT user_id, MIN(date) AS start_date, MAX(date) AS end_date, COUNT(*) AS days_count
FROM (
    SELECT 
        user_id, 
        date,
        -- 1. 计算每个用户的行号
        ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY date ASC) AS rn
    FROM (SELECT DISTINCT user_id, date FROM login) t -- 记得去重
) temp
-- 2. 日期减去行号，得到一个常数日期
GROUP BY user_id, DATE_SUB(date, INTERVAL rn DAY)
-- 3. 筛选天数满足条件的组
HAVING COUNT(*) >= 3;
```

------

**5. 常见变形与应对**

| **题目要求**                 | **应对策略**                                                 |
| ---------------------------- | ------------------------------------------------------------ |
| **最大连续登录天数**         | 在上述代码基础上，外层套一个 `SELECT user_id, MAX(days_count)`。 |
| **连续 3 天且每天积分 > 10** | 在最内层去重时，先加一个 `WHERE score > 10`。                |
| **连续活跃（不一定是登录）** | 将 `login` 表换成行为明细表（点击、购买等），思路一致。      |



#### 更形象的例子

数据库先看这个用户的所有登录日期，并给它们排上序号 $rn$。**注意：序号永远是连续的 1, 2, 3...**

| **日期 (date)** | **序号 (rn)** | **状态说明**             |
| --------------- | ------------- | ------------------------ |
| 2020-10-12      | 1             | 正常起步                 |
| 2020-10-13      | 2             | 连续                     |
| 2020-10-14      | 3             | 连续                     |
| **2020-10-17**  | **4**         | **这里发生了日期跳跃！** |
| 2020-10-18      | 5             | 恢复连续                 |

**第二步：时光倒流 (`DATE_SUB`)**

现在，我们让每一行日期都减去它的序号 $rn$。你可以把这想象成每个人都在往回走，**走出的步数就是他的序号。**

1. **10-12** 往回走 **1** 天 $\rightarrow$ 停留在了 **10-11**
2. **10-13** 往回走 **2** 天 $\rightarrow$ 停留在了 **10-11**
3. **10-14** 往回走 **3** 天 $\rightarrow$ 停留在了 **10-11**
   - *你看，前三个人步调一致，日期涨 1 天，步数也多走 1 步，结果全停在了 10-11。*
4. **10-17** 往回走 **4** 天 $\rightarrow$ 停留在了 **10-13**
   - *由于日期突然跳到了 17 号，而步数才走到第 4 步，差值变大了！它停在了 10-13。*
5. **10-18** 往回走 **5** 天 $\rightarrow$ 停留在了 **10-13**
   - *18 号比 17 号多 1 天，步数也比 4 多了 1 步，两相抵消，它也停在了 10-13。*

**第三步：结果聚拢 (`GROUP BY`)**

现在数据库手里拿到了这样一张经过处理的清单：

| **日期**   | **减法后的“身份证” (base_date)** |
| ---------- | -------------------------------- |
| 2020-10-12 | **2020-10-11**                   |
| 2020-10-13 | **2020-10-11**                   |
| 2020-10-14 | **2020-10-11**                   |
| 2020-10-17 | **2020-10-13**                   |
| 2020-10-18 | **2020-10-13**                   |

**最后，`GROUP BY user_id, base_date` 登场：**

- 它把身份证为 `2020-10-11` 的记录放进一个篮子 $\rightarrow$ `COUNT(*)` 结果是 **3**。
- 它把身份证为 `2020-10-13` 的记录放进另一个篮子 $\rightarrow$ `COUNT(*)` 结果是 **2**。

如果你要找连续登录 **3 天** 的人，数据库看一眼篮子：第一个篮子够 3 个，选中！第二个篮子只有 2 个，淘汰！



#### DATE_SUB和DATE_ADD

**`DATE_SUB`（连续登录法）**：是用来**“分堆”**的。它处理的是一个用户在长达一个月或一年里，到底有多少段连续的记录。

**`DATE_ADD`（固定偏移法）**：是用来**“打靶”**的。它只关注一个特定的“目标点”——也就是新登录后的**第 2 天**。

**两种思路的系统对比**

| **维度**     | **次日留存率 (本题思路)**             | **连续登录 (之前讲的思路)**           |
| ------------ | ------------------------------------- | ------------------------------------- |
| **核心函数** | `DATE_ADD`                            | `DATE_SUB`                            |
| **关注点**   | **特定两点**：第 1 天和第 2 天        | **全局片段**：任意长度的连续区间      |
| **比较对象** | 比较 `l2.date` 是否等于“计算出的明天” | 比较 `date - rn` 的结果是否等于“常数” |
| **结果形态** | 一个百分比 (Ratio)                    | 用户的名单或连续的天数                |







### SQL269 接上 总留存率VS每日留存率 

<span style="color:teal">**267和269可以总结为“全站大盘留存”** vs **“每日新增留存”两种类型**。</span>

| **维度**           | **SQL267 (总留存率)**                              | **SQL269 (每日留存率)**                               |
| ------------------ | -------------------------------------------------- | ----------------------------------------------------- |
| **查询目标**       | 计算全站**所有**新用户作为一个整体的次日留存表现。 | 计算**每一天**产生的新用户在各自次日的留存表现。      |
| **结果形式**       | **单一数值**（如 `0.500`）。                       | **日期列表**（如 `2020-10-12                          |
| **分母定义**       | 全站出现过的**所有**新用户总数。                   | **具体某一天**出现的新用户人数。                      |
| **是否需要骨架表** | **不需要**。没新用户就不产生分母，不需要占位。     | **需要**。为了显示留存率为 0 的日期，必须用日期骨架。 |

接下来我们来看题目:

**SQL269** **牛客每个人最近的登录日期(五)**

**描述**

牛客每天有很多人登录，请你统计一下牛客每个日期新用户的次日留存率。
有一个登录(login)记录表，简况如下:

| id   | user_id | client_id | date       |
| ---- | ------- | --------- | ---------- |
| 1    | 2       | 1         | 2020-10-12 |
| 2    | 3       | 2         | 2020-10-12 |
| 3    | 1       | 2         | 2020-10-12 |
| 4    | 2       | 2         | 2020-10-13 |
| 5    | 1       | 2         | 2020-10-13 |
| 6    | 3       | 1         | 2020-10-14 |
| 7    | 4       | 1         | 2020-10-14 |
| 8    | 4       | 1         | 2020-10-15 |

第1行表示user_id为2的用户在2020-10-12使用了客户端id为1的设备登录了牛客网，因为是第1次登录，所以是新用户
......
第4行表示user_id为2的用户在2020-10-13使用了客户端id为2的设备登录了牛客网，因为是第2次登录，所以是老用户
......
最后1行表示user_id为4的用户在2020-10-15使用了客户端id为1的设备登录了牛客网，因为是第2次登录，所以是老用户

请你写出一个sql语句查询每个日期新用户的次日留存率，结果保留小数点后面3位数(3位之后的四舍五入)，并且查询结果按照日期升序排序，上面的例子查询结果如下:

| date       | p     |
| ---------- | ----- |
| 2020-10-12 | 0.667 |
| 2020-10-13 | 0.000 |
| 2020-10-14 | 1.000 |
| 2020-10-15 | 0.000 |

查询结果表明:
2020-10-12登录了3个(user_id为2，3，1)新用户，2020-10-13，只有2个(id为2,1)登录，故2020-10-12新用户次日留存率为2/3=0.667;
2020-10-13没有新用户登录，输出0.000;
2020-10-14登录了1个(user_id为4)新用户，2020-10-15，user_id为4的用户登录，故2020-10-14新用户次日留存率为1/1=1.000;

2020-10-15没有新用户登录，输出0.000;

(注意:sqlite里查找某一天的后一天的用法是:date(yyyy-mm-dd, '+1 day')，sqlite里1/2得到的不是0.5，得到的是0，只有1*1.0/2才会得到0.5)



<span style = "color:teal">思路:</span>

<span style = "color:teal">①面对这个问题,首先确定是每个日期新用户的留存率,我们首先应该设置一个框架表0,也就是表0是原封不动来自于题目的login表,我们要把它当作最左侧的基准也就是股价表,所以要用DISTINCT date 或者 GROUP BY date确保只有一个,否则后面JOIN 的时候就是几百个重复的date与目标表进行相乘.</span>



<span style = "color:teal">②紧接着我们要join第二张表:表1,通过表1我们要选择哪些是新用户,这个通过MIN结合GROUP BY user_id就可以办到,然后关键的部分来了:</span><span style = "color:red">我们需要`ON l0.date = l1.base_date`</span>,<span style = "color:teal">很多人可能会疑惑为什么不用`ON l0.user_id = l1.user_id`,这是因为题目要求统计：**“每个日期”新用户的次日留存**, 这意味着，我们的报表每一行都是一个日期。</span>

<span style="color:teal">**- $l0$ 是“排队窗口”**：它提供了 10-12、10-13、10-14 这样一个个整齐的窗口。</span>

<span style="color:teal">**- $l1$ 是“刚出生的婴儿”**：它记录了每个用户以及他们出生的日期（`base_date`）。</span>

<span style="color:teal">当写下 `ON l0.date = l1.base_date` 时，你实际上是在对数据库下令：</span>

> **“请把所有在 10-12 出生的新用户，统统领到 10-12 这个窗口前集合。”**

>  **为什么要“挂在日期上”而不是“挂在人身上”？**
>
> 因为我们要算的分母是**“那一天新增加的总人数”**。
>
> - 如果我们不按日期连接，我们就不知道这些新用户该归属于哪一天。
> - 通过 `ON l0.date = l1.base_date`，我们强行让 $l1$（新用户表）里的 `user_id` 找到了它的**归属日期**。
>
> **想象一下这个过程：**
>
> - $l0$ 报出一个日期：**“2020-10-12”**。
> - $l1$ 响应：**“我有 3 个人的 `base_date` 是 10-12，分别是用户 1、2、3。”**
> - **结果**：在临时表里，10-12 这一行就扩展成了 3 行，每行对应一个用户。
>
> 
>
> | **l0.date (窗口)** | **l1.user_id (领到窗口的人)** | **l1.base_date (证明其出生日的凭证)** |
> | ------------------ | ----------------------------- | ------------------------------------- |
> | **10-12**          | 1                             | 10-12                                 |
> | **10-12**          | 2                             | 10-12                                 |
> | **10-12**          | 3                             | 10-12                                 |
> | **10-13**          | **NULL**                      | **NULL**                              |



<span style="color:teal">③ 也就是最简单的一步,找到同一个user_id下两张表,表2是表1(第一天)第二天的情况,说明这个人留存了</span>


```sql
SELECT l0.date,IFNULL(ROUND(COUNT(DISTINCT l2.user_id) * 1.0/ COUNT(DISTINCT l1.user_id),3),0)
FROM
(
SELECT DISTINCT date FROM login 
) AS l0
LEFT JOIN (
SELECT user_id,
		MIN(date) AS base_date
		FROM login 
		GROUP BY user_id
) AS l1
ON l0.date = l1.base_date
LEFT JOIN login l2 
ON l1.user_id = l2.user_id AND l2.date = DATE_ADD(l1.base_date,INTERVAL 1 DAY)
GROUP BY date;
```

