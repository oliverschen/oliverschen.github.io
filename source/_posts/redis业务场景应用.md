---
title: redis业务场景应用
date: 2019-11-23 21:38:33
tags: redis
category: 分布式锁
---

![Photo by lumberjacck on wallhaven.cc](/redisLock.png)

redis 最常见的场景就是分布式缓存，分布式锁的场景。它本身提供的数据结构在实现一些功能时要比关系型数据库方便很多，比如点赞，好友关系等功能，并且不用担心并发以及性能问题。这里记录 redis 经常使用的一些场景。  ヽ(ˋ▽ˊ)ノ

<!--more-->

