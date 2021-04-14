---
title: SpringCloud
date: 2019-04-18 22:09:55
tags: SpringCloud
category: SpringCloud
comments: true
---

![大多数人的生活，是无数次选择的结果](springcloud/springcloud.png)

最近两年微服务架构特别火，很多公司都投身到微服务的怀抱。面试的时候面试官也一定会问有没有用过或者了解过微服务，微服务的火热可见一斑。很多公司都在微服务的路上试探，我们公司也不例外。目前一般 Java 为基础的公司大多用的都是 spring 全家桶，而近年出了 spring boot 和 spring cloud，为微服务提供了一站式解决方案。

<!-- more -->

#### 什么是 Spring boot

官方是这样介绍的：“spring boot 可以轻松创建独立的企业级应用。”
它是对 spring 框架的封装，简化了开发流程，专注于开发功能。去除传统 xml 配置的方式。以 java 代码配合注解的方式实现配置，更易于理解和编写。因为它的这些特性，它也成为了开发微服务系统的基础。之前 spring 能开发的它都可以做到，并且更快速，更高效。

#### 什么是 Spring Cloud

spring cloud 是一系列框架的集合，它包含了 服务的注册、发现，服务网关，配置中心、消息总线、负载均衡、断路器、数据监控等，而这些组件中大部分也不是 spring 公司研发的，而是基于现有的一些优秀开源框架进行二次封装的。当然这也是 spring 不重复造轮子的理念。而这些服务的开发都是基于 spring boot 这款优秀的框架的。spring cloud 现有组件都有 20 多个，而且还在新增中。

##### 优点
1. 可扩展性强：可以针对需求方便的进行扩展
2. 弹性：各个组件独立部署，单个组件故障不会导致雪崩效应，系统还是正常运行
3. 组合性高：不同服务之间的不同组合，可以完成不同业务的开发。

##### 缺点
1. 运维成本增高
2. 系统复杂性增高


#### spring boot 和 spring cloud

上面分别了解了 spring boot 和 spring cloud 之后相信对他们的区别和联系也有了认识。简单来说，没有 spring cloud，spring boot 是可以独立创建应用，并且投入生产不会有影响。而 spring cloud 的开发却离不开 spring boot 的支持。也就是说  spring cloud 是基于 spring boot 的，分清楚两者关系在学习微服务架构的时候就会有一个整体的概念，学好 spring boot 意味着 spring cloud 开发也会顺畅很多。


#### dubbo

dubbo 是阿里开源的一款服务治理中间件，在国内特别火，并且其有一个强大的监控后台，最近又开始了更新。相信未来在服务治理方面会做的更好。



#### spring cloud 和 dubbo

下面是一张它们核心功能对比表。

| 组件     |   dubbo   |    spring cloud  |
| ---- | ---- | ---- |
|  服务注册中心    |   zookeeper   | Eureka 等等..     |
|  服务网关    |  无    |  Zuul 等..    |
|  断路器  | 不完善     |  Hystrix    |
|配置中心| 无| Config|
|链路追踪|无|Slenth|
|消息总线|无|Bus|

其实直接把 dubbo 和 spring cloud 放在一起对比是不公平的，spring cloud 是封装了很多优秀组件之后的一个微服务解决方案的集合，而 dubbo 更专注于服务治理，在服务治理和监控方面做的比是spring cloud 还要优秀。

总的来说 spring cloud 有 spring 这个强大的组织做背书，在未来微服务方向肯定是所向披靡的，但是 dubbo 在服务治理方面也会一枝独秀，毕竟在阿里这样体量的公司和实际的业务场景下，它只会进化的越来越完善，越来越优秀。


***

<center>愿你 平安喜乐</center>