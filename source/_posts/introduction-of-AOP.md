---
title:
- (FS计划)AOP和Spring中AOP的简单介绍
date:
- 2017-02-25 00:05:48
tags:
- Java
- Spring
- AOP
---
![](http://upload-images.jianshu.io/upload_images/1112615-f513cbc11f698921.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 0x00 AOP是什么
AOP _`(Aspect Oriented Programming)`_面向切面编程，Spring是通过Java动态代理实现的。

AOP的目的是为了将组件和业务代码分离开，让日志、事务处理、缓存、性能统计、权限控制等等模块跟实际的业务模块解耦。

这么说还是很笼统，Java的类中，属性和方法才是主流，切面是什么，完全没这个概念。
那么以拍电影举个栗子：
```
1.前期筹备，剧本确定、演员试镜确定
############分割线a
2. 开拍、剪辑、制作
############分割线b
3. 营销、宣传、公映
```

1->2->3这个流程是拍电影的业务流程，但是如果我要拍花絮怎么办，这个流程不属于现在的电影业务流程，我们需要安排一个零时的业务，那我们就要在`############分割线a
`分割线那里添加摄花絮的计划，分配一些人员。
那么分割线那里，就是切面。这些切面干什么，由切面代理去管理和运行，这就是AOP。
<!--more-->

## 0x01 AOP的用途
来说说为什么要用AOP：
AOP中面向切面编程的这个切面就是针对流程的。当某些服务与业务无关，比如日志系统，日志系统与现实业务应该是相互独立的，但是，在编写代码的时候我们需要这些日志来调试定位程序执行的位置。
这些日志代码如果直接写在程序里会扰乱原本的业务模块，会影响到系统的持续集成

因此我们希望日志系统能够独立于业务代码，但是每当业务执行到某个位置时，日志又会输出。这就要借助于AOP了_（想象一下，业务代码被切开了，中间插入了日志执行的标记，走到标记时就会输出日志，这个标记就是切面）_

当然，还有业务中登录状态验证，权限检查，全局的事务处理，共用的某些模块，都可以用AOP来实现，这是OOP的进步

# 0x02 代理模式
Java动态代理是基于代理模式的一种实现，可以查询一下代理模式，这里将动态代理的对应关系图贴一下

![java中的动态代理模式](http://upload-images.jianshu.io/upload_images/1112615-c3cc7c540bc86019.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们通过代理接口Subject去执行RealSubject的subjectMethod()方法，而java的动态代理是，通过InvocationHandler中invoke()方法去调用Proxy事先生成好实例对象。说的有点绕口，配合看图。
再不明白：可以看我的：[(FS计划)Java里的动态代理模式和代理方法增强](http://www.jianshu.com/p/893ffc0b7a73)
或者：[看别人的详细介绍](http://www.cnblogs.com/xiaoluo501395377/p/3383130.html)

## 0x03 Spring里的动态代理
因为Spring中拥有了DI的高级特性，所以在Spring中实现动态代理是可以依靠配置Bean来实现的。Spring中的动态代理依赖下面几个接口和类
```java
import org.springframework.aop.BeforeAdvice;//前置通知
import org.springframework.aop.AfterAdvice;//后置通知
import org.springframework.aop.ThrowsAdvice;//异常通知
import org.aopalliance.intercept.MethodInterceptor;//环绕通知
import org.springframework.aop.IntroductionAdvisor;//引介通知
```

可以写一个例子来看看
+ 接口

```java
package com.zing.aoptest;

public interface IRocketLaunching {
    boolean launch();
}
```
+ 目标对象 

```java
package com.zing.aoptest;

public class RocketLaunchingImpl implements IRocketLaunching {
    @Override
    public boolean launch() {
        System.out.println("倒计时：987654321 发射！");
        return false;
    }
}
```
+ 织入类  

```java
package com.zing.aoptest;

import java.lang.reflect.Method;
import org.springframework.aop.MethodBeforeAdvice;

public class PrepareForLaunching implements MethodBeforeAdvice {

    public void before(Method method, Object[] params, Object obj) throws Throwable {
        System.out.println("给火箭家燃料\n");
    }
}
```
+ 代理对象 

交给Spring 来做
```xml

    <!--前置增强类-->
    <bean id="beforeLaunch"  class="com.zing.aoptest.PrepareForLaunching"></bean>
    <!--目标类-->
    <bean id="launchingControl" class="com.zing.aoptest.RocketLaunchingImpl"></bean>

    <!--工厂-->
    <!--optimize为true表示使用CGLib，false为Java动态代理。默认为false-->
    <!--当织入对象需要使用单例模式的时候，optimize建议为true，这样比较节省资源-->
    <!--target表示需要代理的类，会为这个目标织入前置增强和后置增强-->
    <bean id="rocketProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target" ref="launchingControl"></property>
        <property name="proxyInterfaces" value="com.zing.aoptest.IRocketLaunching"></property>
        <property name="interceptorNames" value="beforeLaunch"></property>
        <property name="proxyTargetClass" value="true"></property>
    </bean>
```
+ 测试一下 
  
```
package com.zing.aoptest;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.testng.annotations.Test;

public class AOPTest {

    @Test
    public  void AopTest() {
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring/spring-dao.xml");

        RocketLaunchingImpl rocketLaunching = (RocketLaunchingImpl) context.getBean("rocketProxy");

        rocketLaunching.launch();

    }
}
```

测试结果

![result](http://upload-images.jianshu.io/upload_images/1112615-eeeff8cf57d0f994.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 0x04 Spring 里的AOP简单介绍
Spring的AOP是基于JDK的代理模式和CGLib为技术基础，完成织入横切代码的一项功能。

朕要去撸一个AOP的代码例子：
****
例子好了，但是依旧没搞清楚AOP配置的语法，先不急

放上例子
```
package com.zing.aspect_oriented_test;
/**
 * 方法切面
 */
public class SubjectService implements InterfaceSubjectService {
    @Override
    public void AspectMethod(String str) {
        System.out.println("yo yo yo!\t" + str);
    }
}
```

```
package com.zing.aspect_oriented_test;
/**
 * 对切面的处理，之前与之后
 */public class AspectTarget {
    public void beforeYouTalk() {
        System.out.println("************beforeYouTalk, I know everything,so do not lie!");
    }
    private void afterYouTalk() {
        System.out.println("************afterYouTalk, ha ha ha,good boy");
    }
}
```

我们知道了切面和切面处理方法，接下来告诉Spring，切面位置和处理方法
在XML配置文件中将两个bean配置到IoC容器中
```
<bean id="aspectService" class="com.zing.aspect_oriented_test.SubjectService"></bean>
<bean id="aspect" class="com.zing.aspect_oriented_test.AspectTarget"></bean>
```
再配置切面和切面方法
```
<aop:config>
    <aop:pointcut id="pointCut" expression="execution(* com.zing...*.*(..))"></aop:pointcut>
    <aop:aspect ref="aspect">
        <aop:before pointcut-ref="pointCut" method="beforeYouTalk"></aop:before>
        <aop:after pointcut="execution(* com.zing..*.*(..))"  method="afterYouTalk"></aop:after>
    </aop:aspect>
</aop:config>
```

上面的配置可能不太明白，因为用的是AspectJ语法，`* com.zing..*.*(..)`表示`com.zing`包下的所有子包的类与方法

例子撸完，写个Junit测试一下
```
package com.zing.aspect_oriented_test;

import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
/**
 * Created by zing on 16/4/23.
 */
public class AspectJunitTest {
    @Test
    public void AopTest(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-config.xml");
        InterfaceSubjectService subjectService = applicationContext.getBean("aspectService",InterfaceSubjectService.class);
        subjectService.AspectMethod("Monster");
    }
}
```
运行结果
```
************beforeYouTalk, I know everything,so do not lie!
yo yo yo!	Monster
************afterYouTalk, ha ha ha,good boy
```

![还是上图吧](http://upload-images.jianshu.io/upload_images/1112615-f10097b73ad03a88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码没问题，_（全程注意你引得包不要出问题，可以参考我之前的小白文[IDEA 构建SpringMVC+MAVEN项目](http://www.jianshu.com/p/2101d176555b)）_下面聊一下AOP的配置

吐槽：觉得XML配置好烦人
这篇再写就累赘了，算了，这个Spring的动态代理的写法并不常用，配置写起来也异常麻烦，后面会详细介绍AOP的常规使用方法，敬请期待

如果有错误，麻烦留言告诉我。转载请注明出处：https://micorochio.github.io/2017/02/25/introduction-of-AOP/

----
[FS全栈计划目录：https://micorochio.github.io/fs-plan/](https://micorochio.github.io/fs-plan/)