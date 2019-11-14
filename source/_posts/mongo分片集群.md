---
title: mongo分片集群
date: 2019-11-12 22:59:17
tags: mongo
category: mongo
---

![by https://wallhaven.cc/w/lq9pxq](mongo分片集群/replicaSet.png)

mongdb 通过分片机制将数据分布在多台机器上面，实现了横向扩展。也支持了非常大数集和高吞吐量操作。mongodb 可以很好的利用机器的内存资源，内存越大，查询就会越快。mongo 对数据的结构没有其他限制，对开发者很友好，很适合迭代很快，表字段变化很多的场景。

##### 分片

mongo 通过分片机制，将数据拆分后存储在不同机器上面。这样就可以存储更多的数据，并且能以很快的速度读取出来。mongo 原生就支持了分布式特性，在使用方面也是非常方便。下面先[下载](https://www.mongodb.org/dl/linux/x86_64)进行部署。









[官方文档](https://docs.mongodb.com/manual/sharding/)