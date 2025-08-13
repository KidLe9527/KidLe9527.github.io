---
title: 'MySQL学习过程心得体会'

date: 2025-08-10T22:01:00-08:00
lastmod: 2025-08-10T22:02:00-08:00
categories: ['笔记']
tags: ['MySQL']
cover: https://kidle9527.github.io/images/55.png
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

***



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

---



### 聚合函数的使用位置

* 聚合函数只能用于 ` SELECT列表或HAVING子句 `中，不能直接用于WHERE子句或GROUP BY子句。如果需要基于聚合结果进行筛选，应该使用HAVING而不是WHERE ！！

---



### 小难的题目

1. [3642. 查找有两极分化观点的书籍 - 力扣（LeetCode）](https://leetcode.cn/problems/find-books-with-polarized-opinions/description/)

* 别人写的代码

```
# Write your MySQL query statement below
select b.book_id, title, author, genre, pages, rating_spread, polarization_score
from books b
join (
    select book_id, 
    round(sum(if(session_rating>3 or session_rating<3, 1, 0))/count(session_rating),2) as polarization_score, 
    max(session_rating)-min(session_rating) as rating_spread
    from reading_sessions
    group by 1
    having count(session_id)>4 and sum(if(session_rating>3,1,0))>0 and sum(if(session_rating<3,1,0))>0
    ) tb
using(book_id)
where polarization_score>=0.6
order by polarization_score desc, title desc

作者：RJSolution
```

1. using(book_id)

* 在SQL中，`USING(book_id)`是一种简化JOIN条件的语法，它等价于`ON table1.book_id = table2.book_id`。

* **`USING(book_id)`的作用**

  当两个表通过**相同名称的列**（这里是`book_id`）进行JOIN时，可以用`USING`替代`ON`，使代码更简洁。

* **在结果中，JOIN列只出现一次**

  - 如果用`ON b.book_id = r.book_id`，查询结果会包含`b.book_id`和`r.book_id`两列。
  - 如果用`USING(book_id)`，查询结果只会显示一个`book_id`列（不会重复）。

* 自己写的然后用yb改的

```
SELECT 
    b.*,
    MAX(r.session_rating) - MIN(r.session_rating) AS rating_spread,
    ROUND(SUM(CASE WHEN r.session_rating <= 2 OR r.session_rating >= 4 THEN 1 ELSE 0 END) / COUNT(*), 2) AS polarization_score
FROM 
    books b
JOIN 
    reading_sessions r ON b.book_id = r.book_id
GROUP BY 
    b.book_id
HAVING 
    COUNT(*) >= 5 
    AND SUM(CASE WHEN r.session_rating <= 2 OR r.session_rating >= 4 THEN 1 ELSE 0 END) / COUNT(*) >= 0.6
    AND SUM(CASE WHEN r.session_rating >= 4 THEN 1 ELSE 0 END) >= 1  -- 至少一个≥4的评分
    AND SUM(CASE WHEN r.session_rating <= 2 THEN 1 ELSE 0 END) >= 1  -- 至少一个≤2的评分
ORDER BY 
    polarization_score DESC, 
    b.title DESC;
```

2. ROUND(x, y)函数 --> 求参数x的四舍五入值，保留y位小数
3. `CASE WHEN r.session_rating <= 2 OR r.session_rating >= 4 THEN 1 ELSE 0 END`
   * case when xxx , true -> then , false -> else; 注意最后有一个 `END`

* 然后就是总结，对于比较复杂的数据库问题，可以在select的时候就筛选条件，也可以在from后面的派生表里筛选条件（注意要有别名），聚合函数不能出现在where里面等等...
