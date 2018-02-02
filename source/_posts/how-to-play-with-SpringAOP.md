---
title:
- (FS计划)理清楚Spring的AOP到底怎么玩
date:
- 2017-02-28 00:03:50
tags:
- Java
- Spring
- AOP
---

![](http://upload-images.jianshu.io/upload_images/1112615-6895c50cea7f837d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
前面简单介绍了Spring中AOP的使用，是基于XML配置。这次详细介绍一下Spring中AOP的使用和实现。
<!--more-->
## 0x00 回顾
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

想必看完云里雾里的，这里面用到了AOP的配置，完全没有概念，接下来详细解释一下，配置的含义

## 0x01 Aspect切面
Aspect，是织入的节点，前面两篇文章已经介绍了代理，而代理的节点，就是用Aspect来标记的，因为Spring封装了优秀的AspectJ解决方案，Aspect作为一个既定的接口，被Spring扩展了多个具体类型的通知`BeforeAdvice`,`AfterAdvice`,`ThrowsAdvice`

从`BeforeAdvice`中分析，Spring中将前置接口设定为`MethodBeforeAdvice`，这个接口中只需要实现`before`方法
```
public interface MethodBeforeAdvice extends BeforeAdvice {
    void before(Method var1, Object[] var2, Object var3) throws Throwable;
}
```

这个方法将会在目标方法执行前被回调。

同样`AfterAdvice`是后置通知，具体的继承有`AfterReturningAdvice`
```
public interface AfterReturningAdvice extends AfterAdvice {
    void afterReturning(Object var1, Method var2, Object[] var3, Object var4) throws Throwable;
}
```
实现这个接口的`afterReturning`方法，在目标方法成功执行并返回值之后，AOP会回调`afterReturning`方法

最后`ThrowsAdvice`，其实是`AfterAdvice`子接口，在目标方法执行发生异常时，会被回调。
具体可以这么实现
```

class BoomException implements ThrowsAdvice{
    public void afterThrowing(IOException ioEx){
        System.out.println("IO异常:"+ioEx.getMessage());
    }

    public void afterThrowing(ClassCastException ccEx){
        System.out.println("转换异常:"+ccEx.getMessage());
    }
}
```

## 0x02 Pointcut切点
切点定义了一个代理接入位置，决定了通知作用的连接点。当然，也可以是一堆连接点，一般用一个正则表达式标识。

```
public interface Pointcut {
    Pointcut TRUE = TruePointcut.INSTANCE;

    ClassFilter getClassFilter();

    MethodMatcher getMethodMatcher();
}
```
其中`getMethodMatcher()`就是获取一个方法过滤器，这个方法过滤器将符合标准的方法，作为切面的连接点。
关于MethodMatcher，可以查看一下Spring源码。这里不做过多解释。

## 0x03 Advisor通知器
配置中并没有写Advisor，所以简单介绍一下，一个完整的模块，当要进行AOP编程时，需要将方法标记为切面，并定义了切面前置通知、后置通知、异常通知。定义完成，需要通过通知器，将切面和通知绑定起来，这个通知器就是Advisor。

Advisor将Advice和Pointcut结合起来，通过IoC容器来配置AOP来使用。

## 0x04 ProxyFactoryBean
ProxyFactoryBean是Spring利用Java的代理模式或者CGLIB来实现Aop的一种方式，如何在XML中配置ProxyFactoryBean？
+ 通知器Advisor使用Bean来配置。
+ 织入方法类使用Bean配置
+ 定义ProxyFactoryBean，为这个bean配置几个参数：
  + 目标：target
  + 代理接口：proxyInterface
  + 织入类：interceptName

如果不清楚可以看[AOP和Spring中AOP的简单介绍](http://www.jianshu.com/p/d03c6c8daab9)中的第0x03小节的例子。这里就扣下代码，里面还有配置后置方法织入，异常方法通知织入，不一一介绍。
```xml
  <bean id="aspectService" class="com.zing.aspect_oriented_test.SubjectService"></bean>
  <bean id="aspect" class="com.zing.aspect_oriented_test.AspectTarget"></bean>

  <bean id="rocketProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target" ref="launchingControl"></property>
        <property name="proxyInterfaces" value="com.zing.aoptest.IRocketLaunching"></property>
        <property name="interceptorNames" value="beforeLaunch"></property>
        <property name="proxyTargetClass" value="true"></property>
    </bean>
```
因为ProxyFactoryBean是依靠Java或CGLIB的Proxy方式来获取对象的，使用依靠代理的`getObject()`方法来作为入口。使用接下来看一下这个方法的实现方式
```
public Object getObject() throws BeansException {
//初始化通知器
        this.initializeAdvisorChain();
//区分单例模式和原始模式prototype
        if(this.isSingleton()) {
            return this.getSingletonInstance();
        } else {
            if(this.targetName == null) {
                this.logger.warn("Using non-singleton proxies with singleton targets is often undesirable. Enable prototype proxies by setting the \'targetName\' property.");
            }
            return this.newPrototypeInstance();
        }
    }
```
具体可以追一追`newPrototypeInstance()`和`getSingletonInstance()`两个方法，得到实现方式的完整过程。留给感兴趣的小伙伴。因为再往里挖就挖到CGLIB和JDK对象生成里去了，感觉刨过头了。

##0x05 Schema的AOP配置
前面啰嗦了一大堆，我觉得应该介绍一下具体的配置方式

```xml
 <!--aop定义开始-->
    <aop:config>
        <!--定义通知器-->
        <aop:advisor ref="aspectSupportBean"></aop:advisor>
        <!--定义切面 ref表示引用的bean-->
        <aop:aspect ref="aspectSupportBean">
            <!--定义切面增强位置-->
            <aop:pointcut id="pcut" expression="execution(* cn.javass..*.*(..))" ></aop:pointcut>
            <!--前置通知，下面的参数跟第一个类似-->
            <aop:before pointcut="切入点表达式" pointcut-ref="切入点Bean引用"  method="前置通知实现方法名" arg-names="前置通知实现方法参数列表参数名字"/>
            <!--后置返回通知-->
            <aop:after-returning></aop:after-returning>
            <!--异常通知-->
            <aop:after-throwing></aop:after-throwing>
            <!--最终通知-->
            <aop:after></aop:after>
            <!--环绕通知-->
            <aop:around></aop:around>
            <!--引入定义-->
            <aop:declare-parents  types-matching="AspectJ语法类型表达式" implement-interface=引入的接口"  default-impl="引入接口的默认实现"  delegate-ref="引入接口的默认实现Bean引用"/>
        </aop:aspect>
    </aop:config>
```
如果你看完前面的东西应该不难理解这些配置
但是有一个execution，为此查阅了一下文档，切入点指示符。执行表达式的格式如下：
```
execution（modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern（param-pattern） throws-pattern?）
```
除了返回类型模式（上面代码片断中的ret-type-pattern），名字模式和参数模式以外， 所有的部分都是可选的。返回类型模式决定了方法的返回类型必须依次匹配一个连接点。 你会使用的最频繁的返回类型模式是`*`，它代表了匹配任意的返回类型。 一个全限定的类型名将只会匹配返回给定类型的方法。名字模式匹配的是方法名。 
你可以使用`*`通配符作为所有或者部分命名模式。 参数模式稍微有点复杂：()

匹配了一个不接受任何参数的方法， 而(..)

匹配了一个接受任意数量参数的方法（零或者更多）。 模式(*)

匹配了一个接受一个任何类型的参数的方法。 模式(*,String)

匹配了一个接受两个参数的方法，第一个可以是任意类型， 第二个则必须是String类型。

## 0x06 @AspectJ的AOP

是基于注解的AOP，默认Spring是不开启的，需要再XML里添加一行配置
```xml
<aop:aspectj-autoproxy/>
```
之后便可以用注解的方式使用AOP了,我列举一下
```
//配置切面
@Aspect() 
Public class Aspect{ 
…… 
}
```
```
//配置织入方法
@Pointcut(value="切入点表达式", argNames = "参数名列表") 
public void pointcutName(……) {}
```
```java
//前置通知
@Before(value = "切入点表达式或命名切入点", argNames = "参数列表参数名")

//后置返回通知
@AfterReturning( 
value="切入点表达式或命名切入点", 
pointcut="切入点表达式或命名切入点", 
argNames="参数列表参数名", 
returning="返回值对应参数名") 
// value：指定切入点表达式或命名切入点；
// pointcut：同样是指定切入点表达式或命名切入点，如果指定了将覆盖value属性指定的，pointcut具有高优先级；
// argNames：与Schema方式配置中的同义；
// returning：与Schema方式配置中的同义。

//后置通知
@After ( 
value="切入点表达式或命名切入点", 
argNames="参数列表参数名") 
//value：指定切入点表达式或命名切入点；
// argNames：与Schema方式配置中的同义；


//异常通知
@AfterThrowing ( 
value="切入点表达式或命名切入点", 
pointcut="切入点表达式或命名切入点", 
argNames="参数列表参数名", 
throwing="异常对应参数名")

//环绕通知
@Around ( 
value="切入点表达式或命名切入点", 
argNames="参数列表参数名")

//引用
@DeclareParents( 
value=" AspectJ语法类型表达式", 
defaultImpl=引入接口的默认实现类) 
private Interface MyInterface;
```
只是简化了配置，用起来跟Schema类似



##参考
http://www.importnew.com/17795.html
http://www.importnew.com/17813.html
《Spring 内幕技术》

----
[FS全栈计划目录：https://micorochio.github.io/fs-plan/](https://micorochio.github.io/fs-plan/)