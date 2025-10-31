---
title: 'MySQL学习过程心得体会'
date: 2025-08-10T22:01:00-08:00
lastmod: 2025-08-10T22:02:00-08:00
categories: ['笔记']
tags: ['MySQL']
cover: https://kidle9527.github.io/images/55.png
---

# MySQL学习记录

## 1. MySQL学习笔记

### 求字符串的长度

* 在 MySQL 中计算字符串长度，主要根据您想计算的是 **“字符数”** 还是 **“字节数”**，需要使用不同的函数。

|                     函数                     |                 作用                 |                           适用场景                           |
| :------------------------------------------: | :----------------------------------: | :----------------------------------------------------------: |
| `CHAR_LENGTH(str)`或 `CHARACTER_LENGTH(str)` | 返回字符串 `str`包含的**字符**个数。 | **绝大多数情况推荐使用**。计算实际的字符数量，无论每个字符占用多少字节。 |
|                `LENGTH(str)`                 |  返回字符串 `str`占用的**字节**数。  | 需要知道字符串在计算机中实际存储空间时使用。结果取决于字符编码（如 utf8mb4, latin1）。 |

### 函数group_concat()

* [1484. 按日期分组销售产品 - 力扣（LeetCode）](https://leetcode.cn/problems/group-sold-products-by-the-date/description/)

* `GROUP_CONCAT`是 MySQL 中一个非常实用的聚合函数，它可以将分组中的多个值连接成一个字符串。

~~~mysql
基本语法：
GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR str_val])
~~~

* 核心功能：
  1. **将多行数据合并为一行字符串**，在 GROUP BY 分组后，将组内的多个值合并为一个字符串，默认使用逗号 `,`作为分隔符
  2. 支持去重（distinct），支持排序（order by），支持自定义分隔符（separator）

~~~mysql
select sell_date, count(distinct product) as num_sold, 
       (select product from Activities q where a.sell_date = q.sell_date) as products
-- 原代码这里求product，要求的是一串产品，但子查询在SELECT子句中只能返回单行单列
-- 因此修改成 group_concat(DISTINCT product ORDER BY product SEPARATOR ',') AS products
~~~







---

## 2. 针对MySQL刷题过程中遇到的一些问题及解决办法

### NULL不能用等号

* **NULL 在 SQL 中是一个特殊值，不能使用等号 `=`进行比较！**

  ~~~sql
  -- ❌ 错误：这种比较永远返回 UNKNOWN（不是 TRUE）
  WHERE id = NULL   -- 结果：UNKNOWN
  WHERE id != NULL  -- 结果：UNKNOWN
  
  -- ✅ 正确：必须使用 IS NULL 或 IS NOT NULL
  WHERE id IS NULL     -- 结果：TRUE 或 FALSE
  WHERE id IS NOT NULL -- 结果：TRUE 或 FALSE
  ~~~

  

  ## 2. 详细解释

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

---

### 判断条件的规范

* [1251. 平均售价 - 力扣（LeetCode）](https://leetcode.cn/problems/average-selling-price/)

~~~
# 一开始的代码
select p.product_id, ifnull(ROUND(sum(p.price * u.units) / sum(u.units), 2), 0) average_price
from Prices p LEFT JOIN UnitsSold u using(product_id)
WHERE u.purchase_date BETWEEN p.start_date AND p.end_date -- 这里需要限制条件在开始结束期内买入，确保只计算有效价格期内的销售
GROUP BY p.product_id;
~~~

* 问题所在：

  1. 首先，`ifnull(value1, value2)`函数的判断是**只有value1为`null`**，才会返回value2，其他不管是0还是什么都会返回value1

     * 跟函数`if(value, t, f`)的区别在于，如果value是0或者null（或者等式但不成立），返回false，其他都是被返回true

  2. `join ....using(xx)  ==  join ... on  a.xx = b.xx`:注意这里必须两表的相关项同名，using才能使用，on没有这个限制

  3. 最重要的一点! 对于销量units为0也就是本应记录平均价格为0的产品会被过滤

     * 原因在于**WHERE子句过滤掉了所有没有销售记录的产品**！

     ~~~
     当使用LEFT JOIN后立即使用WHERE条件时：
     对于有销售记录的产品：u.purchase_date BETWEEN p.start_date AND p.end_date条件成立，记录被保留
     对于没有销售记录的产品：u.purchase_date为NULL，NULL BETWEEN ...的结果为NULL（即false），这些记录被WHERE过滤掉了
     ~~~

  4. 解决方法就是将where条件转移到join的on条件里

     * 区别：join在where操作前，决定哪些记录参与join操作，不影响主表的记录保留；where对所有join后的记录进行筛选，会影响最终结果

~~~
# 最终的代码呈现：
select p.product_id, if(sum(u.units) > 0 ,ROUND(sum(p.price * u.units) / sum(u.units), 2), 0) average_price
from Prices p LEFT JOIN UnitsSold u 
ON p.product_id = u.product_id
AND u.purchase_date BETWEEN p.start_date AND p.end_date
GROUP BY p.product_id;
~~~

### where 和 having group的区别

* 针对力扣题目https://leetcode.cn/problems/sales-analysis-iii/

```mysql
select product_id, product_name
from Sales join Product using(product_id)
where min(sale_date) >= "2019-01-01" and max(sale_date) <=  "2019-03-31";
```

* 这个 SQL 语句的错误在于**在 `WHERE` 子句中使用了聚合函数 `MIN()` 和 `MAX()`**，这在 SQL 语法中是不允许的。
  * 具体原因在于：
    1. **`WHERE` 子句的作用**：用于筛选**单条记录**，它在聚合函数（如 `MIN`/`MAX`）计算之前执行。
    2. **聚合函数的特性**：`MIN()`、`MAX()` 等聚合函数是对**一组记录**进行计算（如计算某个分组的最小值 / 最大值），需要在 `GROUP BY` 分组之后执行。
    3. 冲突点：
       * 你的语句想要通过 `MIN(sale_date)` 和 `MAX(sale_date)` 判断「某个商品的所有销售日期是否都在 2019-01-01 到 2019-03-31 之间」。
       * 但 `WHERE` 子句无法直接使用聚合函数的结果作为筛选条件，因为此时还未进行分组和聚合计算。
  * 因此，**不能够在where语句中使用聚合函数**，应使用 `HAVING` 子句配合 `GROUP BY`，因为 `HAVING` 是在聚合计算后对**分组结果**进行筛选

```mysql
SELECT product_id, product_name
FROM Sales 
JOIN Product USING(product_id)
GROUP BY product_id, product_name  -- 按商品分组
HAVING MIN(sale_date) >= "2019-01-01" 
   AND MAX(sale_date) <= "2019-03-31";  -- 对分组后的聚合结果筛选
```

---

# SQL语言篇温习

### 1. 创建数据库

~~~mysql
create database [ if not exists ] 数据库名 
[ default charset 字符集 ] 
[ collate 排序规则 ] ;
~~~

* 常用字符集和排序规则组合

  ```mysql
  -- UTF8通用设置
  create database myapp_db 
  default charset utf8mb4 
  collate utf8mb4_unicode_ci;
  
  -- 中文环境常用
  create database chinese_db 
  default charset utf8mb4 
  collate utf8mb4_chinese_ci;
  
  -- 大小写敏感
  create database case_sensitive_db 
  default charset utf8mb4 
  collate utf8mb4_bin;
  ```

* 解释：

  * 常用字符集说明：
    * **`utf8mb4`**: 支持4字节UTF8编码（推荐，支持emoji）
    * **`utf8`**: 基本UTF8编码（3字节）
    * **`gbk`**: 中文编码
    * **`latin1`**: 西欧语言编码
  * 常用排序规则说明：
    * **`utf8mb4_unicode_ci`**: 基于Unicode标准的排序，多语言支持好
    * **`utf8mb4_general_ci`**: 简单快速的排序（旧版）
    * **`utf8mb4_bin`**: 二进制排序，区分大小写
    * **`utf8mb4_chinese_ci`**: 针对中文优化的排序



### 2. 数据库相关术语

- **Console** = 控制台 / 查询控制台
- **Database** = 数据库
- **Schema** = 模式 / 架构
- **Query** = 查询
- **Execute** = 执行
- **Connection** = 连接
- **Refresh** = 刷新
- CONSTRAINT = 约束
- column = 列
- INDEX = 索引

### 3. DESC语句查询表结构

* `DESCRIBE table_name`:

  * 返回类似的结果：

    | Field  | Type        | Null | Key  | Default | Extra |
    | :----- | :---------- | :--- | :--- | :------ | :---- |
    | id     | int         | YES  |      | NULL    |       |
    | name   | varchar(50) | YES  |      | NULL    |       |
    | age    | int         | YES  |      | NULL    |       |
    | gender | varchar(1)  | YES  |      | NULL    |       |

  * 解释：

    * ### **ield（字段名）**

      - **含义**: 表中每个列的名称
      - **示例**: `id`, `name`, `age`, `gender`

    * ### **Type（数据类型）**

      - **含义**: 字段的数据类型和长度
      - **示例**:`int`- 整数类型`varchar(50)`- 最大50字符的变长字符串`datetime`- 日期时间类型

    * ### **Null（是否允许空值）**

      - **含义**: 该字段是否允许存储 NULL 值
      - **取值**:`YES`- 允许为 NULL`NO`- 不允许为 NULL（必须要有值）
      - **在你的表中**: 所有字段都是 `YES`，表示都可以为空

    * ###  **Key（键类型）**

      - **含义**: 字段是否被定义为键（索引）
      - **常见取值**:
        - `PRI`- 主键 (Primary Key)
        - `UNI`- 唯一键 (Unique Key)
        - `MUL`- 普通索引 (Multiple)
        - 空值 - 不是任何键
      - **在你的表中**: 所有字段都是空，表示没有定义主键或索引

    * ###  **Default（默认值）**

      - **含义**: 当插入数据未指定该字段值时使用的默认值
      - **常见取值**:
        - `NULL`- 默认为空值
        - 具体值如 `0`, `'默认文本'`, `CURRENT_TIMESTAMP`等
      - **在你的表中**: 所有字段默认都是 `NULL`

    * ###  **Extra（额外信息）**

      - **含义**: 字段的额外属性
      - **常见取值**:
        - `auto_increment`- 自动递增（常用于主键ID）
        - `on update CURRENT_TIMESTAMP`- 更新时自动设置时间戳
        - 空值 - 无特殊属性
      - **在你的表中**: 所有字段都是空，表示没有特殊属性

  * `show create table 表名` ： 查询指定表的建表语句

### 4. sql中的数据类型

* MySQL中的数据类型有很多，主要分为三类：数值类型、字符串类型、日期时间类型。
  * `DECIMAL`，用于计算精确小数：依赖于M(精度)和D(标度)的值。（即总位数和小数位数）
    * eg： decimal(4，3) ---> 即精确三位小数，一位整数

* 时间类型，DATETIME和TIMESTAMP类型有什么区别？

| 特性         | DATETIME                                   | TIMESTAMP                                  |
| :----------- | :----------------------------------------- | :----------------------------------------- |
| **范围**     | 1000-01-01 00:00:00 到 9999-12-31 23:59:59 | 1970-01-01 00:00:01 到 2038-01-19 03:14:07 |
| **存储空间** | 8字节                                      | 4字节                                      |
| **时区处理** | 存储输入的日期时间，不涉及时区转换         | 存储为UTC时间，检索时转换为当前时区        |
| **自动更新** | 不支持自动更新（MySQL 5.6.5+支持）         | 支持自动更新（如更新时间戳）               |
| **默认值**   | 允许默认值为常量                           | 可以设置为 CURRENT_TIMESTAMP               |

---

### 5. 创建表--案例

* 设计一张员工信息表，要求如下：

  1. 编号（纯数字）

  2. 员工工号 (字符串类型，长度不超过10位)

  3. 员工姓名（字符串类型，长度不超过10位）

  4. 性别（男/女，存储一个汉字）

  5. 年龄（正常人年龄，不可能存储负数）

  6. 身份证号（二代身份证号均为18位，身份证中有X这样的字符）

  7. 入职时间（取值年月日即可）

~~~mysql
CREATE TABLE `employee` (
    `id` INT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '编号',
    `employee_id` VARCHAR(10) NOT NULL UNIQUE COMMENT '员工工号',
    `name` VARCHAR(10) NOT NULL COMMENT '员工姓名',
    `gender` ENUM('男', '女') NOT NULL COMMENT '性别',
    `age` TINYINT UNSIGNED NOT NULL COMMENT '年龄',
    `id_card` CHAR(18) NOT NULL UNIQUE COMMENT '身份证号',
    `hire_date` DATE NOT NULL COMMENT '入职时间',
    `create_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `update_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (`id`),
    UNIQUE KEY `uk_employee_id` (`employee_id`),
    UNIQUE KEY `uk_id_card` (`id_card`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='员工信息表';
~~~

* 1.解释性别那里的枚举语法；2.既然前面的部分已经写过wmployee_id和id_card的unique了，后面那个有什么用，3.主键可以在id那一行后面直接加primary key吗，有什么区别。4.时间戳的两句解释下具体语法。5. ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='员工信息表';解释

* ----------------------------------> 解释

1. 枚举语法

  ~~~MySQL
  `gender` ENUM('男', '女') NOT NULL COMMENT '性别'
  ~~~

  * **ENUM 语法**：`ENUM('值1', '值2', '值3', ...)`

    * ### 工作原理：

      - **定义时**：指定允许的值列表 `('男', '女')`
      - **插入时**：只能插入定义的值或 NULL（如果允许）
      - **存储时**：实际存储的是值的索引（1字节），不是字符串本身

2. unique约束的两种写法区别

   * 写法一：列级约束

     ~~~mysql
     `employee_id` VARCHAR(10) NOT NULL UNIQUE
     ~~~

     * 写法二：表级约束

     ~~~mysql
     UNIQUE KEY `uk_employee_id` (`employee_id`)
     ~~~

     * 区别：

     | 特性         | 列级约束         | 表级约束         |
     | :----------- | :--------------- | :--------------- |
     | **功能**     | 完全相同         | 完全相同         |
     | **命名**     | 系统自动生成名称 | 可以自定义名称   |
     | **多列唯一** | 不支持           | 支持多列组合唯一 |
     | **可读性**   | 简洁             | 更清晰，便于管理 |

3. 主键的两种写法区别：

   * 列级主键和表级主键

   ~~~mysql
   `id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY
   -- 上列下表
   `id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
   PRIMARY KEY (`id`)
   ~~~

   * 区别：

   | 特性         | 列级主键   | 表级主键     |
   | :----------- | :--------- | :----------- |
   | **功能**     | 完全相同   | 完全相同     |
   | **简洁性**   | 更简洁     | 稍显冗长     |
   | **多列主键** | 不支持     | 支持复合主键 |
   | **可读性**   | 适合单主键 | 适合复杂约束 |

4. 时间戳语法

~~~mysql
`create_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
`update_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间'
~~~

- **DEFAULT CURRENT_TIMESTAMP**：插入数据时，如果不指定该字段值，自动设置为当前时间戳
- **效果**：记录数据创建的时间

- **DEFAULT CURRENT_TIMESTAMP**：插入时自动设置当前时间
- **ON UPDATE CURRENT_TIMESTAMP**：更新数据时，自动更新为当前时间
- **效果**：记录数据最后修改的时间

---

5. 表选项详细解释

~~~mysql
ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='员工信息表'
~~~

* ENGINE=InnoDB

  - **存储引擎**：使用 InnoDB 引擎

  - **优势**：支持事务、行级锁、外键约束、崩溃恢复

  - **对比**：MyISAM（不支持事务，表级锁）


* DEFAULT CHARSET=utf8mb4

  - **默认字符集**：utf8mb4

  - **支持范围**：完整的 Unicode 字符（包括 emoji 表情）

  - **对比**：utf8（只支持基本多文种平面）


* COLLATE=utf8mb4_unicode_ci

  - **排序规则**：utf8mb4_unicode_ci

  - **unicode_ci**：基于 Unicode 标准的排序，支持多语言

  - **ci**：Case Insensitive（不区分大小写）

  - **对比**：utf8mb4_general_ci（较老的通用排序）


* COMMENT='员工信息表'

  - **表注释**：为表添加描述信息

  - **用途**：便于文档化和维护

---

# SQL 表修改语法详解

SQL 提供了多种修改表结构的语句，主要包括创建表、修改表结构、重命名表和删除表等操作。

## 1. CREATE TABLE - 创建表

### 基本语法

```mysql
CREATE TABLE table_name (
    column1 datatype constraint,
    column2 datatype constraint,
    ...
    constraint_constraints
);
```

### 示例

```sql
-- 创建学生表
CREATE TABLE students (
    student_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    age INT CHECK (age >= 0),
    email VARCHAR(100) UNIQUE,
    enrollment_date DATE DEFAULT CURRENT_DATE
);

-- 创建带外键的课程表
CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL,
    instructor VARCHAR(50),
    student_id INT,
    FOREIGN KEY (student_id) REFERENCES students(student_id)
);
```

## 2. ALTER TABLE - 修改表结构

### 添加列

```sql
ALTER TABLE table_name 
ADD column_name datatype constraint;

-- 示例
ALTER TABLE students 
ADD phone_number VARCHAR(15);

-- 添加多个列
ALTER TABLE students 
ADD (
    address VARCHAR(200),
    birth_date DATE
);
```

### 修改列定义

```sql
-- 修改数据类型
ALTER TABLE table_name 
MODIFY column_name new_datatype;

-- 示例
ALTER TABLE students 
MODIFY name VARCHAR(100) NOT NULL;

-- 修改多个列
ALTER TABLE students 
MODIFY (
    age SMALLINT,
    email VARCHAR(150)
);
```

### 重命名列

```sql
-- MySQL
ALTER TABLE table_name 
CHANGE old_column_name new_column_name datatype;

-- SQL Server / PostgreSQL
ALTER TABLE table_name 
RENAME COLUMN old_name TO new_name;

-- 示例
ALTER TABLE students 
CHANGE name full_name VARCHAR(100);
```

### 删除列

```sql
ALTER TABLE table_name 
DROP COLUMN column_name;

-- 示例
ALTER TABLE students 
DROP COLUMN phone_number;

-- 删除多个列
ALTER TABLE students 
DROP COLUMN address, 
DROP COLUMN birth_date;
```

### 添加约束

```sql
-- 添加主键
ALTER TABLE table_name 
ADD PRIMARY KEY (column_name);

-- 添加外键
ALTER TABLE table_name 
ADD FOREIGN KEY (column_name) REFERENCES other_table(column_name);

-- 添加唯一约束
ALTER TABLE table_name 
ADD UNIQUE (column_name);

-- 添加检查约束
ALTER TABLE table_name 
ADD CHECK (condition);

-- 示例
ALTER TABLE students 
ADD CONSTRAINT chk_age CHECK (age BETWEEN 0 AND 120);

ALTER TABLE courses 
ADD CONSTRAINT fk_student 
FOREIGN KEY (student_id) REFERENCES students(student_id);
```

### 删除约束

```sql
-- 删除约束（需要知道约束名）
ALTER TABLE table_name 
DROP CONSTRAINT constraint_name;

-- 删除主键
ALTER TABLE table_name 
DROP PRIMARY KEY;

-- 删除外键
ALTER TABLE table_name 
DROP FOREIGN KEY constraint_name;

-- 示例
ALTER TABLE students 
DROP CONSTRAINT chk_age;
```

## 3. RENAME TABLE - 重命名表

```sql
-- MySQL
RENAME TABLE old_name TO new_name;

-- 或使用 ALTER TABLE
ALTER TABLE old_name RENAME TO new_name;

-- SQL Server
sp_rename 'old_name', 'new_name';

-- 示例
RENAME TABLE students TO university_students;
```

## 4. TRUNCATE TABLE - 清空表数据

```sql
TRUNCATE TABLE table_name;

-- 示例
TRUNCATE TABLE courses;
```

## 5. DROP TABLE - 删除表

```SQL
DROP TABLE table_name;

-- 删除多个表
DROP TABLE table1, table2, table3;

-- 带条件删除（如果表存在）
DROP TABLE IF EXISTS table_name;

-- 示例
DROP TABLE IF EXISTS old_students;
```

## 6. 复杂示例

### 完整的表修改流程

```SQL
-- 创建初始表
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    hire_date DATE
);

-- 添加列
ALTER TABLE employees 
ADD email VARCHAR(100), 
ADD department VARCHAR(50);

-- 修改列
ALTER TABLE employees 
MODIFY first_name VARCHAR(100) NOT NULL;

-- 添加约束
ALTER TABLE employees 
ADD CONSTRAINT uk_email UNIQUE (email),
ADD CONSTRAINT chk_hire_date CHECK (hire_date >= '2000-01-01');

-- 重命名列
ALTER TABLE employees 
CHANGE department dept_name VARCHAR(60);

-- 删除列
ALTER TABLE employees 
DROP COLUMN dept_name;

-- 最终重命名表
ALTER TABLE employees RENAME TO company_employees;
```

## 7. 数据库特定的语法差异

### MySQL 特有语法

```sql
-- 添加自增属性
ALTER TABLE table_name 
MODIFY column_name INT AUTO_INCREMENT;

-- 添加索引
ALTER TABLE table_name 
ADD INDEX index_name (column_name);
```

### PostgreSQL 特有语法

```sql
-- 添加自增序列
ALTER TABLE table_name 
ALTER COLUMN column_name SET DEFAULT nextval('sequence_name');

-- 修改列类型（更严格的语法）
ALTER TABLE table_name 
ALTER COLUMN column_name TYPE new_data_type USING expression;
```

### SQL Server 特有语法

```sql
-- 添加标识列
ALTER TABLE table_name 
ADD column_name INT IDENTITY(1,1);

-- 修改列为空性
ALTER TABLE table_name 
ALTER COLUMN column_name data_type NULL|NOT NULL;
```

## 注意事项

1. **备份数据**：在修改表结构前，务必备份重要数据
2. **影响性能**：大型表的修改操作可能很耗时
3. **依赖关系**：修改有外键关联的表时需要特别小心
4. **事务处理**：在事务中执行修改操作，以便在出错时回滚

## 最佳实践

```sql
-- 使用事务确保操作原子性
BEGIN TRANSACTION;

ALTER TABLE employees 
ADD salary DECIMAL(10,2);

ALTER TABLE employees 
ADD CONSTRAINT chk_salary CHECK (salary >= 0);

COMMIT;

-- 检查表结构
DESCRIBE employees;  -- MySQL
-- 或
SELECT * FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'employees';
```

这些语法涵盖了 SQL 中表修改的主要操作，具体使用时请参考你所使用的数据库系统的文档，因为不同数据库管理系统可能会有细微的语法差异。

---



# DML 语言详解：增删改操作

## 1. DML 概述

**DML**（Data Manipulation Language，数据操作语言）用于对数据库中的**数据进行操作**，主要包括：

- **INSERT** - 插入数据
- **UPDATE** - 更新数据
- **DELETE** - 删除数据
- **SELECT** - 查询数据（通常也归为DML）

## 2. INSERT - 插入数据

### 基本语法

```SQL
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);
```

### 详细语法变体

#### 方式1：指定列名插入

```SQL
INSERT INTO employees (employee_id, name, department, salary, hire_date)
VALUES (1, '张三', '技术部', 8000.00, '2023-01-15');
```

#### 方式2：省略列名（必须提供所有列的值）

```SQL
INSERT INTO employees
VALUES (2, '李四', '销售部', 7000.00, '2023-02-20');
```

#### 方式3：插入多行数据

```SQL
INSERT INTO employees (employee_id, name, department, salary, hire_date)
VALUES 
    (3, '王五', '人事部', 6000.00, '2023-03-10'),
    (4, '赵六', '财务部', 7500.00, '2023-04-05'),
    (5, '钱七', '技术部', 8500.00, '2023-05-12');
```

#### 方式4：从其他表插入数据

```SQL
-- 从临时表插入
INSERT INTO employees (employee_id, name, department)
SELECT temp_id, temp_name, temp_dept FROM temp_employees
WHERE temp_status = 'active';
```

### 实际示例

#### 创建测试表

```SQL
CREATE TABLE students (
    student_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    age INT,
    gender ENUM('男', '女'),
    major VARCHAR(100),
    enrollment_date DATE DEFAULT CURRENT_DATE
);
```

#### 插入操作示例

```SQL
-- 插入单条记录（指定列）
INSERT INTO students (name, age, gender, major)
VALUES ('张三', 20, '男', '计算机科学');

-- 插入单条记录（省略列名）
INSERT INTO students 
VALUES (NULL, '李四', 21, '女', '软件工程', '2023-09-01');

-- 插入多条记录
INSERT INTO students (name, age, gender, major) VALUES
('王五', 19, '男', '人工智能'),
('赵六', 22, '女', '数据科学'),
('钱七', 20, '男', '网络安全');

-- 插入默认值
INSERT INTO students (name, age, gender) 
VALUES ('孙八', 23, '男');
```

## 3. UPDATE - 更新数据

### 基本语法

```SQL
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

### 详细语法变体

#### 更新单列

```SQL
UPDATE employees 
SET salary = 9000.00 
WHERE employee_id = 1;
```

#### 更新多列

```SQL
UPDATE employees 
SET salary = 9500.00, department = '高级技术部' 
WHERE employee_id = 1;
```

#### 基于表达式的更新

```SQL
-- 工资上涨10%
UPDATE employees 
SET salary = salary * 1.1 
WHERE department = '技术部';
```

#### 使用子查询更新

```SQL
-- 根据平均工资调整
UPDATE employees 
SET salary = salary * 1.05 
WHERE salary < (SELECT AVG(salary) FROM employees);
```

### 实际示例

#### 创建测试数据

```SQL
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10,2),
    stock_quantity INT,
    last_updated DATE
);

INSERT INTO products VALUES
(1, 'iPhone 14', '手机', 5999.00, 100, '2023-01-01'),
(2, 'MacBook Pro', '电脑', 12999.00, 50, '2023-01-01'),
(3, 'iPad Air', '平板', 4399.00, 80, '2023-01-01'),
(4, 'AirPods', '耳机', 1299.00, 200, '2023-01-01');
```

#### 更新操作示例

```SQL
-- 更新单个产品的价格
UPDATE products 
SET price = 5799.00 
WHERE product_id = 1;

-- 更新多个字段
UPDATE products 
SET price = price * 0.9, last_updated = CURRENT_DATE 
WHERE category = '手机';

-- 条件更新：库存不足时补充库存
UPDATE products 
SET stock_quantity = stock_quantity + 50, last_updated = CURRENT_DATE 
WHERE stock_quantity < 60;

-- 使用CASE语句的条件更新
UPDATE products 
SET price = CASE 
    WHEN category = '手机' THEN price * 1.1
    WHEN category = '电脑' THEN price * 1.05
    ELSE price * 1.02
END,
last_updated = CURRENT_DATE;
```

## 4. DELETE - 删除数据

### 基本语法

```SQL
DELETE FROM table_name WHERE condition;
```

### 详细语法变体

#### 删除特定记录

```SQL
DELETE FROM employees WHERE employee_id = 5;
```

#### 删除满足条件的多条记录

```SQL
DELETE FROM employees WHERE department = '临时部';
```

#### 清空整个表

```SQL
DELETE FROM employees;  -- 删除所有记录，但表结构保留
```

#### 使用子查询删除

```SQL
-- 删除工资低于平均工资的员工
DELETE FROM employees 
WHERE salary < (SELECT AVG(salary) FROM employees);
```

### 实际示例

#### 创建测试数据

```SQL
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    quantity INT,
    order_date DATE,
    status ENUM('pending', 'shipped', 'delivered', 'cancelled')
);

INSERT INTO orders VALUES
(1, 101, 1, 2, '2023-10-01', 'delivered'),
(2, 102, 2, 1, '2023-10-02', 'shipped'),
(3, 101, 3, 3, '2023-10-03', 'pending'),
(4, 103, 1, 1, '2023-09-15', 'cancelled'),
(5, 104, 4, 5, '2023-10-04', 'pending');
```

#### 删除操作示例

```SQL
-- 删除特定订单
DELETE FROM orders WHERE order_id = 4;

-- 删除已取消的订单
DELETE FROM orders WHERE status = 'cancelled';

-- 删除一个月前的待处理订单
DELETE FROM orders 
WHERE status = 'pending' 
AND order_date < DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY);

-- 删除某个客户的所有订单
DELETE FROM orders WHERE customer_id = 101;
```



## 5. 总结

**DML三大操作核心要点：**

* INSERT

  - 用于添加新记录
  - 可以单条或多条插入

  - 支持从其他表插入数据

* UPDATE

  - 用于修改现有数据

  - 必须使用WHERE条件避免误更新

  - 支持基于表达式和子查询的更新

* DELETE

  - 用于删除记录

  - 必须使用WHERE条件避免误删除

  - 表结构保留，只删除数据

**重要提醒：**

- 始终在UPDATE和DELETE中使用WHERE条件
- 生产环境操作前先备份数据
- 使用事务保证操作原子性
- 测试环境验证后再在生产环境执行



## alter 和 update、modify、add的区别！

### alter 和 update的核心区别

| 特性         | ALTER                  | UPDATE              |
| :----------- | :--------------------- | :------------------ |
| **操作对象** | 表结构（容器本身）     | 表数据（容器内容）  |
| **修改什么** | 表定义、列、约束、索引 | 记录中的字段值      |
| **SQL类型**  | DDL（数据定义语言）    | DML（数据操作语言） |
| **事务支持** | 有限，多数自动提交     | 完整事务支持        |
| **回滚能力** | 通常不能回滚           | 可以回滚            |
| **使用频率** | 较低（结构变化时）     | 很高（日常操作）    |

* 实际语法对比：

~~~sql
-- ALTER：修改表结构
ALTER TABLE employees ADD COLUMN email VARCHAR(100);
ALTER TABLE employees DROP COLUMN phone;
ALTER TABLE employees RENAME TO staff;

-- UPDATE：修改表数据
UPDATE employees SET salary = 5000 WHERE id = 1;
UPDATE employees SET department = 'IT', title = '经理' WHERE name = '张三';
~~~

### ADD vs MODIFY 的核心区别

##### *  根本区别（都是ALTER的子命令）

| 特性         | ADD                    | MODIFY           |
| :----------- | :--------------------- | :--------------- |
| **操作目的** | 添加新元素             | 修改现有元素     |
| **操作对象** | 新列、新约束、新索引   | 现有列的定义     |
| **影响范围** | 增加表结构元素         | 改变现有结构属性 |
| **数据影响** | 新列初始为NULL或默认值 | 可能涉及数据转换 |

* 实际语法对比：

~~~sql
-- ADD：添加新元素
ALTER TABLE employees ADD COLUMN birth_date DATE;                    -- 添加新列
ALTER TABLE employees ADD CONSTRAINT uk_email UNIQUE (email);        -- 添加约束
ALTER TABLE employees ADD INDEX idx_department (department);         -- 添加索引

-- MODIFY：修改现有元素
ALTER TABLE employees MODIFY COLUMN name VARCHAR(100) NOT NULL;     -- 修改列定义
ALTER TABLE employees MODIFY COLUMN salary DECIMAL(12,2) DEFAULT 0;  -- 修改默认值
~~~

`ALTER TABLE（表级操作）`
    `├── ADD（添加元素）`
    `│   ├── ADD COLUMN（添加列）`
    `│   ├── ADD CONSTRAINT（添加约束）`
    `│   └── ADD INDEX（添加索引）`
    `│`
    `├── MODIFY（修改元素）`
    `│   └── MODIFY COLUMN（修改列定义）`
    `│`
    `├── DROP（删除元素）`
    `└── RENAME（重命名）`

`UPDATE（数据级操作，独立于ALTER）`
    `└── 修改记录内容`

---

# DQL语句详解

~~~sql
-- DQL 查询语句，语法结构如下：
SELECT
	字段列表
FROM
	表名列表
WHERE
	条件列表
GROUP BY
	分组字段列表
HAVING
	分组后条件列表
ORDER BY
	排序字段列表
LIMIT
	分页参数
~~~

好的，我们来逐一解答这些问题，这些问题都非常核心，能帮助你深入理解 SQL 的执行逻辑。

------

### 1. 聚合函数只能用在 SELECT 语句和 HAVING 里，不能在 WHERE 里对吗？

**对，完全正确。**

- **原因：** SQL 语句的执行顺序是关键。`WHERE`子句是在 **分组前（GROUP BY）** 对**每一行原始数据**进行过滤。此时，数据还没有被分组，聚合函数（如 `COUNT`, `SUM`, `AVG`）根本无从计算，因为它们的意义就是对一组数据进行运算。
- `HAVING`子句则是在 **分组后（GROUP BY）** 对**已经分好的组**进行过滤。此时，每个组的数据已经聚合完毕，所以可以使用聚合函数来设置过滤条件。

**错误示例（会报错）：**

```
SELECT department, AVG(salary)
FROM employees
WHERE AVG(salary) > 10000  -- 错误！WHERE 不能使用聚合函数
GROUP BY department;
```

**正确示例：**

```
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 10000; -- 正确！HAVING 在分组后执行
```

------

### 2. SELECT 语句每个部分的执行顺序

这是一个非常重要的问题！SQL 的**书写顺序**和**执行顺序**是不同的。

- **书写顺序：** `SELECT`-> `FROM`-> `WHERE`-> `GROUP BY`-> `HAVING`-> `ORDER BY`-> `LIMIT`
- **执行顺序：** `FROM`-> `WHERE`-> `GROUP BY`-> `HAVING`-> `SELECT`-> `ORDER BY`-> `LIMIT`

**详细解释：**

1. **FROM**: 首先确定数据来源，加载整个表或多表连接后的结果集。
2. **WHERE**: 对 FROM 阶段得到的所有原始数据行进行过滤，排除不满足条件的行。
3. **GROUP BY**: 将 WHERE 过滤后的数据，按照指定的列进行分组。
4. **HAVING**: 对 GROUP BY 生成的分组结果进行过滤，排除不满足条件的分组。
5. **SELECT**: **最后才轮到 SELECT！** 从经过前面所有步骤处理后的结果中，选择需要显示的列，并计算表达式或聚合函数。这也是为什么别名不能在 WHERE 或 GROUP BY 中使用（因为还没执行到 SELECT），但可以在 ORDER BY 中使用。
6. **ORDER BY**: 对 SELECT 出来的最终结果集进行排序。
7. **LIMIT**: 限制返回的最终行数。

------

### 3. `COUNT(1)`是什么意思？`SELECT COUNT(1) FROM emp`有什么用？

- **`COUNT(expression)`的作用：** 统计表中**非 NULL** 的 `expression`的行数。
- **常见用法：** `COUNT(*)`: 统计所有行的数量，**包括 NULL 行**。这是最常用的方式。 `COUNT(column_name)`: 统计指定列中**非 NULL 值**的数量。 `COUNT(1)`/ `COUNT(0)`/ `COUNT('任何常量')`: 这里的 `1`不是一个列名，而是一个常量表达式。对于每一行，这个表达式的值都是 `1`（非 NULL），所以 `COUNT(1)`也会统计**所有行的数量**。
- **`SELECT COUNT(1) FROM emp`的作用：** 这条语句的作用就是**统计 `emp`表中的总记录数**。它与 `SELECT COUNT(*) FROM emp`的结果是完全一样的。
- **性能差异：** 在现代主流数据库（如 MySQL, PostgreSQL）中，优化器会对 `COUNT(*)`进行特殊优化，其性能与 `COUNT(1)`几乎没有区别，甚至更好。因此，**更推荐使用语义更清晰的 `COUNT(\*)`**。

------

### 4. 分组查询中多字段分组是满足两个条件的分一组还是按照两个字段分成两个组？

**是“满足两个字段组合的唯一值”分为一组。** 它不是分成两个独立的组，而是将 `GROUP BY`后面的多个字段看作一个**复合键**。

**示例：** `GROUP BY workaddress, gender`

这会将数据先按 `workaddress`分组，然后在每个 `workaddress`内部，再按 `gender`进行细分。

结果会是：

- (北京, 男)
- (北京, 女)
- (上海, 男)
- (上海, 女) ...等等，每一个唯一的 `(workaddress, gender)`组合都会形成一个独立的分组。

------

### 5. & 6. SELECT列表中的非聚合列必须出现在GROUP BY子句中，但GROUP BY里的非聚合列不必出现在SELECT中，对吗？为什么？

**对，这个理解是正确的。**

- **规则：** 在包含 `GROUP BY`的查询中，`SELECT`子句中出现的列，要么是 **聚合函数** 的参数，要么就必须出现在 **GROUP BY** 子句中。这是为了确保每一行结果的意义明确：每一行代表一个分组，`SELECT`中的非聚合列必须是用来定义这个分组的列。
- **为什么 GROUP BY 中的列可以不出现在 SELECT 中？** **因为 GROUP BY 的目的是定义如何分组，而 SELECT 的目的是决定最终显示什么。** 有时我们分组只是为了进行聚合计算，但并不关心分组依据的具体值。 **示例：** 我们只想知道公司总共有多少个不同的部门，但不关心是哪些部门。 `SELECT COUNT(*) AS dept_count FROM employees GROUP BY department; -- `department` 用于分组，但不需要显示出来`结果可能只是一行：`dept_count: 3`。

**现在分析你提供的两段代码：**

```
-- 代码1
SELECT workaddress, gender, count(*) as '员工数量'
from emp
group by workaddress, gender;

-- 代码2
select workaddress, gender, count(*) '数量' from emp group by gender , workaddress;
```

- **它们求的是什么？** 两段代码的功能**完全一样**。都是**统计各个工作地址下，男性员工和女性员工分别有多少人**。例如： 北京 男 15人 北京 女 20人 上海 男 10人 上海 女 18人
- **有什么区别？** **唯一的区别是 `GROUP BY`子句中字段的书写顺序**：第一段是 `workaddress, gender`，第二段是 `gender, workaddress`。 **在结果上，这没有任何区别。** 因为分组依据是这两个字段的组合，与顺序无关。`(A, B)`和 `(B, A)`定义的组是完全相同的。 **在性能上，** 如果为这些字段建立了复合索引，那么 `GROUP BY`的顺序如果与索引顺序一致，可能会带来性能优势。但在逻辑结果上，两者等价。

------

### 7. ORDER BY / HAVING 里的非聚合列需要出现在 GROUP BY 里吗？

- **ORDER BY： 不需要。** `ORDER BY`是在 `SELECT`之后执行的，它是对最终的结果集进行排序。你可以根据最终结果集中的任何列（包括不在 `GROUP BY`中的列，只要它能被唯一确定地选出）进行排序。但要注意，如果排序列不是分组列或聚合列，结果可能不符合预期。 **示例（可能无意义但语法正确）：** `SELECT department, AVG(salary) FROM employees GROUP BY department ORDER BY name; -- 语法上允许，但逻辑错误。一个分组对应多个人名，排序依据不明确。`
- **HAVING： 不一定需要，但通常会是聚合列。** `HAVING`用于过滤分组。它的条件可以基于： **聚合函数**（最常见）：`HAVING AVG(salary) > 10000` **分组列**（即出现在 `GROUP BY`中的列）：`HAVING department = '技术部'`。这种情况下，这个条件其实完全可以（并且更高效地）放在 `WHERE`子句中。

**分析你提供的最后两段代码：**

```
-- 代码A (错误/低效)
select gender, count(*) '人数'
from emp
group by gender, age  -- 按性别和年龄分组
having age < 60;      -- 再过滤掉 age>=60 的组

-- 代码B (正确/高效)
select gender, count(*) from emp where age < 60 group by gender;
```

- **代码A 的问题：** **逻辑错误/低效：** `GROUP BY gender, age`会把数据按“性别和年龄的每个唯一组合”分成非常细的组（例如：男-25岁一组，男-26岁一组...）。然后 `HAVING age < 60`会过滤掉那些年龄分组键大于等于60的组。这完全不是我们想要的结果。我们只是想先过滤掉年龄大于60岁的**个人**，然后再按性别统计人数。 **性能差：** 它先对全表进行了非常细粒度的分组，然后再过滤，如果数据量大，这会消耗大量资源。
- **代码B 的正确性：** 它使用 `WHERE age < 60`在分组**前**就过滤掉了所有年龄大于等于60岁的员工记录。然后对剩下的记录直接按 `gender`分组计数。这完全符合“统计60岁以下男女员工人数”的需求，并且性能好得多。

**结论：** **凡是能放在 WHERE 子句中的过滤条件，都不要放在 HAVING 子句中。** HAVING 只应用于那些必须在分组后才能确定的过滤条件（即涉及聚合函数的条件）。

---





