---
title:
- JSP学习笔记
date:
- 2016-08-25 22:17:53
tags:
- JSP
- Java
- Web
---

## 前言
#### 1.Tomcat的目录结构
|||
|----|----|
|bin：      |存放各种平台的启动、停止等命令|
|conf：     |存放服务器的各种配置文件|
|lib：      |存放服务器需要的jar包|
|log：      |服务器的日志|
|temp：     |服务器启动时，存放临时文件
|webapps：  |Web应用文件存放于此
|work：     |Tomcat吧JSP生成的servlet存放在这个文件夹下
<!-- more -->
#### 2. Web应用目录

WEB-INF外，存放公共资源，一般为页面、图片等


WEB-INF目录：是JavaWeb程序的安全目录，非客户端无法访问，只有服务端可以访问<br>
classes:  java文件生成的字节码<br>
lib：      Web应用依赖的jar包<br>
web.xml：  项目部署文件，映射JSP和servlet<br>

虚拟路径: 默认的是文件的路径，可以通过修改Web content-root来更改虚拟路径

#### 3. 修改Tomcat的默认端口

进入Tomcat安装目录的conf文件夹，打开server.xml

修改8080，定义为自己选择的端口号
```
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />

```

## JSP基础语法
#### 页面元素构成
1.静态内容：
2.注释
3.声明
4.脚本
5.表达式
6.指令


> 指令

+ page指令：
```
  <%@ page 属性1="值1" 属性2="值2,值3"%>
```
language      指定jsp的脚本予语言，默认是Java
import        引用脚本语言中使用到的类文件
contentType   指定JSP的编码方式 默认是ISO-8859-1

+ include指令：引用其他文件
+ taglib指令：指定jsp标签库

> 注释

HTML注释

JSP注释(客户端不可见)
`<%--注释--%>`

JAVA注释(客户端不可见)
```
//单行注释
/*
多行注释
*/
```
> 脚本

<br>
在JSP中执行的Java代码
```
<%
out.println("Hello World")
%>
```

> JSP 声明

在`<%!  %>`中间的就是JSP声明
可以用来声明变量和函数

```
<%!
 int a = 1;

 int add(int x , int y){
   return x + y;
 }
%>
```

##  JSP表达式
在`<%=   %>`中间的，就是表达式
> 表达式不以分号`';'`为结束

如：
```
<%!
 int a = 1;

 int add(int x , int y){
   return x + y;
 }
%>

//表达式
<%= a%>
<%= add(3,4)%>
```

## JSP的生命周期
![JSP生命周期图](http://upload-images.jianshu.io/upload_images/1112615-2fa98119a5313afa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

JSPService()方法是一个多线程调用的方法，虽然快，但是容易出现线程同步问题。
Servlet常驻内存，所以响应还是很快的

JSP被修改之后，JSP引擎需要重新编译JSP文件，重新生成Servlet


## JSP的内置对象
 1.out
 2.request
 3.response
 4.session
 5.application
 6.page
 7.pageContext
 8.exception
 9.config

> out对象

|常用方法||
|----|----|
| void println()        |向客户端打印字符串|
| void clear()          |清除缓存区的内容，flush之后调用会报错|
| void clearBuffer()    |清除缓冲区的内容，flush之后调用不会异常|
| void flush()          |将缓冲区的内容输出到客户端|
| void getBufferSize()  |返回缓冲区字节数大小|
| void getRemaining()   |缓冲区还有多少可用|
| boolean isAutoFlush() |缓冲区满了会不会报异常|
| void close()          |关闭输出流|

> request对象

客户端的请求，是HttpServletRequest类的实例

|常用方法||
|----|----|
| String getParameter(String name)            |返回name被指定的参数|
| String[] getParameterValues(String name)    |返回name指定的参数们|
| void setAttribute(String name)              |存储指定name属性的值|
| Object getAttribute(String name)            |返回name下的属性值|
| String getContentType()                     |得到请求体中的MIME类型|
| String getProtocol()                        |返回请求用的协议类型和主机名|
| String getServerName()                      |返回接受请求的服务器主机名|
| int getServerPort()                         |获得服务器接受请求的端口号|
| String getCharacterEncoding()               |返回字符编码方式|
| void setCharacterEncoding()                 |设置请求的字符编码|
| int getContentLength()                      |返回请求的长度|
| String getRemoteAddress()                   |返回发送此请求的客户端IP地址|
| String getRealPath()                        |返回虚拟路径的真实路径|
| String getContextPath()                     |返回上下文路径|


*URL传递中文参数，可在Tomcat下设置*
```
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443"  URIEncoding="utf-8"/>
```


> response对象

很少使用，是HttpServletResponse的对象，具有页作用域，只对当前页面请求有作用

|常用方法：||
|-----|-----|
| String getCharacterEncoding()       |返回响应用的字符编码
| void setContentType(String type)    |设置响应的MIME类型
| PrintWriter getWriter()  |返回可以向客户端输出字符的一个对象，总是提前内置out对象进行输出（与out对象有区别）|
| void sendRedirect(String location)  |重定向用户请求到location|

重定向和请求转发
重定向：客户端行为，相当于两次请求，地址栏会发生变化
请求转发：服务器行为，转发后请求对象会保存


> session对象

session的意义是一次对话，在Web中表示用户浏览网站的时间，session只会存在一段时间，超过时间后session就会消失

|常用方法：||
|----|----|
|long getCreationTime()   |返回Session创建时间
|String getId()  |                     sesion创建时，JSP分配给Session的ID|
|Object setAttribute(String name, Object value) |使用指定的名称将对象绑定到Session
|Object getAttribute(String name)      |获取Session中name属性的值，没有则为null|
|String[] getValueNames()              |返回Session中可用属性的名称
|int getMaxInactiveInterval()          |返回两次请求间隔多长时间，Session被取消|
|void setMaxInactiveInterval(int second)  |返回两次请求间隔多长时间，Session被取消|

生命周期：
回话中打开新的页面属于同一个会话，在会话超时前，只要有一个页面没有关闭，则新打开的页面属于同一个会话
超时之后，或者关闭所有页面之后重新打开页面会重新创建新的会话。
区别是，关闭所有页面后，服务端的session仍然存在，但是客户端不会携带相同的sessionId去和服务端校验

销毁session：调用session.invalidate()方法，Session过期，服务器重新启动

*Tomcat默认session超时时间为30分钟，可在web.xml中修改*
```
<session-config>
  <session-timeout>10</session-timeout>
  <!--这里的单位是分钟-->
</session-config>
```

> application对象

application对象实现了用户间的数据共享，全局变量
application对象创建于服务器启动，到服务停止销毁
不同用户操作的application为同一对象，任何人修改都会对其他用户产生影响

> page对象

指向当前JSP页面本身，是object的实例


> pageContext对象

提供了JSP页面内所有对象以及名字空间的访问，包括session application等

|常用方法||
|----|----|
| JspWriter getOut()                          |返回客户端响应的JspWriter流|
|HttpSession getSession()                     |返回当前页面的session|
|ServletRequest getRequest()                  |当前页面request对象|
|ServletResponse getResponse()                |当前页面的response对象|
|void setAttribute(String name, Object attr)  |设置名称为name的属性值为attr|
|Object getAttribute(String name)             |获取名称为name的属性值|
|int getAttributeScope(String name)           |name属性值的作用范围|
|void forward(String relativeUrlPath)         |页面重新导到另一个页面|
|void include(String relativeUrlPath)         |当前位置包含另一文件|


> config对象

 在Servlet初始化时，JSP引擎向Servlet传递初始化参数和服务器相关信息用的。
 |常用方法||
 |----|----|
 |ServletContext getServletContext()  |返回含有服务器相关信息的ServletContext对象|
 |String getInitParameter(String name)|返回初始化参数|
 |Enumeration getInitParameter()      |返回Servlet初始化参数枚举|

> exception对象

明显，当JSP中出现异常时就会出现这个对象，要使用这个对象，把isErrorPage设置为true，否则无法编译
`<% page isErrorPage="true"%>`

|常用方法||
|----|----|
|String getMessage()            |返回描述异常的消息|
|String printStackTrace()       |返回关于异常的栈轨迹|
|Throwable FillInStackTrace()   |重写异常的执行栈轨迹|


## JSP动作元素
为处理请求提供信息，大致分为5类

|||
|----|----|
|存取JavaBean相关的|`<jsp:useBena>、<jsp:setProperty>、<jsp:getproperty>`|
|JSP的本元素|`<jsp:include>、<jsp:forward>、<jsp:param>、<jsp:plugin>、<jsp:params>、<jsp:fallback>`|
|JSP2.0新增元素|`<jsp:root>、<jsp:declaration>、<jsp:scriptlet>、<jsp:exception>、<jsp:text>、<jsp:output>`|
|JSP2.0新增的动作元素 用来动态生成元素值|`<jsp:attribute>、<jsp:body>、<jsp:element>`|
|JSP2.0新增的动作元素，用在Tag File中|`<jsp:invoke>、<jsp:dobody>`|

+ `<jsp:useBean>`使用方法
`<jsp:useBean id="标识符" class="java类名" scope="作用范围" />`

+ `<jsp:setProperty>`使用方法
```
<jsp:setProperty name="JavaBean实例名" property="*"/>(跟表单关联)
//会根据表单的name去匹配实例的属性

<jsp:setProperty name="JavaBean实例名" property="JavaBean属性名"/>(跟表单关联)
//将表单中的某个属性匹配到JavaBean属性名上
<jsp:setProperty name="JavaBean实例名" property="JavaBean属性名" value="BeanValue"/>(手动设置)
//手动设置Bean的属性
<jsp:setProperty name="JavaBean实例名" property="propertyName" param="request对象中的参数名"/>(跟request参数关联)
//
```
scope的作用域有四个“page,request,session,application”

## JSP操作Cookie
+ 创建Cookie
Cookie acookie = new Cookie(String key, Object value);

+ 写入Cookie
reponse.addCookie(acookie);

+ 读取Cookie
Cookie[] cookies = request.getCookies();

Cookie对象的常用方法
|||
|----|----|
|void setMaxAge(inr second)     |设置cookie的有效期，秒为单位
|void setValue(String value)    |设置cookie的值|
|String getName()               |获取cookie的名称|
|String getValue()              |获取cookie的值|
|int getMaxAge()                |获取cookie的存活时间，秒为单位|

## 指令与动作

+ include指令`
`<%@ include filde="Url"%>`
+ include动作
`<jsp:include page="URL" flush="true|false">`
flush表示页面是否从缓冲区读取 ，默认为false
+ include动作与指令的区别

||指令|动作|
|----|----|----|
|语法格式          |`<%@ include%>`  | `<jsp:include>`|
|发生作用的时间     |页面转换期间      | 请求期间|
|包含的内容        |文件实际内容      | 页面内|
|转换成的Servlet   |生成一个Servlet  | 各自生成Servlet|
|编译的时间        |较慢             | 较快|
|执行的时间        |较快             | 每次重新解析资源|
