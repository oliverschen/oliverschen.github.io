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
|boot jdk| 1.8.0_202|



#### 安装依赖

下载 mercurial 来克隆 jdk 代码

``` bash
sudo pacman -S mercurial

```

克隆代码到指定目录
``` bash
# 先进入到要放目标目录
sudo hg clone http://hg.openjdk.java.net/jdk8u/jdk8u/
```
当然也可以在官网下载代码，官网[地址](http://hg.openjdk.java.net/jdk8u/jdk8u/)在这里，进去之后左侧有 gz 的包，直接下载。我选择的是第一种方式，直接在仓库克隆代码到本地。

#### 编译

##### README
先看下 README 文件，这个里面已经有详细的说明
``` bash
cat ./README

README:
  This file should be located at the top of the OpenJDK Mercurial root
  repository. A full OpenJDK repository set (forest) should also include
  the following 6 nested repositories:
    "jdk", "hotspot", "langtools", "corba", "jaxws"  and "jaxp".

  The root repository can be obtained with something like:
    hg clone http://hg.openjdk.java.net/jdk8/jdk8 openjdk8
  
  You can run the get_source.sh script located in the root repository to get
  the other needed repositories:
    cd openjdk8 && sh ./get_source.sh

  People unfamiliar with Mercurial should read the first few chapters of
  the Mercurial book: http://hgbook.red-bean.com/read/

  See http://openjdk.java.net/ for more information about OpenJDK.

Simple Build Instructions:
  
  0. Get the necessary system software/packages installed on your system, see
     http://hg.openjdk.java.net/jdk8/jdk8/raw-file/tip/README-builds.html

  1. If you don't have a jdk7u7 or newer jdk, download and install it from
     http://java.sun.com/javase/downloads/index.jsp
     Add the /bin directory of this installation to your PATH environment
     variable.

  2. Configure the build:
       bash ./configure
  
  3. Build the OpenJDK:
       make all
     The resulting JDK image should be found in build/*/images/j2sdk-image

where make is GNU make 3.81 or newer, /usr/bin/make on Linux usually
is 3.81 or newer. Note that on Solaris, GNU make is called "gmake".

Complete details are available in the file:
     http://hg.openjdk.java.net/jdk8/jdk8/raw-file/tip/README-builds.html

```
那就按照 README 文件的步骤来执行编译

##### Getting the Source
``` bash
# 进入目录
cd jdk8u
# 执行脚本,拉取所有代码
bash ./get_source.sh

```
下载完成之后目录结构是这样
``` bash 
-rw-r--r--  1 root root   1522  4月 14 17:11 ASSEMBLY_EXCEPTION
drwxr-xr-x  6 root root   4096  4月 14 17:11 common
-rw-r--r--  1 root root   1588  4月 14 17:11 configure
drwxr-xr-x  6 root root   4096  4月 14 17:56 corba
-rw-r--r--  1 root root   3107  4月 14 17:11 get_source.sh
drwxr-xr-x  8 root root   4096  4月 14 18:06 hotspot
drwxr-xr-x  7 root root   4096  4月 14 17:15 jaxp
drwxr-xr-x  7 root root   4096  4月 14 17:15 jaxws
drwxr-xr-x  7 root root   4096  4月 14 18:46 jdk
drwxr-xr-x  7 root root   4096  4月 14 18:04 langtools
-rw-r--r--  1 root root  19274  4月 14 17:11 LICENSE
drwxr-xr-x  6 root root   4096  4月 14 17:11 make
-rw-r--r--  1 root root   6232  4月 14 17:11 Makefile
drwxr-xr-x 13 root root   4096  4月 14 17:27 nashorn
-rw-r--r--  1 root root   1549  4月 14 17:11 README
-rw-r--r--  1 root root 129333  4月 14 17:11 README-builds.html
drwxr-xr-x  3 root root   4096  4月 14 17:11 test
-rw-r--r--  1 root root 152511  4月 14 17:11 THIRD_PARTY_README

```
不过下载代码的时候我遇到了多次中断回滚，如果发生中断，会有警告在终端提醒， **上面的脚本可以重复拉取。如果有更新的话会自动拉取最新代码**。

##### Building
``` bash 
# 这一步会检查你的 /usr/bin 下的的命令是否齐全，如果缺少会抛错误出来，提示缺少某个命令，只需要安装再次执行就好
bash ./configure

# 如果有以下信息打印出来，说明第一步已经执行完成
A new configuration has been successfully created in
/usr/local/jdk8u/build/linux-x86_64-normal-server-release
using default settings.

Configuration summary:
* Debug level:    release
* JDK variant:    normal
* JVM variants:   server
* OpenJDK target: OS: linux, CPU architecture: x86, address length: 64

Tools summary:
* Boot JDK:       java version "1.8.0_202" Java(TM) SE Runtime Environment (build 1.8.0_202-b08) Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)  (at /usr/lib/jvm/default)
* Toolchain:      gcc (GNU Compiler Collection)
* C Compiler:     Version 8.2.1 (at /usr/bin/gcc)
* C++ Compiler:   Version 8.2.1 (at /usr/bin/g++)

Build performance summary:
* Cores to use:   7
* Memory limit:   7859 MB

```
编译

``` bash
make all
```

##### 完成

编译完成之后会打印下面的信息
``` bash
----- Build times -------
Start 2019-04-14 22:26:22
End   2019-04-14 22:36:39
00:00:18 corba
00:00:14 demos
00:02:26 docs
00:03:32 hotspot
00:00:18 images
00:00:13 jaxp
00:00:17 jaxws
00:02:16 jdk
00:00:25 langtools
00:00:17 nashorn
00:10:17 TOTAL
-------------------------
Finished building OpenJDK for target 'all'
```
编译时会出现各种问题，可能是依赖版本，或者是包下载不下来导致问题，还有写是因为配置文件设置的问题。下面是我碰到的几个问题

##### 设置环境变量

现在已经拥有一个自己的 jdk 了，设置环境变量使用起来。

先在 jdk8u/build/linux-x86_64-normal-server-release/images 下找到 j2sdk-image 目录，这个就是编译好的 jdk，把它复制到自己的 java 目录下，然后设置环境变量。 
``` bash
# 先卸载之前配置的 bootjdk 之后然后设置新的
# 设置环境变量
sudo vim /etc/profile
# JDK
export JAVA_HOME=/usr/local/java
export JRE_HOME=${JAVA_HOEM}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib 

```
重启配置文件
 source /etc/profile

现在 java 环境变量已经设置好了，输入 java -version 可以查看到环境下面信息，而且还有**自己的机器名**在上面，开心一分钟
``` bash
➜  ~ java -version
openjdk version "1.8.0-internal"
OpenJDK Runtime Environment (build 1.8.0-internal-chenkui_2019_04_14_22_21-b00)
OpenJDK 64-Bit Server VM (build 25.71-b00, mixed mode)
```

##### 问题

在编译时出现的 

```
Warning: ××× ‘int readdir_r(DIR*, dirent*, dirent**)’ is deprecated (declared at /usr/include/dirent.h:183) [-Wdeprecated-declarations] ×××
```
解决方案
``` bash
cd /usr/local/jdk8u/hotspot/make/linux/makefiles
vim ./gcc.make
# 编辑 gcc.make
# 找到 WARNINGS_ARE_ERRORS = -Werror 修改为
WARNINGS_ARE_ERRORS = -Wno-all

# WARNING_FLAGS = -Wno-deprecated-declarations -Wno-unused-parameter -Wno-sign-compare -Wno-error 修改为

WARNING_FLAGS = -w

```
参考[链接](https://blog.csdn.net/desiyonan/article/details/80802066) 这个问题是解决最没有头绪的一个，不过在编译的过程中要仔细看报错信息，这样方便 Google 时能准确的找到问题所在。现在就可以愉快的调试源码来玩一下了。


#### the end
记得第一次接触 Java 时，在命令行打印出 `hello world` 时真的很疑惑，就这么一个东西，能有什么用。但是随着一点一点的了解，慢慢发现它的强大之处。每天能进步一点，一年后看今天碰到的问题，其实也不难解决，不积跬步无以至千里，要加油啊！！！  ∩▂∩

官网提供的编译[文档](http://hg.openjdk.java.net/jdk8/jdk8/raw-file/tip/README-builds.html)

***

<center>我最爱海南，每个地方都有挥之不去的记忆</center>