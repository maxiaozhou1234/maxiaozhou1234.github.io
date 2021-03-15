---
title: 【sqlite】测试题
date: 2020-02-26 14:54:21
categories: sqlite
tags: [sqlite]
---

数据库测试题

<<<<<<< HEAD

### 数据库测试，写出对应的操作 sql 语句

#### 1.筛选出表A和表B中列【id,name,age】去重后合并输出

#### 2.表1和表2存在id关联关系，将两表列合并输出所有数据（无重复数据）

#### 3.表1和表2彼此独立，请列出两表的所有可能组合

#### 4.表1为员工表，manager_id 为该员工领导id，请通过单表列出员工的[id,name,manager_name]
=======
### 数据库测试，写出对应的操作 sql 语句

#### 1.筛选出表A和表B中列【id,name,age】去重后合并输出
合并选择 union/union all 去重 union 
前提两个表均包含列【id,name,age】

select id,name,age from table_A union select id,name,age from table_B;

#### 2.表1和表2存在id关联关系，将两表列相关联数据全部输出
1. 存在关联，明显是用 join
2. 对于单表都有扩展的列，可以用 outer join，inner join
3. 只要相关联数据，说明是两表都有的数据，使用 inner join。使用 outer join 输出会把缺少的补位 null，属于全量。

```
mysql> select * from company;
+----+-------+------+------------+--------+
| id | name  | age  | address    | salary |
+----+-------+------+------------+--------+
|  1 | Paul  |   32 | California |  20000 |
|  2 | Allen |   25 | Texas      |  15000 |
|  3 | Teddy |   23 | Norway     |  20000 |
|  4 | Mark  |   25 | Rich-Mond  |  65000 |
|  5 | David |   27 | Texas      |  85000 |
|  6 | Kim   |   22 | South-Hall |  45000 |
|  7 | James |   24 | Houston    |  10000 |
|  8 | Kit   |   30 | NY         |  12000 |
| 10 | Mike  |   17 | NY         |   2000 |
+----+-------+------+------------+--------+
9 rows in set (0.00 sec)
```

```
mysql> select * from department;
+----+-------------+--------+
| id | dept        | emp_id |
+----+-------------+--------+
|  1 | IT Billing  |      1 |
|  2 | Engineering |      2 |
|  3 | Finance     |      7 |
|  4 | Engineering |      3 |
|  5 | Finance     |      4 |
|  6 | Engineering |      5 |
|  7 | Finance     |      6 |
+----+-------------+--------+
7 rows in set (0.00 sec)
```

使用 inner join 输出：

> mysql> select c.id,c.name,c.salary,d.dept from company c inner join department d on c.id=d.emp_id;

```
+----+-------+--------+-------------+
| id | name  | salary | dept        |
+----+-------+--------+-------------+
|  1 | Paul  |  20000 | IT Billing  |
|  2 | Allen |  15000 | Engineering |
|  7 | James |  10000 | Finance     |
|  3 | Teddy |  20000 | Engineering |
|  4 | Mark  |  65000 | Finance     |
|  5 | David |  85000 | Engineering |
|  6 | Kim   |  45000 | Finance     |
+----+-------+--------+-------------+
7 rows in set (0.00 sec)
```

---
**对比外连接**
---

1.使用 company 左外连接 department

> mysql> select c.id,c.name,c.salary,d.dept from company c left outer join department d on c.id=d.emp_id;

```
+----+-------+--------+-------------+
| id | name  | salary | dept        |
+----+-------+--------+-------------+
|  1 | Paul  |  20000 | IT Billing  |
|  2 | Allen |  15000 | Engineering |
|  3 | Teddy |  20000 | Engineering |
|  4 | Mark  |  65000 | Finance     |
|  5 | David |  85000 | Engineering |
|  6 | Kim   |  45000 | Finance     |
|  7 | James |  10000 | Finance     |
|  8 | Kit   |  12000 | NULL        |
| 10 | Mike  |   2000 | NULL        |
+----+-------+--------+-------------+
9 rows in set (0.00 sec)
```
2.使用 company 右外连接 department （sqlite不支持右外连接可以使用左外连接，调整两个表位置）

> mysql> select c.id,c.name,c.salary,d.dept from company c right outer join department d on c.id=d.emp_id;

```
+------+-------+--------+-------------+
| id   | name  | salary | dept        |
+------+-------+--------+-------------+
|    1 | Paul  |  20000 | IT Billing  |
|    2 | Allen |  15000 | Engineering |
|    7 | James |  10000 | Finance     |
|    3 | Teddy |  20000 | Engineering |
|    4 | Mark  |  65000 | Finance     |
|    5 | David |  85000 | Engineering |
|    6 | Kim   |  45000 | Finance     |
+------+-------+--------+-------------+
7 rows in set (0.00 sec)

```

上面可以看到，外连接输出与左表、右表相关，题目只要相关联，使用inner join。

#### 3.表1和表2彼此独立，请列出两表的所有可能组合
交叉连接
select * from table_1 cross join table_2;

#### 4.表1为员工表，manager_id 为该员工领导id，请通过单表列出员工的[id,name,manager_name]

mysql> desc employ;

```
+------------+----------+------+-----+---------+-------+
| Field      | Type     | Null | Key | Default | Extra |
+------------+----------+------+-----+---------+-------+
| id         | int(11)  | YES  |     | NULL    |       |
| name       | char(20) | YES  |     | NULL    |       |
| salary     | double   | YES  |     | NULL    |       |
| manager_id | int(11)  | YES  |     | NULL    |       |
+------------+----------+------+-----+---------+-------+
4 rows in set (0.01 sec)
```

mysql> select * from employ;

```
+------+------+--------+------------+
| id   | name | salary | manager_id |
+------+------+--------+------------+
|    1 | aa   |   2000 |          3 |
|    2 | bb   |   1000 |          3 |
|    3 | cc   |   2500 |          4 |
|    4 | dd   |   6000 |        100 |
|  100 | ee   |   6000 |          0 |
|    5 | vv   |   3000 |        100 |
+------+------+--------+------------+
6 rows in set (0.00 sec)
```

mysql> select e.id,e.name,e2.name as manager_name from employ e,employ e2 where e.manager_id=e2.id;

```
+------+------+--------------+
| id   | name | manager_name |
+------+------+--------------+
|    1 | aa   | cc           |
|    2 | bb   | cc           |
|    3 | cc   | dd           |
|    4 | dd   | ee           |
|    5 | vv   | ee           |
+------+------+--------------+
5 rows in set (0.00 sec)
```

#### 5.触发器，表1是用户数据表，uid是操作者id（假设是运维管理），id是新增用户id，需要把表1的增删改操作记录到表log中[uid操作者,action动作,id被操作对象,date时间]
表1：create table user (id int primary key auto_increment,name char(20),age int check(age>0),uid int);

mysql> desc user;

```
+-------+----------+------+-----+---------+----------------+
| Field | Type     | Null | Key | Default | Extra          |
+-------+----------+------+-----+---------+----------------+
| id    | int(11)  | NO   | PRI | NULL    | auto_increment |
| name  | char(20) | YES  |     | NULL    |                |
| age   | int(11)  | YES  |     | NULL    |                |
| uid   | int(11)  | YES  |     | NULL    |                |
+-------+----------+------+-----+---------+----------------+
4 rows in set (0.01 sec)
```

表2：create table user_log (uid int,action char(20),id int,time timestamp default CURRENT_TIMESTAMP);

mysql> desc user_log;

```
+--------+-----------+------+-----+-------------------+-------+
| Field  | Type      | Null | Key | Default           | Extra |
+--------+-----------+------+-----+-------------------+-------+
| uid    | int(11)   | YES  |     | NULL              |       |
| action | char(20)  | YES  |     | NULL              |       |
| id     | int(11)   | YES  |     | NULL              |       |
| time   | timestamp | NO   |     | CURRENT_TIMESTAMP |       |
+--------+-----------+------+-----+-------------------+-------+
4 rows in set (0.01 sec)
```

创建触发器。由于 MySQL 5.5只支持单事件触发，所以需要三个对应触发器
1. 增
> mysql> create trigger user_insert after insert on user
    -> for each row
    -> BEGIN
    -> insert into user_log(uid,action,id) values (new.uid,'create',new.id);
    -> END;
    -> //
Query OK, 0 rows affected (0.06 sec)

2. 改
> mysql> create trigger user_update after update on user
    -> for each row
    -> BEGIN
    -> insert into user_log(uid,action,id) values (old.uid,'update',old.id);
    -> END;
    -> //
Query OK, 0 rows affected (0.05 sec)
3. 删
> mysql> create trigger user_delete after delete on user
    -> for each row
    -> BEGIN
    -> insert into user_log(uid,action,id) values (old.uid,'delete',old.id);
    -> END;
    -> //
Query OK, 0 rows affected (0.08 sec)

测试：

	mysql> select * from user;
	Empty set (0.00 sec)
	
	mysql> select * from user_log;
	Empty set (0.00 sec)

	mysql> insert into user (uid,name,age)values (1000,'aka',10);
	Query OK, 1 row affected (0.03 sec)

	mysql> insert into user (uid,name,age)values (1000,'mille',19);
	Query OK, 1 row affected (0.03 sec)

	mysql> insert into user (uid,name,age)values (1003,'jim',22);
	Query OK, 1 row affected (0.03 sec)

	mysql> update user set name='Sare' where id='1';
	Query OK, 1 row affected (0.02 sec)
	Rows matched: 1  Changed: 1  Warnings: 0

	mysql> delete from user where id=2;
	Query OK, 1 row affected (0.03 sec)

结果显示：
mysql> select * from user;
```
+----+-------+------+------+
| id | name | age  | uid  |
+----+-------+------+------+
|  1 | Sare |   10 | 1000 |
|  3 | jim  |   22 | 1003 |
+----+-------+------+------+
2 rows in set (0.00 sec)
```

mysql> select * from user_log;

```
+------+--------+------+---------------------+
| uid  | action | id   | time                |
+------+--------+------+---------------------+
| 1000 | create |    1 | 2020-02-26 10:59:26 |
| 1000 | create |    2 | 2020-02-26 10:59:43 |
| 1003 | create |    3 | 2020-02-26 11:00:12 |
| 1000 | update |    1 | 2020-02-26 11:01:02 |
| 1000 | delete |    2 | 2020-02-26 11:01:39 |
+------+--------+------+---------------------+
5 rows in set (0.00 sec)
```
>>>>>>> 44295f6b3470b9af3de24b5f8792d7898b2cbfa2
