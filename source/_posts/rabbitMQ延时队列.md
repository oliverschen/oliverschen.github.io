---
title: rabbitMQ延时队列
date: 2019-11-02 16:28:50
tags: RabbitMQ
category: MQ
---

![Photo by on Unsplash](/rabbitMqDelayQueue.png)

在一些业务场景中，会有延迟这种场景出现，比如常见的订单提交之后付款计时，还有一些针对特定用户，比如会员的一些定时优惠的发放，都会在一个时间点去执行，这种一般定时器也是可以实现的，但是如果涉及数据量大，使用定时器处理不是很优的解决方案。这时候就需要了解下 mq 的延迟队列，来更好的实现这个问题。

<!-- more -->

#### 组件

1. Time-To-Live(TTL)

mq 允许给队列设置过期时间 TTL，单位是毫秒，当设有过期时间的消息进入到队列后，说明它只能在队列中 ‘存活’ TTL 时间。过期之后会成为 Dead letter (死信)。

2. Dead Letter Exchange(DLX)

消息过期成为死信之后，如果队列设置了 DLX，则会被 push 到 DLX 中等待绑定在 DLX 上的队列消费

#### 过程

通过上面 2 个机制的配合，就可以实现 mq 延迟队列。给要延迟处理的消息设置指定的过期时间，到期之后被 push 到 DLX，监听 DLX 上的队列进行消费。通过这个机制可以比较优雅的实现延迟机制。


#### Direct 交换机

1. 创建一个 direct 交换机

死信交换机可以用任意类型交换机实现。
```java

@Configuration
public class DeadLetterExchangeConfig {

    @Bean
    public DirectExchange delayExchange() {
        return new DirectExchange(MqExchangeEnum.DELAY_EXCHANGE_TEST.getKey(),
                true,false);
    }
}
```

2. 配置枚举类
``` java

@Getter
@AllArgsConstructor
public enum MqExchangeEnum {

    EXCHANGE_TEST("exchange_test", "direct交换机"),
    FANOUT_EXCHANGE_TEST("fanout_exchange_test", "fanout交换机"),
    TOPIC_EXCHANGE_TEST("topic_exchange_test","topic交换机"),
    DELAY_EXCHANGE_TEST("delay_exchange_test","死信交换机");


    private String key;
    private String desc;
}
```

3. 常量类
```java
@Getter
@AllArgsConstructor
public enum MqArgsEnum {

    X_DEAD_LETTER_EXCHANGE("x-dead-letter-exchange","死信交换机参数"),
    X_DEAD_LETTER_ROUTING_KEY("x-dead-letter-routing-key","死信路由参数"),
    X_MESSAGE_TTL("x-message-ttl", "过期时间");

    private String key;
    private String desc;

}
```
4. 队列
创建队列是重要的步骤
```java
// 绑定到 DLX 交换机上面，监听并消费。
@Bean
public Queue orderQueue() {
    return new Queue(MqQueueEnum.ORDER_QUEUE.getKey(),
            true, false, false);
}

// 死信队列，消息发送到这个队列中
@Bean
public Queue orderDeadQueue() {
    return QueueBuilder.durable(MqQueueEnum.DEAD_ORDER_QUEUE.getKey())
            .withArgument(MqArgsEnum.X_DEAD_LETTER_EXCHANGE.getKey(), MqExchangeEnum.DELAY_EXCHANGE_TEST.getKey())
            .withArgument(MqArgsEnum.X_DEAD_LETTER_ROUTING_KEY.getKey(), MqRouteKeyEnum.DEAD_LETTER_KEY.getKey())
            .withArgument(MqArgsEnum.X_MESSAGE_TTL.getKey(), 10000)
            .build();
}
```

5. 绑定
```java
@Bean
public Binding deadLetterBinding() {
    return BindingBuilder.bind(queueConfig.orderQueue())
            .to(deadLetterExchangeConfig.delayExchange())
            .with(MqRouteKeyEnum.DEAD_LETTER_KEY.getKey());
}
```

6. 发消息
```java
// 这里最好指定编码，不然会乱码。
@Test
public void deadLetterSender() {
    String msg = "死信消息测试";
    rabbitTemplate.convertAndSend(MqQueueEnum.DEAD_ORDER_QUEUE.getKey(),msg);
}
```
发完之后在后台界面可以看到死信队列中已经有一条消息，等待过期之后刷新，消息会到绑定的死信交换机上的队列。

7. 消费
```java
@Component
public class DeadLetterConsumer {

    @RabbitListener(queues = "order_queue")
    public void handlerDeadLetter(String msg) {
        System.out.println("DEAD_LETTER:" +msg);
    }
}
```

延迟在很多场景中都会使用的到，如果使用定时任务处理在大数据情况下会产生延迟等问题，这是不能被容忍的，并且使用任务来执行定时任务也不是很优的处理方式。使用 mq 延迟队列的话可以很好的解决对延迟场景下的业务处理，而且姿势比较帅。

mq (官方文档)[https://www.rabbitmq.com/dlx.html]

***

<center>我命由我不由天</center>


