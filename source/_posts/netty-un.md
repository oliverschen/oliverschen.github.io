---
title: netty
date: 2020-01-14 23:54:00
tags: netty
category: netty
---

![Photo by deleted on wallhaven.cc](/netty.png)


介绍
1. 高性能，时间驱动，异步非阻塞
2. 基于 NIO 的客户端，服务端编程框架
3. 可靠的稳定性和伸缩性

应用场景
1. 高性能领域
2. 多线程并发领域
3. 异步通信领域


Java IO 通信

BIO 通信

1. 一个客户端对应一个线程，当客户端个数增多时，服务端线程也随之增多，而线程对于服务端来说是一种有限资源。

伪异步 IO 通信

1. 线程池负责连接，当有大量请求进来时，也不会导致线程剧增而宕机。

NIO 通信

1. 缓冲区 buffer
2. 通道 channel （支持同时可读可写）
3. 多路复用器 selector


AIO 通信

1. 连接注册读写时间和回调函数
2. 读写方法异步
3. 主动通知程序



Netty 

优点：
1. API 简单
2. 入门门槛低
3. 性能高
4. 成熟，稳定





http: 半双工协议（）
webSocket：全双工协议
