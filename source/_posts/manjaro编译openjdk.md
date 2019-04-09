---
title: manjaro编译openjdk
date: 2019-04-09 23:01:53
tags: openjdk
category: openjdk
---



刚接触到 Java 时就被一些陌生的英文缩写吓到了，什么 jdk,jre,jvm 等等。但是随着后面对它的了解越多，对这些基础的概念越来越清晰。 jdk 就是我们经常使用的开发工具包，它不仅包含 jre,还包含了编译器等其他基础包，而 jre 则是 Java 代码运行的最小环境。而 jvm 则是 Java 口号 “一处编译，到处运行” 的支撑者。从学习开始到现在一直都和它们间接接触，今天准备和它们亲密接触。

<!-- more -->

#### 下载源码

我下载的版本是 jdk8u60，官网![地址](http://hg.openjdk.java.net/jdk8u/jdk8u60/jdk/file/935758609767)，进去之后左侧有 zip 的包，直接下载。

#### 