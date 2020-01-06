---
title: Linux-shell
date: 2020-01-03 00:15:01
tags: Linux
category: Linux
---

![Photo by deleted on wallhaven.cc](/linux-shell.png)


shell 是用户和 Linux 系统交互的一个应用程序，就像 Windows 操作系统可以用过界面进行交互。shell 脚本就是通过 shell 来执行的，就像 Java 程序执行在 JVM 上类似。类似 Java 虚拟机，Linux 下也有很多的 shell 程序，但是最常用的就是 bash，因为其使用简单，并且是免费的，所以是 Linux 的默认 shell。

<!--more-->

#### shell script

##### hello world

任何语言都离不开一个 hello world ，现在感受下 shell 的第一个脚本。
```bash
#!/bin/bash
echo "hello world"
```
一般 shell 脚本命名都是以 .sh 来写的，这是一个约定俗成的写法，这样别人在看到 .sh 结尾的文件就知道是一个 shell 脚本，当然想以其他结尾也是可以的。

> #!
> 这个是告诉系统这个脚本使用哪个 shell 来执行。
> echo
> 向窗口输出文本

##### 变量

规则
> 1. 只能使用数字，字母，下划线，且不能以数字开头
> 2. 变量名区分大小写
> 3. 变量赋值通过（=）号，且不能有空格

使用
```bash
[root@jihe shell]# a=1
[root@jihe shell]# echo $a
1
[root@jihe shell]# echo ${a}
1
[root@jihe shell]# echo $ab

[root@jihe shell]# echo ${a}b
1b
```

###### 特殊变量

**$?**

接收上一天命令的返回状态码，上条命令执行成功是 0 ，失败为其他 1-255 之间的值。

**$#**

脚本执行的参数个数

```bash
#!/bin/bash
$1
$2
$#
```

`$*`

所有参数

**$$**

获取当前脚本的进程号，可以实现脚本自杀，或者使用 exit 退出。

#### 循环和判断

##### 循环

shell 中也是 for 循环，但是格式区别于 Java 中的 for 循环，而且 shell 中 for 循环有多种写法，下面是比较容易记住的写法。

打印 0 到 9
```bash
#!/bin/bash
for((i=0;i<10;1++))
do
echo "i="$i
done
```
这样程序会打印 `i=0` 到  `i=9` 输出到窗口。`do` 作为循环体的开始，`done` 作为循环体的结束。

##### 比较

test EXPR
[ EXPR ] : 中括号和表达式之间有空格
###### 整型

`-gt` : 大于， [ $1 -gt $2 ] 或者 test $1 -gt $2 判断变量 1 大于 变量 2
`-lt`：小于
`-ge`：大于等于
`-le`：小于等于
`-eq`：等于
`-ne`：不等于

###### 字符串

`=`：等于，判断变量是否为空 [ "$str" = "" ] 或者 [ -z $str ]
`!=`：不等于


##### 判断

类似 Java 中的 if，而且是以 fi 作为判断的结尾

if [condition 1]
then
    first tree
elif [conditin 2]
    second tree
else 
    third tree
fi

```bash
#!/bin/bash
num=$1
if [ $num -eq 1 ]
then
        echo one
elif [ $num -eq 2 ]
then
        echo two
else
        echo none
fi
```

执行结果
```bash
[root@jihe shell]# ./if.sh 1
one
[root@jihe shell]# ./if.sh 2
two
[root@jihe shell]# ./if.sh 3
none
```


#### 表达式

`${}`：获取变量的值
`$()`：等于 ``，会执行里面的命令，并且获取命令执行的结果
`$[]`：可以对方括号里面的公式进行算术运算
`$(())`：可以对双括号里面的公式进行算数运算


#### 日期

##### date

显示当前时间
```bash
date
```

##### 格式化

```bash
[root@jihe shell]# date +%Y-%m-%d
2020-01-05
```
##### 获取时间戳

```bash
[root@jihe shell]# date +%s
1578188573
```

##### 指定时间

```bash
# 获取前一天时间，按照指定格式输出
[root@jihe shell]# date --date='1 days ago' +%Y-%m-%d
2020-01-04
# 获取后一天时间，按照指定格式输出
[root@jihe shell]# date --date='-1 days ago' +%Y-%m-%d
2020-01-06
# 获取指定时间的前一天
[root@jihe shell]# date --date='20191111 1 days ago' +%Y-%m-%d
2019-11-10
```

#### 后台执行

在后台执行脚本

##### nohup
不挂断的运行命令，忽略所有挂断（hangup）信号
```bash
nohup for.sh &
```
nohup 会忽略进程的挂断信号，也会在当前执行目录生成一个 nohup.out 日志文件。


#### 标准输入，输出，重定向

标准输入，输出，错误可以使用文件描述符 0，1，2 引用

##### 重定向
```bash
# > 将输出结果重定向输出到 file 文件中
# 1> 将输出的正确结果重定向到 file 文件中
# >> 将输出的结果追加到 file 文件中，不做覆盖处理
ls >file 或者 ls 1>file (ls >>file)

# 2> 将错误输出到 file 文件中，haha 是一个错误命令
haha 2>file 
```
特殊

```bash
# 把输出信息直接丢失，既不保存也不打印到控制台
ls >/dev/null
```

#### 定时器

Linux 下有专门执行定时任务的配置
```bash
vim /etc/crontab

# 下面是 crontab 文件的内容
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

# 在这里追加定时任务，都为 * 时为 1 分钟执行一次。root 为执行用户，后面跟脚本绝对路径。
* * * * * root /usr/local/shell/crontab.sh

```
crontab.sh
脚本将文本内容输出到指定文件 cron.txt。
```bash
#!/bin/bash
echo "hello world" >>/usr/local/shell/cron.txt
```
查看 cron.txt 文件
```bash
# 实时输出内容
tail -f cron.txt
```


***

<center>美好的事即将发生</center>






