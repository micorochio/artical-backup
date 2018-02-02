---
title:
- (FS计划)SpringMVC的IoC容器对bean的管理
date:
- 2016-07-28 00:03:50
tags:
- Java
- Spring
- IoC
---

首先，加载bean需要先写好配置文件（其实用注解也可以，只是不方便理解）

## 配置

```xml
<beans>
    <!--导入其他资源XML-->
    <import resource=”resource1.xml”/>
    <!--Bean定义，class定义Bean的实现类，id称为标识符，其余id为别名-->
    <bean id=”bean1”class=””></bean>
    <bean id=”bean2”class=””></bean>
    <!--指定name，这样name就是“标识符”-->
    <bean name=”bean2”class=””></bean>
    <!--alias为bean指定别名-->
    <alias alias="bean3" name="bean2"/>
    <!--导入其他依赖-->
    <import resource=”resource2.xml”/>
</beans>
```

多个配置文件是可以编制成数组，交给ApplicationContext加载的。
但是大家现在用习惯了注解


SpringMVC的IoC容器中，管理的bean 内部表示为[BeanDefinition](https://github.com/spring-projects/spring-framework/blob/master/spring-beans%2Fsrc%2Fmain%2Fjava%2Forg%2Fspringframework%2Fbeans%2Ffactory%2Fconfig%2FBeanDefinition.java)
> 
**一个bean需要以下属性**
+ 全限定类名（FQN）：用于定义Bean的实现类;
+ Bean行为定义：这些定义了Bean在容器中的行为；包括作用域（单例、原型创建）、是否惰性初始化及生命周期等；
+ Bean创建方式定义：说明是通过构造器还是工厂方法创建Bean；
+ Bean之间关系定义：即对其他bean的引用，也就是依赖关系定义，这些引用bean也可以称之为同事bean 或依赖bean，也就是依赖注入。

bean使用时必须有一个唯一的指定：
id必须唯一不可重复，一个bean可以有多个id，其余的id是别名；
没有id可以用唯一不重复的name，若有需要可以使用alias来指定别名，或者用`,`隔开；
同时指定id和name，id就是标识符，而name就是别名，总之必须在IoC容器中唯一；
若id和name都没有指定，则bean在使用的时候会使用全限定的类名进行注入。

## 实例化
SpringMVC是通过反射来实例化bean的。
**1.通过构造器实例化bean：**
Spring IoC容器即能使用默认空构造器也能使用有参数构造器两种方式创建Bean，无参数的直接在XML文件中直接定义bean节点，带参数的可以在XML文件中bean节点下增加`constructor-arg`参数属性：
```xml
<bean name="test" class="com.zing.test.HelloMvc">
<!-- 指定构造器参数 -->
    <constructor-arg index="0" value="第一个参数"/>
</bean>
```
index是参数所在的位置，value表示注入常量值，也可以指定引用，指定引用使用ref来引用另一个Bean定义，当然注入还可以根据参数类型和名字来配置

```xml
<!-- 根据变量名注入 -->
    <constructor-arg name="testIoC" value="第一个参数"/>
<!-- 根据参数类型注入-->
    <constructor-arg type="java.lang.String" value="第一个参数"/>
```

```xml
<!--构造器里注入另一个bean-->
<!-- 通过构造器注入 -->
<bean id="helloApi" class="com.zing.test.NormalBean"/>
<bean id="bean2" class="com.zing.test.TestProperty">
    <property name="helloApi"><ref bean=" helloApi"/></property>
</bean>
```



**2.使用静态工厂来实例化bean**
需要编写工厂方法，这样实例化的时候即可使用工厂方法来实例化bean

```java
package com.zing.test.factory_test;

import com.zing.test.hello.HelloImpl;
//工厂类
public class HelloIoCFactory {
    public static HelloImpl newInstance(String name) {
        System.out.println(name);
        return new HelloImpl(name);
    }
}
```
第一种工厂配置

```xml
<!--为bean配置工厂方法-->
<bean class="com.zing.test.factory_test.HelloIoCFactory" id="helloIoC" factory-method="newInstance">
    <constructor-arg index="0" value="哈哈"/>
</bean>
```
第二种工厂配置

```xml
<bean id="instanceFactory" class="com.zing.test.factory_test.HelloIoCFactory"/>
<bean id="byIndex" factory-bean="instanceFactory"  factory-method="newInstance">
    <constructor-arg index="0" value="Hello World!"/>
    <constructor-arg index="1" value="1"/>
</bean>
```

```java
//通过工厂获取实例
BeanFactory beanFactory = new ClassPathXmlApplicationContext("spring-config.xml");
HelloInterface hello = beanFactory.getBean("helloIoC",HelloImpl.class);
hello.sayHello();
```

**3.通过setter方法进行注入**
个人觉得：有必要吗？都是用注解的好不好，就列一下，不详细记录了

```xml
<!-- 通过setter方式进行依赖注入 -->
    <bean id="bean" class="com.zing.test.factory_test.HelloImpl4">
        <property name="message" value="Hello World!"/>
        <property name="index">
            <value>1</value>
        </property>
    </bean>
```

**4.延时实例化**
延时初始化又叫惰性初始化，作用是消除资源占用巨大的bean在程序初始化时的影响，防止因为这个bean，导致启动缓慢。延迟初始化的Bean通常会在第一次使用时被初始化；
配置如下：
```xml
<!-- 通过配置lazy-init="true"实现惰性初始化 -->
<bean id="helloApi" class="com.zing.test.HelloImpl lazy-init="true"/>
```

## 自动装配
自动装配就是指由Spring来自动地注入依赖对象，无需人工参与。

目前Spring3.0支持`no`、`byName`、`byType`、`constructor`四种自动装配，默认是“no”指不支持自动装配的，其中Spring3.0已不推荐使用之前版本的`autodetect`自动装配，推荐使用Java 5+支持的（`@Autowired`）注解方式代替；

```xml
<bean id="helloApi" class="com.zing.test.HelloImpl"/>
<bean id="bean" class="com.zing.test.bean.HelloApiDecorator" autowire="byName"/>
```

+ `byName`使用setter方法根据参数名称注入
+ 根据类型注入，**在根据类型注入时，将把当前自己排除在外！**如果是`List`，`Map`这样的类型，用`autowire-candidate`属性为`false`来让指定的`Bean`放弃作为自动装配的候选者。使用`primary`属性为`true`来指定某个`Bean`为首选`Bean`
+ `constructor`构造器注入，只用在构造器中。
+ _`autodetect`自动匹配，不推荐使用！_

> 不是所有的类型都支持自动匹配，**`Object`、基本数据类型（`Date、CharSequence、Number、URI、URL、Class、int`**不支持
`<bean>`标签的autowire-candidate属性可被设为false，这个bean将不能作为可依赖注入的参数
**数组类型、集合（`Set、Collection、List`）接口类型**：将根据泛型获取匹配的所有候选者并注入到数组或集合中
**字典（Map）接口类型：**同样根据泛型信息注入，键必须为`String`类型的`Bean`名字，值根据泛型信息获取


**缺点**
> 如果你用的是自动装配的话，出了问题会比较难找。不知道是哪里注入进来的，唉，这就是我等菜比为什么要先从配置开始看。现在流行用注解，会好不少。

## 依赖检查
用于检查Bean定义的属性都注入数据了，不管是自动装配的还是配置方式注入的都能检查，如果没有注入数据将报错，从而提前发现注入错误，只检查具有setter方法的属性。
对于注解来说没什么用，简单介绍一下

```xml
<bean id="test"/>
<!-- 注意我们没有注入，所以测试时会报错 -->
<bean id="bean" dependency-check="objects">
  <property name="message" value="Haha"/>
</bean>
```
**`dependency-check`**配置检查类型，一共有4种：`null，object，simple，all`
null=不检查，为默认配置。
object=检查除基本类型外的依赖对象。
simple=对基本类型进行依赖检查，包括数组类型，其他依赖不报错；
all=所有类型都检查

----
[FS全栈计划目录：https://micorochio.github.io/fs-plan/](https://micorochio.github.io/fs-plan/)