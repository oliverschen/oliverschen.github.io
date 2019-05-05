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

##### pom
``` bash
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.jihe</groupId>
    <artifactId>jihe-consumer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>jihe-consumer</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

##### 启动类
```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class JiheConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(JiheConsumerApplication.class, args);
    }

}
```

##### 回调

之前我们使用 feigin 进行远程调用，现在 feign 配合 hystrix 进行使用，首先使用 @FeignClient 注解中的回调参数 fallback 指定具体的回调类。

```java
@FeignClient(name = "jihe-producer", fallback = UserRemoteHystrix.class)
public interface UserRemote {

    /**
     * 远程调用
     * @param name 参数
     * @return 调用结果
     */
    @RequestMapping("/user/info/{name}")
    String whoImI(@PathVariable("name") String name);

}
```

##### 回调实现类

编写具体的回调实现类，继承远程调用接口,编写相应的回调处理逻辑。 

```java
public class UserRemoteHystrix implements UserRemote {

    @Override
    public String whoImI(String name) {
        return "I'm api,my name is " + name + " and my service has down";
    }
}

```

到这里工程就搭建好了，一次启动项目。

首先访问服务提供方 producer `http://localhost:8081/user/info/jihe`,这时会直接访问到 producer 返回的消息。现在通过 consumer 进行远程调用 `http://localhost:8082/api/user/jihe` 这时可以看到返回的结果是一致的，远程调用成功，那如果我关掉服务的提供方在访问服务消费方会发生什么呢？这时会执行 fallback 指定的类中的逻辑，实现熔断作用。

***

<center>天涯远不远？人就在天涯，天涯怎会远呢</center>








