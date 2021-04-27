---
title: MySQL
date: 2020-01-18 22:16:51
tags: Mysql
category: Mysql
comments: true
---

![Photo by WoshWosh on wallhaven.cc](/mysql.png)


趁着放假，准备系统的在理一理 MySQL 的知识，通过看网上大神的一些学习记录和自己工作中的一些经验总结。把 MySQL 数据库重新学习一边，加深对 MySQL 的理解，也方便以后复习时查看。

<!--more-->

## 结构

MySQL 是由两大部分 server 部分和存储引擎构成的。而 server 部分主要由 连接器，分析器，优化器和执行器组成的，server 部分主要负责客户端的连接，SQL 语句的解析，优化以及执行。 存储引擎主要负责数存储，并且提供数据的读写接口。

### MySQL组件

![MySQL组成](/mysql-comp.jpg)
#### server

MySQL 前置操作都在这里执行。

#### 连接器

负责管理客户端连接，权限的验证等操作。当验证通过一个用户名和密码之后，此后的操作都是依赖当前权限，也就是说当一个用户连接成功之后，即使管理员修改密码，但是他还是能完成操作。

#### 分析器

当用户通过连接器，SQL 到达之后，一般会先查询缓存（8.0 之前）。如果缓存中存在的话直接返回。没有的话则对 SQL 语句进行语法分析等动作。对于缓存不建议使用，因为当有表涉及到更新时，所有关于当前表的缓存会失效。除非是数据不经常变更的数据表。

#### 优化器

优化器主要是分析 SQL 语句怎样执行最优，然后生成执行方案。

#### 执行器

执行 SQL 语句，返回结果集。

## 日志系统

### redo log

在 MySQL 语句更新过程中，如果每次都在磁盘找到相应记录，并且修改记录，写入磁盘的话，效率会很低。所以在执行过程中一些会将修改结果先保存在内存中，然后在 redo log 中记录当前的修改，当事务提交之后。服务在空闲的时候，或者日志空间不够时，在刷新 redo log 中 commit 的数据到磁盘中去。WAL （write-ahead logging）技术, 先写日志，在写磁盘。redo log 是存储引擎 InnoDB 独有的日志记录系统，它保证了在 MySQL 服务宕机之后保证数据不丢失。 可以通过设置参数 `innodb_flush_log_at_trx_commit=1` 保证每次 redo lod 都写在磁盘上，保证重启之后数据不丢失。

### binlog

MySQL server 层提供的日志系统，支持所有引擎。bin log 是追加的形式写入的，不像 redo log 有大小限制。binlog有两种记录模式，statement格式的话是记sql语句， row格式会记录行的内容，记两条，更新前和更新后都有。


## 事务

在网上购物从确认商品，下订单，扣减库存，到付款成功。这是一个整体的操作。对于这个操作来说，如果有其中一步操作失败，那意味着在它之前的操作都要回滚。这就是一个事务。在这一系列操作中，要么都成功。要么都失败。

事务特性：ACID，原子性（atomictity），一致性（consistency），隔离性（isolation），持久性（durablity）


### 隔离级别
隔离级别越高，效率越低。

#### 读未提交（read uncommitted）

> 一个事务还没有提交，它的变更能别其他事务看到




#### 读提交（read commit）

> 事务提交之后，变更才能被其他事务看到


#### 可重复读（repeatable read）

> 事务启动之后，在事务提交之前，看到的数据和启动之前一致。没有提交的事务对其他事务不可见。


#### 串行化（serialiable）

> 对于同一行记录，写会加写锁，读会加读锁，当出现读写锁冲突时，必须等上个事务完成之后才能执行下一个事务。

#### 开启
```sql
-- 开启
begin (start) transaction;
-- 提交
commit;
-- 回滚
rollback;
```
或者设置参数
```sql 
-- 0 关闭自动提交，查询也会自动开启，并且不会自动提交。一直持续到commit/rollback，或者断开连接。
set autocommit=0
```

由于 MySQL MVCC 多版本并发控制，每个操作都会有相应的回滚操作保存下来，每个事务都会记录。当设置为自动提交时，连接成功就会执行 set autocommit=0 的操作，每个操作都会在事务中，如果时长连接就会导致意外长事务。**所以一般 set autocommit=1,显示开启事务**

## 主从复制

### 部署

Docker 部署 Mysql5.7 

#### 配置

```
[mysqld]
## 设置server_id，一般设置为IP，注意要唯一
server_id=6
## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql
## 开启二进制日志功能，可以随便取，最好有含义（关键就是这里了）
log-bin=replicas-mysql-bin
## 为每个session分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=1M
## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed
## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
```

#### 启动

指定不同的配置和端口启动两个容器

> docker run --name mysql-6 -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -v ~/develop/mysql/mysql-6/data:/var/lib/mysql -v ~/develop/mysql/mysql-6/conf/my.cnf:/etc/mysql/my.cnf mysql:5.7



> docker run --name mysql-7 -d -p 3307:3306 -e MYSQL_ROOT_PASSWORD=root -v ~/develop/mysql/mysql-7/data:/var/lib/mysql -v ~/develop/mysql/mysql-7/conf/my.cnf:/etc/mysql/my.cnf mysql:5.7

#### 主节点

连接主节点授权

```
GRANT REPLICATION SLAVE ON *.* to 'sync'@'%' identified by 'root';
# 查看主节点状态
show master status;
```

#### IP

```
docker inspect --format '{{ .NetworkSettings.IPAddress }}' <container-ID> 
```

#### 从节点

连接主节点，指定主节点的地址信息

```
change master to master_host='172.17.0.2',master_user='sync',master_password='root',master_log_file='mysql-bin.000001',master_log_pos=437,master_port=3306;
```

开启从节点

```
reset slave;

start slave;
```



***

<center>我该选哪个？</center>