---
title: rabbitMQ延时队列
date: 2019-11-02 16:28:50
tags: RabbitMQ
category: MQ
---

![Photo by Vadim Sadovski on Unsplash](RabbitMQ延时队列/rabbitMqDelayQueue.png)


#### 延迟

在一些业务场景中，会有延迟这种场景出现，比如常见的订单提交之后付款计时，还有一些针对特定用户，比如会员的一些定时优惠的发放，都会在一个时间点去执行，这种一般定时器也是可以实现的，但是如果涉及数据量大，使用定时器处理不是很优的解决方案。这时候就需要了解下 mq 的延迟队列，来更好的实现这个问题。


#### 流程

