---
title:
- (笔记备份)CentOS 7 安装与卸载 MySQL 5.7
date:
- 2017-01-15 23:13:05
tags:
- Linux
- CentOS
- mysql
---
还挺麻烦的，老折腾了，CentOS 7 用MariaDB代替了MySQL，
yum源里也没有MySQL，只能去Mysql官网去找源，这次用的是rpm安装，
不过由于服务器在国外，我的小水管4kb/s 安装了十几个小时。
## 0x00 先介绍卸载

防止重装<br>

+ yum方式

```
 #查看yum是否安装过mysql
yum list installed mysql*
```

如或显示了列表，说明系统中有MySQL
![](http://upload-images.jianshu.io/upload_images/1112615-bba6b2eec0c79a1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!--more-->
+ yum卸载

根据列表上的名字
```
yum remove mysql-community-client mysql-community-common mysql-community-libs mysql-community-libs-compat mysql-community-server mysql57-community-release
rm -rf /var/lib/mysql  
rm /etc/my.cnf  
```
+ rpm查看安装

```
 rpm -qa | grep -i mysql  
```
![](http://upload-images.jianshu.io/upload_images/1112615-2a22af8a7764a6e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ rpm 卸载

```
rpm -e mysql57-community-release-el7-9.noarch
rpm -e mysql-community-server-5.7.17-1.el7.x86_64
rpm -e mysql-community-libs-5.7.17-1.el7.x86_64
rpm -e mysql-community-libs-compat-5.7.17-1.el7.x86_64
rpm -e mysql-community-common-5.7.17-1.el7.x86_64
rpm -e mysql-community-client-5.7.17-1.el7.x86_64
cd /var/lib/  
rm -rf mysql/  
```

+ 清除余项

```
whereis mysql
mysql: /usr/bin/mysql /usr/lib64/mysql /usr/local/mysql /usr/share/mysql /usr/share/man/man1/mysql.1.gz
 # 删除上面的文件夹
rm -rf /usr/bin/mysql
…
…
```

我就省略了

+ 删除配置



```
rm –rf /usr/my.cnf
rm -rf /root/.mysql_sercret

```

剩余配置检查
<br>
```
chkconfig --list | grep -i mysql
chkconfig --del mysqld
```
根据上面的列表，删除 ,如：mysqld

## 0x01 再介绍安装

+ 注意

*yum源，阿里的CentOS7.repo是没有的，国外源相当慢，做好心理准备。*
+ 下载地址

http://dev.mysql.com/downloads/
![下载页面](http://upload-images.jianshu.io/upload_images/1112615-2f064863299d04d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按照官方的文档进行安装
http://dev.mysql.com/doc/refman/5.7/en/installing.html
![http://dev.mysql.com/doc/refman/5.7/en/linux-installation.html](http://upload-images.jianshu.io/upload_images/1112615-917573d14e0e88cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
文档地址：http://dev.mysql.com/doc/refman/5.7/en/linux-installation.html
3是各种安装方式列表
CentOS用yum安装相对省事，省去很多配置环节

yum安装，先要搞到源
```
wget http://repo.mysql.com/mysql57-community-release-el7-9.noarch.rpm
sudo rpm -ivh mysql57-community-release-el7-9.noarch.rpm
```
接下来使用yum安装

```
  # 更新yum软件包
yum check-update  
  # 更新系统 
yum update
  # 安装mysql
yum install mysql mysql-server
```
接下来是漫长的等待。如果中途关机，或者下载挂了，请执行卸载步骤后，再来一次。

完成后

记住要给root上密码
```
/usr/local/mysql/bin/mysqld_safe --skip-grant-tables --user=mysql &
systemctl start mysqld
mysql -u root
mysql> update mysql.user set authentication_string=password('new_password') where user='root' and Host ='localhost';
mysql> flush privileges;
mysql> quit;
```

+  启动与开放远程访问

```
systemctl start mysqld
mysql -u root -p
 # 授权远程访问
mysql> use mysql;
mysql> rant all privileges  on *.* to root@'%' identified by "root";
mysql> FLUSH RIVILEGES;
```
建议root不要授权远程访问，请创建新mysql用户


## 0x02 编译安装
这个略坑，我按照官方文档安装，安好了不会配置，唉，吐槽自己太菜！导致没启动成功，后来换成了yum安装
http://dev.mysql.com/doc/refman/5.7/en/installing-source-distribution.html
我还是要把脚本贴出来
```
 # 添加mysql用户
shell> groupadd mysql
shell> useradd -r -g mysql -s /bin/false mysql
shell> rpmbuild --rebuild --clean MySQL-VERSION.src.rpm
 # 源码编译安装
shell> tar zxvf mysql-VERSION.tar.gz
shell> cd mysql-VERSION
shell> mkdir build
shell> cd build
shell> cmake ..
shell> make
shell> make install
 # 结束 source-build specific instructions
 # 权限步骤
shell> cd /usr/local/mysql
shell> chown -R mysql .
shell> chgrp -R mysql .
shell> bin/mysql_install_db --user=mysql    # MySQL 5.7.6执行
shell> bin/mysqld --initialize --user=mysql # MySQL 5.7.6 更高版本执行
shell> bin/mysql_ssl_rsa_setup              # MySQL 5.7.6 更高版本执行
shell> chown -R root .
shell> chown -R mysql data
shell> bin/mysqld_safe --user=mysql &
 # 配置命令
shell> cp support-files/mysql.server /etc/init.d/mysql.server
```
大致是以上的安装脚本，官网上有详细解释每一条的作用。可以参照一下。如果安装失败，可以参照最上面的卸载教程。
祝你好运。

## 0x03 参考
http://dev.mysql.com/doc/
http://blog.csdn.net/typa01_kk/article/details/49057073

----
*如果有错别字或者其他错误，请告知我修改。
转载请务必注明出处！