---
title:
- OS X 中Launchpad 图标行列数量更改
date:
- 2016-08-25 22:17:53
tags:
- Launchpad
- Mac
- macOS Sierra
- OS X
---

改变Launchpad内图标排列的行数和列数

启动**终端**

 + 改变行数量
`defaults write com.apple.dock springboard-rows -int X`

+ 改变列数量
`defaults write com.apple.dock springboard-columns -int X`

+ 使改变生效
`killall Dock`

 **将X换成数字**

如果想恢复原样
```
defaults write com.apple.dock springboard-rows Default
defaults write com.apple.dock springboard-columns Default
killall Dock
```

+ _提示：千万不要改太小了，一行最少放5个，否则改回来你得拖好多图标到某页上_<br>
** 本人实践，13寸笔记本上，最佳是一页5行，每行9个（rows=5，columns=9）**
<!-- more -->

![Look！！！](http://upload-images.jianshu.io/upload_images/1112615-9320419eb72981b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
