---
title: 【sqlite】unions 语法笔记
date: 2020-2-25 11:30:03
categories: sqlite
tags: [sqlite]
---

SQLite unions 合并查询

> SQLite的 UNION 子句/运算符用于合并两个或多个 SELECT 语句的结果，不返回任何重复的行。
为了使用 UNION，每个 SELECT 被选择的列数必须是相同的，相同数目的列表达式，相同的数据类型，并确保它们有相同的顺序，但它们不必具有相同的长度。

### SQLite unions 合并查询

将表1和表2的内容合并输出，**前提是两个表中的列必须一致(列名、顺序)**

#### 1.内联查询
> mysql> select emp_id,name,dept from company inner join department on company.id=department.emp_id;

```
+--------+-------+-------------+
| emp_id | name  | dept        |
+--------+-------+-------------+
|      1 | Paul  | IT Billing  |
|      2 | Allen | Engineering |
|      7 | James | Finance     |
|      3 | Teddy | Engineering |
|      4 | Mark  | Finance     |
|      5 | David | Engineering |
|      6 | Kim   | Finance     |
+--------+-------+-------------+

7 rows in set (0.00 sec)
```


#### 2.左外连接查询
> mysql> select emp_id,name,dept from company left outer join department on company.id=department.emp_id;

```
+--------+-------+-------------+
| emp_id | name  | dept        |
+--------+-------+-------------+
|      1 | Paul  | IT Billing  |
|      2 | Allen | Engineering |
|      3 | Teddy | Engineering |
|      4 | Mark  | Finance     |
|      5 | David | Engineering |
|      6 | Kim   | Finance     |
|      7 | James | Finance     |
+--------+-------+-------------+

7 rows in set (0.00 sec)
```

#### 3.联合输出
> mysql> select emp_id,name,dept from company inner join department on company.id=department.emp_id union select emp_id,na
me,dept from company left outer join department on company.id=department.emp_id;

```
+--------+-------+-------------+
| emp_id | name  | dept        |
+--------+-------+-------------+
|      1 | Paul  | IT Billing  |
|      2 | Allen | Engineering |
|      7 | James | Finance     |
|      3 | Teddy | Engineering |
|      4 | Mark  | Finance     |
|      5 | David | Engineering |
|      6 | Kim   | Finance     |
+--------+-------+-------------+

7 rows in set (0.00 sec)
```

上面这个例子结果不能很好展现合并的效果，我们更改一下第二条查询的条件，增加员工id<5，如下
> mysql>select emp_id,name,dept from company left outer join department on company.id=department.emp_id and company.id<5;

```
+--------+-------+-------------+
| emp_id | name  | dept        |
+--------+-------+-------------+
|      1 | Paul  | IT Billing  |
|      2 | Allen | Engineering |
|      3 | Teddy | Engineering |
|      4 | Mark  | Finance     |
|   NULL | David | NULL        |
|   NULL | Kim   | NULL        |
|   NULL | James | NULL        |
+--------+-------+-------------+

7 rows in set (0.00 sec)
```

这里我们看到左外连接后，id>=5的员工数据被置为null，此时再联合输出：

> mysql> select emp_id id,name,dept from company inner join department on company.id=department.emp_id union select emp_id
,name,dept from company left outer join department on company.id=department.emp_id and company.id<5;

```
+------+-------+-------------+
| id   | name  | dept        |
+------+-------+-------------+
|    1 | Paul  | IT Billing  |
|    2 | Allen | Engineering |
|    7 | James | Finance     |
|    3 | Teddy | Engineering |
|    4 | Mark  | Finance     |
|    5 | David | Engineering |
|    6 | Kim   | Finance     |
| NULL | David | NULL        |
| NULL | Kim   | NULL        |
| NULL | James | NULL        |
+------+-------+-------------+

10 rows in set (0.00 sec)
```

查询后可以看到，表1的查询结果都在，表2中1-4数据和表1相同合并为一，剩下三条再并表输出，总共数据为7+3 = 10条。

#### 4.union all 合并所有

仅使用 union 关键字，重复的项会被过滤，如果要保留两个表中的所有内容，可以使用**union all**合并所有数据。
将上面的例子修改一下：

> mysql> select emp_id id,name,dept from company inner join department on company.id=department.emp_id union all select em
p_id,name,dept from company left outer join department on company.id=department.emp_id and company.id<5;

```
+------+-------+-------------+
| id   | name  | dept        |
+------+-------+-------------+
|    1 | Paul  | IT Billing  |
|    2 | Allen | Engineering |
|    7 | James | Finance     |
|    3 | Teddy | Engineering |
|    4 | Mark  | Finance     |
|    5 | David | Engineering |
|    6 | Kim   | Finance     |
|    1 | Paul  | IT Billing  |
|    2 | Allen | Engineering |
|    3 | Teddy | Engineering |
|    4 | Mark  | Finance     |
| NULL | David | NULL        |
| NULL | Kim   | NULL        |
| NULL | James | NULL        |
+------+-------+-------------+

14 rows in set (0.00 sec)
```

结果如上。
