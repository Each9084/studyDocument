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

## 描述

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

