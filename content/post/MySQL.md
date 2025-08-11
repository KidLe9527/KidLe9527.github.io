---
title: 'MySQL学习过程心得体会'

date: 2025-08-10T22:01:00-08:00
lastmod: 2025-08-10T22:02:00-08:00
categories: ['笔记']
tags: ['MySQL']
---

# MySQL学习笔记

## 针对MySQL刷题过程中遇到的一些问题及解决办法

### 派生表必须要有别名

```
1. Every derived table must have its own alias
```

* 这个错误消息的意思是"每个``派生表``必须有自己的别名"，这是 MySQL 在执行 SQL 查询时常见的语法错误。

```
-- 错误的写法：子查询没有别名
SELECT * FROM (SELECT * FROM employees);

-- 正确的写法：为子查询指定别名
SELECT * FROM (SELECT * FROM employees) AS emp;
```



### 只有`表`才能取别名

```
2. You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'x
) y' at line 9
```

这个错误是因为你在子查询的末尾添加了不必要的别名 ` x` ，而 MySQL 不允许在这种位置使用它们。

```
错误代码：
SELECT MAX(salary) SecondHighestSalary
FROM (
    SELECT salary
    FROM Employee
    WHERE salary < (
        SELECT max(salary)
        from Employee  
    ) x -- 就是这里没有引用不需要添加别名
) y;
```



--> 在 MySQL 中，` 子查询的别名通常用于 FROM 子句中的表引用 `，但你的查询没有使用它们，所以 MySQL 会报错。



### 聚合函数的使用位置

* 聚合函数只能用于 ` SELECT列表或HAVING子句 `中，不能直接用于WHERE子句或GROUP BY子句。如果需要基于聚合结果进行筛选，应该使用HAVING而不是WHERE ！！

