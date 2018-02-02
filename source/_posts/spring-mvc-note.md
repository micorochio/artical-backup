---
title:
- SpringMVC快速学习笔记
date:
- 2017年03月19日17:59:09
tags:
- Java
- Spring
- SpringMVC
---

![](http://upload-images.jianshu.io/upload_images/1112615-306b3175e2265614.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Spring MVC的工作流程

1 用户请求
2 前端控制器
3 handlerMapping 找到对应的Controller
4 controller 返回执行链
5 前端控制器通过 HandlerAdapter去执行执行链，返回model and view
6 将model and view交给视图渲染器，渲染成视图
<!--more-->
## 0x01使用配置

配置前端控制器,在Web.xml中添加
```xml
<servlet>
    <servlet-name>dispatcher-servlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--集成Mybatis-->
    <!--集成Spring-->
    <!--集成SpringMvc-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring/spring-*.xml</param-value>
    </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcher-servlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
```

配置完成后，在classpath目录冲创建 spring文件夹，文件夹中创建spring Mvc配置文件


![分层配置](http://upload-images.jianshu.io/upload_images/1112615-ffdfafceb94cc87a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

先不考虑service层和dao层，有了前端控制器，就需要配置HandlerMapping 映射器和HandlerAdapter 适配器，最后是视图解析器

为了简化配置，DefaultAnnotationHandlerMapping和AnnotationMethodHandlerAdapter被mvc标签代替
所以 spring-web.xml文件如下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--开启注解模式-->
    <!--简化注解：
        1：自动注册DefaultAnnotationHandlerMapping和AnnotationMethodHandlerAdapter
        2：提供了一系列功能（数据绑定，数字和日期转换 @NumberFormat、@DateTimeFormat） xml和json的读写
    -->
    <mvc:annotation-driven/>


    <!--
        静态资源默认Servlet配置
        对静态资源请求的支持：图片，css，js
        允许使用"/"做整体映射
    -->
    <mvc:default-servlet-handler/>

    <!--静态资源映射-->
    <mvc:resources mapping="/rs/**" location="/WEB-INF/static/"/>

    <!--配置视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/view/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--配置自动扫描bean 如果和Spring其他组件一起使用，不建议全局扫描，只扫描controller就好了-->
    <context:component-scan base-package="com.zing"/>

</beans>
````

## 0x02 Controller
+ URL映射
简单URL映射的配置：
 
```xml
<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">  
    <property name="mappings">  
        <props>  
            <prop key="/userInfo">ControllerA</prop>  
        </props>  
    </property>  
</bean>  
      
<bean id="ControllerA" class="com.zing.controller.ControllerDemo"></bean>
```
不过现在一律使用注解式映射

```java
@Controller()
@RequestMapping("/user")
public class AccountController {

    @RequestMapping(value = "/login")
    public String login(HttpSession session) {
      ...
    }
}
```
通过RequestMapping来限定url访问的方法，注意的是@Controller()和@RequestMapping()是需要成对出现的，否则无法访问，另外RequestMapping还有延伸，通过规范请求方式，请求头等来限定用户请求。

+ 参数的绑定和自定义参数绑定
Spring MVC 自带参数绑定的功能，就是根据request中的key/value数据，找到对应名称和类型的入参，将参数绑定到对应的pojo对象上，特殊类型例如Date，可以通过自定义参数绑定，来讲参数绑定到pojo上

写几个案例：
```html
 <form action="/user/signUp" method="post">
    <input type="text" placeholder="邮箱  name="email"/>
 </form>
```
```java
//注入1
    @PostMapping("user/signUp")
    public static String getToken(String email) {
      ...
    }

//注入2
   @PostMapping("user/signUp")
    public static String getToken(@RequestParam(“email”)String name) {
      ...
    }
```
还有N多注入方式，其原理大致一样：有兴趣百度：SpringMVC参数绑定
自定义参数绑定的方式是参数按照用户自己编写转换器处理后绑定
```xml
<bean id= "convertersBinder" class= "org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <property name= "converters">
       <list>
        <bean class= "com.zing.converter.Converter01"/>
        <bean class= "com.zing,converter.Converter02"/>
        <bean class= "com.zing,converter.Converter03"/>
        </list>
    </property>
</bean>
```

```java
package com.zing.account.interceptor;

import org.springframework.core.convert.converter.Converter;

public class Converter01 implements Converter<I,O> {
    @Override
    public O convert(I input) {
//        将input处理后生成O类型输出
        return new O();
    }
}
```

这样前端的字符串，就可以转换成其他任意类型了。

## 0x03 SpringMVC对事务的支持
service一般用来处理数据逻辑业务，校验数据等。这一层属于业务逻辑层，所以在某些业务上需要进行事物处理.

如何使用SpringMVC的事务管理呢，有两种方法
1. 开启事务注解
```xml
  <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--数据库连接池，集成之后就不会出现红色-->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--开启注解式事物-->
    <tx:annotation-driven transaction-manager="transactionManager"/>
```
使用
```java
//注册Service接口中的注解式事物
@Transactional(readOnly = false, rollbackFor = DataAccessException.class)   
    Account register(Account account);   
```

2. 配置AOP自动事务管理
```xml

    <!--切面通知-->
    <tx:advice id="transactionInterceptor" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="save*" propagation="REQUIRED"/>
            <tx:method name="insert*" propagation="REQUIRED"/>
            <tx:method name="update*" propagation="REQUIRED"/>
            <tx:method name="delete*" propagation="REQUIRED"/>
            <tx:method name="select*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="get*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
        </tx:attributes>
    </tx:advice>

    <!--配置AOP-->
    <aop:config>
        <aop:advisor advice-ref="transactionInterceptor"
                     pointcut="execution(* com.zing.*.service.*.*(..))"></aop:advisor>
    </aop:config>
```
前者简单，后者可以规范代码。都可以选择。

## 0x04 RESTful链接和静态资源的支持
可以将URL的链接某部分作为参数传入方法
![](http://upload-images.jianshu.io/upload_images/1112615-e6f7447b1d9f4860.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

静态资源可以用`<mvc:resource >`标签来映射，否则可能会会被拦截

![](http://upload-images.jianshu.io/upload_images/1112615-769800662cdea7cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如图的配置，表示将WEB-INF下static文件夹用/rs/来映射，这样就可以通过
`http://localhost:8000/rs/js/jquery.min.js`来访问static目录下/js/jquery.min.js文件了

## 0x05 拦截器
Spring拦截器，编写一个类实现`org.springframework.web.servlet.HandlerInterceptor;
`就可以作为拦截器使用了

拦截器中有3个方法
```java
//请求到达方法前执行
boolean preHandle(HttpServletRequest var1, HttpServletResponse var2, Object var3) throws Exception;

//执行方法后，没有返回ModuleAndView之前，可以执行
    void postHandle(HttpServletRequest var1, HttpServletResponse var2, Object var3, ModelAndView var4) throws Exception;

//方法执行后执行
    void afterCompletion(HttpServletRequest var1, HttpServletResponse var2, Object var3, Exception var4) throws Exception;

```

配置拦截器方式
```xml
<mvc:interceptors>
  <mvc:interceptor>
    <mvc:mapping path="/**"/>
    <mvc:exclude-mapping path="/rs/**"/>
    <mvc:exclude-mapping path="/getCaptcha"/>
    <mvc:exclude-mapping path="/user/**"/>    <bean class="com.zing.account.interceptor.LoginInterceptor"/>
  </mvc:interceptor>
</mvc:interceptors>
```

## 0x06全局异常的处理 ExceptionResolver
Spring中有现成的全局异常处理解决方法
```java
public class MyExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {

        if (e instanceof MyException) {
            //处理逻辑
        } else {
            //处理逻辑
        }
        return null;
    }
}
```
可以看到返回的依旧是 ModelAndView ，处理完成后，可将想返回给用户的友好的界面。

将这个全局异常处理器配置到xml中, 直接当Bean放进去就行，Spring会默认将异常处理交给这个Resolver。
```xml
<bean class="com.zing.exp-resolver.MyExceptionResolver"></bean>
```



