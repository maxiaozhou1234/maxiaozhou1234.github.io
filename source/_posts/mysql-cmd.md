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

### MySQL 5.5版本
1. 没有 datatime,如需要使用时间戳，用**timestmap**代替
2. 默认使用分号**；**结束判断，如果需要多行输入，将delimiter ; 更改为delimiter //，按**“//”**为结束判断 

### 待办记录

* [ ] SQLite PRAGMA
* [x] SQLite 约束
* [x] SQLite Join
* [x] SQLite Unions 子句
* [x] SQLite NULL 值 **（null 未知）**
	* is not null
	* is null
* [x] SQLite 别名 **as 省略后效果一致**
* [x] SQLite 触发器
* [x] SQLite 索引
* [ ] SQLite Indexed By **不明**
* [x] SQLite Alter 命令
* [x] SQLite Truncate Table
* [x] SQLite 视图
* [ ] SQLite 事务
* [ ] SQLite 子查询
* [ ] SQLite Autoincrement
* [ ] SQLite 注入
* [ ] SQLite Explain
* [ ] SQLite Vacuum
* [x] SQLite 日期 & 时间
* [ ] SQLite 常用函数