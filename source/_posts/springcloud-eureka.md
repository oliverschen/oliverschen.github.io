---
title: springcloud-eureka
date: 2019-04-18 22:08:39
tags: springcloud
category: springcloud
---

![不如吃茶去](springcloud-eureka/eureka.png)

spring cloud netflix eureka 是对 Netflix 公司开源组件封装后的服务注册与发现组件。主要负责对服务的治理功能。也是微服务的核心组件之一，它由 euraka server 和 eureka client 组成，服务的提供方将服务注册到 eureka server，服务的消费方通过 eureka client 调用在注册中心的服务。这样就完成了服务的提供和调用。

<!-- more -->

***

#### eureka server

##### pom 坐标

``` xml

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
    <artifactId>jihe-eureka</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>jihe-eureka</name>
    <description>demo of the eureka server</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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
如果是第一次引入的话要下载依赖包，可能需要一会。

##### 开启 eureka server

在启动类加入注解

``` java
@SpringBootApplication
@EnableEurekaServer  // 开启服务
public class JiheEurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(JiheEurekaApplication.class, args);
    }
}

```

##### 配置


``` yml
# 日志
debug: false

spring:
  application:
    name: jihe-eureka

# 服务端口
server:
  port: 8080

eureka:
  client:
    register-with-eureka: false # 为 true 时注册中心会尝试注册自己，这里关闭。但是集群时需要打开，因为注册中心会相互注册
    fetch-registry: false # 为 true 时服务中心进行服务检索
    service-url:
      defaultZone: http://localhost:${server.port}/eureka/  # 注册中心地址

```

现在启动服务，访问 http://localhost:8080 就可以看到 eureka server 页面了。

#### 服务提供者

##### pom 坐标

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.jihe</groupId>
    <artifactId>jihe-producer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>jihe-producer</name>
    <description>Demo of the service producer</description>

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

###### 注册服务
``` java
@SpringBootApplication
@EnableDiscoveryClient # 开启服务发现
public class JiheProducerApplication {
    public static void main(String[] args) {
        SpringApplication.run(JiheProducerApplication.class, args);
    }
}
```
##### 配置

``` yml
debug: false

server:
  port: 8081

spring:
  application:
    name: jihe-producer

# 注册到服务中心
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8080/eureka/

```
这样就注册好服务了，现在访问 http://localhost:8080 注册中心页面就可以看到已经注册的服务

**Instances currently registered with Eureka**
|Application|AMIs|Availability Zones|Status
|----|----|----|----|
|JIHE-PRODUCER| 	n/a (1) |(1) |UP (1) - 192.168.0.104:jihe-producer:8081|


到这里服务的注册和发现就完成了，可以看到服务提供者讲服务注册到服务中心，供服务的消费者调用服务，如果熟悉 spring boot 的话其实 spring cloud 的简单使用并不难，继续加油。
以上代码[地址](https://github.com/oliverschen/spring-cloud-example)

***
<center>不积跬步，无以至千里</center>
