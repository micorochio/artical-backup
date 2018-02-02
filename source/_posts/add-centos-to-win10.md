---
title:
- Win10安装CentOS7填坑记
date:
- 2017-01-13 19:31:15
tags:
- linux
- Win
- 系统
- CentOS
---

## 0x00 血泪！一人装机到深夜
淘来一个120G的固态，打算装个CentOS7，玩玩服务器，万万没想到遇到了一个神奇的坑。整到大半夜都没搞定
都是因为百度来的东西大部分已经是二手货了，很多操作并不知道为什么。自己也是棵白菜。唉
朕要把这个坑记住，以后以此为鉴。

## 0x01 准备
+ __硬件__ 你需要一个PC或笔记本电脑（Mac建议别装双系统了，蛋疼，还是虚拟机去吧）、一个8GB的U盘，如果还有一台可以做U盘引导的电脑更好，最好是Windows的系统


![我的古董](http://upload-images.jianshu.io/upload_images/1112615-5595612452c2483d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ __软件__  [CentOS 7 x86_64的ISO文件](https://www.centos.org/download/)和[老毛桃U盘装机软件最新版](http://www.laomaotao.net/down/) 

ps:老毛桃会被windows误杀，装之前先关闭各种杀毒软件和电脑管家

## 0x02 第一步：准备制作U盘
<!--more-->
注意会格式化U盘请，务必将U盘数据做好备份，否则就没了。
管理员模式打开老毛桃，开始制作。

![使用老毛桃制作USB启动盘](http://upload-images.jianshu.io/upload_images/1112615-e30a8e0d48faffa6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![点击写入，等待大约2~12分钟，看U盘速度](http://upload-images.jianshu.io/upload_images/1112615-e4e36782a4aba6e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意：写入方式：我选择的是USB-HDD+ v2，据说，兼容性更好。其他默认就行！但是建议不要隐藏启动分区。

制作完成之后就可以装机了

## 0x03 第二步：装机
这步可以参考百度了，但是百度这个贱人搜出来一大堆比人抄袭的东西，比人的坑也被抄了下来。唉。不多说了下面按照步骤，开始装机
+ 选择USB启动，有的电脑是按住F12有的是F10，有的是Delete键，请根据自己的电脑，按键选择boot顺序
![选择U盘](http://upload-images.jianshu.io/upload_images/1112615-ef5c9ba13ebb3217.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果你的U盘制作的没有问题的话，就会进入安装模式

![界面](http://upload-images.jianshu.io/upload_images/1112615-c8ce6b5682cff55b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

__这里有坑！__
到达这个界面的时候，先直接按enter键，如果运气好的话，会直接进入安装界面，运气不好的话，会找不到U盘挂载。

![像这样](http://upload-images.jianshu.io/upload_images/1112615-9282f15332802096.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个时候不要随便改
`vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet`
改为
` vmlinuz initrd=initrd.img inst.stage2=hd:/dev/sdb4 quiet`
__这是别人抄的，不能全抄！！！__

## 0x04 正确的填坑方式

+    __1.__
强制关机！按住电源，直到PC关闭！然后重新启动，进入到下面的界面，立刻按Tab键（有的要按e,总之你看着按吧）
![界面](http://upload-images.jianshu.io/upload_images/1112615-c8ce6b5682cff55b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后最下面一行字出来了，用笔记下来！

![vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x88_64    rd.live.check quiet
](http://upload-images.jianshu.io/upload_images/1112615-b8d102b42cbe6985.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
一定记下来！
现在先看看我们的U盘在这里叫什么名字，挂载在哪里
将下面的文字改成
```shell
vmlinuz initrd=initrd.img linux dd rd.live.check quiet
```

![](http://upload-images.jianshu.io/upload_images/1112615-5a42ebe5c31d0621.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
会有一个列表，你会看到，一个格式为vfat的磁盘，挂载到sda4上，标签为`CentOS 7 x8`
因为U盘是fat格式所以这货是sda4！不是百度到的sdb4！

__那我换成sda4就好了啊__，错！万一你重启，U盘挂载到sdb4上，或者sdc4上，你会崩溃的，我就这样折腾到了深夜，简直哔了狗！

+ __2.__
 现在记下红框里的一切，我们按住电源键再强制重启到
![界面](http://upload-images.jianshu.io/upload_images/1112615-c8ce6b5682cff55b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
按一下Tab键（你看着改）
我们开始改最下面一行
```shell
vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64   rd.live.check quiet
```
刚刚看到红框里的LABLE是`CentOS 7 x8`,现在将空格改为`\x20`，根据你自己的LABLE，看看名字，跟着改。
为什么改为`\x20`？你可以看看Lunix Unicode，ASCII编码,Url转码什么的，就懂了
```shell
vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x8  rd.live.check quiet
```
然后回车就可以跳过这个坑了。

## 0x05 继续安装
接下来我就不多说了，按照别人博客继续来就行了
比如：http://blog.sina.com.cn/s/blog_135027f480102uyug.html
为了方便你们阅读，我搬个砖
> __2.3、修改后，按Ctrl+x执行修改，正常情况下，将进入安装界面，如下图：__
![](http://upload-images.jianshu.io/upload_images/1112615-4317a823071e11c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
选择想要安装的语言，点继续，如下图：
![](http://upload-images.jianshu.io/upload_images/1112615-1f44bd5b1717e3f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
__2.3.1、时区的选择__ <br>
如果你安装的是英文版，需要将时区改为上海<br>
__2.3.2、键盘__ <br>
英文键盘和中文键盘布局是一样的！！<br>
__2.3.3、语言支持__<br>
可同时选择支持多种语言
__2.3.4、安装源（可以默认不动）__<br>
程序将自动选择，进入可以手动制定，还可以直接指定为网络位置！！<br>
__2.3.5、软件选择（注意）__<br>
进入后，可以看到有多个选项，根据需要选择，如下图：<br>
![](http://upload-images.jianshu.io/upload_images/1112615-47b39586461a5717.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)<br>
注意：
默认为最小安装，安装后是没有界面的哦！！！！<br>
__2.3.6、安装位置（重要）__<br>
 这里我没有截图，下图来自网络，与实际有些不同<br>
![](http://upload-images.jianshu.io/upload_images/1112615-27a2f33453da82cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)<br>
1、__本地标准磁盘__ 项中，应该为 本地磁盘 和 U盘 ，选择本地硬盘<br>
2、__其他存储选项__ 中，选择 我要配置分区<br>
3、注意： 最下面的 完整磁盘摘要以及引导程序，打开，
选择不添加引导（后面再添加），<br>
*不知道是我这里的问题还是共性问题，先选择 安装引导 下一步将出错！！！，最后点击完成，进入如下界面*
![](http://upload-images.jianshu.io/upload_images/1112615-2b87b8ba2e8dcbac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
__注意：__
这里的分区最好选择 为标准分区。
因为：在用grub引导时，无法挂载LVMPV分区，根本不识别！！ <br>
  *最好不要用自动创建*
![](http://upload-images.jianshu.io/upload_images/1112615-b877544eb23d452b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
1、点击 + 号，分别添加 swap 和 /  两个分区<br>
大小 ： swap 一般为内存的两倍；<br>             /  为剩余的所有空间<br>
这步需要格外注意：
<br>（1）在选择自动创建分区时，分区信息将不能在更新，（这可能是我自己的问题，等待大家测试）
*完成后，点完成，返回配置摘要界面*
![](http://upload-images.jianshu.io/upload_images/1112615-5dec763ff017a640.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/1112615-02263eaf9cb57639.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>注意：此时，我们再选择安装位置项，将上面没添加的分区引导一项选上，直接点完成，直到返回配置界面！！
<br>__2.3.6、网络和主机名__
默认网络是关着的，可以再此处打开
<br>__2.4、最后，点击开始安装，如下：__
![](http://upload-images.jianshu.io/upload_images/1112615-252d430e79cac200.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>在安装过程中，可以设置 root 密码 和 新的用户 （安装后半部分不能再修改了！！）等待安装即可！！
<br>*注意： 安装过程中，如果密码太简单，需要点击两次完成来确认！*
![](http://upload-images.jianshu.io/upload_images/1112615-f2ca5949e745e768.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/1112615-d4f513db27f8e347.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/1112615-735325e67c0e542f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>至此，安装已完成，重启，就剩下引导了！！！！！！！！！！

## 0x06 恢复Windows引导&&恢复CentOS引导

+ 在CentOS中恢复Windows
```
# 切换到root 用户
su
# 编辑引导
 vim /etc/grub.d/40_custom
```
在结尾处添加
```
menuentry "Windows" {
         set root=(hd0,1)
         chainloader +1
}
```
保存 esc，`:wq!`，
编译重新自动生成grub所需要的配置文件
```
grub2-mkconfig --output=/boot/grub2/grub.cfg
```
好了，重启就能找到Windows了

+ 在Windows中恢复Linux
如果启动你只看到Windows，看不到Linux系统的，推荐你使用EasyBCD来恢复启动项

![](http://upload-images.jianshu.io/upload_images/1112615-2c352de9826c0c9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

教程在：http://www.360doc.com/content/12/0828/15/6140124_232817242.shtml
<br>我就不详细说了，目前还没找到其他靠谱的启动项恢复方法，EasyBCD下载地址的话，建议正版，否则的话，小心全家桶。




##0x07参考

<br>[Win 10 + CentOS 7 双系统安装与CentOS美化小记](http://www.voidcn.com/blog/superbfly/article/p-5768790.html )
<br>[Win8.1+CentOS7 双系统 U盘安装](http://blog.sina.com.cn/s/blog_135027f480102uyug.html)
<br>[安装 Windows 10 + Centos 7 双系统共存](http://www.techweb.com.cn/network/system/2016-12-21/2456741.shtml)
<br>[win7/10+centOS7双系统，默认启动win10](http://blog.sina.com.cn/s/blog_14a5924d30102w9pc.html)

文章转载请标明出处！











