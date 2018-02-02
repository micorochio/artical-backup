---
title:
- Java里的动态代理模式与代理加强
date:
- 2016-12-10 14:28:32
tags:
- Java
- 设计模式
- 代理模式
- AOP
- 动态代理
---
![](http://upload-images.jianshu.io/upload_images/1112615-f59cb26d91d9c8b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代理模式是设计模式中的一种，是比较常用的。这种模式在面向切面编程中发挥了很大的作用。
直接说代理模式比较抽象，我们先从使用场景开始入手。

### 那么，举个栗子来说一下代理模式

比如，烧开水泡茶这个活动，这件事情前面需要洗茶壶，后面需要把茶杯洗好收起来。我们将烧开水这件事抽象起来与其他的事情剥离，来降低耦合。前后的事情交给代理增强来做。


![就像这样](http://upload-images.jianshu.io/upload_images/1112615-d62dbb07b41d8f28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




### JDK 里面的动态代理
JDK是自带动态代理的，我们今天就介绍一下JDK中自带的动态代理的使用。 

![java中的动态代理模式模型图](http://upload-images.jianshu.io/upload_images/1112615-c3cc7c540bc86019.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![简单翻译](http://upload-images.jianshu.io/upload_images/1112615-f88241fdfb09d45a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过代码，我们来看看具体的。

首先，我们创建一个人类
```java
package com.zing.proxy_demo;
/**
 * Created by zing on 2016/12/1.
 */
public class Person {
    public String name = "Zing"; 
}
```

再创建一个工作接口，并实现这个接口
```java
package com.zing.proxy_demo;
/**
 * Created by zing on 2016/12/1.
 */
public interface iWorkingSomething {
     void work(Person who);
}
```

```java

package com.zing.proxy_demo;
/**
 * Created by zing on 2016/12/1.
 */
public class GuDuWork implements iWorkingSomething {
    @Override
    public void work(Person someOne) {
        System.out.println(someOne.name + "正在咕嘟咕嘟烧开水……\n一年之后终于烧好了\n");
    }
}
```

有人要问为什么不直接写Work，要加层接口，这是要问Java里的动态代理是面向接口的，必须这么玩。

从代码里能看出来，Zing要放到work方法中，来烧开水。
我们还需要：
+  在烧水泡茶之前洗洗水壶，准备柴火。
+ 在喝完茶之后，把茶杯洗好收起来

这俩动作，我们放在织入Handler中，让织入Handler，帮我们把这两个方法织入到work方法里。

我们创建一个织入组件
```java
package com.zing.proxy_demo;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * Created by zing on 2016/12/1.
 */
public class WavingInvocationHandler implements InvocationHandler {

    //需要代理的目标对象
    private Object target;

    /**
     * 将目标对象存入，一定得是具体的对象，而不是class
     * @param target
     */
    public WavingInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //前置增强 
        System.out.println("洗水壶，准备茶叶，茶点\n");

        //生成织入类，作为代理
        Object obj = method.invoke(target,args);

        //后置增强
        System.out.println("喝完，洗茶杯，收拾……\n");
        
        //将包含织入方法的代理对象返回  
        return obj;
    }
}
```
这个组件用来织入并生成新对象，那如何使用呢，我们可以用一个JUnit来验证一下。
```java
@Test
public void testProxy() {
    //需要代理的目标对象
    GuDuWork guDuGudu = new GuDuWork();
    //将原来目标放进织入组件中
    WavingInvocationHandler handler = new WavingInvocationHandler(guDuGudu);
    //生成织入增强的代理对象
    iWorkingSomething somethingWork = (iWorkingSomething) Proxy.newProxyInstance(
            guDuGudu.getClass().getClassLoader(),
            guDuGudu.getClass().getInterfaces(),
            handler
    );
    //代理对象增强版测试
    somethingWork.work(new Person());
}
```
我们看到了整个流程涉及到了4个方面
> + 目标对象 `GuDuWork`
+ 织入类  `WavingInvocationHandler`
+ 代理对象 `Proxy.newProxyInstance()方法生成的对象`
+ 接口 `iWorkingSomething`
整个流程就是`iWorkingSomething`的子类在执行`work(Person who)`方法的时候，会在执行前完成一些任务，执行之后，也会完成一些任务。而这些任务不会让`iWorkingSomething`的实现类去操心。
这就是完整的代理模式使用的简单演示。

![演示结果](http://upload-images.jianshu.io/upload_images/1112615-16550a1cd0e932e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 在业务中，代理模式的应用
很多人会觉得，要是在work方法里面直接加前置方法，和后置方法不是更简单吗？
在这个demo里确实会省略很多代码。
> 
不过：
+ 当你在写完一个完整的业务模块，需要测试并打印日志。你会发现你的业务模块了充满了日志代码，而这些代码跟主业务逻辑没有半毛钱关系。还会影响代码阅读。
+ 还有当一些一部消息需要传递到其他模块的时候，你不会希望发送消息的代码（一堆判断，验证，分析逻辑）出现在你主业务模块里面，而是自动触发。

以上两个场景只是简单的描述了一下，你会发现面向切面编程就是为了解决这些跟主业务垂直逻辑无关，但是却很重要的业务。

可是我们能不能更简单的实现代理模式呢？
答案是：可以，用CGLib。那么我来简单介绍一下CGLib的代理模式。
# 用CGLib实现代理模式
CGLib是利用字节码技术来创建代理子类的，可以再字节码层直接拦截父类方法，并且可以织入增强方法。只需要
> + 目标对象
+ 织入类
+ 代理类

这3个即可，话不多说，亮代码
```java
//目标对象依旧是 GuDuWork.class
```
织入类CGLibProxy
```java
package com.zing.proxy_demo;

import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;
/**
 * Created by zing on 2016/12/10.
 */
public class CGLibProxy implements MethodInterceptor {
    private Enhancer enhancer = new Enhancer();
    /**
     * 创建代理对象
     * @param proxiedClass
     * @return
     */
    public Object madeAnProxiedObject(Class proxiedClass) {
        enhancer.setSuperclass(proxiedClass);
        enhancer.setCallback(this);
        return enhancer.create();
    }
    /**
     * 对代理类的方法进行增强！
     * @param o
     * @param method
     * @param params
     * @param methodProxy
     * @return
     * @throws Throwable
     */
    @Override
    public Object intercept(Object o, Method method, Object[] params, MethodProxy methodProxy) throws Throwable {
        //前置增强
        System.out.println("Zing 要请大家喝茶，快去通知小伙伴\n");
        //执行方法
        Object result = methodProxy.invokeSuper(o, params);
        //后置增强
        System.out.println("终于喝完了，大家都走了，留下Zing一个人默默的刷杯子\n");
        return result;
    }
}
```
代理对象（使用）
```java
@Test
public void testCGLibProxy() {
    CGLibProxy cgLibProxy = new CGLibProxy();
    GuDuWork workingSomething = (GuDuWork) cgLibProxy.madeAnProxiedObject(GuDuWork.class); 
   workingSomething.work(new Person());
}
```


![执行结果](http://upload-images.jianshu.io/upload_images/1112615-a149d1f9f097570a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上就是CGLib实现代理模式的代码。

# 小结
代理模式是重要的设计模式之一，有了代理模式，后面，也就有了著名的面向切面编程，_Aspect Oriented Programming(AOP)_。是Spring AOP的基石。希望大家看完有所收获

感兴趣的可以再了解一下CGLib，因为深入到字节码，CGLib能干的事情非常多，是一个非常强的Java库。

[我的博客   Zing ，欢迎点击，来踩](https://micorochio.github.io)
____
love & peace
[FS全栈计划目录：https://micorochio.github.io/fs-plan/](https://micorochio.github.io/fs-plan/)
