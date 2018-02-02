---
title:
- 函数式接口和Lambda表达式深入理解
date:
- 2016-12-25 12:27:06
tags:
- Java
- Java8
- Lambda
- 函数式编程
---
![](http://upload-images.jianshu.io/upload_images/1112615-3eab911b3c5091b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我上一篇文章介绍了函数式接口和Lambda表达式 ，以及Java解决所谓的闭包。http://azing.xyz/2016/12/22/lambda-and-Functional-Programming
这次深入一下。
## 0x00 函数式接口
前面讲了一下函数式接口，不过可能只是讲了个大概，大致讲了一下什么是函数式接口
> + 函数式接口就是：一个interface，里面只有一个抽象方法，其他什么都没有。
+ FunctionalInterface注解标注一个函数式接口，不能标注`类`，`方法`，`枚举`，`属性`这些。
+ 如果接口被标注了`@FunctionalInterface`，这个类就必须符合函数式接口的规范
+ 即使一个接口没有标注`@FunctionalInterface`，如果这个接口满足函数式接口规则，依旧被当作函数式接口。

这次我们来用代码来深入了解函数式接口

<!-- more -->
![Demo01](http://upload-images.jianshu.io/upload_images/1112615-b2f5132dd4857d68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如上图，只包含一个抽象方法是最普通的函数式接口

![两个抽象方法，报错](http://upload-images.jianshu.io/upload_images/1112615-3fe1ad8f57634766.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再看，当接口有两个抽象方法的时候，就不在是函数式接口了，使用`@FunctionalInterface`标注编译时会报错

![特例-01](http://upload-images.jianshu.io/upload_images/1112615-42538ac6cf9ae118.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
奇怪的是这里有3个抽象方法，为什么不报错？
我们知道`toString`和`equals`方法是Object的方法，Java基础告诉我们，Object是所有类的默认父类，也就是说任何对象都会包含Object里面的方法，即使是函数式接口的实现，也会有Object的默认方法，所以：**重写Object中的方法，不会计入接口方法中**，除了final不能重写的，Object中所能复写的方法，写到接口中，不会影响函数式接口的特性

![特例-02](http://upload-images.jianshu.io/upload_images/1112615-c34dd07136009926.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Java8 允许接口中含有非抽象方法，这种在接口中使用`default`修饰的非抽象方法称为默认方法，默认方法也不会影响函数式接口的特性。我们依然可以认为`DemoConsumer`是一个函数式接口。

## 0x01 Lambda表达式深入
Lambda表达式的形式如下
```java
（param1, param2, param3, param4…）->{ doing……}；

```
>由此引申出多种写法：
 ```java
 //1.
() -> System.out.println("Hello Lambda");
//2.
number1 -> int a = number1 * 2;
//3.
(number1, number2) -> int a = number1 + number2;
//4.
(number1, number2) -> {
  int a = number1 + number2;
  System.out.println(a);
}
```

下面通过重构一段代码，来深入了解一下Lambda表达式
```
public class FunctionalInterfaceTest {
    public static void main(String[] args) {
        List<String> demoList = Arrays.asList( "Zing", "阿三", "小明", "小红", "赵日天");
       rollCall(demoList);
    }
    public static void rollCall(List<? extends String> list){
        for(String name : list){
            if(name.startsWith("小")){
                System.out.println(name);
            }
        }
    }
}
```

如果希望筛选的条件能自由定义，而不是`name.startsWith("小")`写死，并且希望找到人名后，不是简单的` System.out.println(name);`，而是能做一些其他的事情。
继续重构：
```java
/**
 * 函数式接口
 * @param <T>
 */
@FunctionalInterface
interface Checker<T extends String>{
    boolean check(T t);
}

@FunctionalInterface
interface Out<T>{
    void achievement(T t);
}

public class FunctionalInterfaceTest {
    /**
     * 点名
     */
    @Test
    public void testLambda() {
        List<String> demoList = Arrays.asList("小明", "Zing", "阿三", "小红", "赵日天");
        rollCall(demoList,
                name-> name.startsWith("Z"),
                name->{
                    String rate = name + "是单身狗!";
                    System.out.println(rate);
                });
    }

    /**
     * 点名逻辑
     * @param list
     * @param checker
     */
    public void rollCall(List<? extends String> list, Checker checker,Out out){
        for(String name : list){
            if(checker.check(name)){
                out.achievement(name);
            }
        }
    }
}

```

![运行结果](http://upload-images.jianshu.io/upload_images/1112615-e0b9a6da4bda1b31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
一不小心暴露了什么。哈哈哈

通过上面的重构，很明显，这么写也是合法的
```java
        Checker checker =  name-> name.startsWith("Z"),
        Out estimator = name->{
            String rate = name + "是单身狗!";
            System.out.println(rate);
        };
```
由此可以知道，Lambda和函数式接口是等价的。

## 0x02 补充
+ **类型**

有人会很奇怪，为什么<br>`Checker checker = name-> name.startsWith("Z")`<br>这样写的时候，name会被当成String 类型？

这是Java的类型推断，大致逻辑是编译器知道函数式接口方法的输入参数类型，所以无论前面的参数是什么名字，都会被当成方法所需要的参数类型。

+ **简单缩写**

还有一个奇怪的地方`name->name.startsWith("Z")`为什么这样写也可以？
为什么不是写成``name->{ return name.startsWith("Z");}` 。
很明显，后面的写法是没有错的，

![](http://upload-images.jianshu.io/upload_images/1112615-f32ee64c93382c58.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
但是Idea会有一个虚线，说明不需要写return
![展开看说明](http://upload-images.jianshu.io/upload_images/1112615-0d9b2930abe06bdb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
当只需要执行一条语句的时候，lambda支持这种简洁返回。所以为什么拒绝呢？

+ **外部参数**

Lambda表达式是不能操作外部对象的，因为Lambda 实质上是接口的子对象，只能访问静态资源和本身的内部变量。

![报错！](http://upload-images.jianshu.io/upload_images/1112615-40ba305a25e2587f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
编译器会要求将外部变量使用`final`修饰。

+ **和方法引用结合**

**方法引用**`Method References`是Java8配合Lambda一起做出的新特性，当Lambda表达式里面只执行已知的方法的时候，可以使用方法引用来写出跟简洁易读的代码
```java
List<String> demoList = Arrays.asList("小明", "Zing", "阿三", "小红", "赵日天");
demoList.forEach(System.out::println);
```

看到这里想必心里不禁想说，我擦，好简洁！
官方给出了4种方法引用
> Kinds of Method References

> | Kind | Example|
| ---- |----|
|Reference to a static method|ContainingClass::staticMethodName|
|Reference to an instance method of a particular object|containingObject::instanceMethodName|
|Reference to an instance method of an arbitrary object of a particular type|ContainingType::methodName|
|Reference to a constructor|    ClassName::new|
来源：
http://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html

我想我就不用翻译了吧，出门百度翻译😉
____

love&peace
[FS全栈计划目录：https://micorochio.github.io/fs-plan/](https://micorochio.github.io/fs-plan/)