---
title: RabbitMQ安装和使用
date: 2019-09-05 00:16:09
tags: [MQ,RabbitMQ]
category: [MQ,RabbitMQ]
comments: true
---

![Photo by Vadim Sadovski on Unsplash](RabbitMQ安装和使用/rabbitmq.png)

作为消息中间件，MQ （Message Queue） 在系统中有至关重要的作用，在高并发场景下，MQ 异步处理请求，缓解系统高峰期压力。在系统之间的调用中解耦，降低各个系统之间的依赖，提交系统的可扩展性。在一写复杂业务处理时，进行异步处理，类似乐观锁，及时返回给用户操作状态，至于业务逻辑通知 MQ 之后在进行处理，提高用户体验，也提高了服务器的处理能力。
<!--more-->


#### 安装

|  系统    |   版本   | 
| ---- | ---- | ---- |
| CenterOS |  7.6       |  
|   RabbitMq  | 3.6.10    | 

因为 RabbitMQ 是 erlang 实现的，所以先要安装 erlang 环境。

```bash
# 没有 wget 命令的先安装 wget
yum -y install wget
# 下载
wget http://www.rabbitmq.com/releases/erlang/erlang-19.0.4-1.el7.centos.x86_64.rpm

# 升级包
rpm -ivh erlang-19.0.4-1.el7.centos.x86_64.rpm

# 安装
yum -y install erlang

# 下载 rabbitmq
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.6/rabbitmq-server-3.6.6-1.el6.noarch.rpm

# 安装 mq
yum -y install rabbitmq-server-3.6.6-1.el6.noarch.rpm

```

在安装过程中碰到了缺少一些依赖包的情况，但是 Google 上都有解决方案，下载安装后可以解决。

#### 使用

##### 启动
```bash
cd /usr/sbin
service rabbitmq-server start
# 查看进程
ps -ef | grep rabbitmq 
```

##### 后台运行
```bash
rabbitmq-server -detached
```

##### 开机启动
```bash
chkconfig rabbitmq-server on
```
##### 安装 web 管理插件
```bash
rabbitmq-plugins enable rabbitmq_management

```
访问： http://ip:15672 就可以进入到管理页面了

##### 创建用户

rabbitmq 有个默认的 guest/guest 用户，但是只能在 localhost 下访问，所以要创建一个用户来可以进行远程访问

```bash
# 创建用户
abbitmqctl add_user admin admin
# 授予权限
rabbitmqctl set_user_tags admin administrator
# 分配权限
rabbitmqctl add_vhost admin

rabbitmqctl set_permissions -p admin admin ".*" ".*" ".*"

```


##### 关键字

1. Broker：可以理解成 rabbitmq 服务。
2. vhost:  虚拟主机，一个 broker 可以创建多个 vhost，用作权限分离。
3. Producer：消息生产者，消息的来源。
4. Consumer：消息消费者，负责消费生产者生产的消息。
5. Exchange：交换机，可以看作是一个消息的中转站。
6. Queue：队列，消息的载体，消息会被投递到一个或者多个队列中。
7. Binding：绑定，按照一定规则绑定交换机和队列。
8. Routing Key：路由key，交换机根据这个关键字投递到对应的队列。
9. Channel：通道，在 client 端每个链接，可以建立多个通道。

#### 模式

##### 简单队列

生产者将消息推到队列，消费者从队列消费消息。一个生产者对应一个消费者。

###### work 模式

一个生产者对应多个消费者，但是一个消息只能被一个消费者获取。

1. 创建一个 Direct 交换机
``` java
@Configuration
public class ExchangeConfig {

    @Bean
    public DirectExchange directExchange(){
        // params
        // 1. name:交换机名字
        // 2. durable:持久化，当为 true 时，在 mq 重启之后会重新加载此交换机
        // 3. autoDelete:自动删除，当交换机长时间不使用时，自动删除此交换机
        return new DirectExchange(RabbitMqConfig.EXCHANGE_TEST,
                true, false);
    }
}
```
2. 配置
``` java
@Configuration
public class RabbitMqConfig {

    public static final String EXCHANGE_TEST = "exchange_test";
    public static final String FIRST_QUEUE = "first_queue";
    public static final String SECOND_QUEUE = "second_queue";

    /** 队列key1*/
    public static final String FIRST_ROUTING_KEY = "first_routing_key";
    /** 队列key2*/
    public static final String SECOND_ROUTING_KEY = "second_routing_key";

    private final ConnectionFactory connectionFactory;
    private final ExchangeConfig exchangeConfig;
    private final QueueConfig queueConfig;


    public RabbitMqConfig(ConnectionFactory connectionFactory, ExchangeConfig exchangeConfig,
                          QueueConfig queueConfig) {
        this.connectionFactory = connectionFactory;
        this.exchangeConfig = exchangeConfig;
        this.queueConfig = queueConfig;
    }

    @Bean
    public SimpleMessageListenerContainer simpleMessageListenerContainer(){
        SimpleMessageListenerContainer simpleMessageListenerContainer = new SimpleMessageListenerContainer(connectionFactory);
        simpleMessageListenerContainer.addQueues(queueConfig.firstQueue());
        simpleMessageListenerContainer.setExposeListenerChannel(true);
        simpleMessageListenerContainer.setMaxConcurrentConsumers(5);
        simpleMessageListenerContainer.setConcurrentConsumers(1);
        //设置确认模式手工确认
        simpleMessageListenerContainer.setAcknowledgeMode(AcknowledgeMode.MANUAL);
        return simpleMessageListenerContainer;
    }

    @Bean
    public RabbitTemplate rabbitTemplate() {
        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        template.setConfirmCallback(msgSendConfirmCallBack());
        return template;
    }

    @Bean
    public AckCallback msgSendConfirmCallBack(){
        return new AckCallback();
    }


    /**
     * 根据指定的 key 将具体的队列绑定到 direct 交换机
     */
    @Bean
    public Binding firstBinding() {
        return BindingBuilder.bind(queueConfig.firstQueue())
                             .to(exchangeConfig.directExchange())
                             .with(RabbitMqConfig.FIRST_ROUTING_KEY);
    }
    @Bean
    public Binding secondBinding() {
        return BindingBuilder.bind(queueConfig.secondQueue())
                             .to(exchangeConfig.directExchange())
                             .with(RabbitMqConfig.SECOND_ROUTING_KEY);
    }
}
```
3. 队列
```java
@Configuration
public class QueueConfig {

    @Bean
    public Queue firstQueue() {
        // params
        // 1. name:队列名称
        // 2. durable:持久化，当为 true 时，在 mq 重启之后此队列自动加载
        // 3. exclusive:排他队列，当为 true 时，只有队列声明者才能调用
        // 4. autoDelete:自动删除，当为 true 时，自动删除长时间不适用的队列
        return new Queue(RabbitMqConfig.FIRST_QUEUE,
                true,false,false);
    }

    @Bean
    public Queue secondQueue() {
        return new Queue(RabbitMqConfig.SECOND_QUEUE,
                true, false, false);
    }
}
```
4. 回调
```java
@Slf4j
public class AckCallback implements RabbitTemplate.ConfirmCallback {
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        log.info("AckCallback 回调，ID:{}",correlationData);
        if (ack) {
            log.info("消息消费成功");
        }else {
            log.info("消息消费失败，原因：{}",cause);
        }
    }
}
```
消息消费完之后会有回调

5. 生产者
```java
@Test
public void sender() {
    CorrelationData correlationData = new CorrelationData("123456789");
    String msg = "第一次发送消息";
    rabbitTemplate.convertAndSend(RabbitMqConfig.EXCHANGE_TEST,
            RabbitMqConfig.SECOND_ROUTING_KEY,msg,correlationData);
}
```
我这里直接用 JUnit 进行测试

6. 消费费者
```java
@Component
public class MqConsumer {
    // 监听的队列名称，链接工厂实例
    @RabbitListener(queues = {RabbitMqConfig.SECOND_QUEUE, RabbitMqConfig.FIRST_QUEUE},
            containerFactory = "rabbitListenerContainerFactory")
    public void handlerMessage(String msg) {
        System.out.println("消费成功" + msg);
    }
}
```

在 Direct 交换机中，一个交换机可以绑定多个队列，通过不同的路由 key 来指定消息发送到对应的队列中进行消费。当消费者启动时，会自动监听消费相应的消息。


##### Fanout 交换机（订阅模式）

订阅模式类似公众号，推送一条消息之后，关注者都可以收到这天推送。

1. 枚举类

交换机配置枚举类
``` java
@Getter
@AllArgsConstructor
public enum MqExchangeEnum {

    FANOUT_EXCHANGE_TEST("fanout_exchange_test", "fanout交换机");


    private String key;
    private String desc;
}
```

队列枚举类

```java
@Getter
@AllArgsConstructor
public enum MqQueueEnum {

    FANOUT_QUEUE_A("fanout_queue_a","fanout交换机队列1"),
    FANOUT_QUEUE_B("fanout_queue_b", "fanout交换机队列2");

    private String key;
    private String desc;
}
```

交换机配置
``` java 
@Configuration
public class FanoutExchangeConfig {


    @Bean
    public FanoutExchange fanoutExchange() {
        return new FanoutExchange(MqExchangeEnum.FANOUT_EXCHANGE_TEST.getKey(),
                true,false);
    }

}
```
2. 绑定
```java
@Bean
    public Binding fanBindingA() {
        return BindingBuilder.bind(queueConfig.fanoutQueueA()).to(fanoutExchangeConfig.fanoutExchange());
    }

    @Bean
    public Binding fanBindingB() {
        return BindingBuilder.bind(queueConfig.fanoutQueueB()).to(fanoutExchangeConfig.fanoutExchange());
```

3. 发送消息
```java 
@Test
    public void fanoutSender() {
        for (int i = 0; i < 20; i++) {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            CorrelationData correlationData = new CorrelationData(i + "消息");
            String msg = "fanout exchange first test, time: " + System.currentTimeMillis() + " . msg num: " + i;
            rabbitTemplate.convertAndSend(MqExchangeEnum.FANOUT_EXCHANGE_TEST.getKey(),"",msg,correlationData);
        }
    }
```

4. 消费消息
监听相应的队列，进行消息处理。
``` java
@Component
@RabbitListener(queues = {"fanout_queue_a", "fanout_queue_b"})
public class FanoutConsumer {

    @RabbitHandler
    public void process(String msg) {
        System.out.println("fanout exchange 消息内容：" + msg);
    }

}
```
fanout 交换机推送消息到所有绑定到它的队列，这里指定具体的路由也是不会生效的


##### Topic 交换机（匹配模式）

topic 交换机绑定队列之后，可以根据路由 key 来匹配具体的队列，进行消息消费，以通配符的方式绑定到相应的队列，生产者发送相应的路由后，会按照规则匹配到具体的队列

1. 交换机配置

```java
@Configuration
public class TopicExchangeConfig {

    @Bean
    public TopicExchange topicExchange() {
        return new TopicExchange(MqExchangeEnum.TOPIC_EXCHANGE_TEST.getKey());
    }
}
```

2. 队列
```java
@Bean
    public Queue topicQueueA() {
        return new Queue(MqQueueEnum.TOPIC_QUEUE_A.getKey(),
                true, false, false);
    }

@Bean
public Queue topicQueueB() {
    return new Queue(MqQueueEnum.TOPIC_QUEUE_B.getKey(),
            true, false, false);
}
```

3. 路由

> `#` 表示 零个或者多个单词，匹配任何字符， 以 # 作为路由，topic 交换机的工作模式会和 fanout 交换机工作模式相同
> *   表示一个单词
```java
@Bean
    public Binding topicBindingA() {
        return BindingBuilder.bind(queueConfig.topicQueueA())
                .to(topicExchangeConfig.topicExchange()).with("#.user.#");
    }

@Bean
public Binding topicBindingB() {
    return BindingBuilder.bind(queueConfig.topicQueueB())
            .to(topicExchangeConfig.topicExchange()).with("topic.#");
}
```

4. 生产者
```java
@Test
public void topicSender() {
    for (int i = 0; i < 20; i++) {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        String msg = "top message:" + i;
        rabbitTemplate.convertAndSend(MqExchangeEnum.TOPIC_EXCHANGE_TEST.getKey(), "topic.user", msg);
    }
}
```

5. 消费者

```java
@Component
public class TopicConsumer {

    @RabbitListener(queues = "topic_queue_a")
    public void topicConsumerOne(String msg) {
        System.out.println("topic_queue_a 1 消费" + msg);
    }

    @RabbitListener(queues = "topic_queue_b")
    public void topicConTow(String tow) {
        System.out.println("topic_queue_b 2 消费" + tow);
    }
}
```

##### Headers
通过匹配消息 headers 中的 token 属性，会忽略「路由key」，如果一个消息 headers 中的 value 和队列绑定时的一样，那就会收到这条消息。

参考[链接](https://blog.csdn.net/zhuzhezhuzhe1/article/details/80454956)
参考[RabbitMq教程](https://www.kancloud.cn/digest/rabbitmq-for-java/122041)

***

<center>When life gives you lemons,make Lemonade</center>












