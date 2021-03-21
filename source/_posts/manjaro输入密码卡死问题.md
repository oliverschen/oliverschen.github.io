---
title: Manjaro输入密码卡死问题
date: 2019-04-24 23:01:19
tags: Manjaro
category: Manjaro
comments: true
---

![年轻才不怕](manjaro输入密码卡死问题/密码卡死.png)


最近玩 manjaro 还是听开心的，该有的软件都有了，用起来也挺方便的。事故发生在一次更新之后，我第二天晚上打开电脑输入密码，卡死在输密码的界面...（嗯...小问题，开启重启。我太天真了，还是没有解决问题）因为最近比较忙，手头也没有多余的电脑，就搁置了几天。最近查了各种资料，问了一些大佬，终于解决了问题。

<!-- more -->

#### 环境

|    ev  |  version    |   
| ---- | ---- |    
|    os  |  Mangaro 18.0.4    |       
|    Kernel |    x86_64 Linux 4.19.34-1-MANJARO   |      
|DE| KDE 5.56.0 / Plasma 5.15.4 |
|GPU|Mesa DRI Intel(R) Haswell Mobile |

#### 问题定位

Google 了很多同学碰到的问题，说卡死是双显卡导致的，因为我也是双显卡，所以准备先在这里排除问题。首先 CTRL + ALT + F3 进入到 tty3 终端。使用 root 登录之后就有问题了，所有的命令都报错

``` bash
## 报错信息
# ll
-bash: 口口口口口
```
基本所有命令都不能使用，很难受。后面请教了一些大佬说应该是环境变量的问题，后面的`口口口口` 应该是中文不支持原因，实际应该是下面的情况

``` bash
## 报错信息
-bash: 未找到命令
```
#### 环境变量

上面的问题终于有了眉目，命令不存在应该是环境变量导致的，所以先要设置环境变量

``` bash
# 编辑环境变量
$ /usr/bin/vim /etc/profile
# 添加下面的值
export PATH=/usr/local/sbin:/usr/local/bin:/bin
# 重启
source /etc/profile
```

到这里所有的命令应该就可以使用了。

#### 更新显卡依赖

``` bash
pacman -S virtualgl lib32-virtualgl lib32-primus primus
```

更新完成之后

``` bash
reboot
```
重启之后，终于进去了...切还是那样熟悉 。

***
<center>白马非马</center>
