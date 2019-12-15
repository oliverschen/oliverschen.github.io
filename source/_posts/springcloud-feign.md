---
title: springcloud-feign
date: 2019-12-15 09:47:39
tags: springcloud
category: springcloud
---

![Photo by kejsirajbek on wallhaven.cc](/springcloud-feign.png)


在远程调用时，Java 提供了 UrlConnection 来进行远程调用，也有 Apace 封装的 HttpClient 库进行调用。 在[eureka](https://fengzhu.top/2019/04/18/springcloud-eureka/)中的例子使用了 RestTemplate 来进行服务调用，feign 也一样，是一个远程调用组件，它是以接口的方式来进行调用，时调用姿势更加优雅，使用起来更加方便。[官网地址](https://spring.io/projects/spring-cloud-openfeign#overview)

<!--more-->

#### feign

改造 [eureka](https://fengzhu.top/2019/04/18/springcloud-eureka/)中的例子，将 restTemplate 修改成 feign 调用。

##### 引入

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

##### 开启 feign

```java
@SpringBootApplication
// 开启 feign 客户端
@EnableFeignClients
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

##### 接口方式调用

feign 调用是以接口的方式进行 http 调用
```java
@FeignClient(value = "jihe-user")
public interface FeignService {

    @RequestMapping(value = "/user/{id}",method = RequestMethod.GET)
    String getUser(@PathVariable("id") int id);
}
```
1. @FeignClient 指定具体要调用的服务应用名。
2. @RequestMapping 指定要调用的服务路径，请求方式。


##### 调用

```java
@RestController
public class OderController {

    // @Autowired
    // private OderService oderService;

    // @RequestMapping("/order/{name}/{id}")
    // public String getOrderAndUserInfo(@PathVariable("name") String name, @PathVariable("id") int id) {
    //     String user = oderService.getUser(id);
    //     return "我是：" + name + "," + user;
    // }

    @Autowired
    private FeignService feignService;

    @RequestMapping("/order-feign/{name}/{id}")
    public String getOrderAndUserInfoByFeign(@PathVariable("name") String name, @PathVariable("id") int id) {
        String user = feignService.getUser(id);
        return "FEIGN-我是：" + name + "," + user;
    }
}
```

引入 feign 接口，直接调用接口中的方法，实现对远程服务的调用。实现方式很快捷方便。访问 `http://localhost:8082/order-feign/jihe/1002` 这个路径，可以看到具体的结果。


***

<center></center>