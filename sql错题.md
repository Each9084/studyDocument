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

