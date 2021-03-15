---
title: 【sqlite】Trigger 触发器
date: 2020-2-25 13:34:25
categories: sqlite
tags: [sqlite]
---

sqlite Trigger 触发器

### Trigger 触发器

SQLite 触发器（Trigger）是数据库的回调函数，它会在指定的数据库事件发生时自动执行/调用。
指定在特定的数据库表发生 DELETE、INSERT 或 UPDATE 时触发，或在一个或多个指定表的列发生更新时触发。

[Trigger 详细资料链接](https://www.runoob.com/sqlite/sqlite-trigger.html)

#### 1.语法
> CREATE  TRIGGER trigger_name [BEFORE|AFTER] event_name[INSERT|UPDATE|DELETE] ON table_name
FOR EACH ROW（对 SQLite 可加可不加）
BEGIN
 -- 触发器逻辑....
END;

#### 2.SQLite 使用触发器
##### 1）创建记录表，用于存储触发后数据
create table log(uid int,entry_date text);

##### 2)创建触发器
create TRIGGER emp_log AFTER INSERT ON employ
BEGIN
INSERT INTO log(uid,rtime) values (new.id,datetime('now','localtime'));
END;

##### 3)向 employ 插入数据，查看log中记录
> insert into employ values(5,'vv',3000,100);

因为没有创建新表，所以存在之前的数据，旧数据再触发器之前存在是不会新增记录在 log 表中

| id   | name | salary | manager_id |
|-:|:-:|:-|
|    1 | aa   |   2000 |          3 |
|    2 | bb   |   1000 |          3 |
|    3 | cc   |   2500 |          4 |
|    4 | dd   |   6000 |        100 |
|  100 | ee   |   6000 |          0 |
|    5 | vv   |   3000 |        100 |

6 rows in set (0.00 sec)

log 中记录
> select * from log;

| uid  | entry_date|
|-:|:-:|
|5 | 2020-02-25 13: 07:02 |

1 row in set (0.00 sec)

#### 2.MySQL 使用触发器
使用 MySQL 添加触发器语法和上面一致
**注意:**
* 如果是在命令行中执行MySQL，需要将delimiter ; 更改为delimiter //，MySQL 默认使用分号 **";"** 结束判断，不更改在遇到分号执行语句会导致提示语法错误导致创建触发器失败
* 需要增加 **FOR EACH ROW**， MySQL 不支持语句触发器,仅仅支持行级触发器，必须增加这句

示例如下：
> mysql> delimiter //
mysql> CREATE TRIGGER emp_log AFTER INSERT ON employ
    -> FOR EACH ROW
    -> BEGIN
    -> INSERT INTO log(uid) values (new.id);
    -> END;
    -> //
Query OK, 0 rows affected (0.08 sec)

最后可以重新把分号执行修改回来，在命令行输入
> mysql> delimiter ;

#### 3.MySQL 触发器查询，删除
#####１)查询所有触发器
> show triggers;

##### 2)删除触发器
> drop trigger xxx; (xxx 为触发器名称）