---
title:
- Mac系统安装mysql填坑
date:
- 2016-03-25 22:17:53
tags:
- mysql
- Java
- Web
- macOS Sierra
- OS X
---
先去下载mysql-5.7.9-osx10.9-x86\_64.dmg
安装（一直下一步，输入密码即可）mysql-5.7.9-osx10.9-x86\_64.pkg好了，启动MySQL服务.
![Untitled.png](https://upload-images.jianshu.io/upload_images/1112615-e1162b756dc419ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
坑开始了
+ mysql指令不识别
```
 $ mysql
 -bash: mysql: command not found
```
+ root用户初始密码不给力
```
Access denied for user 'root'@'localhost' (using password: YES)
```
+ 链接不上mysql
```
Access denied for user 'root'@'localhost' (using password: NO)
```

+ 你以为OK了？错！
```
You must reset your password using ALTER USER statement before executing this statement.
```

----
<!-- more -->

针对
```
mysql: command not found
```
这个最简单了，你要不会的话，唉，我还是告诉你吧,打开终端工具
输入命令
```
$ ln -s /usr/local/mysql/bin/mysql /usr/bin
```
假如你人品不好，被打脸了,提示你权限不够：
```
ln: /usr/bin/mysql: Operation not permitted
```
不要紧，我们把权限升高点
```
$ sudo ln -s /usr/local/mysql/bin/mysql /usr/bin
```
然后输入你的密码，要是没有的话，唉，你还是不要当程序员了，一点安全意识都没有要是上帝抛弃你了，sudo执行还是不可以
```
ln: /usr/bin/mysql: Operation not permitted
```
还报楼上的错，靠，真是比了狗的！！！别慌，先找个临时解决的办法，这个大招只能在当前窗口下放，记住哦！
```
$ alias mysql=/usr/local/mysql/bin/mysql
```
这下还不行，你老老实实去配置bin吧，不会去Google，或者百度

针对
```
Access denied for user 'root'@'localhost' (using password: YES)
```

你高高兴兴的输入
```
$ mysql -u root
```
或者拿工具链接本地数据库，结果
```
Access denied for user 'root'@'localhost' (using password: YES)
```
如果还记得安装的时候，弹了个小窗窗，那么恭喜你，里面有密码提示
![Upload mysql.user.png failed. Please try again.]](https://upload-images.jianshu.io/upload_images/1112615-9c2b135adadfcea9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
因为第一次安装，密码不记得怎么办?
凉拌，你都没密码
只能重置了，重置密码
```
$ mysqladmin -u root password 1
```
你会发现没用，我擦为什么百度到的会没用呢！
于是抖个机灵用Google，你都没密码，随便让你改吗
```
$ sodo mysqladmin -u root password 1
```
依旧没有将密码改成功.
#前方大招：先停掉MySQL所有服务！
请先移步到/usr/local/mysql/bin/

```
$ sudo su
```
```
$ cd /usr/local/mysql/bin/
$ ./mysqld_safe --skip-grant-tables --skip-networking &
```

这时候，新建一个终端窗口，不要关闭当前的
输入
```
$ cd /usr/local/mysql/bin/
$ mysql -u root
```

好开心，进入了.
![表结构](https://upload-images.jianshu.io/upload_images/1112615-21edde00b0a08a75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你发现，有4个库.
赶紧百度看怎么改root密码
```
 mysql> UPDATE mysql.user SET password=PASSWORD(’新密码’) WHERE User=’root’;
 mysql> FLUSH PRIVILEGES;
```

可惜啊，老版本是有password这个字段的，但是mysql-5.7.9并没有.

看看表结构
``mysql> show columns from mysql.user;

好长啊！！！我就上图吧

![用户表字段](https://upload-images.jianshu.io/upload_images/1112615-49ef55704b144fcf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你发现没有password这个字段，仔细看看，authentication\\_string这个字段很可疑，text类型，select看看
```
mysql> select authentication_string from mysql.user;
 +-------------------------------------------+
 | authentication_string                     |
 +-------------------------------------------+
 | *3850A8D8B8D396B78359100B96A8D4A19884F930 |
 | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
 +-------------------------------------------+
```

果然，加密后的密码，那就好办了，想好新密码
```
 mysql> UPDATE mysql.user SET authentication_string=PASSWORD(’新密码’) WHERE User=’root’;
 mysql> FLUSH PRIVILEGES;```

好了，已经OK了。
我个人喜好，重启MySQL服务，你可以不照做。现在回去测试重连

```
$ mysql -u root -p
$ 新密码
```

恩进来了Â
然后
```
 mysql> select * from mysql.user where user = root;
```
又来问题了
```
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```

我擦,刚刚设置的密码,又过期了?
当然不是,
还得再设置一下下--命令:
```
mysql> SET PASSWORD = PASSWORD('新密码');
```
这下就好了.
**love&peace**
