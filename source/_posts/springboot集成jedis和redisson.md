---
title: Springboot集成Jedis和Redisson
date: 2019-11-23 21:38:33
tags: [Lock,Redis]
category: [Lock,Redis]
comments: true
---

![Photo by lumberjacck on wallhaven.cc](/redisLock.png)

redis 最常见的场景就是分布式缓存，分布式锁的场景。它本身提供的数据结构在实现一些功能时要比关系型数据库方便很多，比如点赞，好友关系等功能，并且不用担心并发以及性能问题。这里记录下springboot整合jedis和redisson框架 ヽ(ˋ▽ˊ)ノ

<!--more-->


##### redisson

redis Java 客户端，类似 jedis，但是有更加丰富的解决方案，在分布式锁方面也提供了一套 API 来轻松实现分布式锁。[reddison wiki](https://github.com/redisson/redisson/wiki/)

##### spring boot 整合

1. 引入 jar 包
```xml
<!-- redis -->
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.8.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2. 配置文件

```yml
server:
  port: 8080

# redis 配置
spring:
  redis:
    host: 192.168.31.107
    port: 6379
    password: 123456
    jedis:
      pool:
        max-active: 10
        max-idle: 8
        max-wait: 3000ms
        min-idle: 4
```

3. jedis 配置类

```java
@Configuration
@EnableCaching
public class JedisConfig extends CachingConfigurerSupport {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        // key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // hash的key也采用String的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        // value序列化方式采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        // hash的value序列化方式采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }

}
```

4. redisson 配置类

```java
@Configuration
public class RedissonConfig {

    @Value("${spring.redis.host}")
    private String host;

    @Value("${spring.redis.port}")
    private String port;

    @Value("${spring.redis.password}")
    private String password;


    @Bean
    public RedissonClient redissonClient(){
        Config config = new Config();
        config.useSingleServer().setAddress("redis://" + host + ":" + port).setPassword(password);
        return Redisson.create(config);
    }

}
```

5. jedis 工具类

```java
@Component
public class JedisUtil {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    /**
     * 设置 String 类型键值对
     * @param key 键
     * @param value 值
     * @return true or false
     */
    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 获取键对应值
     * @param key 键
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }
}
```

6. redisson 工具类

```java
@Component
public class RedissonUtil {

    @Autowired
    private RedissonClient redissonClient;


    /**
     * 获取锁
     * @param key key
     * @return lock
     */
    public RLock getLock(String key) {
        return redissonClient.getLock(key);
    }

}
```

7. 测试
```java
@Slf4j
@RestController
public class RedisController {

    @Autowired
    private JedisUtil jedisUtil;
    @Autowired
    private RedissonUtil redissonUtil;

    /**
     * 测试 jedis
     * @return name
     */
    @RequestMapping("/jedis")
    public String setAndGetRedis() {
        Random random = new Random();
        String key = random.nextInt(1000) + ":name";
        jedisUtil.set(key, "jihe" + key);
        log.info("获取到的key:{}",key);
        return (String) jedisUtil.get(key);
    }

    @RequestMapping("/redisson")
    public String lockAndUnlock() {
        RLock lock = redissonUtil.getLock("2001:product");
        log.info("获取到了锁对象");
        // 模拟处理业务
        try {
            sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
        return "处理完成";
    }
}
```


以上只是简单的将 2 个框架整合在一起，在具体业务场景下的一些使用会抽空继续更新。代码[github 地址](https://github.com/oliverschen/springBoot/tree/master/springboot-redisson-jedis)

***
<center>喂马，劈柴，周游世界</center>