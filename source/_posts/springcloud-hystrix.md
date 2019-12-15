---
title: springcloud-hystrix(二)
date: 2019-04-25 22:38:29
tags: springcloud
category: springcloud
---

![行到水穷处，坐看云起时](springcloud-hystrix/hystrix.png)

springcloud hystrix 熔断器，顾名思义。在现实生活中也有很多熔断器，像家里的过载保护开关就充当这个角色，当有用电器过载或者发生短路时，就能及时切断电路，避免造成更大的损坏。hystrix 在整个微服务系统中也充当类似这种角色，但是它的功能要远比刚刚这个例子要丰富的多。
<!-- more -->

#### 雪崩效应

通俗来讲雪崩效应就是有一处发生雪崩可能会造成大面积雪崩的现象。在微服务系统中，多个服务之间存在相互之间的调用，如果有一处服务出现物理故障或者其他故障，如果不及时处理，可能会造成更多服务的故障甚至会影响整个系统瘫痪。这种现象可能发生在大量请求调用某个服务，导致此服务出现等待甚至崩溃后，致使其他服务也出现等待等现象，直到导致整个系统崩溃。这是一件可怕的事情，不过好在我们部署服务的时候一般都是集群部署，保证高可用。但是在实际业务场景中，可能会存在很多情况发生，熔断机制是保护服务高可用的最后一道防线。


#### hystrix

hystrix 一般部署在服务的调用方，也就是服务的消费放，我这里创建的项目和昨天的一样，今天会多创建一个服务的调用方，来部署 hystrix

##### 特性

1. 请求熔断：当请求到达服务失败的数量到达一定比例（默认50%），断路器会自动切换到 Open 状态，这时所有的请求会直接失败，不会发送到具体的服务，一般断路器 Open 状态一段时间（默认5秒）后，自动切换到 HALF-Open 半开状态。这时会判断下次请求，如果成功，则切换到 CLOSE 关闭状态。否则重新回到 Open 状态。
2. 服务降级：FallBack 相当于服务降级操作，当服务不可用时走异常处理逻辑返回一个默认结果，告知调用方服务处于异常状态。

##### pom

也是依赖 [eureka](https://fengzhu.top/2019/04/18/springcloud-eureka/)中的例子来进行集成 Hystrix 使用。
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

##### 启动类
```java
@SpringBootApplication
@EnableFeignClients
// 开启 Hystrix
@EnableHystrix
public class JiheOrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(JiheOrderApplication.class, args);
    }

    @Bean
    @LoadBalanced
    @ConditionalOnMissingBean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }

}
```

##### 开启 hystrix

```yaml
# 开启 hystrix
feign:
  hystrix:
    enabled: true
```

##### fallback

之前我们使用 feigin 进行远程调用，现在 feign 配合 hystrix 进行使用，首先使用 @FeignClient 注解中的回调参数 fallback 指定具体的回调类。

```java
@FeignClient(value = "jihe-user",fallback = FeignServiceFallBack.class)
public interface FeignService {

    @RequestMapping(value = "/user/{id}",method = RequestMethod.GET)
    String getUser(@PathVariable("id") int id);
}
```

##### 回调实现类

编写具体的回调实现类，继承远程调用接口,编写相应的回调处理逻辑。 

```java
@Component
public class FeignServiceFallBack implements FeignService {
    @Override
    public String getUser(int id) {
        return "user service is busy,please wait";
    }
}

```

到这里工程就搭建好了，按顺序启动项目。访问 order 服务 `http://localhost:8082/order-feign/jihe/1002` 会看到 `FEIGN-我是：jihe,我是1002号用户,访问端口：8081` 的结果，这是服务正常情况下的结果，现在将 user 服务直接关机，然后访问，则返回熔断后的结果 `FEIGN-我是：jihe,user service is busy,please wait`。

***

<center>天涯远不远？人就在天涯，天涯怎会远呢</center>








