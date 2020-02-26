---
title: 【sqlite】join 语法笔记
date: 2020-02-25 10:01:00
categories: sqlite
tags: [sqlite]
---

SQLite 数据库 join 语法笔记

### SQLite 关键字 join

#### 1.cross join 交叉连接
把第一个表的每一行和第二个表的每一行进行匹配。如表1有m行，表2有n行，交叉连接之后有 m*n 行。

**select * from company;**

![1.png](https://i.loli.net/2020/02/24/eFaoVbMcN9fzvnU.png)

**select * from department;**

![2.png](https://i.loli.net/2020/02/24/NLHugBocmZ5IdFC.png)

**select emp_id,name,dept from company cross join department;**

![3.png](https://i.loli.net/2020/02/24/YzhkEsmfAxjnSpv.png)

#### 2.inner join 内联
根据连接条件结合两个表的列值创建一个新的结果表。
内连接语法：
1. > SELECT ... FROM table1 [INNER] JOIN table2 ON conditional_expression ...
2. 为了避免冗余，并保持较短的措辞，可以使用 USING 表达式声明内连接（INNER JOIN）条件。这个表达式指定一个或多个列的列表：**注：如果两张表的关联字段名相同，才可以使用USING子句** 
> SELECT ... FROM table1 JOIN table2 USING ( column1 ,... ) ...

3. 自然连接（NATURAL JOIN）类似于 JOIN...USING，只是它会自动测试存在两个表中的每一列的值之间相等值：
> SELECT ... FROM table1 NATURAL JOIN table2...

示例：
**select emp_id,name,dept from company inner join department on company.id=department.emp_id;**

![4.png](https://i.loli.net/2020/02/24/bRU9NXdnQfkBwlr.png)

#### 3.outer join 外连接
外连接（OUTER JOIN）是内连接（INNER JOIN）的扩展。虽然 SQL 标准定义了三种类型的外连接：LEFT、RIGHT、FULL，**但 SQLite 只支持 左外连接（LEFT OUTER JOIN）**。

1. 左连接 left outer join/left join
关联两张或多张表中，根据条件显示匹配的数据。依附于左表进行扩展，扩展后表中内容如果右表没有匹配数据，则用 null 显示。
2. 右连接 right outer join/right join
关联两张或多张表中，根据条件显示匹配的数据。依附于右表进行扩展，扩展后表中内容如果左表没有匹配数据，则用 null 显示。
3. 全连接 full outer join/right join
全外连接就是关联的两张或多张表中，根据关联条件，显示所有匹配和不匹配的记录。
左表中有的记录，但是右表中没有匹配上的，以空(null)显示。右表中有的记录，但是左表中没有匹配上的，也以空(null)显示。
FULL OUTER JOIN也可以简写成FULL JOIN，效果是一样的。

左外连接语法：
1. > SELECT ... FROM table1 LEFT OUTER JOIN table2 ON [conditional_expression 条件] ...

2. 为了避免冗余，可以使用 **using** 声明条件 
> SELECT ... FROM table1 LEFT OUTER JOIN table2 USING ( column1 ,... ) ...

示例：
**select emp_id,name,dept from company left outer join department on company.id=department.emp_id;**

![5.png](https://i.loli.net/2020/02/24/QgiYNUvdqa1JRwm.png)

#### 4.自连接，扩展
只有一张表，通过把表取别名，当作两张表使用，自己和自己关联。

示例：查询经理的名称，通过员工id和经理id相同自连接查询
```
create table employ (id int,name char(20),salary real check(salary >0),manager_id int);

insert into employ values (1,'aa',2000,3);
insert into employ values (2,'bb',1000,3);
insert into employ values (3,'cc',2500,4);
insert into employ values (4,'dd',6000,100);
insert into employ values (100,'ee',6000,0);

默认使用 inner join，同样也可以使用left join
select e.name,e.salary,e2.name manager from employ e ，employ e2 on e.manager_id = e2.id;

使用条件 where
select e.name,e.salary,e2.name manager from employ e , employ e2 where e.manager_id = e2.id;

```
![6.png](https://i.loli.net/2020/02/24/7ut5ZO6Vsm8aNiX.png)

---
图片存储记录：
[1.png](https://i.loli.net/2020/02/24/eFaoVbMcN9fzvnU.png)
[2.png](https://i.loli.net/2020/02/24/NLHugBocmZ5IdFC.png)
[6.png](https://i.loli.net/2020/02/24/7ut5ZO6Vsm8aNiX.png)
[4.png](https://i.loli.net/2020/02/24/bRU9NXdnQfkBwlr.png)
[5.png](https://i.loli.net/2020/02/24/QgiYNUvdqa1JRwm.png)
[3.png](https://i.loli.net/2020/02/24/YzhkEsmfAxjnSpv.png)