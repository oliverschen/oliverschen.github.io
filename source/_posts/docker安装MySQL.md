---
title: Docker安装MySQL
date: 2019-04-09 22:46:42
tags: [Docker,Mysql]
category: [Docker,Mysql]
comments: true
---


![](docker安装MySQL/mysql.png)


自从接触到 docker 之后，就想把所有组件都安装在上面，方便后面使用。今天开始折腾安装 MySQL，虽然目前公司项目使用 MySQL 非常少，但是我很喜欢 MySQL，哈哈。

<!-- more -->
### 单机部署

##### 拉取镜像

``` bash
# 拉去最新版的 mysql
sudo docker pull mysql

```

使用以上命令拉取最新版 MySQL

##### 启动 MySQL

``` bash
docker run -p 3306:3306 --name mysql-ck -v /usr/local/mysql/conf:/etc/mysql/conf.d -v /usr/local/mysql/logs:/logs -v /usr/local/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7

# -p 将 3306 端口映射到本机 3306 端口
# --name 后面跟 MySQL 别名
# -v MySQL 数据保存路径：/usr/local/mysql/conf:/etc/mysql/conf.d   将本机 /usr/local/mysql/conf 映射到 docker 的 /etc/mysql/conf.d 目录
# -e 设置初始密码
# -d 后台运行
```
启动 MySQL 成功之后，容器会返回一个 id 类似 `6d2b3575b78f67481300d01cd65ca47a2a1868f448f2a53bb68d7b056f7ef23d`


##### 进入 MySQL

``` bash
# 进入
sudo docker exec -it mysql bash
# 登录
root@6d2b3575b78f:/# mysql -uroot -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.15 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>

```

##### 客户端安装

建议到[官网](https://www.navicat.com.cn/products/)下载 navicat 客户端，当然条件允许的建议购买正版产品，实在不行就先下载下来，有半个月的试用期，体验一般，个人觉得他们家的产品用起来很不错。下载后解压，直接启动即可使用。

### 主从复制

#### 版本

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

#### 

今天没有加班，下班早早就溜了，最近加班比较多，希望程序狗们都能天天没 bug ,天天早下班。

***

<center>道理都懂，但做不到</center>


