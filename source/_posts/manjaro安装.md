---
title: manjaro安装
date: 2019-03-17 17:52:15
category: OS
tags: manjaro
photos: http://pnvzvh2vw.bkt.clouddn.com//manjaro.png
---
### manjaro 配置和软件安装

一般做开发的话使用 Linux 是躲不开的，在学校的时候挺喜欢折腾，但也是局限于在虚拟机安装各种版本的 Linux 系统，但是有些 win 上必须应用软件在 Linux 找不到代替版，就一直只是当做服务器的角色，没有用在实际使用中，直到去年碰到 manjaro，就把常驻多年的 win 换成了 manjaro，使用起来很方便，应用软件基本都有了，安装过程也很简单，类 win 操作，这里也只是记录下配置和一些必要软件的安装命令，以防我哪天作死搞挂重装....o‿≖✧

> manjaro 安装比较简单，建议使用 [rufus](https://rufus.ie/)，还有 [manjaro](https://manjaro.org/) 官网地址。

#### 配置

**更新中国源**

```
sudo pacman-mirrors -i -c China -m rank
## 弹出后选择清华或者中科大源
```

**配置 AUR 源**

```
[archlinuxcn]
# SigLevel = Optional TrustedOnly
SigLevel = Never
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch

[arch4edu]
SigLevel = Never
Server = https://mirrors.tuna.tsinghua.edu.cn/arch4edu/$arch
```

**更新并选择最快的源列表**

```
sudo pacman-mirrors -g 
```

**更新数据源**

```
sudo pacman -Syy && sudo pacman -S archlinuxcn-keyring
```

#### 安装常用软件

**安装 yaourt**

```
sudo pacman -S yaourt
```
**安装 拼音**

```
##### 安装搜狗输入法
sudo pacman -S fcitx-im
sudo pacman -S fcitx-configtool
sudo pacman -S fcitx-sogoupinyin

添加输入法配置文件 sudo vim ~/.xprofile

export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"


如果安装失败的话执行下面的方式安装
sudo yaourt sogou
```

**安装 chrome**

```
sudo pacman -S google-chrome
```

**安装深度截图**

```
sudo pacman -S deepin-screenshot
```

**安装 TIM**

```
sudo pacman -S deepin-wine-tim
```

**安装网易云音乐**

```
sudo pacman -S netease-cloud-music
```

**安装 pdf 阅读器**

```
sudo yaourt foxit
```

**安装下载工具**

```
sudo yaourt -S uget 
```

**安装 zsh**

```
#最新版本已经默认安装了。
sudo pacman -S zsh
# 安装oh-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
# 更换默认的shell
chsh -s /bin/zsh
```
**安装 wechat**

```
sudo pacman -S electronic-wechat	
```

**安装 git**

```
sudo pacman -S git
```

**安装 wps**

```
sudo pacman -S wps-office

```

**安装护眼软件**

```
sudo pacman -S xflux-gui-git
```

**安装 vscode**

```
sudo pacman -S visual-studio-code-bin
```

**安装 JDK**

```
sudo pacman -S jdk8
### 配置环境变量
export JAVA_HOME=/usr/lib/jvm/default
export JRE_HOME=${JAVA_HOEM}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib 
```

**安装　idea**

```
在官网下载安装包解压
tar　-zxcf　ideaxxx.tar.gz
进到　./bin　目录执行文件
./idea.sh
完成安装
```

**安装 svn 客户端 kdesvn**
```
sudo pacman -S kdesvn
```
**安装 mongodb**
``` bash
$ yay -S mongodb
## 配置
$ sudo vim /etc/mongodb.conf # 编辑 mongodb 数据库路径

sudo chmod  u=rw /home/mj/mongodb  #  设置读写权限
sudo chown -R mongodb:  /home/mj/mongodb  # 更改用户

```
**安装 docker**
``` bash
sudo pacman -S docker

sudo systemctl start docker.service # 启动服务 

sudo systemctl enable docker.service # 设置 docker 自启动

```


**安装 ss**

```
sudo pacman -S shadowsocks-qt5
```

**配置 switchyomega**

```
### 下载
https://github.com/FelisCatus/SwitchyOmega/releases 
配置方式网上有很多博客

```

**pacman 命令**

```
pacman -S  软件名   #安装
pacman -Syu    #更新
pacman -R 软件名    #移除
```

#### other

**设置　ll 命令**

``` bash
编辑　~/.bashrc
$ vim ~/.bashrc
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias ll='ls -i'
保存生效　
$ source ~/.bashrc
```

**设置主目录为英文**

``` bash
$ sudo pacman -S xdg-user-dirs-gtk
$ export LANG=en_US
$ xdg-user-dirs-gtk-update
$ #然后会有个窗口提示语言更改，更新名称即可
$ export LANG=zh_CN.UTF-8
$ #然后重启电脑如果提示语言更改，保留旧的名称即可
```

补充 ing...