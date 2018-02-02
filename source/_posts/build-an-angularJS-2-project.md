---
title:
- 用WebStorm创建AngularJS 2项目
date:
- 2017-01-01 16:31:33
tags:
- JavaScript
- Web
- 前端
- Angular
---

![第一步](http://upload-images.jianshu.io/upload_images/1112615-08353dbd6359df86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ 第二步，创建Angular CLI项目

_ps:[有26%的人称 Angular 2 环境设置是一大入门门槛，有22%的人说环境设置太过复杂。Angular CLI的诞生，正是为了解决这个问题](http://www.infoq.com/cn/news/2016/05/angular-cli-ng-conf-2016)_

如果你没有安装CLI，附上官网给你：https://cli.angular.io/


![](http://upload-images.jianshu.io/upload_images/1112615-081998ce3212083c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--more-->

解释一下3个框框
1. location: 是项目存放位置，最后untitled可以改成你需要的项目名字
2. Node interpreter: 是Node.js安装的位置
3. Angular CLI： 是Angular CLI安装位置，可以安装在任何位置，为了省事，全局安装吧。
**命令行：**
```
// -g 是全局安装的意思
sudo npm install angular-cli -g
```

![龟速的npm](http://upload-images.jianshu.io/upload_images/1112615-be60cb80930fbf66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

配置好Angular CLI的路径后点击 create

+ 第三步, 交给Storm吧

![](http://upload-images.jianshu.io/upload_images/1112615-47ecfd4ea6fc6110.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

还是要吐槽一下npm的速度，WebStorm没有给这玩意儿加速。
跑完就可以Coding了。484超级方便。

+ 第四步， 配置运行

![选择  Edit Configurrations…](http://upload-images.jianshu.io/upload_images/1112615-2476d5776e88ca4b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![添加Node.js配置](http://upload-images.jianshu.io/upload_images/1112615-99cde7e9fc02afac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![配置 npm 命令](http://upload-images.jianshu.io/upload_images/1112615-6a5747598c148c93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击运行，启动服务

![](http://upload-images.jianshu.io/upload_images/1112615-bdd32088e1b01e00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![将地址输到浏览器](http://upload-images.jianshu.io/upload_images/1112615-a0f61e80fff684d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



附上超级英雄的Angular2 的官方教程：https://angular.cn/docs/ts/latest/tutorial/


小伙子，去搞前端吧！

----
love & peace！