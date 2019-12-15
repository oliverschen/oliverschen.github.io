---
title: springcloud-zuul
date: 2019-12-15 17:03:26
tags: springcloud
category: springcloud
---

![Photo by kejsirajbek on wallhaven.cc](/springcloud-ribbon.png)


#### 网关

服务网关时微服务架构中服务的统一入口，除了服务路由分发，负载均衡等，还具备鉴权等功能。zuul 就是 springcloud 中提供服务网关的组件。

![网关作为微服务架构统一入口](/zuul-user.png)

##### 优点

1. 易于监控：可以在网关收集监控数据将其推送到外部系统进行分析。
2. 易于认证：在网关进行鉴权等，通过后在将请求分发到具体服务。请求进入网关之后，各个微服务之间进行无状态调用。
3. 易于重构：客户端和服务之后交互入口统一，在后期服务重构等操作时，可以进行客户端无感知重构。

#### 搭建

##### pom 

引入 zuul pom 坐标。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.jihe</groupId>
    <artifactId>jihe-zuul</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>jihe-zuul</name>
    <description>zuul service</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
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

##### 配置

```yml

debug: false

server:
  port: 9999

spring:
  application:
    name: jihe-zuul

# 注册到服务中心
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8080/eureka/
zuul:
  routes:
    # 自定义路由
    api-order:
      path: /api-order/**
      serviceId: jihe-order
    api-user:
      path: /api-user/**
      serviceId: jihe-user
```

##### 配置启动类
```java
@EnableZuulProxy
@EnableEurekaClient
@SpringBootApplication
public class JiheZuulApplication {

    public static void main(String[] args) {
        SpringApplication.run(JiheZuulApplication.class, args);
    }

}
```

在自定义路由部分可以自己按照需求来自定义路由分发，上面当请求是 `/api-order/**` 开头时，统一走 `jihe-order` 服务，当请求是 `api-user` 时，统一走 `jihe-user` 服务。 `*` 匹配任何路径。

#### 测试

启动注册中心，启动网关和用户服务，访问 user 服务 `http://localhost:8081/user/1` 返回结果 `我是1号用户,访问端口：8081`，现在通过网关访问 user 服务 `http://localhost:9999/api-user/user/1` 返回结果 `我是1号用户,访问端口：8081`，通过网关，将 `api-user` 路由请求分发到了 user 服务，这里也可以用服务名来调用 `http://localhost:9999/jihe-user/user/1` 也是返回相同的结果。

##### 忽略服务名访问

需要在配置文件添加下面配置：
```yml
zuul:
  routes:
    # 自定义路由
    api-order:
      path: /api-order/**
      serviceId: jihe-order
    api-user:
      path: /api-user/**
      serviceId: jihe-user
  # 忽略通过服务名访问
  ignored-services: "*"
```
配置之后通过服务名将访问不到资源。

##### 绑定 url 映射

跳转到指定的路由。

```yml
zuul:
  routes:
    baidu:
      url: http://www.baidu.com
      path: /baidu/**
```

##### url 映射负载均衡

```yml
ribbon:
  eureka:
    enabled: false
# 请求服务负载均衡
jihe-user:
  ribbon:
    listOfServers: http://ke.qq.com/,http://www.imooc.com/

zuul:
  routes:
    class:
      serviceId: jihe-user
      path: /ketang/**
```
请求 `http://localhost:9999/ketang` 请求线性负载到 listOfServers 中的服务。

#### 过滤器

配置统一过滤器，进行鉴权等操作。

```java
package com.jihe.zuul.filter;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import org.apache.commons.lang.StringUtils;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

/**
 * @description: 统一过滤器
 * @author: ck
 * @time: 2019-12-15 22:16
 **/
@Component
public class UnionFilter extends ZuulFilter {


    /**
     * 包含类型：
     * pre:路由代理之前执行
     * route：代理时执行
     * error：代理出错执行
     * post：route || error 之后执行
     */
    @Override
    public String filterType() {
        // 之前执行
        return FilterConstants.PRE_TYPE;
    }

    /**
     * 存在多个过滤器时执行顺序
     * 数字越小，优先级越高
     */
    @Override
    public int filterOrder() {
        return 1;
    }

    /**
     * 是否执行过滤器：true 需要
     */
    @Override
    public boolean shouldFilter() {
        return true;
    }

    /**
     * 具体执行逻辑
     *
     * @throws ZuulException
     */
    @Override
    public Object run() throws ZuulException {
        System.out.println("enter UnionFilter,auth begin");
        RequestContext context = RequestContext.getCurrentContext();
        // 获取到 request 对象，进行鉴权等操作
        HttpServletRequest request = context.getRequest();
        String token = request.getHeader("token");
        if (StringUtils.isBlank(token)) {
            try {
                context.setSendZuulResponse(false);
                context.setResponseStatusCode(400);
                context.getResponse().getWriter().println("auth failed,please reload");
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return null;
    }
}

```
配置好过滤器之后访问 `http://localhost:9999/api-user/user/1` 服务，直接返回 `auth failed,please reload` 结果，鉴权失败，请重试。

网关将鉴权等操作和服务分离出来，并且将整个微服务架构统一入口，也是微服务架构中重要的环节之一。以上就是 zuul 学习中的一些记录。

今天重新看了一遍 `启示录`，还是很震撼，血腥而真实。还有一个月左右就要 2020 了，年初的 flag ，实现了几个？？？

***

<center>看完一本书，一部电影，真正留下来的是什么?</center>