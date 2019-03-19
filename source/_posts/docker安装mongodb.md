---
title: docker安装mongodb
date: 2019-03-19 23:19:44
tags: docker,mongodb
category: docker,mongodb
photos:  http://pnvzvh2vw.bkt.clouddn.com//mongodb.png
---

mongodb 是一款很优秀的开源 nosql 数据库，它内部以 json 作为存储格式，在数据存储方面有很大的收缩性。因为前几天折腾 docker，这里记录一下在 docker 部署 mongdb 的步骤和简单的使用方法，其实之前接触 nosql 最多的就是 redis，目前公司大量使用 mongodb,在这里记录一些关键点，方便回顾。

|    ev  |  version    |   
| ---- | ---- |  
|   docker   |  18.09.3-ce    |       
|    os  |  Mangaro 18.0.1    |       
|    mongodb  |    v4.0.6  |       

关于 docker 的安装，如果是 Manjaro 的话，请在[这里](https://fengzhu.top/2019/03/17/manjaro%E5%AE%89%E8%A3%85/#more)参考，如果是其他系统的话， Google 应该有很多好的博客可以参考。
#### 安装 mongodb

``` bash
sudo docker pull mongo
```

等待下载完成之后，查看 docker 容器中镜像情况

``` bash
sudo docker pull mongo
```
等待 mongo 下载完成之后，进行配置

``` bash
sudo docker run -d -p 27017:27017 -v mongo_configdb:/data/configdb -v mongo_db:/data/db --name mongo docker.io/mongo --auth
# --auth 有此参数的话，在连接时需要用户名密码
# 增加用户名和密码
sudo docker exec -it mongo mongo admin
# 在 shell 中创建用户和密码
db.createUser({ user: 'root', pwd: 'root', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });

```
启动成功之后查看 mongo 运行情况

``` bash
sudo docker ps -a
```

到这里，mongodb 在 docker 中的安装就完成了，建议安装 mongodb 客户端[robo3T](https://robomongo.org/)可以更直观的看到数据的格式和存储的方式。我有个习惯就是在接触到一些新技术时先不会去纠结它的一些具体细节，我先会做个简单的例子自己跑一边，看一遍到底是怎么回事，和自己已经会的东西相比，有什么异同，这样的话会比较容易上手，当然这只是个建议，每个人都有自己的方式去学习新东西。下面我会按照我的思路来看一遍 mongo 的操作过程。

#### springboot 集成 mongodb

这里使用 spring boot 是因为用它的话能最快的搭建一个框架，来使用 mongo，至于 spring boot 工程如何搭建 Google 上有很多教程可以参考。主要引入 mongodb 坐标。
``` xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
```
#### CRUD

mongodb 不像关系型数据库，对于字段的完整性要求很高，通俗点讲的话它的存储结构完全自由，只要是格式正确的数据都能存储在里面，但是在实际开发中，每个数据库中表的数据会是一类数据，比如说 User 表中就是存储用户信息，不可能把订单信息也放在里面，这样就会造成数据的混乱，在实际开发中，每个表也都会有自己对应的 Javabean，它的字段也取决于这个 Javabean。

##### 增


