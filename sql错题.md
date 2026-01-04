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

