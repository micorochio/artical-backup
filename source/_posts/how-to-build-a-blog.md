---
title:
- Mac如何搭建静态博客（Hexo+github）
date:
- 2016-6-10 11:56:07
tags:
- Mac
- macOS Sierra
- 博客搭建
- Hexo
- 静态博客
---

整体步骤先写一下
+ 1.安装Node.js（已安装则跳过）
+ 2.安装Git（已安装则跳过）
+ 3.安装Hexo及插件，配置参数
+ 4.部署到Github
+ 5.修改主题

### 1.安装Node.js（已安装请忽略）

+ 下载这是4.4.4版本，你可以去官网下载最新的稳定版。
node-4.4.4 https://nodejs.org/dist/v4.4.4/node-v4.4.4.pkg 。
下载完双击打开，按提示安装就好.
完成后验证：打开终端，输入
```
 npm -version
```
会有版本号显示

<!-- more -->

### 2. 安装Git（已安装请忽略）
建议图形化安装，蛋不疼不要折腾 MacPorts安装法
图形化的 Git 安装工具：https://sourceforge.net/projects/git-osx-installer/

![安装完成](http://upload-images.jianshu.io/upload_images/1112615-db7d02983c2c7d3c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3. 安装Hexo及其插件
+ 安装Hexo

很简单，打开终端
建议使用阿里的Node源，15分钟跟官方同步一次，速度杠杠的！
使用方法
```
#这两步，可以忽略
npm --registry=https://registry.npm.taobao.org install koa
npm --registry=https://registry.npm.taobao.org install cnpm -g
```
运行完之后执行安装操作
也可以直接安装，可能大陆会比较慢

```
cd ~/Document
# 创建目录
mkdir hexo
# 切换目录
cd hexo
# 安装 Hexo 输入密码
sudo npm install -g hexo-cli
# 初始化 Hexo
hexo init
```
**切换阿里的源后`sudo npm install -g hexo-cli`可以换成`sudo cnpm install -g hexo-cli`**

+ 安装插件

```
npm install hexo-generator-index --save
npm install hexo-generator-archive --save
npm install hexo-generator-category --save
npm install hexo-generator-tag --save
npm install hexo-server --save
npm install hexo-deployer-git --save
npm install hexo-deployer-heroku --save
npm install hexo-deployer-rsync --save
npm install hexo-deployer-openshift --save
npm install hexo-renderer-marked --save
npm install hexo-renderer-stylus --save
npm install hexo-generator-feed --save
npm install hexo-generator-sitemap --save
```
嫌慢，用`cnpm` 代替`npm`


### 4. 部署到Github

这一步跟我之前写的[CentOS下搭建Hexo + github 博客](http://www.jianshu.com/p/0823e387c019)的部署到github一毛一样，可以参考参考。
这里只总结一下步骤
> + 配置git整体环境，已经配置过可忽略
+ 创建博客仓库，仓库名必须规范
+ 生成SSH秘钥，添加到github账户下
+ 配置Hexo文件_config.yml，关联博客仓库和Hexo
+ 编译Hexo，上传静态网页到github

如果全部完成的话，可以通过 http://github用户名.github.io访问你的博客

### 5. 配置Hexo主题
+ 1.安装主题NexTps: 在Hexo安装目录下执行
```
git clone https://github.com/iissnan/hexo-theme-next themes/next
```
并在目录hexo下的_config.yml中
```
# 找到 theme: 修改后面的参数，默认是 landscape
theme: next
```
+ 2.配置主题源码拷贝出来太多了，所以贴出[next的使用说明](http://theme-next.iissnan.com/getting-started.html)供大家参考

+ 3.找主题[https://hexo.io/themes/](https://hexo.io/themes/)
