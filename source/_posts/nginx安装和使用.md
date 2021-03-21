---
title: Nginx安装和使用
date: 2019-11-25 23:25:00
tags: Nginx
category: Nginx
comments: true
---

![Photo by deleted on wallhaven.cc](/nginx.png)

nginx 是互联网公司必不可少的一个中间件。它不仅性能好，而且资源占用极低。nginx [中文文档](http://www.nginx.cn/doc/)，[官方下载地址](http://nginx.org/en/download.html)

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
        # 当前虚拟主机监听的端口
        listen IP:PORT                       # ipV4 地址
        listen       [::]:80 default_server; # ipV6 地址
        # 当前虚拟主机名称 eg：www.baidu.com
        server_name 虚拟主机名
        # root：server 虚拟主机下的根目录
        root        主目录
        location [OPERATOR] URL {  # 指定 URL 的特性
            if CONDITION{

            }
        }
    }
}
```
server_name

```bash
 1. 虚拟主机名称后可以跟多个由空白字符分割的字符串
 2. 支持 * 通配任意长度的任意字符：server_name *.baidu.com www.baidu.*
 3. 支持 ~ 起始的字符做正则表达式匹配，但是存在性能问题：server_name ~^www\d+\.baidu\.com$
 匹配优先级：
 a. 字符串精确匹。b. 左侧 * 通配符。c. 右侧 * 通配符 eg:com.www.baidu.* 。 d. 正则。e. defult_server
```
location 配置

```bash
# 在一个 sever 中 location 配置段可存在多个，用于实现从 uri 到文件系统的路径映射，nginx 会根据用户请求的 uri 来检查定义的所有 location,并找出一个最佳匹配进行应用。
location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
```
location 中也有 root 路径，当请求是具体的 location 指向的路径时，走 location 中的 root 目录
```bash
# 如果访问 http:192.168.31.107/news 则访问的时 location 指向的 /usr/local/html/news 目录下的文件，不带路径则访问的是 server 下 root 目录下的文件
server {
        listen       80 default_server;
        server_name  _;
        root        /usr/local/html;
        location /news {
                root /usr/local/html
        }
    }
```

路径解析
> 在 location 中指定路径有两种方式，一种是 root，一种是 alias。它们之间的区别是：
> root 方式处理结果 = root 路径 + location 路径
> alias 方式处理结果 = alias 路径直接替换 location 路径。



匹配：
`=` ：对 URL 精确匹配；
```bash
location = /{  }  只能精确匹配 http:fengzhu.top/  
```
`^~`：对 URL　的最右边部分做匹配检查，不区分字符大小写
`~`：对 URL 做正则匹配，区分大小写
`~*`：对 URL 做正则匹配，不区分大小写
不带符号：匹配起始于此 URL 的所有 URL
优先级：=,^~,~/~*,不带符号

**还是要自己配置试试，才能知道具体是怎么运行的**

***

<center>just do it</center>