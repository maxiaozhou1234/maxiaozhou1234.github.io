---
title: 【MySQL】操作命令
date: 2020-2-25 10:00:00
categories: sqlite
tags: [MySQL]
---

MySQL 基本操作命令

### MySQL 基本操作命令

1. 查看所有数据库
show databases;
2. 使用数据库
use xxx;
3. 创建数据库
create database xxx charset=utf8;//创建xx数据库，并指定编码
4. 查看创建数据库时的语法命令
show create database xxx;
5. 查看数据库中所有的表
show tables;
6. 删除数据库
drop database xxx;
7. 查看所有触发器
show triggers;
8. 查看指定表触发器（使用 \G 切换视图为纵向输出）
select * from information_schema.triggers where EVENT_OBJECT_TABLE='table_name';
9. 删除触发器
drop trigger trigger_name;
10. 查看表结构
desc table;
describe table;
11. 创建视图
create view view_name as select column1[...] from table where [condition];
12. 删除视图
drop view name; 

### MySQL 5.5版本注意
1. 没有 datatime,如需要使用时间戳，用**timestmap**代替
2. 默认使用分号**；**结束判断，如果需要多行输入，将delimiter ; 更改为delimiter //，按**“//”**为结束判断 
3. 没有 datetime ，时间戳使用 timestamp

