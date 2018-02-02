---
title:
- 在CentOS 7上安装 Nginx
date:
- 2016-6-11 12:06:51
tags:
- Linux
- CentOS
- 博客搭建
- Nginx
---
### 1.准备
+ 安装PCRE

pcre是正则兼容包，可以让Nginx 支持Http Rewrite模块
 CentOS 安装方法
 ```
yum install pcre pcre-devl -y
```
+ 安装openSSL

支持HTTPS
CentOS 安装方法
```
 yum install openssl opnessl-devel -y
```

### 2.编译安装

创建用户
```
 useradd -s /sbin/nologin -M nginx
```
下载Nginx 解压
<!-- more -->

```
wget http://nginx.org/download/nginx-1.10.0.tar.gz
tar -zxvf nginx-1.10.0.tar.gz
```

执行配置
```
./configure --user=nginx --group=nginx --prefix=/application/nginx1.10.0 --with-http_stub_status_module --with-http_ssl_module
```
`--user=nginx`指定用户
`--group=nginx`指定组
`--prifix=/application/nginx1.10.0`指定安装路径
`--with-http_stub_status_module`指定正态模块
` --with-http_ssl_module`指定ssl模块

> 遇到坑
```
checking for OS
 + Linux 3.10.0-327.10.1.el7.x86_64 x86_64
checking for C compiler ... not found
./configure: error: C compiler cc is not found
```
解决：安装gcc，一个C编译器
```
yum -y install make gcc gcc-c++ ncurses-devel 
```

编译安装
```
make install
```

创建软连接
```
ln -s /application/nginx1.10.0 /usr/local/nginx
```

启动Nginx
```
cd /usr/local/nginx
./nginx
```

验证
```
curl 0.0.0.0:80
```

### 排错
+ 1.ping

ping+ip 如果不通，说明物理连接不通

+ 2.telnet

telnet+ip+端口，如果不通，说明服务器和本机不通，看看端口是不是被限制

+ 3.本机curl

curl +localhost ，如果不通，说明服务异常，需要重启。

### 部分坑

+ 查看Nginx是否监听80端口

```
netstat -lntup|grep nginx
```
+ 查看80端口被哪个程序占用

```
lsof -i :80
```
+ 解决80端口被关闭的问题

```
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
```
