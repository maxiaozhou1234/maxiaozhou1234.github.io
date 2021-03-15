---
title: 【sqlite】VIEW 视图
date: 2020-2-25 18:08:50
categories: sqlite
tags: [sqlite]
---

sqlite view 视图

### View 视图

视图（View）可以包含一个表的所有行或从一个或多个表选定行。视图（View）可以从一个或多个表创建，这取决于要创建视图的 SQLite 查询。

#### 1.创建视图语法
> CREATE [TEMP | TEMPORARY] VIEW view_name AS
SELECT column1, column2.....
FROM table_name
WHERE [condition];

#### 2.创建视图实例
你可以只从一个表中取其中几列做为新视图，这里用两个表合并为一个视图。

> mysql> create view company_view as select e.id,e.name,e.age,d.dept from company e inner join department d where e.id=d.e
mp_id;
Query OK, 0 rows affected (0.03 sec)   

 mysql> select * from company_view;
<<<<<<< HEAD

| id | name  | age  | dept        |
|:-:|:-:|-:|:-:
=======
```
+----+-------+------+-------------+
| id | name  | age  | dept        |
+----+-------+------+-------------+
>>>>>>> 44295f6b3470b9af3de24b5f8792d7898b2cbfa2
|  1 | Paul  |   32 | IT Billing  |
|  2 | Allen |   25 | Engineering |
|  7 | James |   24 | Finance     |
|  3 | Teddy |   23 | Engineering |
|  4 | Mark  |   25 | Finance     |
|  5 | David |   27 | Engineering |
|  6 | Kim   |   22 | Finance     |
<<<<<<< HEAD

7 rows in set (0.00 sec)
=======
+----+-------+------+-------------+
7 rows in set (0.00 sec)
```
>>>>>>> 44295f6b3470b9af3de24b5f8792d7898b2cbfa2

#### 3.删除视图
> drop view view_name

mysql> show tables;
<<<<<<< HEAD

| Tables_in_learn_db |
|:-|
=======
```
+--------------------+
| Tables_in_learn_db |
+--------------------+
>>>>>>> 44295f6b3470b9af3de24b5f8792d7898b2cbfa2
| company            |
| company_view       |
| department         |
| employ             |
| log                |
<<<<<<< HEAD

5 rows in set (0.01 sec)
=======
+--------------------+
5 rows in set (0.01 sec)
```
>>>>>>> 44295f6b3470b9af3de24b5f8792d7898b2cbfa2

> mysql> drop view company_view;
Query OK, 0 rows affected (0.00 sec)

mysql> show tables;
<<<<<<< HEAD

| Tables_in_learn_db |
|:-|
=======
```
+--------------------+
| Tables_in_learn_db |
+--------------------+
>>>>>>> 44295f6b3470b9af3de24b5f8792d7898b2cbfa2
| company            |
| department         |
| employ             |
| log                |
+--------------------+
<<<<<<< HEAD
4 rows in set (0.00 sec)
=======
4 rows in set (0.00 sec)
```
>>>>>>> 44295f6b3470b9af3de24b5f8792d7898b2cbfa2
