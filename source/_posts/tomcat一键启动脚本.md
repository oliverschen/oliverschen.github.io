---
title: 启动tomcat shell脚本
date: 2020-02-24 21:46:07
tags: tomcat
category: tomcat
comments: true
---


![Photo by SamUerto on wallhaven.cc](/start-sh.png)


开发环境往往会部署很多服务在一台服务器上面，每次启动都要好几个命令才能启动 tomcat，很麻烦并且很浪费时间。所以写了一个简易脚本，用来在开发环境启动服务。不过这个脚本还有待完善，这个脚本需要手动把编译的包上传到服务器之后再指定重启，大量时间消耗在打包和上传文件的步骤中。可以考虑直接指定代码分支，直接在 GitLab 上拉取代码自动部署在服务器，这样会节省很多事情。当然也可以使用 Jenkins 等自动化部署工具来完成。

<!--more-->

##### 过程

1. 先判断启动参数是否正确
2. 获取到想要启动服务的 PID
3. 如果获取到了，说明改服务是启动状态，kill 进程之后重新启动。如果没有获取到，直接尝试启动服务

##### 脚本

> 很简单的一个小脚本

```bash

#! /bin/bash
param=$1
# 服务的绝对路径  
path=/usr/local/$param;
if [[ "$param" != "" ]]&&[[ $param == tomcat* ]];
then
    echo "服务名称："$param
    # 获取服务的 pid,grep -v grep 去除 grep pid,grep -v "start.sh" 去掉当前脚本的pid
    pid=$(ps -ef | grep $param | grep -v grep | grep -v "start.sh" | awk '{print $2}');
    echo "当前服务的 PID="$pid;
    if [[ "$pid" != "" ]];
    then
        kill -9 $pid;
        echo "杀掉进程执行结果："$?;
        if [ $? -eq 0 ];
        then
            echo "进程已结束";
            $path/bin/startup.sh;
            echo "服务正在重启...";
            tail -f $path/logs/catalina.out;
        else
            echo "重启失败";
        fi
    else
        echo "当前服务没有启动，尝试直接启动";
         $path/bin/startup.sh;
         echo "服务正在启动...";
         tail -f $path/logs/catalina.out;
    fi
else
    echo "请输入正确的服务名称";
fi
```
脚本很简单，有些地方也有说明，需要根据自己服务部署情况做一些相应的修改，就可以使用了。

##### 使用

如果想要启动 tomcat-xx 服务，则直接输入 `./start.sh tomcat-xx` 就可以重启服务。

##### 注意

脚本名称 `start.sh` 因为里面有个地方写死了脚本名称（这里可以优化下）。

***

<center>做一个温暖的人</center>