

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

![sqlJoin](F:\学习\Sql学习\asset\sqlJoin.jpg)