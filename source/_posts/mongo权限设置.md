---
title: mongo权限设置.md
date: 2019-03-28 23:45:33
tags: mongo
category: mongo
photos: 
---

![](mongo权限设置/mongoPower.png)

mongodb 使用起来还是相对简单的，但是在权限设置时，遇到了几个小问题，以至于花了比较久的时间，在给 mongo 设置权限前首先要清楚几点：

<!-- more -->

1. mongo 默认是没有用户的。
2. mongo 每个数据库由独立的用户来管理。
因为 mongo 默认是没有用户的，所以首先要创建管理员用户，这里创建管理员用户时要进入 admin 数据库来设置


``` bash
# 进入 adimn 数据库
> use admin
switched to db admin
> db.createUser(
   {
     user: "admin", # admin 用户名
     pwd: "123456", # 密码
     roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] # admin 此用户作用的目标数据库
   }
);
# 此处创建成功 bash 会返回成功信息
# 验证权限
> db.auth("admin","123456");
1 # 成功会返回1
```
至此，mongo 管理员用户已经创建成功了，现在退出 mongo ,在配置文件中开启验证

``` conf
auth=ture
```
重启 mongo

创建 test 数据库，给此数据库创建 test 用户方能连接到此数据库

``` bash
# 有的话进入，没有则创建此库
> use test
> db.createUser(
   {
     user: "test",
     pwd: "123456",
     roles: [ { role: "readWrite", db: "test" } ]
   }
);

```
此时用户和关联的库已经创建好，在 robo3T 客户端就可以直接连接使用了。