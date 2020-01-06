---
title: nginx安装和使用
date: 2019-11-25 23:25:00
tags: nginx
category: nginx
---

![Photo by deleted on wallhaven.cc](/nginx.png)

nginx 是互联网公司必不可少的一个中间件。它不仅性能好，而且资源占用极低。nginx (http://www.nginx.cn/doc/)[中文文档]，(http://nginx.org/en/download.html)[官方下载地址]

<!--more-->


#### 安装

```bash
# 安装 epel 源，如果 yum 安装时提示没有找到 nginx 的话
yum install epel-release

yum install nginx
```

#### 命令

##### 启动
```bash
nginx
```
##### 关闭
```bash
nginx -s stop
```
##### 重新加载配置
```bash
nginx -s reload
```

##### 配置检查
```bash
nginx -t
```

#### 配置

main 配置

```bash
# 指定 worker 进程的运行身份，如组不指定，默认和用户名同名
user nginx;
# worker 进程的数量，通常为当前主机 cpu 物理核心数
worker_processes auto;
# worker 进程能够打开的文件数量上限（支持的并发数如：65535）
worker_rlimit_nofile number
error_log /var/log/nginx/error.log;
# 指定存储 nginx 主进程 PID 的文件路径
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
# 指明包含进来的其他配置文件片段
include /usr/share/nginx/modules/*.conf;
# 后台执行，默认 on 开启，off 则关闭，适合调试使用
daemon off;
# 是否以 master/worker 模式运行，默认 on ，off 将不启动 worker
master_process on|off
```
events 配置

```bash
events{
   }
# 参数
# 每个 worker 进程所能够打开的最大并发链接数数量
# 总最大并发数：worker_processes * worker_connections
worker_connections number
# 并发连接请求处理方式，默认选择最优 epoll
use epoll/select
# 处理新连接请求方法，on 指 worker 轮流处理新请求，off 指每个新请求到达会通知（唤醒）所有 worker 进程，但是只有一个进程可以获得连接，造成‘惊群’，影响性能。
accept_mutex on | off
```
http 配置
 
```bash
http{
    # 各个 server 的公共配置
    server{
      # 每个 server 用于定义一个虚拟主机  
    }
    server{
        listen IP:PORT
        server_name 虚拟主机名
        root        主目录
        location [OPERATOR] URL {  # 指定 URL 的特性
            if CONDITION{

            }
        }
    }
}
```

location 配置

```bash
# 在一个 sever 中 location 配置段可存在多个，用于实现从 uri 到文件系统的路径映射，nginx 会根据用户请求的 uri 来检查定义的所有 location,并找出一个最佳匹配进行应用。
location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }

```



***

<center></center>