---
title:
- CentOS7 搭建Java Web服务器笔记
date:
- 2017-01-19 17:53:01
tags:
- Linux
- CentOS
- 服务器
- Java
- Java Web
- nginx
- redis
- jenkins
- NodeJS
---
搭建Java服务器就是把JavaEE的Web部署环境和软件都安装好，这是我自己用旧的PC搭建的，所以只是单个服务器，不涉及分布式。这些安装步骤都可以单独找教程

## 0x00 前言

> + gcc gcc-c++
+ JDK
+ Redis
+ Nginx
+ MySQL
+ NodeJS
+ Jenkins

可能需要的附加软件
+ git
+ Python
+ Python pip
…
<!--more-->
## 0x01 JDK安装和配置
其实CentOS 7 自带Java7 为了适应lambda ，更新到Java8

+ 卸载之前的版本

```
[root@localhost ~]#  rpm -qa | grep java
javapackages-tools-3.4.1-6.el7_0.noarch
tzdata-java-2014i-1.el7.noarch
java-1.7.0-openjdk-headless-1.7.0.71-2.5.3.1.el7_0.x86_64
java-1.7.0-openjdk-1.7.0.71-2.5.3.1.el7_0.x86_64
python-javapackages-3.4.1-6.el7_0.noarch
```
根据列表
```
[root@localhost ~]# rpm -e --nodeps tzdata-java-2014i-1.el7.noarch
[root@localhost ~]# rpm -e --nodeps java-1.7.0-openjdk-headless-1.7.0.71-2.5.3.1.el7_0.x86_64
[root@localhost ~]# rpm -e --nodeps java-1.7.0-openjdk-1.7.0.71-2.5.3.1.el7_0.x86_64
```
+ 下载JDK包

[下载地址：http://www.oracle.com/technetwork/java/javase/downloads/index.html](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

因为Oracle搞了cookie验证，所以下载完使用ftp上传到CentOS上也可以，或者，wget带上Cookie
```
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u102-b14/jdk-8u102-linux-i586.rpm"
```

+ 安装rpm

```
rpm -ivh jdk-8u102-linux-i586.rpm
```

+ 配置JAVA_HOME

```
vi /etc/profile
# 最末尾添加
JAVA_HOME=/usr/local/jdk/jdk1.8.0_91
JRE_HOME=$JAVA_HOME/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib/dt.jar
export JAVA_HOME JRE_HOME PATH CLASSPATH
```
保存 退出vi
```shell
# 使环境变量生效
source /etc/profile
```

+ 验证安装成功

```
[root@localhost ~]# java -version
java version "1.8.0_92"
Java(TM) SE Runtime Environment (build 1.8.0_92-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.92-b14, mixed mode)
```

## 0x02 安装Redis
因为Redis不大，可以使用yum工具安装
+ yum 安装

```
# 系统应该安装了EPEL
sudo yum -y install redis
```
如果不行
+ 编译安装

[参考这篇文章：https://linux.cn/article-6719-1.html](https://linux.cn/article-6719-1.html)

+ Redis 相关操作

```
 # 启动服务：
 systemctl start redis.service
 # 停止服务：
systemctl stop redis.service
 # 重启服务：
systemctl restart redis.service
 ```
+ [Redis官网：https://redis.io/](https://redis.io/)

## 0x03 安装Nginx
+ yum 安装

如果不是网络被限制的话，建议用这个方法安装
```
sudo yum install nginx
```


+ 编译安装<br>

+ 检查依赖<br>

```
 rpm -q gcc
 rpm -q openssl
 rpm -q zlib
 rpm -q pcre
```

`package pcre is not installed`表示pcre未被安装，否则表示该项依赖已被安装<br>
+ 安装依赖<br>
选择没有安装的依赖，yum安装<br>

```
 # 建议使用root用户
yum install -y gcc gcc-c++
yum -y install openssl
yum -y install zlib
yum -y install pcre
```

+ 创建目录，安装Nginx<br>

```
     # 请使用root或管理员用户
     # 创建安装目录
     cd /
     mkdir appHome
     cd appHome
     # 下载Nginx，解压
     wget http://nginx.org/download/nginx-1.11.8.tar.gz
     tar zxvf nginx-1.9.15.tar.gz
     cd nginx-1.9.15
     # 编译安装
     ./configure
     make 
     make install
```

+ 启动Ngixn<br>
如果是yum安装

```
sudo systemctl start nginx
```

如果是编译安装

```
cd /usr/local/nginx/bin
sudo  ./nginx
```

记得打开端口

```
 # 查看端口是否开放
firewall-cmd --list-all
```

建议开放非80端口，修改一下Nginx配置，开80端口可能被查水表
```
 # 打开80端口
 firewall-cmd --zone=public --add-port=80/tcp --permanent
 firewall-cmd --zone=public --add-service=http --permanent
 firewall-cmd --reload
```

## 0x04 安装MySQL
这个老折腾了，CentOS 7 用MariaDB代替了MySQL，yum源里也没有MySQL，只能去Mysql官网去找源，这次用的是rpm安装，不过由于服务器在国外，我的小水管4kb/s 安装了10个小时，所以我弄了个新笔记
[CentOS 7 安装与卸载MySQL 5.7跳坑：http://www.jianshu.com/p/e54ff5283f18](http://www.jianshu.com/p/e54ff5283f18)

## 0x05 安装NodeJS
下载安装并配置
+ 下载解压
```
cd /appHome/
wget https://nodejs.org/dist/v6.9.4/node-v6.9.4-linux-x64.tar.xz
tar -zxvf node-v6.9.4-linux-x64.tar.xz
```
+ 安装
```
cd node-v6.9.4-linux-x64
./configure 
make 
make install 
```
+ 添加配置
```
vi /etc/profile
```
末尾添加
```
# NODE_HOME
NODE_HOME =/appHome/node-v6.9.4-linux-x64
 PATH=$PATH:$NODE_HOME/bin
export PATH NODE_HOME
```
保存，编译
```
source /etc/profile
```

## 0x06 安装Jenkins
超级简单
https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Red+Hat+distributions

我贴一下脚本
+ 安装

```
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
sudo yum install jenkins
```
+ 操作

```
# 启动
sudo service jenkins start
# 停止
sudo service jenkins stop
# 重启
sudo service jenkins restart
sudo chkconfig jenkins on
```

+ 开启端口访问

```
# 可访问端口列表
firewall-cmd --list-all
# 开启8080端口外网访问
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```

----
love&peace