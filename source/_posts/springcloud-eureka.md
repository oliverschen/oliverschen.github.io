---
title: Springcloud-eureka
date: 2019-04-18 22:08:39
tags: SpringCloud
category: SpringCloud
comments: true
---

![不如吃茶去](springcloud-eureka/eureka.png)

spring cloud netflix eureka 是对 Netflix 公司开源组件封装后的服务注册与发现组件。主要负责对服务的治理功能。也是微服务的核心组件之一，它由 euraka server 和 eureka client 组成，服务的提供方将服务注册到 eureka server，服务的消费方通过 eureka client 调用在注册中心的服务。这样就完成了服务的提供和调用。

<!-- more -->

#### eureka

##### 续约

服务的续约：应用内的 eureka client 后台会启动一个定时任务，跟 eureka server 保持心跳续约任务。每个一段时间（默认30s）向 eureka server 发送一次 renew 请求进行续约，告诉 eureka server 自己还活着，防止被 eureka server 的 evict 任务剔除掉。

##### 下线

服务下线：应用内的 eureka client 在应用停止后，向 eureka server 发送 cancel 请求，告诉注册中心自己已经关闭了， eureka server 收到请求后会将其移除注册列表，防止消费端消费不可用的服务。

##### 剔除

服务剔除：eureka server 启动后在后台会启动一个 evict 任务，对一定时间没有续约的服务进行剔除。


#### eureka server
##### pom 坐标

``` xml

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
    <artifactId>jihe-eureka</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>jihe-eureka</name>
    <description>eureka server</description>

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
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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
如果是第一次引入的话要下载依赖包，可能需要一会。

##### 开启 eureka server

在启动类加入注解

``` java
@EnableEurekaServer
@SpringBootApplication
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

这里以用户 user 服务来充当服务的提供者
##### pom 坐标

``` xml
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
    <artifactId>jihe-user</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>jihe-user</name>
    <description>user service</description>

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

###### 注册服务
``` java
@EnableEurekaClient
@SpringBootApplication
public class JiheUserApplication {

    public static void main(String[] args) {
        SpringApplication.run(JiheUserApplication.class, args);
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
    name: jihe-user

# 注册到服务中心
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8080/eureka/

```
这样就注册好服务了，现在访问 http://localhost:8080 注册中心页面就可以看到已经注册的服务

**Instances currently registered with Eureka**

| Application     |   AMIs   |   Availability Zones | Status|
| ---- | ---- | ---- | ---- |
|JIHE-USER| 	n/a (1) |(1) |UP (1) - 192.168.0.104:jihe-user:8081|

到这里服务的注册和发现就完成了，可以看到服务提供者讲服务注册到服务中心，供服务的消费者调用服务,那微服务之间是如何调用的呢？下面看下另外一个组件。


#### 提供接口

在 user 服务中创建 UserController,并向外抛一个可以访问的接口
```java
@RestController
public class UserController {

    @RequestMapping("/user/{id}")
    public String getUser(@PathVariable("id") int id) {
        return "我是" + id + "号用户";
    }

}
```

#### 服务消费者

订单 order 服务充当 user 服务的消费者

##### pom

``` xml
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
    <artifactId>jihe-order</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>jihe-order</name>
    <description>order service</description>

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

##### 配置文件

```yml
debug: false

server:
  port: 8082

spring:
  application:
    name: jihe-order

# 注册到服务中心
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8080/eureka/
```

#### RestTemplate

spring 提供了一个 rest 接口调用组件 restTemplate 来调用其他服务，这里使用它来调用 user 服务接口，在订单服务中，创建 restTemplate 实例对象。

##### 启动类配置

消费者端开启服务发现和 Feign 客户端

```java
@SpringBootApplication
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

##### 消费者远程调用接口
创建 OderService 类，调用 user 服务（这里直接写了具体的类，没有使用接口的方式）

```java
@Service
public class OderService {
    @Autowired
    private RestTemplate restTemplate;

    public String getUser(int id) {
        String url = "http://jihe-user/user/{id}";
        return restTemplate.getForObject(url, String.class, id);
    }
}
```


##### 调用

```java
@RestController
public class OderController {

    @Autowired
    private OderService oderService;

    @RequestMapping("/order/{name}/{id}")
    public String getOrderAndUserInfo(@PathVariable("name") String name, @PathVariable("id") int id) {
        String user = oderService.getUser(id);
        return "我是：" + name + "," + user;
    }
}
```
到这里就完成了微服务模块最小的一个结构，`服务注册中心`，`服务提供者`，`服务调用者`。先启动 注册中心，然后在启动服务提供者，在启动服务调用者，在浏览器可以测试，直接访问服务的提供者，是可以访问到。通过调用者，也可以直接访问到服务的提供者，但是访问的端口和路径是不一样的。如果熟悉 spring boot 的话其实 spring cloud 的简单使用并不难，继续加油。
以上代码[地址](https://github.com/oliverschen/springcloud)


***
<center>不积跬步，无以至千里</center>
