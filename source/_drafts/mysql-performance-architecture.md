---
title: MySQL性能管理及架构设计
date: 2018-03-28 10:32:33
tags:
- mysql
category:
- MySQL
---

<!--more -->
事务

事务是数据库系统区别于其他一切文件系统的重要特性之一
事务是一组具有原子性的SQL语句，或是一个独立的工作单元
具备原子性、一致性、隔离性、持久性

原子性：
 一个事务必须被视为一个不可分割的最小工作单元，整个事务中的所有操作要么全部提交成功，要么全部失败，对于一个事务来说，不可能只执行其中的一部分操作。

```mysql
// 初始使用的是随机生成的字符串，登陆后将数据库密码修改为空
mysql> alter usre 'root'@'localhost' identified by '';

// 导入当前目录下的safety.sql到数据库中
mysql 数据库名 -uroot< safety.sql
```
