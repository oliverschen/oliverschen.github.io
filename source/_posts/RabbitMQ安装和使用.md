---
title: RabbitMQ安装和使用
date: 2019-09-05 00:16:09
tags: RabbitMQ
category: MQ
---

![Photo by Vadim Sadovski on Unsplash](RabbitMQ安装和使用/rabbitmq.png)

作为消息中间件，MQ （Message Queue） 在系统中有至关重要的作用，在高并发场景下，MQ 异步处理请求，缓解系统高峰期压力。在系统之间的调用中解耦，降低各个系统之间的依赖，提交系统的可扩展性。在一写复杂业务处理时，进行异步处理，类似乐观锁，及时返回给用户操作状态，至于业务逻辑通知 MQ 之后在进行处理，提高用户体验，也提高了服务器的处理能力。
<!--more-->


#### 安装

|  系统    |   版本   | 
| ---- | ---- | ---- |
| CenterOS |  7.6       |  
|   RabbitMq  | 3.6.10    | 

因为 RabbitMQ 是 erlang 实现的，所以先要安装 erlang 环境。

```bash
# 没有 wget 命令的先安装 wget
yum -y install wget
# 下载
wget http://www.rabbitmq.com/releases/erlang/erlang-19.0.4-1.el7.centos.x86_64.rpm

# 升级包
rpm -ivh erlang-19.0.4-1.el7.centos.x86_64.rpm

# 安装
yum -y install erlang

# 下载 rabbitmq
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.6/rabbitmq-server-3.6.6-1.el6.noarch.rpm

# 安装 mq
yum -y install rabbitmq-server-3.6.6-1.el6.noarch.rpm

```

在安装过程中碰到了缺少一些依赖包的情况，但是 Google 上都有解决方案，下载安装后可以解决。

#### 使用

##### 启动
```bash
cd /usr/sbin
service rabbitmq-server start
# 查看进程
ps -ef | grep rabbitmq 
```

##### 后台运行
```bash
rabbitmq-server -detached
```

##### 开机启动
```bash
chkconfig rabbitmq-server on
```
##### 安装 web 管理插件
```bash
rabbitmq-plugins enable rabbitmq_management

```
访问： http://ip:15672 就可以进入到管理页面了

##### 创建用户

rabbitmq 有个默认的 guest/guest 用户，但是只能在 localhost 下访问，所以要创建一个用户来可以进行远程访问

```bash
# 创建用户
abbitmqctl add_user admin admin
# 授予权限
rabbitmqctl set_user_tags admin administrator
# 分配权限
rabbitmqctl add_vhost admin

rabbitmqctl set_permissions -p admin admin ".*" ".*" ".*"

```


##### 关键字

1. Broker：可以理解成 rabbitmq 服务。
2. vhost:  虚拟主机，一个 broker 可以创建多个 vhost，用作权限分离。
3. Producer：消息生产者，消息的来源。
4. Consumer：消息消费者，负责消费生产者生产的消息。
5. Exchange：交换机，可以看作是一个消息的中转站。
6. Queue：队列，消息的载体，消息会被投递到一个或者多个队列中。
7. Binding：绑定，按照一定规则绑定交换机和队列。
8. Routing Key：路由key，交换机根据这个关键字投递到对应的队列。
9. Channel：通道，在 client 端每个链接，可以建立多个通道。





https://blog.csdn.net/hellozpc/article/details/81436980


https://blog.csdn.net/whoamiyang/article/details/54954780



