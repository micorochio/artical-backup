---
title:
- 初探Gradle
date:
- 2017-05-07 23:06:43
tags:
- Gradle
- Java
---
## 0x00 所谓构建工具
多人合作时，代码需要按照规范统一管理，但是早期一个项目很难做到一处编写到处运行。一方面，开发者自己的编写环境不能绝对统一，另一方面，没有一个方案去解决自动构建项目的问题，而手动机械重复构建往往出现各种问题。

后来，Java构建项目工具诞生了Ant，第一代自动化构建工具。通过XML，约束构建流程，达到自动构建的目的。
Ant构建脚本由 一个project ，多个target，以及可用的task构成。
有兴趣的童鞋可用查一下，Ant构建的XML文件的大致写法，这里不作详解。只介绍缺点
ㄟ( ▔, ▔ )ㄏ
Ant虽然实现了自动构建，但是在大型项目中，XML构建脚本异常臃肿庞大；构建脚本逻辑也会越来越复杂，最后难以维护。
而且Ant没有规范项目结构，可能导致每次编译出来的东西都不太一样，特别当依赖被改来改去，易出现重复拷贝。因为Ant自身本来就没有提供依赖管理工具，只能借助Ivy。
构建时也无法监控内存变化，task执行。
<!--more-->

然后又出现了Maven，现在Maven依然被广泛利用。
Maven本身规范了项目结构，并提供了依赖管理。主要功能如下：
+ 代码编译
+ 测试（单元，集成）
+ 装配（如，jar文件依赖）
+ 部署（将项目部署到本地仓库）
+ 发布（将项目发布到远程仓库）

上面都是看来的，因为Maven解决了Ant的痛点，很快流行起来，其中仓库的概念，更是好用，解决了依赖问题。而且Maven可以添加各种插件，相对来说，比Ant更易用，更稳定。

但是：Maven规范很严格，可能导致你搬运来的非Maven项目，需要大改才能运行。扩展也非常难搞，需要了解Major，早期版本还会自己更新，这对有墙的我们来说很痛苦。

所以今天我要介绍Gradle，一个灵活的构建工具。

## 0x01 Gradle，一个更好的构建工具！
一开始接触gradle会觉得，这玩意儿写的东西很简洁，logo也比Maven好看。
但是估计也不知道这东西咋玩。所以这个东西究竟好在哪里？

1. 基于JVM，Gradle是基于JVM的构建工具（Java跨平台，你值得拥有😁）。
2. 由Groovy的领域语言DSL来表达构建脚本（所以简洁，强大）。
3. 项目迁移到gradle不需要特大改动，而且极易扩展
4. 用的人越来越多了
5. 大型项目持续交付，


![](http://upload-images.jianshu.io/upload_images/1112615-be2d8b2a656cb75c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

唉，看的书作者懒得翻译，我给你们看[wikipedia的解释](https://zh.wikipedia.org/wiki/%E5%81%A5%E5%A3%AE%E6%80%A7_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6))
> 鲁棒 ：**健壮性**（英语：**Robustness**）是指一个计算机系统在执行过程中处理错误，以及算法在遭遇输入、运算等异常时继续正常运行的能力。 诸如[模糊测试](https://zh.wikipedia.org/wiki/%E6%A8%A1%E7%B3%8A%E6%B5%8B%E8%AF%95)之类的[形式化方法](https://zh.wikipedia.org/wiki/%E5%BD%A2%E5%BC%8F%E5%8C%96%E6%96%B9%E6%B3%95)中，必须通过制造错误的或不可预期的输入来验证程序的健壮性。很多商业产品都可用来测试软件系统的健壮性。健壮性也是[失效评定](https://zh.wikipedia.org/wiki/%E5%A4%B1%E6%95%88%E8%AF%84%E5%AE%9A)分析中的一个方面。

## 0x02 Gradle之世界你好
废话一大堆，现在写一个Gradle版的Hello World
+ 准备
1. 安装JDK，配置JAVA_HOME
Windows：http://jingyan.baidu.com/article/6dad5075d1dc40a123e36ea3.html
Lunix：http://jingyan.baidu.com/article/ab0b56308966acc15afa7d18.html
Mac：自行搜索吧
2. 安装Gradle，配置GRADLE_HOME
官方安装文档：https://gradle.org/install
如果你英文烂，请使用百度，我只说一下Lunix
https://gradle.org/releases   找一个最新的稳定版本，不看源码你只需要binary-only 版本的,我以3.5 为例子。

```sh
cd /usr/local
sudo mkdir gradle
cd gradle
sudo wget https://services.gradle.org/distributions/gradle-3.5-bin.zip
sudo unzip gradle-3.5-bin.zip
```
配置环境变量
```sh
vim ~/.profile
```
在最后添加下面
```sh
export GRADLE_HOME=/usr/local/gradle/gradle-3.5
export PATH=$GRADLE_HOME/bin:$PATH
```
按下`esc` 然后` :wq!`保存
然后使环境变量生效
```sh
source ~/.profile
```
好了，安装完成。
> **请务必自己处理好路径**

测试，打开命令行。

![输入  gradle -v ](http://upload-images.jianshu.io/upload_images/1112615-9dd7019706ee29d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来创造爱因斯坦小板凳

随便找一个文件夹，创建一个build.gradle文件，里面写如下代码
```makefile
task helloworld{
  doLast{
    println 'Hello World!'
  }
}
```
然后，在这个文件夹下运行命令行
输入
```sh
gradle -q helloworld
```
看效果

![我是在graldeTest文件夹下运行的](http://upload-images.jianshu.io/upload_images/1112615-e933d43e41ec3b2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里只是运行了一个task，task和action是gradle重要的概念，当然我也是开始学习。后面会把这些概念挨个介绍给各位老爷。

这里介绍一下 gralde -q这个命令
```sh
-q, --quiet             Log errors only.
```
安静模式运行，只打印错误日志。
当然
还有其他的命令，我找几个常用的介绍一下。
```sh
-b xxx.gradle  #运行一个叫xxx的构建脚本
--offline #离线运行（保证本地有离线仓库）
-P --project-prop #向构建脚本中传参
-i --info #设置编译日志输出级别
-s --stacktrace #构建出错输出跟踪栈信息
tasks #显示所有可运行的task
properties #显示项目中可用的属性
```

## 0x03 小结
废话了一大堆，后面写了个helloWorld，想必目前各位看官老爷还是不太明白gradle究竟如何使用。不急，后面我会一一把坑填好的：我会介绍创建Gradle项目，构建脚本，依赖管理等，不过写的会稍微慢一点。
ps:因为脑残升级了node，导致Hexo博客炸了，昨天抢救了一整天。发了一篇小白文。
因为要熟悉项目，还有那个FS计划依然在进行，不过会慢一些了，过些日子还有一堆其他事情要处理，希望我能够保持学习的节奏吧。

欢迎转载，请保留出处：[初探Gradle by:MaxZing](https://micorochio.github.io/2017/05/08/gradle-learning-01/)