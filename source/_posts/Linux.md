---
title: Linux-常用命令
date: 2020-01-01 23:17:14
tags: Linux
category: Linux
comments: true
---

![Photo by kejsirajbek on wallhaven.cc](/linux.png)


Linux 是服务端最常用的操作系统，也是范围最广的，都是通过命令来交互的，记录些常用命令，这里会一直追加，年底了，回家的诱惑越来越强。
<!--more-->

#### 基础命令


1. 远程复制

 **scp**

```bash
# 复制文件到远程
scp 文件 192.168.31.107:/usr/local/
```

静默复制目录到远程主机

```bash
# -r 表示目录，-q 静默复制
scp -rq /目录名 192.168.31.107:/usr/local/
```

2. 权限

说明

> 使用命令 ll 就可以清晰的查看 Linux 文件目录权限。

![文件目录权限](/Linux-权限.png)

具体的命令执行后：
```bash
drwxr-xr-x. 2 root root  4096 May 11  2019 bin
drwxr-xr-x. 2 root root  4096 May 11  2019 etc
drwxr-xr-x. 2 root root  4096 May 11  2019 games
-rw-r--r--. 1 root root 13871 Dec 16 11:38 jihe.sh
drwxr-xr-x. 2 root root  4096 May 11  2019 include
drwxr-xr-x. 2 root root  4096 May 11  2019 lib
```
比较有意思的是，以上 `rwx` 表示的权限也可以用数字来描述：
r=4,w=2,x=1；如果想设置可读可执行：r + x = 5


设置权限

**chmod**

```bash
# 给当前用户设置可执行权限
chmod +x jihe.sh
# 给所有用户最大权限（可读可写可执行）
chmod 777 jihe.sh
# 递归添加权限
chmod -R 777 bin

```

2. 查看文件

**cat**

```bash
# 文件合并 - 把 file1 和 file2 合并输出到 file3
cat file1 file2 > file3
# 显示行号
cat -b file1

```

**more**

```bash
# 分页显示大文件 - 空格下一页
more jihe.sh

```

3. 解压

**tar**

```bash
# 压缩 - 把目录下所有 txt 文件打包
tar -zcvf jihe.tar.gz *.txt
# 解压
tar -zxvf jihe.tar.gz
```
参数说明

> -z 是否使用 gzip 解压缩
> -c 创建压缩文件（create）
> -x 解压压缩文件
> -v 压缩过程显示文件
> -f 使用档案名，此参数为最后一个参数，后面只能接档案名

4. 文件大小

**du**

```bash
# 全部目录及其子目录每个档案所占磁盘空间
du -a 
# 全部目录及其子目录所占磁盘空间
du -h 
# 对应目录及其子目录所占空间
du -ch 目录
# 总大小
du -sh
```

5. 管道

**|**

管道命令使用 `|` 作为界定符号，管道命令需要结合其他命令一块使用。

```bash
# 输出 jihe.txt 中包含 abc 的行。（区分大小写，包含空格必须加引号 "a bc"）
cat jihe.txt | grep abc
# 输出 jihe.txt 文件中包含 abc 的行（忽略大小写）
cat jihe.txt | grep -i abc
# 输出 jihe.txt 文件中不包含 abc 的行
cat jihe.txt | grep -v abc

```

6. 查看

**which**

```bash
# 查找 $PATH 设置命令及安装文件目录所在位置
which ll

```

7. 输出及显示

**echo**
输出内容
```bash 
# 输出文本，一般有特殊字符建议加双引号
echo "hello world"
# 输出环境变量
echo $JAVA_HOME

```
**export**

设置或者显示环境变量
```bash
# 列出当前黄静变量值
export -p
# 设置临时环境变量
export  变量（$JAVA_HOME）

```

8. yum
应用包管理，类似安卓平台上的安卓应用市场，iOS 平台 AppStore 等应用中心

```bash
# 安装软件 -y 忽略在安装中确认的操作，直接默认 yes
yum install `-y` 包名
# 升级
yum update 报名
# 删除
yum remove 报名
# 清除缓存
yum clean all

```

9. 操作历史

**history**
保留了最近执行的命令记录，默认保留 1000 条
历史清单从 0 开始编号到最大值
```bash
# 显示最近 n 条命令
history 10
# 清除所有的历史记录
history -c
# 保存历史记录到文本
history -w jihe.txt
```


#### 高级功能

1. 系统状态

**ps**

查看处于活动状态的服务进程
```bash
ps -ef | grep 进程名[tomcat]

# 查看端口
ss -ntl
```

**netstat**

查看端口号等信息

```bash
netstat -anp | grep 端口号[8080]
```

**top**
动态显示当前系统运行情况，类似 windows 中任务管理器
```bash
# 显示命令执行的全路径
top -c 
# 指定用户名
top -u
# 指定进程
top -p 
# 按进程的 CPU 使用率排序
top P
# 按进程的内存使用率排序
top M

```

**df**
显示磁盘使用率

```bash
df -h
```

**who**
显示当前登录用户
```bash
who

```

**uname**
查看系统信息
```bash
uname -a
```

**free**
查看内存和交换空间使用情况
```bash
# 单个命令显示的是字节，以 M/G 为单位显示
free -m/g

```

2. 关机重启

**reboot**
重启

```bash
# 立刻重启
reboot -h now 
```

**poweroff**

关机
```bash
poweroff
```

3. 查看系统

**lscpu**
```bash
# 查看系统 cpu 信息
lscpu
```

***

<center>越来越好</center>




