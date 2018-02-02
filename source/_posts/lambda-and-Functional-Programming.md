---
title:
- 函数式接口和Java8的Lambda表达式
date:
- 2016-12-21 13:23:42
tags:
- Java
- Java8
- Lambda
- 函数式编程
---


![Lambda](http://upload-images.jianshu.io/upload_images/1112615-bd17c5cff7dbe5bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 0x00 JS中的闭包以及Java中替代方法

在Java中，是不允许将一个方法作为参数传递给另一个函数，也不允许将一个方法作为值，返回给调用者，这就是所谓的不支持闭包。<br/>
但是JavaScript是可以这么玩的。比如：

```javascript
getWebResource(function(result,error){
  if(error){
    doException();
  }else{
    analysisResult(result);
  }
});
```
我们看到直接向`getWebResource`方法中传递了一个`function`，这个function用来解析网络请求结果。


但是Java不能这样，那Java一般是怎么玩耍的呢？
<!-- more -->
<br/>
我们来看一段Android内的Java代码
```java
        loginBtn = (Button) findViewById(R.id.btn_login);
        loginBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Login();
            }
        });

```
很明显，我们为`loginBtn`设置了一个点击监听回调函数，这函数中，传递了`View.OnClickListener`的匿名实例对象，我们通过实例对象，调用`onClick(View view)`方法。<br/>
这样，通过匿名对象，我们可以实现类似js中的闭包的功能，不过我们是通过匿名对象调用函数，而不是直接调用函数，还是有区别的

## 0x01 函数式接口
上面，我们看到`View`类中的`OnClickListener`接口。这个接口负责接收点击事件的回调
```java
    public interface OnClickListener {
        void onClick(View var1);
    }
```
这个接口有且仅有一个抽象方法，这种接口，我们称为函数式接口(`FunctionalInterface`)。这是Java8中添加的新的支持。
可以看看Java8的API
`java.lang.FunctionalInterface`API:https://docs.oracle.com/javase/8/docs/api/java/lang/FunctionalInterface.html
这是个标注函数式接口的注解。里面的注释，解释了函数式接口，和这个注解的用法。
就不翻译了，大致意思是：
+ 函数式接口就是：一个interface，里面只有一个抽象方法，其他什么都没有。
+ FunctionalInterface注解标注一个函数式接口，不能标注`类`，`方法`，`枚举`，`属性`这些。
+ 如果接口被标注了`@FunctionalInterface`，这个类就必须符合函数式接口的规范
+ 即使一个接口没有标注`@FunctionalInterface`，如果这个接口满足函数式接口规则，依旧被当作函数式接口。

> 注意：interface中重写Object类中的抽象方法，不会增加接口的方法数，因为接口的实现类都是Object的子类。

说完这些，我们该说一下神奇的Lambda表达式了


## 0x02 Lambda表达式
> “Lambda 表达式”(Lambda expression)是一个[匿名函数](http://baike.baidu.com/view/3034885.htm)，Lambda表达式基于数学中的[λ演算](http://baike.baidu.com/view/1179241.htm)得名，直接对应于其中的Lambda抽象(Lambda abstraction)，是一个匿名函数，即没有函数名的函数。Lambda表达式可以表示[闭包](http://baike.baidu.com/view/648413.htm)（注意和数学传统意义上的不同）。
——百度百科

Java中的Lambda表达式有三种形式，我们来举3个例子
```java
//1.
() -> System.out.println("Hello Lambda");

//2.
(number1, number2) -> int a = number1 + number2;

//3.
(number1, number2) -> {
  int a = number1 + number2;
  System.out.println(a);
}
```

大致形式就是
```java
（param1, param2, param3, param4…）->{ doing……}；

```
就是一个匿名的函数式接口的缩写。


很明显，在之前的代码中，我们用到了匿名对象，这个匿名对象就是一个函数式接口的子对象，所以，在Java8中可以使用Lambda表达式
我们把刚刚的代码简化成Lambda表达
```java
    loginBtn = (Button) findViewById(R.id.btn_login);
    loginBtn.setOnClickListener(（view)->  Login());
```
瞬间简洁了，有木有？

然后呢？因为有了函数式接口的标准，Java8中的List，有了一个新方法，叫forEach，下面演示一下这个方法。
```java
package com.zing.lambda_demo;

import java.util.Arrays;
import java.util.List;

/**
 * Created by zing on 2016/12/21.
 */
public class FunctionalInterfaceTest {
    public static void main(String[] args) {
        List<Integer> demoList = Arrays.asList(1, 2, 3, 4, 5);
        demoList.forEach((num) -> System.out.println(num));
    }
}
```

![运行结果](http://upload-images.jianshu.io/upload_images/1112615-75007a0d7cb62e6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


当然，Lambda表达式虽然简洁，也是有缺点的：就是降低了代码的可读性，比如一眼看不出来原来的函数式接口是什么。
所以什么情况用Lambda，要多写多练才会用的恰当。

ps:我就不解释forEach具体方法了，贴俩图你看看实现和注解

![forEach方法实现](http://upload-images.jianshu.io/upload_images/1112615-73ee342ec5ffea78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![Consumer函数式接口](http://upload-images.jianshu.io/upload_images/1112615-ac6192c8ede46c99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


谢谢收看，我的博客：http://www.azing.xyz 欢迎光顾。
____
love&peace
[FS全栈计划目录：https://micorochio.github.io/fs-plan/](https://micorochio.github.io/fs-plan/)