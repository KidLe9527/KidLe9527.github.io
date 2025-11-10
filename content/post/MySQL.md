---
title: 'MySQL学习过程心得体会'
date: 2025-08-10T22:01:00-08:00
lastmod: 2025-08-10T22:02:00-08:00
categories: ['笔记']
tags: ['MySQL']
cover: https://kidle9527.github.io/images/55.png
---

# 数据库学习记录

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



## 3.小难的题目

### using 和round函数的妙用

* [3642. 查找有两极分化观点的书籍 - 力扣（LeetCode）](https://leetcode.cn/problems/find-books-with-polarized-opinions/description/)

* 别人写的代码

```sql
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

```sql
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

~~~sql
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

~~~sql
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

## 4. SQL语言篇温习

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

## 5.SQL 表修改语法详解

SQL 提供了多种修改表结构的语句，主要包括创建表、修改表结构、重命名表和删除表等操作。

### 1. CREATE TABLE - 创建表

#### 基本语法

```mysql
CREATE TABLE table_name (
    column1 datatype constraint,
    column2 datatype constraint,
    ...
    constraint_constraints
);
```

#### 示例

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

### 2. ALTER TABLE - 修改表结构

#### 添加列

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

#### 修改列定义

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

#### 重命名列

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

#### 删除列

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

#### 添加约束

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

#### 删除约束

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

### 3. RENAME TABLE - 重命名表

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

### 4. TRUNCATE TABLE - 清空表数据

```sql
TRUNCATE TABLE table_name;

-- 示例
TRUNCATE TABLE courses;
```

### 5. DROP TABLE - 删除表

```SQL
DROP TABLE table_name;

-- 删除多个表
DROP TABLE table1, table2, table3;

-- 带条件删除（如果表存在）
DROP TABLE IF EXISTS table_name;

-- 示例
DROP TABLE IF EXISTS old_students;
```

### 6. 复杂示例

#### 完整的表修改流程

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

### 7. 数据库特定的语法差异

#### MySQL 特有语法

```sql
-- 添加自增属性
ALTER TABLE table_name 
MODIFY column_name INT AUTO_INCREMENT;

-- 添加索引
ALTER TABLE table_name 
ADD INDEX index_name (column_name);
```

#### PostgreSQL 特有语法

```sql
-- 添加自增序列
ALTER TABLE table_name 
ALTER COLUMN column_name SET DEFAULT nextval('sequence_name');

-- 修改列类型（更严格的语法）
ALTER TABLE table_name 
ALTER COLUMN column_name TYPE new_data_type USING expression;
```

#### SQL Server 特有语法

```sql
-- 添加标识列
ALTER TABLE table_name 
ADD column_name INT IDENTITY(1,1);

-- 修改列为空性
ALTER TABLE table_name 
ALTER COLUMN column_name data_type NULL|NOT NULL;
```

### 8. 注意事项

1. **备份数据**：在修改表结构前，务必备份重要数据
2. **影响性能**：大型表的修改操作可能很耗时
3. **依赖关系**：修改有外键关联的表时需要特别小心
4. **事务处理**：在事务中执行修改操作，以便在出错时回滚

### 9. 最佳实践

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

## 6. DML 语言详解：增删改操作

### 1. DML 概述

**DML**（Data Manipulation Language，数据操作语言）用于对数据库中的**数据进行操作**，主要包括：

- **INSERT** - 插入数据
- **UPDATE** - 更新数据
- **DELETE** - 删除数据
- **SELECT** - 查询数据（通常也归为DML）

### 2. INSERT - 插入数据

#### 基本语法

```SQL
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);
```

#### 详细语法变体

##### 方式1：指定列名插入

```SQL
INSERT INTO employees (employee_id, name, department, salary, hire_date)
VALUES (1, '张三', '技术部', 8000.00, '2023-01-15');
```

##### 方式2：省略列名（必须提供所有列的值）

```SQL
INSERT INTO employees
VALUES (2, '李四', '销售部', 7000.00, '2023-02-20');
```

##### 方式3：插入多行数据

```SQL
INSERT INTO employees (employee_id, name, department, salary, hire_date)
VALUES 
    (3, '王五', '人事部', 6000.00, '2023-03-10'),
    (4, '赵六', '财务部', 7500.00, '2023-04-05'),
    (5, '钱七', '技术部', 8500.00, '2023-05-12');
```

##### 方式4：从其他表插入数据

```SQL
-- 从临时表插入
INSERT INTO employees (employee_id, name, department)
SELECT temp_id, temp_name, temp_dept FROM temp_employees
WHERE temp_status = 'active';
```

#### 实际示例

##### 创建测试表

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

##### 插入操作示例

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

### 3. UPDATE - 更新数据

#### 基本语法

```SQL
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

#### 详细语法变体

##### 更新单列

```SQL
UPDATE employees 
SET salary = 9000.00 
WHERE employee_id = 1;
```

##### 更新多列

```SQL
UPDATE employees 
SET salary = 9500.00, department = '高级技术部' 
WHERE employee_id = 1;
```

##### 基于表达式的更新

```SQL
-- 工资上涨10%
UPDATE employees 
SET salary = salary * 1.1 
WHERE department = '技术部';
```

##### 使用子查询更新

```SQL
-- 根据平均工资调整
UPDATE employees 
SET salary = salary * 1.05 
WHERE salary < (SELECT AVG(salary) FROM employees);
```

#### 实际示例

##### 创建测试数据

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

##### 更新操作示例

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

### 4. DELETE - 删除数据

#### 基本语法

```SQL
DELETE FROM table_name WHERE condition;
```

#### 详细语法变体

##### 删除特定记录

```SQL
DELETE FROM employees WHERE employee_id = 5;
```

##### 删除满足条件的多条记录

```SQL
DELETE FROM employees WHERE department = '临时部';
```

##### 清空整个表

```SQL
DELETE FROM employees;  -- 删除所有记录，但表结构保留
```

##### 使用子查询删除

```SQL
-- 删除工资低于平均工资的员工
DELETE FROM employees 
WHERE salary < (SELECT AVG(salary) FROM employees);
```

#### 实际示例

##### 创建测试数据

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

##### 删除操作示例

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

---

### 5. 总结

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

---

## 7. alter 和 update、modify、add的区别！

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

### add vs modify 的核心区别

* 根本区别（都是ALTER的子命令）

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

## 8. DQL语句详解

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

## 9. SQL函数详解

SQL 函数主要可以分为两大类：**聚合函数** 和 **标量函数**。

### 1. 聚合函数

这类函数对一组值执行计算，并返回一个单一的值。它们通常与 `GROUP BY`子句一起使用，用于将多行数据分组并对每个组进行汇总。

- **`COUNT()`**: 返回匹配指定条件的行数。`COUNT(*)`: 计算所有行数，包括 NULL 值。`COUNT(column_name)`: 计算指定列中非 NULL 值的数量。
- **`SUM()`**: 返回数值列的总和。
- **`AVG()`**: 返回数值列的平均值。
- **`MAX()`**: 返回一列中的最大值。
- **`MIN()`**: 返回一列中的最小值。
- **`GROUP_CONCAT()`** (MySQL) / **`STRING_AGG()`** (SQL Server/PostgreSQL): 将一组行的字符串值连接成一个字符串。

~~~sql
-- 查询每个学生选修的所有课程（用分号分隔，去重并按课程名排序）
SELECT 
    student_id,
    GROUP_CONCAT(DISTINCT course ORDER BY course SEPARATOR '; ') AS courses
FROM students
GROUP BY student_id;
-- -------->结果见下
~~~

| student_id | courses    |
| :--------- | :--------- |
| 1          | 数学; 英语 |
| 2          | 化学; 物理 |

### 2. 标量函数

这类函数对单个值进行操作，并基于输入值返回另一个单一的值。它们可以应用于 SELECT、WHERE 等子句中。

标量函数又可以细分为以下几类：

- **字符串函数**：用于处理文本字符串。

  - `UPPER() / LOWER()`: 将字符串转换为大写/小写。
  - `LENGTH() / LEN()`: 返回字符串的长度。
  - `TRIM()`: 去除字符串首尾的空格或指定字符。
  - `CONCAT()`: 将两个或多个字符串连接起来。
  - `SUBSTRING() / SUBSTR()`: 从字符串中提取子串。

  ~~~sql
  -- 正常情况：从位置1开始
  SELECT SUBSTRING('Hello MySQL', 1, 5);  -- 结果: 'Hello'
  
  -- 从位置0开始
  SELECT SUBSTRING('Hello MySQL', 0, 5);  -- 结果: ''
  -- 在 SQL 的世界里，字符串位置从 1 开始计数。
  
  -- 从位置2开始
  SELECT SUBSTRING('Hello MySQL', 2, 5);  -- 结果: 'ello '
  
  -- 省略长度参数，提取到末尾
  SELECT SUBSTRING('Hello MySQL', 7);     -- 结果: 'MySQL'
  
  -- 使用负值（从末尾开始计数）
  SELECT SUBSTRING('Hello MySQL', -5);    -- 结果: 'MySQL'
  ~~~

  

  - `REPLACE()`: 替换字符串中的内容。

- **数值函数**：用于执行数学运算。

  - `ROUND(x,  y)`: 对数值进行四舍五入。
  - `CEIL() / CEILING()`: 向上取整。
  - `FLOOR()`: 向下取整。
  - `ABS()`: 返回绝对值。
  - `RAND()`: 生成一个随机数。（0~1之间）

- **日期和时间函数**：用于处理日期和时间值。

  - `NOW() / GETDATE()`: 返回当前的日期和时间。
  - `CURDATE() / GETDATE()`: 返回当前日期。
  - `CURTIME()`: 返回当前时间。
  - `date_add(date, INTERVAL expr type)`: 返回一个日期/时间值加上一个时间间隔expr后的时间值 ---------------> 好用！
  - `DATEDIFF(date1, date2)`: 返回两个日期 （前-后）之间的差值。
  - `YEAR() / MONTH() / DAY()`: 从日期中提取年、月、日。

- **转换函数**：用于转换数据类型。

  - `CAST()`: 将一种数据类型转换为另一种。
  - `CONVERT()`: 功能类似 `CAST`，但语法可能不同（尤其在 SQL Server 中）。

- **条件函数**：实现类似 `IF...ELSE`的逻辑。

  - `CASE ... WHEN ... THEN ... END`: 强大的条件表达式
  - `IF()`(MySQL): 简单的条件判断。
  - `COALESCE()`: 返回参数列表中第一个非 NULL 的值。
  - `ISNULL()`(SQL Server) / `IFNULL()`(MySQL): 检查是否为 NULL，并返回替代值。

---

## 10. 外键约束完整语法详解

### 1. **基本语法结构**

```sql
ALTER TABLE 子表名称
ADD CONSTRAINT 外键约束名称
FOREIGN KEY (子表字段) 
REFERENCES 父表名称(父表字段)
[ON DELETE 参照动作]
[ON UPDATE 参照动作];
```

### 2. **语法组成部分详解**

#### **📍 子表 (从表)**🍭

包含外键字段的表

```sql
ALTER TABLE orders  -- orders是子表
```

#### **📍 外键约束名称**

自定义的外键标识符

```sql
ADD CONSTRAINT fk_orders_user_id  -- 推荐命名：fk_子表_字段
```

#### **📍 外键字段**

子表中引用父表的字段

```sql
FOREIGN KEY (user_id)  -- orders表中的user_id字段
```

#### **📍 父表 (主表)**

被引用的主表

```sql
REFERENCES users(id)  -- 引用users表（父表）的id字段
```

### 3. **参照动作 (Referential Actions)**

#### **ON DELETE 动作**

| 动作          | 说明                                       | 示例                              |
| ------------- | ------------------------------------------ | --------------------------------- |
| `RESTRICT`    | **默认**，阻止删除父表记录                 | 有订单的用户不能被删除            |
| `CASCADE`     | **级联**删除子表记录                       | 删除用户时，同时删除其所有订单    |
| `SET NULL`    | 将子表外键设为NULL（要求外键允许取值NULL） | 删除用户时，订单的user_id设为NULL |
| `NO ACTION`   | 同RESTRICT                                 |                                   |
| `SET DEFAULT` | 设为默认值（MySQL不支持）                  |                                   |

#### **ON UPDATE 动作**

| 动作       | 说明               | 示例                                |
| ---------- | ------------------ | ----------------------------------- |
| `CASCADE`  | 级联更新子表外键   | 用户id改变时，订单的user_id同步更新 |
| `SET NULL` | 将子表外键设为NULL |                                     |
| `RESTRICT` | 阻止更新父表主键   |                                     |

### 4. **实际示例**

#### **示例1：简单的用户-订单关系**

```sql
-- 父表：用户表
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50)
);

-- 子表：订单表
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    amount DECIMAL(10,2),
    -- 添加外键约束
    CONSTRAINT fk_orders_user 
    FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE RESTRICT  -- 阻止删除有订单的用户
    ON UPDATE CASCADE   -- 用户id更新时，订单同步更新
);
```

#### **示例2：部门-员工关系（级联删除）**

```sql
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(50)
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    dept_id INT,
    emp_name VARCHAR(50),
    CONSTRAINT fk_emp_dept 
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
    ON DELETE CASCADE   -- 删除部门时，员工自动删除
    ON DELETE SET NULL  -- 或者：删除部门时，员工部门设为NULL
);
```

### 5. **外键约束的要求和限制**

#### **🔑 字段要求**

```sql
-- 父表字段必须是主键或唯一键
CREATE TABLE parent (
    id INT PRIMARY KEY,           -- ✅ 可以
    code VARCHAR(20) UNIQUE       -- ✅ 也可以
);

-- 子表外键字段类型必须与父表完全一致
CREATE TABLE child (
    parent_id INT,                -- 必须与父表id类型一致
    FOREIGN KEY (parent_id) REFERENCES parent(id)
);
```

#### **🚫 存储引擎限制**

```sql
-- 只有InnoDB支持外键
SHOW ENGINES;  -- 确认使用InnoDB

-- 创建表时指定引擎
CREATE TABLE example (
    id INT PRIMARY KEY
) ENGINE=InnoDB;
```

### 6. **外键管理操作**

#### **查看外键**

```sql
-- 查看表的外键约束（就是查看建表语句）
SHOW CREATE TABLE orders;

-- 从信息模式查看
SELECT 
    CONSTRAINT_NAME,
    TABLE_NAME,
    COLUMN_NAME,
    REFERENCED_TABLE_NAME,
    REFERENCED_COLUMN_NAME
FROM information_schema.KEY_COLUMN_USAGE 
WHERE REFERENCED_TABLE_NAME IS NOT NULL;
```

#### **删除外键**

```sql
ALTER TABLE orders DROP FOREIGN KEY fk_orders_user;
```

#### **添加外键**

```sql
ALTER TABLE orders 
ADD CONSTRAINT fk_orders_user 
FOREIGN KEY (user_id) REFERENCES users(id)
ON DELETE CASCADE;
```

### 7. **外键使用的最佳实践**

1. **命名规范**：`fk_子表名_字段名`
2. **谨慎使用CASCADE**：避免误删数据
3. **索引优化**：外键字段自动创建索引
4. **数据一致性**：确保引用完整性

**外键的核心作用：维护数据库的引用完整性，防止"孤儿记录"的产生。**

---

## 11. 多表查询详细解释

### 1. **多表查询的基本概念**

#### 什么是多表查询？

从多个表中检索相关数据，通过表之间的关联关系组合结果。

#### 为什么需要多表查询？

- 数据规范化：避免数据冗余
- 关系建模：实体之间的关联
- 信息整合：从多个来源获取完整信息

### 2. **多表查询的语法基础**

#### 基本语法结构

```sql
SELECT 列名1, 列名2, ...
FROM 表1
[连接类型] JOIN 表2 ON 连接条件
[WHERE 过滤条件]
[ORDER BY 排序字段];
```

### 3. **连接类型详解**

#### 3.1 **内连接 (INNER JOIN)**

**返回两个表中匹配的记录**

```sql
-- 语法
SELECT columns
FROM table1
INNER JOIN table2 ON table1.column = table2.column;

-- 示例：查询员工及其部门信息
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
```

**维恩图表示：**

```
表A ∩ 表B (交集)
```

#### 3.2 **左外连接 (LEFT JOIN)**

**返回左表所有记录 + 右表匹配记录**

```sql
-- 语法
SELECT columns
FROM table1
LEFT JOIN table2 ON table1.column = table2.column;

-- 示例：查询所有员工，包括没有部门的员工
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

**维恩图表示：**

```
表A ∪ (表A ∩ 表B)
```

#### 3.3 **右外连接 (RIGHT JOIN)**

**返回右表所有记录 + 左表匹配记录**

```sql
-- 语法
SELECT columns
FROM table1
RIGHT JOIN table2 ON table1.column = table2.column;

-- 示例：查询所有部门，包括没有员工的部门
SELECT e.name, d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id;
```

**维恩图表示：**

```
表B ∪ (表A ∩ 表B)
```

#### 3.4 **全外连接 (FULL OUTER JOIN)**

**返回两个表的所有记录（MySQL不支持，但可以模拟）**

```sql
-- MySQL模拟全外连接
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
UNION
SELECT e.name, d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id;
```

**维恩图表示：**

```
表A ∪ 表B
```

#### 3.5 **交叉连接 (CROSS JOIN)**

**返回两个表的笛卡尔积**

```sql
-- 语法
SELECT columns
FROM table1
CROSS JOIN table2;

-- 示例：生成所有可能的员工-部门组合
SELECT e.name, d.dept_name
FROM employees e
CROSS JOIN departments d;
```

### 4. **多表查询的高级技巧**

#### 4.1 **自连接 (Self Join)**

```sql
-- 查询员工及其经理的信息（假设manager_id引用emp_id）
SELECT 
    e1.name AS 员工姓名,
    e2.name AS 经理姓名
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.emp_id;
```

#### 4.2 **多对多关系查询**

```sql
-- 学生选课系统（需要中间表）
SELECT 
    s.student_name,
    c.course_name
FROM students s
INNER JOIN student_courses sc ON s.student_id = sc.student_id
INNER JOIN courses c ON sc.course_id = c.course_id;
```

#### 4.3 **使用USING简化连接**

```sql
-- 当连接字段名称相同时可以使用USING
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d USING(dept_id);  -- 代替 ON e.dept_id = d.dept_id
```

### 5. **性能优化建议**

#### 5.1 **索引优化**

```sql
-- 为连接字段创建索引
CREATE INDEX idx_employees_dept ON employees(dept_id);
CREATE INDEX idx_projects_emp ON projects(emp_id);
```

#### 5.2 **查询优化技巧**

```sql
-- 1. 只选择需要的列
SELECT e.name, d.dept_name  -- 而不是 SELECT *

-- 2. 尽早过滤数据
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
WHERE e.salary > 5000  -- 在JOIN前先过滤

-- 3. 使用EXPLAIN分析查询计划
EXPLAIN SELECT e.name, d.dept_name FROM employees e JOIN departments d...
```

### 6. **常见错误和注意事项**

#### 6.1 **笛卡尔积问题**

```sql
-- 错误：忘记连接条件，产生大量无意义数据
SELECT e.name, d.dept_name
FROM employees e, departments d;  -- 错误！

-- 正确：明确连接条件
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
```

#### 6.2 **表别名使用**

```sql
-- 推荐使用表别名提高可读性
SELECT emp.name, dept.dept_name
FROM employees emp
INNER JOIN departments dept ON emp.dept_id = dept.dept_id;
```

#### 6.3 **NULL值处理**

```sql
-- 左连接时注意NULL值
SELECT 
    e.name,
    COALESCE(d.dept_name, '未分配部门') AS 部门名称
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

### 7.  **综合实战练习**

```sql
-- 复杂查询：统计每个部门薪资最高的员工信息
WITH DeptMaxSalary AS (
    SELECT 
        dept_id,
        MAX(salary) as max_salary
    FROM employees
    GROUP BY dept_id
)
SELECT 
    e.name AS 员工姓名,
    e.salary AS 薪资,
    d.dept_name AS 部门名称
FROM employees e
INNER JOIN DeptMaxSalary dms ON e.dept_id = dms.dept_id AND e.salary = dms.max_salary
INNER JOIN departments d ON e.dept_id = d.dept_id
ORDER BY e.salary DESC;
```

**多表查询的核心要点：理解表之间的关系，选择合适的连接类型，优化查询性能。**

---

## 12. 子查询的易错点详解

### 1. **子查询返回多行错误**

#### ❌ 常见错误：使用 = 比较多行子查询

```sql
-- 错误：子查询可能返回多个结果
SELECT name FROM employees 
WHERE salary = (SELECT MAX(salary) FROM employees GROUP BY dept_id);

-- 正确：使用 IN 或 ANY/SOME
SELECT name FROM employees 
WHERE salary IN (SELECT MAX(salary) FROM employees GROUP BY dept_id);

-- 或者使用 ANY
SELECT name FROM employees 
WHERE salary = ANY (SELECT MAX(salary) FROM employees GROUP BY dept_id);
```

### 2. **子查询性能问题**

#### ❌ Nested Loops 导致的性能问题

```sql
-- 低效：对每行员工都执行一次子查询
SELECT e1.name, e1.salary
FROM employees e1
WHERE e1.salary > (SELECT AVG(salary) FROM employees e2 WHERE e2.dept_id = e1.dept_id);

-- 高效：使用窗口函数或JOIN重写
SELECT name, salary
FROM (
    SELECT name, salary, AVG(salary) OVER (PARTITION BY dept_id) as avg_salary
    FROM employees
) t
WHERE salary > avg_salary;
```

### 3. **NULL值处理不当**

#### ❌ NULL值导致意外结果

```sql
-- 如果子查询返回NULL，整个条件可能失效
SELECT name FROM employees 
WHERE dept_id NOT IN (SELECT dept_id FROM departments WHERE status = 'inactive');

-- 问题：如果子查询返回NULL，NOT IN 会返回空结果
-- 解决：确保子查询不返回NULL
SELECT name FROM employees 
WHERE dept_id NOT IN (SELECT dept_id FROM departments 
                      WHERE status = 'inactive' AND dept_id IS NOT NULL);
```

### 4. **关联子查询的循环引用**

#### ❌ 错误的关联条件

```sql
-- 错误：可能导致无限循环或错误结果
SELECT e1.name
FROM employees e1
WHERE EXISTS (
    SELECT 1 FROM employees e2 
    WHERE e2.manager_id = e1.emp_id 
    AND e1.salary > e2.salary  -- 混乱的关联
);

-- 正确：明确父子查询关系
SELECT e1.name
FROM employees e1
WHERE EXISTS (
    SELECT 1 FROM employees e2 
    WHERE e2.manager_id = e1.emp_id 
    AND e2.salary > 10000  -- 只引用父查询的字段s
);
```

### 5. **SELECT列表中的子查询返回多行**

#### ❌ 在SELECT中使用可能返回多行的子查询

```sql
-- 错误：子查询返回了多行
SELECT 
    name,
    (SELECT dept_name FROM departments WHERE dept_id = employees.dept_id) as dept_name
FROM employees;

-- 正确：确保子查询只返回一行
SELECT 
    name,
    (SELECT dept_name FROM departments WHERE dept_id = employees.dept_id LIMIT 1) as dept_name
FROM employees;

-- 更好的做法：使用JOIN
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

### 6. **子查询中的ORDER BY误解**

#### ❌ 错误使用ORDER BY

```sql
-- 错误：子查询中的ORDER BY通常无效（除非有LIMIT）
SELECT name FROM employees 
WHERE salary > (SELECT salary FROM employees ORDER BY salary DESC LIMIT 1);

-- 正确：明确使用LIMIT
SELECT name FROM employees 
WHERE salary > (SELECT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 0);
```

### 7. **数据一致性问题**

#### ❌ 子查询读取到未提交的数据

```sql
-- 在事务中可能读到不一致的数据
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;

-- 子查询可能读到更新前的数据
SELECT balance FROM accounts 
WHERE balance > (SELECT balance FROM accounts WHERE id = 1);

COMMIT;

-- 解决：使用事务隔离级别或避免在事务中混合读写
```

### 8. **EXISTS vs IN 的性能陷阱**

#### ❌ 错误选择EXISTS和IN

```sql
-- 情况1：外表大，子查询结果集小 - 适合IN
SELECT * FROM large_table 
WHERE id IN (SELECT id FROM small_table WHERE condition);

-- 情况2：外表小，子查询需要索引 - 适合EXISTS
SELECT * FROM small_table s
WHERE EXISTS (SELECT 1 FROM large_table l WHERE l.id = s.id AND l.condition);
```

### 9. **子查询中的聚合函数错误**

#### ❌ 错误的聚合使用

```sql
-- 错误：在WHERE中直接使用聚合函数
SELECT name FROM employees 
WHERE salary > AVG(salary);  -- 语法错误！

-- 正确：使用子查询
SELECT name FROM employees 
WHERE salary > (SELECT AVG(salary) FROM employees);

-- 错误：分组后的子查询
SELECT dept_id, name, salary
FROM employees e1
WHERE salary > (SELECT AVG(salary) FROM employees e2 WHERE e2.dept_id = e1.dept_id)
GROUP BY dept_id, name, salary;  -- 可能不是想要的结果
```

### 10. **CTE vs 子查询的选择**

#### ❌ 过度使用嵌套子查询

```sql
-- 难以阅读的嵌套子查询
SELECT * FROM (
    SELECT dept_id, AVG(salary) as avg_sal
    FROM (
        SELECT * FROM employees WHERE status = 'active'
    ) active_emps
    GROUP BY dept_id
) dept_avg WHERE avg_sal > 5000;

-- 使用CTE提高可读性
WITH active_emps AS (
    SELECT * FROM employees WHERE status = 'active'
),
dept_avg AS (
    SELECT dept_id, AVG(salary) as avg_sal
    FROM active_emps
    GROUP BY dept_id
)
SELECT * FROM dept_avg WHERE avg_sal > 5000;
```

### 11. **实际案例：常见的子查询错误**

#### 案例1：找部门最高薪员工

```sql
-- ❌ 错误写法
SELECT name, salary, dept_id
FROM employees
WHERE salary = (SELECT MAX(salary) FROM employees GROUP BY dept_id);

-- ✅ 正确写法
SELECT e1.name, e1.salary, e1.dept_id
FROM employees e1
INNER JOIN (
    SELECT dept_id, MAX(salary) as max_salary
    FROM employees
    GROUP BY dept_id
) e2 ON e1.dept_id = e2.dept_id AND e1.salary = e2.max_salary;
```

#### 案例2：存在性检查

```sql
-- ❌ 低效的NOT EXISTS
SELECT name FROM employees e1
WHERE NOT EXISTS (
    SELECT 1 FROM employees e2 
    WHERE e2.manager_id = e1.emp_id
);

-- ✅ 使用LEFT JOIN + IS NULL（通常更高效）
SELECT e1.name
FROM employees e1
LEFT JOIN employees e2 ON e2.manager_id = e1.emp_id
WHERE e2.emp_id IS NULL;
```

### 12. **调试和优化技巧**

#### 检查子查询执行计划

```sql
-- 使用EXPLAIN分析
EXPLAIN 
SELECT name FROM employees 
WHERE dept_id IN (SELECT dept_id FROM departments WHERE status = 'active');

-- 单独测试子查询
SELECT dept_id FROM departments WHERE status = 'active';
```

#### 性能优化建议

1. **使用JOIN重写关联子查询**
2. **为子查询中的连接字段创建索引**
3. **避免在子查询中使用SELECT ***
4. **考虑使用临时表或CTE替代复杂子查询**

### 💡 **总结：子查询最佳实践**

1. **明确返回行数**：确保子查询返回预期的行数
2. **注意NULL值**：处理子查询可能返回的NULL
3. **优先使用JOIN**：能用JOIN解决的问题不用子查询
4. **测试性能**：使用EXPLAIN分析查询计划
5. **保持简洁**：避免过度嵌套，使用CTE提高可读性

没看懂~~~2025-11-01

---

## 13. 窗口函数详解

* 例子对比

~~~SQL
-- GROUP BY：每个部门一行
SELECT dept_id, AVG(salary)
FROM employees
GROUP BY dept_id;

-- 窗口函数：每个人一行，但带部门平均
SELECT name, dept_id, salary,
       AVG(salary) OVER (PARTITION BY dept_id) AS avg_salary
FROM employees;

~~~

### 1. **什么是窗口函数？**

窗口函数（Window Function）是一种**对一组相关行进行计算**的特殊函数，它不会像普通聚合函数那样将多行合并为一行，而是**为每一行都返回一个计算结果**。

#### 核心特点：

- ✅ **保持原有行数**：不合并行，每行都有计算结果
- ✅ **分组计算**：可以按指定字段分组计算
- ✅ **排序支持**：可以在窗口内排序
- ✅ **灵活窗口**：可以定义计算范围（当前行前后N行）

### 2. **基本语法结构**

```SQL
聚合函数([参数]) OVER (
    [PARTITION BY 分组字段]
    [ORDER BY 排序字段]
    [ROWS/RANGE 窗口范围]
)
```

### 3. **您的查询详细解释**

```SQL
SELECT name, dept_id, salary,
       AVG(salary) OVER (PARTITION BY dept_id) AS avg_salary
FROM employees;
```

#### 执行过程：

1. **按部门分组**：`PARTITION BY dept_id`
2. **在每个部门内计算平均薪资**：`AVG(salary)`
3. **为每个员工显示其部门的平均薪资**

#### 示例结果：

| name | dept_id | salary | avg_salary |
| ---- | ------- | ------ | ---------- |
| 张三 | 1       | 5000   | **5500**   |
| 李四 | 1       | 6000   | **5500**   |
| 王五 | 2       | 7000   | **7500**   |
| 赵六 | 2       | 8000   | **7500**   |

**注意**：同一部门的员工显示相同的`avg_salary`值！

### 4. **窗口函数 vs 聚合函数**

#### 传统聚合函数（行数减少）

```SQL
-- 结果：每个部门一行
SELECT dept_id, AVG(salary) as avg_salary
FROM employees
GROUP BY dept_id;

-- 结果：
-- dept_id | avg_salary
-- 1       | 5500
-- 2       | 7500
```

#### 窗口函数（行数不变）

```SQL
-- 结果：每个员工一行，包含部门平均薪资
SELECT name, dept_id, salary, AVG(salary) OVER (PARTITION BY dept_id) as avg_salary
FROM employees;

-- 结果：
-- name | dept_id | salary | avg_salary
-- 张三 | 1       | 5000   | 5500
-- 李四 | 1       | 6000   | 5500
-- 王五 | 2       | 7000   | 7500
-- 赵六 | 2       | 8000   | 7500
```

### 5. **常用窗口函数分类**

#### 5.1 **聚合窗口函数**

```SQL
-- 基本聚合
AVG(salary) OVER (PARTITION BY dept_id)
SUM(salary) OVER (PARTITION BY dept_id)
COUNT(*) OVER (PARTITION BY dept_id)
MAX(salary) OVER (PARTITION BY dept_id)
MIN(salary) OVER (PARTITION BY dept_id)
```

#### 5.2 **排名窗口函数**

```SQL
-- 部门内薪资排名
SELECT name, salary,
       ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) as rank1, -- 连续排名(1,2,3)
       RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) as rank2,       -- 并列排名(1,1,3)
       DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) as rank3  -- 密集排名(1,1,2)
FROM employees;
```

#### 5.3 **位移窗口函数**

```SQL
-- 查看前后行的数据
SELECT name, salary,
       LAG(salary) OVER (ORDER BY salary) as prev_salary,    -- 上一行
       LEAD(salary) OVER (ORDER BY salary) as next_salary,   -- 下一行
       FIRST_VALUE(salary) OVER (PARTITION BY dept_id) as first_in_dept, -- 部门第一个
       LAST_VALUE(salary) OVER (PARTITION BY dept_id) as last_in_dept    -- 部门最后一个
FROM employees;
```

### 6. **窗口范围控制**

#### 6.1 **ROWS vs RANGE**

```SQL
-- 计算移动平均（最近3行）
SELECT date, sales,
       AVG(sales) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as moving_avg
FROM sales_data;

-- 计算累计汇总
SELECT date, sales,
       SUM(sales) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as cumulative_sum
FROM sales_data;
```

#### 6.2 **常用窗口范围**

```SQL
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW    -- 从开始到当前行
ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING            -- 前后各一行
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING    -- 从当前行到最后
```

### 7. **实际应用场景**

#### 场景1：员工薪资分析

```SQL
-- 查看员工薪资在部门内的位置
SELECT name, dept_id, salary,
       AVG(salary) OVER (PARTITION BY dept_id) as dept_avg,
       salary - AVG(salary) OVER (PARTITION BY dept_id) as diff_from_avg,
       RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) as dept_rank
FROM employees;
```

#### 场景2：销售趋势分析

```SQL
-- 计算月度销售增长
SELECT month, sales,
       LAG(sales) OVER (ORDER BY month) as prev_month_sales,
       sales - LAG(sales) OVER (ORDER BY month) as month_growth,
       AVG(sales) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as moving_avg_3m
FROM monthly_sales;
```

#### 场景3：市场份额计算

```SQL
-- 计算每个产品在品类中的份额
SELECT product_id, category, sales,
       sales / SUM(sales) OVER (PARTITION BY category) as market_share,
       RANK() OVER (PARTITION BY category ORDER BY sales DESC) as category_rank
FROM products;
```

### 8. **高级窗口函数技巧**

#### 8.1 **多个窗口定义**

```SQL
SELECT name, dept_id, salary,
       AVG(salary) OVER (PARTITION BY dept_id) as dept_avg,           -- 部门平均
       AVG(salary) OVER (PARTITION BY dept_id, title) as title_avg,   -- 部门职位平均
       AVG(salary) OVER () as company_avg                             -- 公司平均
FROM employees;
```

#### 8.2 **命名窗口（Window Aliasing）**

```SQL
-- MySQL 8.0+ 支持
SELECT name, salary,
       AVG(salary) OVER w as avg_salary,
       MAX(salary) OVER w as max_salary
FROM employees
WINDOW w AS (PARTITION BY dept_id ORDER BY hire_date);
```

### 9. **性能优化建议**

#### 9.1 **索引策略**

```SQL
-- 为窗口函数的PARTITION BY和ORDER BY字段创建索引
CREATE INDEX idx_employees_dept_hire ON employees(dept_id, hire_date);
CREATE INDEX idx_employees_dept_salary ON employees(dept_id, salary);
```

#### 9.2 **避免全表扫描**

```SQL
-- 不好的写法：窗口函数应用在大量数据上
SELECT *,
       AVG(salary) OVER (PARTITION BY dept_id)
FROM employees
WHERE salary > 100000;  -- 过滤在窗口函数之后

-- 好的写法：先过滤再应用窗口函数
SELECT *,
       AVG(salary) OVER (PARTITION BY dept_id)
FROM (SELECT * FROM employees WHERE salary > 100000) filtered_emps;
```

### 10. **常见错误**

#### 错误1：忘记PARTITION BY

```SQL
-- 错误：计算整个公司的平均，而不是部门平均
SELECT name, dept_id, salary,
       AVG(salary) OVER () as avg_salary  -- 缺少PARTITION BY dept_id
FROM employees;
```

#### 错误2：窗口范围错误

```SQL
-- 可能不是想要的结果
SELECT name, salary,
       SUM(salary) OVER (ORDER BY salary) as running_total  -- 默认窗口范围可能有问题
FROM employees;

-- 明确指定窗口范围
SELECT name, salary,
       SUM(salary) OVER (ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as running_total
FROM employees;
```

## 🔍 窗口函数与GROUP By**核心区别：数据处理阶段不同**

#### **GROUP BY 的数据处理流程**

```sql
SELECT dept_id, AVG(salary)
FROM employees
GROUP BY dept_id;
```

**执行步骤：**

1. **读取所有数据**：从employees表读取所有行
2. **按dept_id分组**：将数据分成多个组（每个部门一个组）
3. **聚合计算**：对每个组计算AVG(salary)
4. **输出结果**：**每个组只输出一行**

```sql
原始数据：
name    | dept_id | salary
张三    | 1       | 5000
李四    | 1       | 6000  
王五    | 2       | 7000
赵六    | 2       | 8000

GROUP BY处理后：
dept_id | AVG(salary)
1       | 5500
2       | 7500
```

#### **窗口函数的数据处理流程**

```sql
SELECT name, dept_id, salary,
       AVG(salary) OVER (PARTITION BY dept_id) AS avg_salary
FROM employees;
```

**执行步骤：**

1. **读取所有数据**：从employees表读取所有行
2. **按dept_id逻辑分组**：只是逻辑上分组，**不合并行**
3. **为每行计算窗口函数**：在每个逻辑组内计算AVG(salary)
4. **输出结果**：**保持原始行数不变**

```sql
原始数据 + 窗口计算结果：
name    | dept_id | salary | avg_salary
张三    | 1       | 5000   | 5500
李四    | 1       | 6000   | 5500
王五    | 2       | 7000   | 7500  
赵六    | 2       | 8000   | 7500
```

## 🎯 **关键理解：SELECT语句的执行顺序**

#### **GROUP BY的执行顺序**

```sql
-- 实际执行顺序：
1. FROM employees
2. WHERE [条件]  -- 如果有WHERE
3. GROUP BY dept_id  -- ⭐ 这里行数减少了！
4. SELECT dept_id, AVG(salary)  -- 只能选择分组字段和聚合结果
5. HAVING [条件]  -- 如果有HAVING
```

**GROUP BY在SELECT之前执行**，所以当执行到SELECT时，数据已经被合并了。

#### **窗口函数的执行顺序**

```sql
-- 实际执行顺序：
1. FROM employees
2. WHERE [条件]  -- 如果有WHERE  
3. SELECT name, dept_id, salary  -- ⭐ 先选择所有字段
4. 窗口函数计算 AVG() OVER()  -- ⭐ 然后为每行添加窗口计算结果
```

**窗口函数在SELECT之后执行**，所以能访问到所有原始行数据。

### 💡 **总结**

**窗口函数的优势：**

- 📊 **保持明细数据**：不丢失原始行信息
- 🔄 **灵活计算**：支持分组、排序、范围控制
- ⚡ **性能优秀**：比自连接或子查询更高效
- 🎯 **功能强大**：排名、位移、累计计算等

**您的查询正是一个经典的窗口函数应用：为每个员工显示其部门的平均薪资，同时保留每个员工的详细信息！**

---

## 14. 对窗口函数的辩证✅️

### 举例说明

~~~sql
--  查询 "研发部" 员工的平均工资
select e.name, avg(e.salary)
from emp e join dept d on e.dept_id = d.id
where d.name = '研发部';
~~~

* 这是错的，因为`avg(e.salary)`是聚合函数，会把所有相同的部门的工资算出平均值然后聚合成一行结果，但e.name是非聚合列，这里表示每个职工的名字，一个部门里有很多职工，所以这里一行对多行产生了`only_full_group_by`的错误
* 也就是说当select句中出现聚合函数，说明多行聚合成一行，所有非聚合列（就是多列）必须出现在GROUP BY子句中--->通过分组成为一行

~~~sql
-- ---> 没有查询 "研发部" 员工的平均工资的作用，但是代码没有bug
select e.name
from emp e join dept d on e.dept_id = d.id
where d.name = '研发部';
~~~

* 和上面的代码比较，去掉`AVG(e.salary)`后代码就能正常跑起来（~没有bug，但是意义不明~），这是因为现在不会出现一对多的情况

~~~sql
-- 如果想知道每个员工的薪资和部门平均薪资的对比
SELECT e.name, e.salary, AVG(e.salary) OVER() as avg_salary
FROM emp e JOIN dept d ON e.dept_id = d.id
WHERE d.name = '研发部';

-- 或者使用窗口函数按部门分区
SELECT e.name, e.salary, 
       AVG(e.salary) OVER(PARTITION BY d.id) as dept_avg_salary
FROM emp e JOIN dept d ON e.dept_id = d.id
WHERE d.name = '研发部';
~~~

* 这两段写法不会出现`only_full_group_by`错误的原因是：**它们使用了窗口函数而不是聚合函数**。
* 具体来说就是，窗口函数是在聚合函数（比如avg）求每个部门平均薪资的情况下，把这个结果分给每一个这个部门的人的那一行
* `AVG(e.salary) OVER()`是窗口函数， 它**不减少行数**，而是在每行后面添加计算结果
*  `AVG(e.salary) OVER()` 和`AVG(e.salary) OVER(PARTITION BY d.id)`的区别
  * 主要区别就是计算范围一个是整个查询结果集，一个是按照部门分局计算；由于where过滤了只有研发部，所以没区别

### 为什么窗口函数不会触发only_full_group_by

1. **不改变结果集行数**：窗口函数只是为每行添加计算列，不进行行聚合
2. **保持列的一致性**：所有原始列都保持原样，没有聚合与非聚合列的冲突
3. **符合SQL标准**：窗口函数是现代SQL的标准特性，设计时就考虑了这种使用场景

### 总结 窗口 vs 聚合 函数

窗口函数`AVG() OVER()`和传统聚合函数`AVG()`的关键区别：

- **传统AVG()**：多行输入 → 单行输出（需要GROUP BY）
- **窗口AVG() OVER()**：多行输入 → 多行输出（每行添加计算结果）

这就是为什么窗口函数写法不会触发`only_full_group_by`错误的原因。

---

## 15. 外连接遇到空值的情况

​	很好的问题！外连接的不同类型会对NULL值处理产生不同的结果。我用具体例子来说明：

* 一句话，**保留主连接的所有情况，对不匹配的情况的另一张表的所有内容显示NULL**

### 数据准备

假设有两个表：

**部门表(dept)**：

| id   | name   |
| ---- | ------ |
| 1    | 研发部 |
| 2    | 销售部 |
| 3    | 人事部 |

**员工表(emp)**：

| id   | name | dept_id |
| ---- | ---- | ------- |
| 101  | 张三 | 1       |
| 102  | 李四 | 1       |
| 103  | 王五 | 2       |
| 104  | 赵六 | NULL    |

### 1. LEFT JOIN（左外连接）

```sql
SELECT d.name as dept_name, e.name as emp_name
FROM dept d LEFT JOIN emp e ON d.id = e.dept_id
ORDER BY d.id, e.id;
```

**结果**：

| dept_name | emp_name |
| --------- | -------- |
| 研发部    | 张三     |
| 研发部    | 李四     |
| 销售部    | 王五     |
| 人事部    | NULL     |
|           |          |

**特点**：保留左表所有记录，右表不匹配的显示NULL

### 2. RIGHT JOIN（右外连接）

```sql
SELECT d.name as dept_name, e.name as emp_name  
FROM dept d RIGHT JOIN emp e ON d.id = e.dept_id
ORDER BY d.id, e.id;
```

**结果**：

| dept_name | emp_name |
| --------- | -------- |
| 研发部    | 张三     |
| 研发部    | 李四     |
| 销售部    | 王五     |
| NULL      | 赵六     |
|           |          |

**特点**：保留右表所有记录，左表不匹配的显示NULL

### 3. FULL OUTER JOIN（全外连接）

```sql
-- MySQL不支持FULL JOIN，用UNION模拟
SELECT d.name as dept_name, e.name as emp_name
FROM dept d LEFT JOIN emp e ON d.id = e.dept_id
UNION
SELECT d.name as dept_name, e.name as emp_name  
FROM dept d RIGHT JOIN emp e ON d.id = e.dept_id;
```

**结果**：

| dept_name | emp_name |
| --------- | -------- |
| 研发部    | 张三     |
| 研发部    | 李四     |
| 销售部    | 王五     |
| 人事部    | NULL     |
| NULL      | 赵六     |

**特点**：保留两个表的所有记录，不匹配的都显示NULL

### 4. INNER JOIN（内连接）

```sql
SELECT d.name as dept_name, e.name as emp_name
FROM dept d INNER JOIN emp e ON d.id = e.dept_id
ORDER BY d.id, e.id;
```

**结果**：

| dept_name | emp_name |
| --------- | -------- |
| 研发部    | 张三     |
| 研发部    | 李四     |
| 销售部    | 王五     |
|           |          |

**特点**：只返回两个表都能匹配的记录，排除所有NULL情况

### 5. 总结对比

| 连接类型       | 左表NULL处理     | 右表NULL处理     | 结果行数示例       |
| -------------- | ---------------- | ---------------- | ------------------ |
| **LEFT JOIN**  | 保留所有记录     | 不匹配的显示NULL | 4行（3部门+1NULL） |
| **RIGHT JOIN** | 不匹配的显示NULL | 保留所有记录     | 4行（3员工+1NULL） |
| **FULL JOIN**  | 保留所有记录     | 保留所有记录     | 5行（所有组合）    |
| **INNER JOIN** | 排除不匹配的     | 排除不匹配的     | 3行（仅匹配的）    |

### 实际应用场景

#### 场景1：查看所有部门及员工（包括空部门）

```sql
-- 使用LEFT JOIN
SELECT d.name, COUNT(e.id) as emp_count
FROM dept d LEFT JOIN emp e ON d.id = e.dept_id
GROUP BY d.id, d.name;
```

#### 场景2：查看所有员工及部门（包括未分配部门的员工）

```sql
-- 使用RIGHT JOIN  
SELECT e.name, d.name as dept_name
FROM dept d RIGHT JOIN emp e ON d.id = e.dept_id;
```

#### 场景3：完整的人员-部门关系视图

```sql
-- 使用FULL OUTER JOIN（模拟）
SELECT d.name as dept_name, e.name as emp_name
FROM dept d LEFT JOIN emp e ON d.id = e.dept_id
UNION
SELECT d.name as dept_name, e.name as emp_name
FROM dept d RIGHT JOIN emp e ON d.id = e.dept_id;
```

**关键记忆点**：

- **LEFT JOIN**：要左表的全部数据
- **RIGHT JOIN**：要右表的全部数据
- **FULL JOIN**：两个表的全部数据都要
- **INNER JOIN**：只要两个表都有的数据

---

## 16. where和外连接的区别

很好的对比！这两个SQL语句展示了**WHERE连接**和**外连接**的重要区别。让我详细解释：

### 两个SQL语句的对比分析

#### 语句1：WHERE连接（隐式内连接）

```sql
SELECT s.name, s.no, GROUP_CONCAT(c.name SEPARATOR ',') as '课程'
FROM student s, course c, student_course sc
WHERE s.id = sc.student_id AND sc.course_id = c.id
GROUP BY s.name, s.no, '课程';  -- 注意：这里GROUP BY '课程'是错误的
```

#### 语句2：显式左外连接

```sql
SELECT 
    s.name as 学生姓名,
    s.no as 学号,
    GROUP_CONCAT(c.name SEPARATOR ', ') as 所选课程
FROM student s 
LEFT JOIN student_course sc ON s.id = sc.student_id
LEFT JOIN course c ON sc.course_id = c.id
GROUP BY s.id, s.name, s.no
ORDER BY s.no;
```

### 关键区别

#### 1. **连接类型的本质区别**

**WHERE连接（语句1）**：

- 实际上是**隐式的INNER JOIN**
- 只会返回**三个表都能匹配**的记录
- **没有选课的学生不会显示**

**LEFT JOIN（语句2）**：

- 显式的**左外连接**
- 返回**左表（student）的所有记录**
- 即使学生没有选课也会显示

#### 2. **结果差异示例**

假设数据：

- 学生：张三（有选课）、李四（有选课）、王五（没有选课）
- 课程：数学、英语
- 选课关系：张三选了数学和英语，李四选了数学

**语句1（WHERE连接）结果**：

| name | no   | 课程             |
| ---- | ---- | ---------------- |
| 张三 | 001  | 数学,英语        |
| 李四 | 002  | 数学             |
|      |      | ← 王五不会出现！ |

**语句2（LEFT JOIN）结果**：

| 学生姓名 | 学号 | 所选课程  |
| -------- | ---- | --------- |
| 张三     | 001  | 数学,英语 |
| 李四     | 002  | 数学      |
| 王五     | 003  | NULL      |

#### 3. **语句1中的GROUP BY错误**

```sql
GROUP BY s.name, s.no, '课程'  -- 错误！
```

- `'课程'`是别名，不应该出现在GROUP BY中
- 正确的应该是：`GROUP BY s.id, s.name, s.no`

### WHERE在两种连接中的不同作用

#### 在隐式连接中（语句1）

```sql
WHERE s.id = sc.student_id AND sc.course_id = c.id
```

- **WHERE子句同时承担连接条件和过滤条件**
- 这些条件必须同时满足，否则记录被排除

#### 在显式外连接中

```sql
FROM student s 
LEFT JOIN student_course sc ON s.id = sc.student_id  -- 连接条件在ON中
LEFT JOIN course c ON sc.course_id = c.id
WHERE ...  -- 额外的过滤条件可以放在这里
```

### 实际应用场景

#### 场景1：只想看有选课的学生（使用WHERE连接）

```sql
-- 只关心有选课记录的学生
SELECT s.name, s.no, GROUP_CONCAT(c.name) as 课程
FROM student s, course c, student_course sc
WHERE s.id = sc.student_id AND sc.course_id = c.id
GROUP BY s.id, s.name, s.no;
```

#### 场景2：查看所有学生的选课情况（使用LEFT JOIN）

```sql
-- 需要看到所有学生，包括没选课的
SELECT s.name, s.no, 
       COALESCE(GROUP_CONCAT(c.name), '未选课') as 选课情况
FROM student s 
LEFT JOIN student_course sc ON s.id = sc.student_id
LEFT JOIN course c ON sc.course_id = c.id
GROUP BY s.id, s.name, s.no;
```

#### 场景3：结合WHERE过滤（外连接+过滤）

```sql
-- 查看所有学生，但只显示选了"数学"课程的情况
SELECT s.name, s.no,
       CASE WHEN c.name = '数学' THEN '已选数学' ELSE '未选数学' END as 数学选课
FROM student s 
LEFT JOIN student_course sc ON s.id = sc.student_id AND sc.course_id = (
    SELECT id FROM course WHERE name = '数学'
)
LEFT JOIN course c ON sc.course_id = c.id
GROUP BY s.id, s.name, s.no;
```

### 总结 where语句 vs 外连接

| 特性             | WHERE连接（隐式内连接） | 左外连接                     |
| ---------------- | ----------------------- | ---------------------------- |
| **连接条件位置** | WHERE子句中             | ON子句中                     |
| **返回结果**     | 只返回匹配的记录        | 返回左表所有记录             |
| **NULL处理**     | 排除不匹配的记录        | 保留不匹配的记录（显示NULL） |
| **适用场景**     | 只需要关联数据时        | 需要完整数据视图时           |

**选择建议**：

- 如果确定所有学生都有选课 → 使用WHERE连接
- 如果需要看到所有学生（包括没选课的）→ 使用LEFT JOIN
- 现代SQL推荐使用显式的JOIN语法，更清晰易懂

---

## 17. 事务

好的，我们来详细介绍一下 SQL 语言中事务的四大特性，通常简称为 **ACID** 特性。

### 什么是事务？

​	在介绍特性之前，首先要明白什么是事务。事务是数据库操作的一个**逻辑工作单元**，它由一个或多个 SQL 语句组成。这些语句要么**全部成功**，要么**全部失败**，不允许有中间状态。

​	一个经典的例子就是银行转账：从A账户扣款和向B账户加款这两个操作，必须作为一个整体来执行。如果只扣了A的钱，而B没收到，那就会造成严重问题。事务就是为了解决这类问题而生的。

------

### 事务的四大特性 (ACID)

ACID 是四个英文单词的首字母缩写，它们共同保证了数据库事务的可靠性和数据一致性。

#### 1. 原子性 (Atomicity)

- **核心思想：** “All or Nothing”（要么全部完成，要么全部不做）
- **解释：** 事务被视为一个不可分割的原子单元。事务中的所有操作要么全部成功提交，如果其中有任何一个操作失败，那么整个事务中的所有操作都要被回滚到事务开始前的状态，就像这个事务从来没执行过一样。
- **实现机制：** 数据库通常通过**日志**来实现原子性。在事务执行过程中，所有修改操作都会先记录到日志（如 Undo Log）中。如果事务失败或需要回滚，数据库系统就可以根据这些日志，将数据恢复到事务开始前的状态。

#### 2. 一致性 (Consistency)

- **核心思想：** 事务必须使数据库从一个一致性状态变换到另一个一致性状态。
- **解释：** 一致性确保了数据库的完整性约束不会被破坏。这包括预定义的规则，如主键唯一、外键约束、数据类型、用户自定义的业务规则等。事务执行前后，数据库都必须处于一致的状态。
- **举例：** 在银行转账事务中，一致性要求是“转账前后，A和B账户的总金额必须保持不变”。如果A有1000元，B有1000元，无论转账成功与否，最终他们的总金额都应该是2000元。原子性保证了“要么转成功，要么转失败”，而一致性则保证了“成功或失败后，总金额这个业务规则是正确的”。
- **实现机制：** 一致性是由应用程序和数据库系统共同维护的。应用程序需要确保它发起的事务在业务逻辑上是正确的，而数据库系统则通过原子性、隔离性和持久性来共同保证一致性的实现。**可以说，一致性是原子性、隔离性、持久性的最终目的。**

#### 3. 隔离性 (Isolation)

- **核心思想：** 并发执行的事务之间互不干扰。
- **解释：** 当多个事务并发执行时，隔离性确保每个事务都感觉不到有其他事务在同时执行。一个事务的中间状态（未提交的数据）不应该被其他事务看到，以防止出现“脏读”、“不可重复读”、“幻读”等问题。
- **实现机制：** 数据库通过**锁机制**或**多版本并发控制（MVCC）** 来实现隔离性。数据库提供了不同的事务隔离级别（如读未提交、读已提交、可重复读、序列化），允许开发人员在性能和数据一致性之间进行权衡。隔离级别越高，一致性越强，但并发性能越低。

#### 4. 持久性 (Durability)

- **核心思想：** 一旦事务提交，其对数据的修改就是永久性的。
- **解释：** 即使系统发生崩溃（如断电、宕机），只要事务成功提交，修改后的数据也必须持久化地保存在数据库中，不会丢失。
- **实现机制：** 数据库通过**重做日志（Redo Log）** 来实现持久性。事务提交时，首先会将所有修改信息写入重做日志文件（这个过程通常很快，因为日志是顺序写入的）。之后，数据库会在一个合适的时机将数据从内存刷新到磁盘的数据文件中。即使系统在数据刷盘前崩溃，重启后数据库也能通过重做日志来重新执行（Redo）已提交的事务，从而保证数据的持久性。

------

### 总结与比喻

为了更好地理解，我们可以用一个简单的比喻：

想象你要完成一个任务：**从抽屉A（A账户）拿100元放到抽屉B（B账户）里。**

- **原子性 (Atomicity)：** 这个任务包含两个动作：`从A取钱`和 `向B放钱`。这两个动作必须作为一个整体。你不能只完成了“从A取钱”，而“向B放钱”没做成。如果放钱失败，你必须把取出的钱放回A抽屉。
- **一致性 (Consistency)：** 任务执行前后，两个抽屉里的总钱数必须保持不变。这是你要遵守的规则。
- **隔离性 (Isolation)：** 在你执行这个任务的过程中（钱正拿在手上，还没放进B抽屉），你的家人不能来清点A和B抽屉的总金额，否则他会看到一个“总金额少了100元”的中间不一致状态。隔离性就是为了避免这种干扰。
- **持久性 (Durability)：** 一旦你完成了整个放钱动作（事务提交），即使家里突然停电（系统崩溃），来电后，这100元也必须在B抽屉里，结果不会丢失。

### SQL 中的事务控制语句

在 SQL 中，你可以使用以下语句来控制事务：

- `BEGIN`或 `START TRANSACTION`： 显式地开始一个事务。
- `COMMIT`： 提交事务，使所有修改永久生效。
- `ROLLBACK`： 回滚事务，撤销所有未提交的修改。

很多数据库管理系统（DBMS）默认是自动提交模式（Auto-Commit），即每条单独的 SQL 语句都被视为一个事务并自动提交。要使用多语句事务，你需要先关闭自动提交，或显式地使用 `BEGIN TRANSACTION`。

### 常见的隔离级别

- `READ UNCOMMITTED`
- `READ COMMITTED`（最常用）
- `REPEATABLE READ`
- `SERIALIZABLE`







