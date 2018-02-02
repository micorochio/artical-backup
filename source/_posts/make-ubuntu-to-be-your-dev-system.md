---
title:
- 使用Ubuntu快速搭建Java Web开发环境
date:
- 2017-05-06 21:06:11
tags:
- Linux
- Ubuntu
- Java
---
## 0x00 从Windows安装
我是Mac用户，新公司用PC，为了适应，决定用Linux做主力开发系统，首先需要安装系统

有两种方式：
> 1. 使用EasyBCD 添加引导
2. 使用UltraISO制作安装U盘

EasyBCD安装的坑：
1. 需要自己写ISO引导位置
```sh
title Install Ubuntu 16.04
root (hd0,1)
kernel (hd0,1)/vmlinuz.efi boot=casper iso-scan/filename=/ubuntu-16.04-desktop-amd64.iso ro quiet 
splash locale=zh_CN.UTF-8
initrd (hd0,1)/initrd.lz
```
hd(0,1)表示第一个物理分区的第二个盘，当然不一定是C盘。所以ISO文件的位置一定会坑到一堆人。
其实可以安装时更改hd位置，参考：http://www.jianshu.com/p/9b4e4137bc11

2. 要将ISO文件解压，将其中 .disk 文件夹，initrd.lz，vmlinuz.efi 文件放到C盘中，不能少。

UltraISO安装的坑
UltraISO版本一定要新版的，我用的是UltraISO 9，不然U盘启动会报错，老毛桃的刻录工具版本太低。建议自己下载一个。
<!--more-->

## 0x01安装配置
Ubuntu 安装过程中一般情况不用特殊配置，不过我建议修改一下分区，因为是图形化界面，使用`/home`目录较多，建议使用分区大一些。
大致如下：
```
分区：swap    格式：swap    大小：与内存一样大小
分区：/boot   格式：xfs     大小：500M   （启动区以前大约40M，且不能随便更改）
分区：/       格式：xfs     大小：剩下空间的一半
分区：/home   格式：xfs     大小：剩下的空间（如果没有特殊要求）
```
其实这只是我单方面建议，因为home太小，系统被我玩的卡的不行，尤其VBOX装虚拟机 大约35G，瞬间占满home

所以，要装虚拟机，建议home分大一些。，其实这些也不是绝对
在CentOS中，只有`/boot`和`/`两个物理分区。至于分区做什么用的，可以参考Linux 资料

如果不会分区，大佬的建议是，除了`swap`和`/boot` 其余空间全部分给`/`

## 0x02 环境变量的配置
这个很重要，也很简单，只要记住三点
1. vim ~/.profile
2. 写准软件bin文件夹的位置，（不一定是bin文件夹，只要是存放可执行文件的文件夹都可以）
3. 保存好文件，然后一点要运行`source ~/.profile`才能生效

这里以Maven，和JDK举例
```sh
vim ~/.profile
```
将jdk文件的位置和maven的位置准确的写进去
```sh
#JAVA_HOME
export JAVA_HOME=/usr/lib/jdk/jdk1.8
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH  
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH 

#MAVEN_HOME
export MAVEN_HOME=/usr/local/maven/apache-maven-3.3.9
export MAVEN_OPTS="-Xms256m -Xmx512m"
export PATH=${MAVEN_HOME}/bin:$PATH 
```
写完 按`esc` 输入`:wq`保存退出
执行source命令，使环境变量生效
```sh
source ~/.profile
```

> **记住路径一定要写对，不然环境变量配了也白配。**

## 0x03 IDE和其他软件的安装
JDK安装我就不提了，教程特别多，只要解压后配置好环境变量就能使用了。

1.eclipse
即下即用，想方便可以再运行后，将应用绑定到dock上

2.IDEA
也是即下即用，不过可以将IDEA 文件夹下bin目录中 idea.sh 使用ln -s 命令链接到桌面上，或者配置IDEA_HOME。这样找运行文件会快一些。

3.Node
这个可以用apt-get命令安装，这样会方便很多，编译安装需要安装gcc编译器，另外
然后执行configure 
```bash
# prefix 参数是配置软件安装的位置
./configure  --prefix=/usr/local/node
make
sudo make install
```
4.Python
建议apt-get安装

5.Git
一样建议apt-get安装

6.Nginx
建议编译安装，步骤自己查，我只提醒一下，记住先安装依赖，zlib，pcre，openssl。

7.虚拟机安装
因为Wine 特别难用，所以需要安装个虚拟机，跑个QQ和微信什么的，这里推荐使用VisualBox，开源免费。
Ubuntu 16.04 apt源中有，如果想尝试deb安装的话，可以先apt-get 安装deb工具
安装完成后，可以使用 win+tab 搜索 virtual box，就能找到。
我用deb安装一直失败，使用后来换apt-get安装了。

8.搭建梯子
这个有太多教程，我建议看看秋水逸冰的博客：
https://shadowsocks.be/9.html
如果用docker搭建梯子，可以看看mritd现有的docker镜像
https://hub.docker.com/r/mritd/shadowsocks/


## 0x04 OpenVPN的使用
也许公司代码在内网中，
http://blog.csdn.net/w746805370/article/details/51774609
知乎大神的原文被知乎河蟹了，这里包括如何搭建Server 和运行Client，自行取用。

这里只写一个Client链接VPN命令。
找服务端拿到client.ovpn 文件，注意位置，可能跟我的不一样。
```sh
openvpn /etc/openvpn/client.ovpn
```
如果你不想一直开着命令窗口，可以用下面的命令
```bash
nohub openvpn /etc/openvpn/client.ovpn &
```

## 最后补充
MySQL安装我没写，建议用docker 跑MySQL，我也才开始看docker，现在只会跑个mysql，还是没问题的。国外镜像需要良好的梯子。

UI美化我也没写，你可以自己搜索unity-tweak-tool ，然后自行玩耍。

![奏是姐样的](http://upload-images.jianshu.io/upload_images/1112615-0579862bb72c3641.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

----
其他的坑，以后遇到就给补上。
爱转不转吧
作者[zing](https://micorochio.github.io/2017/05/07/make-ubuntu-to-be-your-dev-system/)