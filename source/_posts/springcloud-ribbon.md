---
title: springcloud-ribbon
date: 2019-12-15 00:09:27
tags: springcloud
category: springcloud
---

![Photo by kejsirajbek on wallhaven.cc](/springcloud-ribbon.png)


一般负载均衡指的是服务端的负载均衡，硬件有 F5 等设备，软件方面有 nginx 等。负载均衡是系统在高可用和容灾性等方面的重要的解决方案之一。而 Ribbon 是一款基于客户端的负载均衡应用。
我的理解是，请求没有到达应用之前，是由 nginx 来进行请求分发，决定进入到那个服务，当请求进入到服务之后，在服务调用时由 Ribbon 来进行服务之间请求分发，实现内网应用的负载均衡。
<!--more-->


#### ribbon

一般 `spring-cloud-starter-netflix-eureka-client` 已经引入了 ribbon 组件，在使用时直接进行使用就可以了。


##### @LoadBalanced

负载均衡注解，官方解释 `Annotation to mark a RestTemplate bean to be configured to use a LoadBalancerClient.` 大概意思是 让 restTemplate 实例配置使用负载均衡客户端。


##### 负载均衡算法


###### 轮询



###### 随机获取


##### 使用

使用在 [eureka](https://fengzhu.top/2019/04/18/springcloud-eureka/)中的例子来进行使用。

###### 引入
在订单服务启动时加载负载均衡配置
```java
@SpringBootApplication
public class JiheOrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(JiheOrderApplication.class, args);
    }

    /**
     * 负载均衡注解
     **/
    @Bean
    @LoadBalanced
    @ConditionalOnMissingBean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}

```
请求到服务时打印端口信息，方便测试

```java

@RestController
public class UserController {

    @Value("${server.port}")
    String port;

    @RequestMapping("/user/{id}")
    public String getUser(@PathVariable("id") int id) {
        return "我是" + id + "号用户" + ",访问端口：" + port;
    }

}
```

###### 测试

1. 首先要在 idea 开启并行启动，在 idea 右上角编辑服务栏打开 `Edit Configurations` ，进入到编辑页面后，右上角会有和复选框 `Allow Parallel run` ，选择之后项目修改端口后就可以启动多个实列了。

2. 修改端口号，同时启动 3 和 user 服务。

3. 使用 order 服务调用 user 服务，通过不同的端口号打印信息，会看到 ribbon 默认是轮询的方式来负载均衡请求客户端的。


***

<center></center>