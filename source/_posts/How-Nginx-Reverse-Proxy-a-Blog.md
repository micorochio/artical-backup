---
title:
- 如何用Nginx反向代理静态博客
date:
- 2016-7-10 11:56:07
tags:
- Linux
- Nginx
- 博客搭建
- 反向代理
- 静态博客
---

之前使用Hexo在各个平台搭建了一遍博客，最后发现，最好的办法是，本地搭建Hexo，编译文件后的文件放在git上，在克隆一个仓库到自己的服务器上，最后在自己的服务器上跑一个Nginx ，代理静态文件就行了。
折腾了老么久。

至于Nginx是什么，我就一句话：老牛逼的反向代理了，还能做负载均衡啥的。

![博客方案](http://upload-images.jianshu.io/upload_images/1112615-62140cb11bcc3ed2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

假设你在自己电脑上已经搭建好了Hexo 并且已经上传到了Github上。现在教你怎么让自己的服务器运行自己的博客


> 步骤如下
+ 第一步：安装Nginx
+ 第二步：克隆Git仓库到服务器
+ 第三步：修改Nginx配置文件，重启Nginx

<!-- more -->


##安装Nginx
最复杂的一步，我说一下流程，中途遇到任何坑，可以自行Google或百度

+ 安装openSSL等编译环境
`yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel`

+ 安装PCRE

```
# 下载PCRE
wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
# 解压
tar zxvf pcre-8.35.tar.gz
# 配置环境
cd pcre-8.35
./configure
# 编译安装
make && make install
```

+ 安装Nginx

```
# 下载并解压
wget http://nginx.org/download/nginx-1.6.2.tar.gz
tar zxvf nginx-1.6.2.tar.gz
# 配置环境
cd nginx-1.6.2

# 编译安装，注意安装的openSSL和PCRE的位置，我们把Nginx安装在/usr/local/webserver/nginx内

./configure --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.35
```

查看Nginx版本
`/usr/local/webserver/nginx/sbin/nginx -v`


## 克隆git仓库
这个很简单，首先确认本机已经安装了git，没有的话需要先安装，配置好Git，
很简单。我简单介绍下步骤，不展开说了。
>
+ yum命令安装git-core
+ 配置git.name 和 git.email
+ 生成 SSH证书，将pub公钥存到Githab的SSH钥匙库里


<br>
**做完这些，在命令行中，使用 git clone +仓库名把你的Hexo博客down下来**
```
git clone git@github.com:micorochio/micorochio.github.io.git
```


![这是我的博客，懒癌泛滥，就一个Hello World](http://upload-images.jianshu.io/upload_images/1112615-e0a4eb3a153c2ae9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

克隆结束后，你的本地会多一个文件夹，就是你的博客内容了

![我的就是文件夹名字就是micorochio.github.io](http://upload-images.jianshu.io/upload_images/1112615-b564a2c7a0cb028f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 配置、启动Nginx
还记得我们安装Nginx的目录吗？
`/usr/local/webserver/nginx`

接下来我们对Nginx进行配置
```
cd /usr/local/webserver/nginx/conf/
vi  nginx.conf
```

你会看到老长一段配置文件，很简单。如果看不明白，我直接告诉你，如何配置


![1：配置用户](http://upload-images.jianshu.io/upload_images/1112615-580f010013afe80e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我是zing，其实这是更你博客文件夹有关的，我的文件夹归属用户是zing,所以为了能有文件的读写权利，我的用户配置是zing，如果你的博客文件夹(`xxx用户.github.io的拥有者`)，和你的Nginx，配置的用户不一样，你需要将文件夹和子文件转到nginx配置的用户名下。

_（不知道会不会影响git 更新博客文章，建议还是改Nginx）_



![2：把博客目录配置到server的location节点内](http://upload-images.jianshu.io/upload_images/1112615-34a20a79d9fa3289.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我已经用红框标记了，你博客的文件夹，配置在`root` 节点下
index.html 配置在 `index` 节点下 如果你感兴趣，可以配置下全局的 404，5xx

当然 如果你的域名需要 `http://xxxxxx.com/blog`这种样子的，你可以把location边上的`/`改成 `blog/`这样就可以了。

ps：还可以配置成`http://blog.xxxxxx.com`这样的，感兴趣的小伙伴，自己查一下

保存退出
```
# 先按esc
:wq!
```
要是保存不了，检查下你的用户是不是没有权限修改 nginx.conf



最后，启动Nginx

```
cd /usr/local/webserver/nginx/sbin/
./nginx
```

完成。可以测试域名访问博客了
http://azing.xyz/

zing love&peace
