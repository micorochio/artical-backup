---
title:
- (笔记备份)CentOS 7 换yum源
date:
- 2017-01-15 23:13:05
tags:
- Linux
- CentOS
---

## 0x00 备份原来的源
```shell
$ sudo mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bk
```

## 0x01 下载阿里源
```shell
$ cd /etc/yum.repos.d
$ sudo wget -nc http://mirrors.aliyun.com/repo/Centos-7.repo
```

## 0x02 更改阿里yum源为默认源
```shell
$ sudo mv CentOS-7.repo CentOS-Base.repo
```
<!--more-->
## 0x03 更新本地yum缓存
```shell
# 全部清除
$ sudo yum clean all
# 更新列表
$ sudo yum list
# 缓存yum包信息到本机，提高搜索速度
$ sudo yum makecache
```

## 0x04 国内yum源
1. 阿里yum源:[http://mirrors.aliyun.com/repo/](http://mirrors.aliyun.com/repo/)
+ 163(网易)yum源: [http://mirrors.163.com/.help/](http://mirrors.163.com/.help/)
+ 中科大的Linux安装镜像源：[http://centos.ustc.edu.cn/](http://centos.ustc.edu.cn/)
+ 搜狐的Linux安装镜像源：[http://mirrors.sohu.com/](http://mirrors.sohu.com/)
+ 北京首都在线科技：[http://mirrors.yun-idc.com/](http://mirrors.yun-idc.com/)