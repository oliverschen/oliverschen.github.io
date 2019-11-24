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

###### daemonize 

守护进程模式，可选参数 yes/on，yes 时 redis 服务以守护进程在后台执行

```bash
daemonize yes
```
###### pidfile

当指定守护进程方式启动时，会在 pidfile 参数指定的目录下生成 PID 文件

```bash
pidfile /var/run/redis_6379.pid
```

###### databases

redis 数据库数量，默认 16

```bash
databases 16
```

##### redis 数据结构

redis 支持很多数据结构，String，List，Hash，Sort，ZSort等，简单理解它每种数据结构，都是它的 value 对应的数据结构。

###### String

1. 命令

存入/获取字符串键值对 `GET`,`SET`
```bash
SET key value
GET key  
```

批量存入/获取字符串键值对 `MSET`,`MGET`
```bash
MSET key value [key value]
MGET key [key]
```
将用户对象使用批量字符串的方式保存在 redis

| userId     |   userName   |  age |  
| ---- | ---- | ---- |  
|1001| zhangsan |20 | 
|1002|lisi|21|

```bash
MSET user:1001:name zhangsan user:1001:age 20
127.0.0.1:6379> MGET user:1001:name user:1001:age
1) "zhangsan"
2) "20"
```

以上命令将用户表记录按照批量字符的方式设置到了redis，如果将对象序列化成 json 格式保存一般字符串格式，这样在用户对象年龄或者其他单个字段有改变时会涉及到 JSON 格式转化设置之后在保存的操作，加重了代码量，代码可读性降低。如果用这种方式保存的话可以直接进行针对字段的修改。
```bash
127.0.0.1:6379> MSET user:1001:name zhangsanMOD
OK
127.0.0.1:6379> MGET user:1001:name
1) "zhangsanMOD"
127.0.0.1:6379>
```

存入不存在的键值对 `SETNX`
当设置的 key 存在时，则设置不成功，当不存在时，设置成功。因为 redis 是单线程模型，此命令可以用来实现**分布式锁功能**
```bash
# 设置送出15001礼物价值1000元
SETNX gift:15001 1000
```

删除一个键值对 `DEL`
```bash
DEL gift:15001
```
计数器 `INCR`
这个命令可以很方便的对 key 的 value 加 1 操作，而且不用考虑并发等问题，在实际场景中**文章阅读量可以使用此功能来实现**
```bash
# 111111 文章对应的阅读量
INCR article:readcount:111111
```

增量添加 `INCRBY`
将 key 中储存的数字加上指定的增量值。
```bash
# 键不存在时初始化为 0 之后在加 50
INCRBY gift:15001:count 50
```

2. 应用

> 分布式锁功能简单实现
> 文章阅读量
> 分布式 session 保存，用于权限认证。

###### Hash

1. 命令

存储/获取键值对。`HSET`,`HGET`

Hash 结构的 key 对应的值类似于 Java 中的 HashMap 结构。
```bash
HSET key field value
HGET key field
```
上面将用户信息用 `MSET` 以批量字符串的形式存储，现在将用户表中用户信息用 Hash 结构存储：

| userId     |   userName   |  age |  
| ---- | ---- | ---- |  
|1001| zhangsan |20 | 
|1002|lisi|21|

```bash
HSET user 1001:name zhangsan
HSET user 1001:age 20
# 获取 key 中的属性对应的值
127.0.0.1:6379> HGET user 1001:name
"zhangsan"
```
存储多个键值对（批量）`HMSET`,`HMGET`
```bash
HMSET key field value [key field valye]
HMGET key field [field]
```
使用批量的方式存储用户 `lisi` 的信息
```bash
HMSET user 1002:name lisi 1002:age 21
HMGET user 1002:name 1002:age
# 批量获取
127.0.0.1:6379> HMGET user 1002:name 1002:age
1) "lisi"
2) "21"
# 获取多个
127.0.0.1:6379> HMGET user 1001:name 1001:age 1002:name 1002:age
1) "zhangsan"
2) "20"
3) "lisi"
4) "21"
```
存储一个不存在的键值对 `HSETNX`
```bash
HSETNX key field value
```
```bash
127.0.0.1:6379> HSET gift 15000:count 1
(integer) 1
#第二次设置不成功
127.0.0.1:6379> HSET gift 15000:count 1
(integer) 0

```

删除键对应的属性 `HDEL`
```bash
HDEL key field [field]
```
```bash
HDEL gift 15000:count
# 删除之后重新设置成功
127.0.0.1:6379> HSET gift 15000:count 1
(integer) 1
```

Hash 表大小 `HLEN`
```bash
HLEN key
```
```bash
HLEN user
# 用户 Hash 表大小 4 
127.0.0.1:6379> HLEN user
(integer) 4
```
Hash 表所有的键值对 `HGETALL`
```bash
HGETALL key
```
```bash
HGETALL user
# 用户表中所有键值对
127.0.0.1:6379> HGETALL user
1) "1001:name"
2) "zhangsan"
3) "1001:age"
4) "20"
5) "1002:name"
6) "lisi"
7) "1002:age"
8) "21"
```
Hash 表 key 中属性的键的值设置增量（increment）`HINCRBY`
```bash
HINCRBY key field increament
```
```bash
# 用户 1001 年龄 +1
127.0.0.1:6379> HINCRBY user 1001:age 1
(integer) 21
```

2. 应用

购物车
```bash
1. 用户 ID 为 key
2. 商品 ID 为 field
3. 商品数量为 value
# 购物车操作
1. 添加：                 HSET cart:1001 2001 1
2. 增加数量：             HINCRBY cart:1001 2001 1
3. 商品总数：             HLEN cart:1001
4. 删除商品：             HDEL cart:1001 2001
5. 获取购物车所有商品：    HGETALL cart:1001
```
> 1. 用户 1001 添加 了 1 个 2001 商品到购物车
> 2. 用户 1001 又新增了 1 个商品 2001
> 3. 用户 1001 购物车商品总数
> 4. 用户 1001 删除购物车商品 2001 
> 5. 用户 1001 获取购物车所有商品 


###### List
List 结构中 key 对应的 value 是一个链表结构，**实现常用数据结构**
> Stack(栈) = LPUSH + LPOP -> FILO（先进后出）
> Queue(队列) = LPUSH + RPOP 
> Blocking MQ(阻塞队列) = LPUSH + BRPOP 在阻塞队列中，LPUSH 一条数据之后，使用 BRPOP 获取数据，区别于 RPOP 的是，当 List 中没有数据时 BRPOP 会一直监听这个 List，有值被 push 进来它会立马获取
```bash
# 结构
           LPUSH                   RPUSH
key  ---> |   a   |   b   |   c   |   d   |
           LPOP                    RPOP
```
1. 命令

插入/取出一个或者多个值插入列表头部（最左边） `LPUSH`,`LPOP`
```bash
LPUSH key value [value]
LPOP key
```
将礼物按照分类存入列表中
```bash
# 普通礼物列表存入 4 个礼物
LPUSH ordinary 4001 4002 4003 4004
LPOP ordinary
# 4001 在最右边，第一次取出最左边的 4004
127.0.0.1:6379> LPOP ordinary
"4004"
127.0.0.1:6379> LPOP ordinary
```

插入/取出一个或者多个值插入列表头部（最右边） `RPUSH`,`RPOP`
这个命令和上面的命令是相同的结果，只是取出的位置不一样。
```bash
RPUSH key value [value]
RPOP key
```
列表头/尾取出一个元素，如果没有则阻塞等待，timeout = 0 时一直阻塞等待（timeout/s） `BLPOP`,`BRPOP`
```bash
BLPOP key [key] timeout
BRPOP key [key] timeout
```
监听礼物列表是否有商品，有则在右边阻塞取出
```bash
LPUSH ordinary 4001 4002 4003 4004

# 执行 BRPOP 每次取出一条，如果没有，则一直等待
127.0.0.1:6379> BRPOP ordinary 0
1) "ordinary"
2) "4001"
```
`等待时添加一个元素，等待中的 BRPOP 立马输出新添加的元素`

获取指定 key 列表区间的元素 `LRANGE`
```bash
LRANGE key start stop
```
获取 0-2 区间的值
```bash
127.0.0.1:6379> LPUSH ordinary 4001 4002 4003 4004
(integer) 4
# 
127.0.0.1:6379> LRANGE ordinary 0 2
1) "4004"
2) "4003"
3) "4002"

```

2. 应用

微信公众号的推送信息可以使用 List 结构，用户关注的公众号每推送一个消息，在用户对应的列表增加一个文章ID

```bash
# 用户 1001 有 5001，5002 两个推送文章消息
LPUSH 1001:msg 5001,5002
```

###### SET

























参考 [testerhome](https://testerhome.com/topics/16402),[redis](http://redis.io)



