---
title: redis安装和使用
date: 2019-11-20 00:53:22
tags: redis
category: 缓存
---

![Photo by Richs on wallhaven.cc](/redis.png)

在互联网项目中，缓存中间件是一个必不可少的组件。由于磁盘 IO 和 内存 IO 在性能上的差异，通常一些热点数据都会放在缓存中，既提高了用户访问速度，缓存也在很大程度上减轻了数据库的压力，提高了系统整体的吞吐量，redis 是很成熟的一款 NoSql 数据库，是目前使用最多的缓存中间件，当然它的作用不仅仅可以用来做缓存，可以做分布式锁，简单消息队列等。性能稳定且高效，是居家必备用品。┑(￣▽ ￣)┍ 

<!--more-->

##### 安装

1. 解压

目标路径
> /usr/local
```bash
tar -zxvf ./redis-4.0.6.tar.gz 
```

redis 压缩包是一个源码包，需要编译安装，[redis官网](https://redis.io/)有下载地址

```bash
# 安装依赖
yum -y install gcc gcc-c++ kernel-devel
# 编译
make
# 安装到指定目录
make PERFIX=/usr/local/redis  install
```

> 若在编译过程中出现错，可以尝试删除掉解压的 redis 包重新解压在编译一次。

2. 启动

将编译包中的 redis.conf 和 sentinel.conf 文件复制到 redis/conf 目录下，方便配置

```bash
# 指定配置文件启动
redis/bin/redis-server redis/conf/redis.conf
```

启动之后会看到 redis 的logo:

```
               _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 4.0.6 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 24482
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'            
```

使用 redis-cli 连接测试

```bash
# 客户端连接
redis/bin/redis-cli
```
测试
```bash
127.0.0.1:6379> ping
PONG
```
完成以上步骤，redis 就使用默认配置启动成功了。

##### 参数

redis 在配置文件中提供了很多可供修改的参数，在实际使用中需要对这些参数进行配置在使用。

###### bind
绑定地址，0.0.0.0(默认) 允许任何互联网上的机器访问，这种配置方式很不安全，127.0.0.1 只允许本机客户端连接，这种是最安全的。配置多个主机访问为 `bind 127.0.0.1 192.268.31.108`
```bash
bind 127.0.0.1
```

###### protected-mode

保护模式，可选参数 yes/on，yes 为开启状态。在开启状态时，当没有指定 bind 等参数时，默认 `bind 127.0.0.1`，如果设置为 no 时，则默认指定 bind 为 `bind 0.0.0.0`

```
protected-mode
```

###### port

访问端口，默认 6379

```bash
port 6379
```

###### timeout

连接断开时间（秒），当客户端连续空闲指定 timeout 时间后，就断开该连接，为 0 时为禁止该功能。

```bash

timeout 0
```














参考 [testerhome](https://testerhome.com/topics/16402),[redis](http://redis.io)
