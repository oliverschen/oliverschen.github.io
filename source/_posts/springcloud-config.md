---
title: springcloud-config
date: 2019-12-15 22:39:09
tags: springcloud
category: springcloud
---

![Photo by kejsirajbek on wallhaven.cc](/springcloud-ribbon.png)

项目拆分成微服务架构之后，各个服务的配置文件会增多，并且每个环境（开发，测试，预发布，生产）都会有各自环境对应的配置，单独管理起来很容易出问题，springcloud-config 就是来解决这个问题的，它将配置文件统一管理，它支持将配置放在配置服务内存中，也支持放在远程 git 仓库中，它和注册中心结构类似，也是由两个部分组成，config server & config client。

<!--more-->

#### springcloud-config

##### 创建工程

创建 config 工程，引入 config 坐标
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
    <artifactId>jihe-config</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>jihe-config</name>
    <description>config service</description>

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
            <artifactId>spring-cloud-config-server</artifactId>
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

##### 开启 config server
```java
@EnableConfigServer
@SpringBootApplication
public class JiheConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(JiheConfigApplication.class, args);
    }

}
```

##### 配置文件
```yml
debug: true
server:
  port: 9090

spring:
  application:
    name: jihe-config
  cloud:
    config:
      server:
        git:
          uri: https://github.com/oliverschen/springcloud.git
          search-paths: /jihe-config/**
          username: 
          password:
      # 分支名
      label: master
```
##### 创建配置文件
创建 `user-config-dev.yml`，并提交到远程仓库
```yml
user:
  username: jihe
  age: 18
  email: XXX@gmail.com
```

##### 启动
访问远程仓库的配置文件 `http://localhost:9090/user-config-dev.yml` 就可以访问到用户配置文件了。

#### 动态更新配置

springcloud-config + springcloud-bus 实现配置文件动态刷新。




#### 高可用配置中心

