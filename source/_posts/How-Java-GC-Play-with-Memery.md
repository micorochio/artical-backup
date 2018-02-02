---
title:
- Java的GC如何玩弄对象
date:
- 2017-03-30 11:09:07
tags:
- Java
- GC
- JVM
- FS计划
---
Java的GC是什么，应该做Java的人都知道。但是其实GC的历史要比Java早，Java出现之前，人们就开始研究：
+ 哪些内存需要回收
+ 哪些内存什么时候回收
+ 怎样回收

然后就有了GC，而Java解决的这3件事情，就目前看来，效果还可以。解决了很多的内存问题。但是，GC并不能解决所有内存动态分配的问题，尤其是高并发的软件中，了解GC，方便解决内存溢出问题，更好的控制和调节程序的回收和分配内存。
<!--more-->
## I 对象生死判定算法
感叹一下乔布斯当年看中的两项技术Internet 和OOP Language，现在都如日中天，不愧是乔帮主。

Java是OOP的经典语言之一，OOP语言号称万物皆对象，对象用不到了，那就应该离开内存了，这就是对象的死亡。是不是死掉的呢

死掉的对象，自然需要GC来处理，但是GC怎么知道对象已经死掉了呢？

### 1x01  **引用计数算法**
就是给每个对象添加一个引用计数器，当对象被引用一次，计数器就+1；引用失效时，计数器就-1。当计数器为0，就说明对象死了。这个方法实现简单，效率也可观。但是，主流的JVM没用这个算法，因为这个算法很难解决循环引用的问题。

> __什么是循环引用？__
就是多个对象，互相引用对方作为属性，下面就是A依赖B，B依赖C，C依赖A的循环引用。<br>![这就是，百度一下就知道了](http://upload-images.jianshu.io/upload_images/1112615-ac9f5b3e247bc4a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当然，还有更细致的分法：

 + _强引用_    
 `String a = new String()`就是这样的，强引用存在，GC就不能回收对象
 +  _软引用_   
有用单非必须的对象，这些对象在内存快溢出就会被回收，回收内存依旧不够才会抛出异常
 + _弱引用_   
比软引用还非必要，每次GC来的时候都会被回收
 + _虚引用_   
 最弱的引用，无法通过这个引用来获取对象，唯一的作用是在被回收事给系统一个通知

### 1x02 **可达性算法**
这个比上面高大上一点，Java通过可达性分析来判定对象是否还被引用。什么的可达性分析呢：
Java会从一些叫做GCRoot的对象开始向下遍历，可以遍历到的对象，就是被引用的对象，不可以遍历到的对象就是不可达对象，就是死掉的对象了：

![蓝色表示可达对象
灰色表示不可达对象](http://upload-images.jianshu.io/upload_images/1112615-70117103618610c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*蓝色表示可达对象灰色表示不可达对象*

在图上可以看到，从GCRoot开始，蓝色部分的对象都可以被遍历到，儿灰色部分，即使 Object A 可以遍历到 Object B 和Object C，但是却没有了GCRoot 引用，所以就属于不可达的死亡对象了
<br>（是不是找不到对象就可以死了`T^T`）。

GCRoot 包括：栈中引用对象，方法区静态引用对象，方法区常量引用对象，本地方法引用对象（Native层的）


##  II GC回收垃圾的算法
既然已经能判断了垃圾有哪些，接下来就简单讲讲对垃圾对象如何清理

###  2x01  标记-清除算法
跟名字一样，先把死掉的对象标记出来，然后清除，大部分算法是基于这个思想，不足之处也很明显，1是效率问题，标记和清除的过程都慢，2是空间问题，清除之后会带啦大量的不连续碎片空间。小的碎片会放不下大对象，导致大对象创建时又会触发一次回收

![回收前](http://upload-images.jianshu.io/upload_images/1112615-f653d6c28c45b880.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*回收前*

![回收后](http://upload-images.jianshu.io/upload_images/1112615-206b89e6f5936577.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*回收后*

![](http://upload-images.jianshu.io/upload_images/1112615-e89082cc4f16a68f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2x02 复制算法
为了解决效率问题，有了复制算法，这种算法将内存分成相同大小的两块


![回收前](http://upload-images.jianshu.io/upload_images/1112615-ffaf64836f7d71d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*回收前*

![回收后](http://upload-images.jianshu.io/upload_images/1112615-81fc977d72535b59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*回收后*

![](http://upload-images.jianshu.io/upload_images/1112615-bbd542e94ead259d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其实并不是非要等比划分内存的，大部分对象死的很早Hotspot是划分了三块区域，一块大的两块小的，大的叫Eden，小的叫survivor，大小比例为8:1。清理时将Eden和survivor中存活的对象复制到另一块survivor内存上，然后，清理掉用过的两块内存，下次再用。当survivor不够大的时候，需要依靠新的分配担保去拓展空间。

### 2x03 标记-整理算法

综合复制和标记算法，整理算法会把有用的存活对象向y，一端移动，这样避免了复制算法浪费那么多内存，也不会像普通标记回收算法一样导致内存碎片过于严重。

![回收前](http://upload-images.jianshu.io/upload_images/1112615-141b22c5c8f8efce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*回收前*


![回收后](http://upload-images.jianshu.io/upload_images/1112615-821ce46430e3de22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*回收后*

### 2x04 分代收集算法
将java堆内存分成老年代，新生代。根据经验，新生代死亡比较快，老年代比较持久。所以一般新生代区域使用复制方法，只需要复制几个就可以了，老年代比较持久，所以一般用标记清除，或标记整理来回收。

## III 小结
GC是Java中最诱人的处理内存的方式，也是最令人难受的处理方式。想要深入java，GC是绕不过的必经之路。了解GC的运作方法，可以帮助程序员处理更深层次的Java问题，做出更深层次的系统优化。希望我的小总结能给你带来帮助

**转载请注明出处。**https://micorochio.github.io/2017/03/31/How-Java-GC-Play-with-Memery/
ps我的博客：https://micorochio.github.io/