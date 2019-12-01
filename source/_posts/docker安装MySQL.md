---
title: docker安装MySQL
date: 2019-04-09 22:46:42
tags: [docker,mysql]
category: [docker,mysql]
---


![](docker安装MySQL/mysql.png)


自从接触到 docker 之后，就想把所有组件都安装在上面，方便后面使用。今天开始折腾安装 MySQL，虽然目前公司项目使用 MySQL 非常少，但是我很喜欢 MySQL，哈哈。

<!-- more -->

##### 拉取镜像

``` bash
sudo docker pull mysql

```

使用以上命令拉取最新版 MySQL

##### 启动 MySQL

``` bash
sudo docker run -p 3306:3306 --name mysql -v  /home/chenkui/database/mysql/data  -e MYSQL_ROOT_PASSWORD=root -d mysql

# -p 将 3306 端口映射到本机 3306 端口
# --name 后面跟 MySQL 别名
# -v MySQL 数据保存路径
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

此时就进入到我们熟悉的客户端了，接下来的操作就和在物理机上的操作是一样的了。虽然公司项目可能接触 MySQL 会很少，但是 MySQL 依然是很多公司数据存储不可替代的部分，所以嘛，好好学习，天天向上。

今天没有加班，下班早早就溜了，最近加班比较多，希望程序狗们都能天天没 bug ,天天早下班。

***

<center>道理都懂，但做不到</center>


