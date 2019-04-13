---
title: manjaro编译openjdk
date: 2019-04-09 23:01:53
tags: openjdk
category: openjdk
---

![](manjaro编译openjdk/openjdk.png)

刚接触到 Java 时就被一些陌生的英文缩写吓到了，什么 jdk,jre,jvm 等等。但是随着后面对它的了解越多，对这些基础的概念越来越清晰，对这些概念也有了一些认识。 jdk 就是我们经常使用的开发工具包，它不仅包含 jre,还包含了编译器等其他基础包，而 jre 则是 Java 代码运行的最小环境。而 jvm 则是 Java 虚拟机， “一处编译，到处运行” 就是因为这位大佬的存在。从学习开始到现在一直都和它们间接接触，但是对它们的了解还是有很大的局限，今天拉了 openjdk 的源码准备好好研究一下。

<!-- more -->
#### 环境

|    ev  |  version    |   
| ---- | ---- |    
|    os  |  Mangaro 18.0.1    |       
|    openjdk  |    jdk8u  |      


#### 安装依赖

下载 mercurial

``` bash
sudo pacman -S mercurial

```

克隆代码到指定目录
``` bash
sudo hg clone http://hg.openjdk.java.net/jdk8u/jdk8u  /home/chenkui/Documents/openjdk8u
```
当然也可以在官网下载代码，官网![地址](http://hg.openjdk.java.net/jdk8u/jdk8u/)在这里，进去之后左侧有 gz 的包，直接下载。

解压

``` bash
tar zxvf ./jdk8u
```
#### 