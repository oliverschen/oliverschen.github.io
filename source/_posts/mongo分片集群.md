---
title: mongo分片集群
date: 2019-11-12 22:59:17
tags: mongo
category: mongo
---

![Photo by Unsplash](mongo分片集群/replicaSet.png)

mongdb 通过分片机制将数据分布在多台机器上面，实现了横向扩展。也支持了非常大数集和高吞吐量操作。mongodb 可以很好的利用机器的内存资源，内存越大，查询就会越快。mongo 对数据的结构没有其他限制，对开发者很友好，很适合迭代很快，表字段变化很多的场景。

##### 分片

mongo 通过分片机制，将数据拆分后存储在不同机器上面。这样就可以存储更多的数据，并且能以很快的速度读取出来。mongo 原生就支持了分布式特性，在使用方面也是非常方便。下面先[下载](https://www.mongodb.org/dl/linux/x86_64)进行部署。

###### 环境

|    ev  |  version    |   
| ---- | ---- |    
|    os  | CentOS Linux release 7.6.1810 (Core)   |       
|    mongo  |    mongodb-linux-x86_64-v4.0-latest  |   


###### 配置服务搭建

创建 3 台虚拟机，分别搭建 mongo 实例
机器分配
|    os  |  ip    |   
| ---- | ---- |    
|  centos7   | 192.168.31.107    |       
|    centos7  |  192.168.31.107  |  
|centos7|192.168.31.107 | 

1. 下载并安装
```bash
# 创建目录 /usr/local
mkdir mongo
# 解压到 mongo 目录
tar -zxvf mongodb-linux-x86_64-v4.0-latest.gz
```

2. 配置文件
先创建下面对应的文件夹

```properties

port = 27017                                            #端口，默认 27017 
bind_ip = 0.0.0.0                                       #绑定地址，默认127.0.0.1只能通过本地连接，0.0.0.0允许任何机器连接
maxConns=2000                                           #最大连接数
logpath = /usr/local/mongo/config/log/mongo.log         #指定日志文件
logappend = true                                        #写日志的模式：设置为true为追加。默认是覆盖
pidfilepath = /usr/local/mongo/config/log/mongo.pid     #进程ID，没有指定则启动时候就没有PID文件。默认缺省
fork = true                                             #是否后台运行，设置为true 启动 进程在后台运行的守护进程模式。默认false
dbpath = /usr/local/mongo/config/data                   #数据存放目录。默认： /data/db/
replSet=configs                                         #使用此设置来配置复制副本集。指定一个副本集名称作为参数，所有主机都必须有相同的名称作为同一个副本集。
configsvr = true                                        ##设置是否是配置服务，默认端口27019，默认目录/data/configdb

```

3. 启动服务
```bash

/usr/local/mongo/bin/mongod -f /usr/local/mongo/conf/config.conf

#启动成功
about to fork child process, waiting until server is ready for connections.
forked process: 8982
child process started successfully, parent exiting
```

**备注**
其他两台机器也按照以上方式同样配置


4. 初始化副本集

```json
#config变量
config = {
    _id : "configs",
     members : [
         {_id : 0, host : "192.168.31.107:27017" },
         {_id : 1, host : "192.168.31.108:27017" },
         {_id : 2, host : "192.168.31.109:27017" }
    ]
 }

 #初始化配置
rs.initiate(config)
```
configs:配置文件中副本集名称


###### 分片副本集

1. 配置文件

```properties

port = 27001                                            #端口，默认 27017 
bind_ip = 0.0.0.0                                       #绑定地址，默认127.0.0.1只能通过本地连接，0.0.0.0允许任何机器连接
maxConns=2000                                           #最大连接数
logpath = /usr/local/mongo/shard1/log/mongo.log         #指定日志文件
logappend = true                                        #写日志的模式：设置为true为追加。默认是覆盖
pidfilepath = /usr/local/mongo/shard1/log/mongo.pid     #进程ID，没有指定则启动时候就没有PID文件。默认缺省
fork = true                                             #是否后台运行，设置为true 启动 进程在后台运行的守护进程模式。默认false
dbpath = /usr/local/mongo/shard1/data                   #数据存放目录。默认： /data/db/

replSet=shard1                                          #使用此设置来配置复制副本集。指定一个副本集名称作为参数，所有主机都必须有相同的名称作为同一个副本集。
shardsvr = true                                         #设置是否分片，默认端口27018

```

2. 启动 shard1 服务

按照相同的方式配置其他 2 台服务并启动

```bash
bin/mongod -f conf/shard1.conf 
```


3. 初始化副本集
进入 shell 命令行
```bash
mongo --port 27001
```

初始化
```json
#使用admin数据库
use admin
#定义副本集配置，第三个节点的 "arbiterOnly":true 代表其为仲裁节点。
config = {
    _id : "shard1",
     members : [
         {_id : 0, host : "192.168.31.107:27001" },
         {_id : 1, host : "192.168.31.108:27001" },
         {_id : 2, host : "192.168.31.109:27001" }
    ]
 }

#初始化副本集配置
rs.initiate(config);

```







[官方文档](https://docs.mongodb.com/manual/sharding/)