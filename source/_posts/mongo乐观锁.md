---
title: mongo乐观锁
date: 2019-03-31 21:04:22
tags: mongo
category: mongo
---

![](mongo乐观锁/mongoLock.png)

前几天完了一下 mongo 的一些基础操作，在工作中遇到最多的就是 CRUD 这种操作，但是随着业务复杂度增加，访问数量增加之后，对系统的需求也会随之增加。我在业务中使用最多的就是乐观锁，主要原因是因

<!-- more -->
为乐观锁有一下几点好处：
1. 在执行读操作时不会对数据进行加锁处理，这样提高了数据访问速度。
2. mongo 本身不支持事物，所以没有像关系型数据库那样完善的锁机制。

下面看下 mongo 是怎样实现乐观锁的，看下具体的乐观锁是如何实现对数据加锁的。

###### 乐观锁
乐观锁在读取数据时一般会认为数据没有被修改，所以在读的时候不会对数据加锁，但是在更新数据时会对其进行`加锁`,这里所谓的加锁一般是会设置一个版本号，在更新时看下这个版本号有没有变化，如果更新时版本号不一致，则说明数据已经被更新，当前更新操作不会被执行。
下面是 mongo 中实现乐观锁的一些具体操作

###### version
在实体类中添加版本号
``` java
@Document(collection = "user")
@Data
public class User {

    @Version
    private Long version; // 这里添加版本字段，并添加版本注解

    @Id
    private String id;

    private String userName; // 姓名
    private Integer age; // 年龄
    private String sex; // 性别

    public User(String userName,Integer age,String sex) {
        this.setUserName(userName);
        this.setSex(sex);
        this.setAge(age);
    }

    public User() { }
}

```

###### 测试
配置好版本号之后进行测试

``` java

    @Test
    public void optimisticLockTest() {

        User user = template.insert(new User("feng", 18, "female")); // 插入一条记录

        User one = template.findOne(new Query(new Criteria("id").is(user.getId())), User.class); // 将插入的记录查出来

        one.setUserName("管你啊"); // 给查出来的对象设置行名称

        template.save(one); // 保存查出来的对象

        template.save(user); // 这里再次保存之前的对象时会抛出 OptimisticLockingFailureException 异常

    }

```
乐观锁使用比较简单，但是也很大程度的避免了在更新操作时的加锁问题，但是也会存在脏读等问题，以上就是 mongo 使用乐观锁的一些具体操作，这些在官网例子中也有说明，这里附上[官网连接](https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/#mongo-template.optimistic-locking) 

开始写东西不久，在很多表达方面可能不是很到位，但是会坚持下去的，这里也算是对自己进行约束的一个地方。我一直是一个不敢表达自我的人，希望在这里用书写的方式先把自己的东西写出来，能在回顾的时候不断提高自己。good luck...

