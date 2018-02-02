---
title:
- MyBatis使用笔记
date:
- 2017-02-06 15:56:19
tags:
- Java
- MyBatis
- Java Web
---
![](http://upload-images.jianshu.io/upload_images/1112615-9f3500165cc54a1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 1. 编写配置文件
<!--more-->
```xml
<?xml version='1.0' encoding='UTF-8'?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!--使用JDBC的getGeneratedKeys获取自增Key数据库主键值-->
        <setting name="useGeneratedKeys" value="true"/>
        <!--使用别名替换列名字-->
        <setting name="useColumnLabel" value="true"/>
        <!--开启驼峰命名转换-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```
将连接数据库的库信息，写入Spring的xml文件中
```xml
<!--配置MybatisSQLSession-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--注入连接池-->
        <property name="dataSource" ref="dataSource"/>
        <!--注入Mybatis全局配置-->
        <property name="configLocation" value="classpath:database/mybatis-config.xml"/>
        <!--扫描entity包-->
        <property name="typeAliasesPackage" value="com.zing.account.entity"/>
        <!--扫描mapper文件-->
        <property name="mapperLocations" value="classpath:database/mapper/*.xml"/>
    </bean>

```

## 2.获取SqlSessionFactory
Spring 管理下，用获取bean的方法就能获取SqlSessionFactory
```java
ApplicationContext acontext = new FileSystemXmlApplicationContext("xxxx.xml");
acontext.getBean("sqlSessionFactory");
```
还有其他方式获取Bean，不一一列举。

>*不使用Spring管理SQLSession的话,可以按照官方文档，补全Mybatis-config.xml,再通过下面代码获得SqlSession*

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

## 3.编写SQL映射文件，添加到mybatis-config.xml
创建`email.xml`
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.zing.account.dao.LoginAndSignUpDAO">
    <select id="isEmailAlreadyExists">
        SELECT count(0)
        FROM cmc_login_oauth
        WHERE login_email=#{email};
    </select>
</mapper>
```
吧mapper文件添加到`mybatis-config.xml`的`<configuration>`内,注意路径要正确
```xml
<mappers>
    <mapper resource="classpash:/mapper/email.xml"/>
</mappers>
```


## 4.SqlSession进行数据库访问
```java
SqlSession session = sqlSessionFactory.openSession();
try {
  // 通过SqlSession，进行数据库访问,如查询
  //session.select("nameSpace.SqlID",param)
  session.select("com.zing.account.dao.LoginAndSignUpDAO.isEmailAlreadyExists","1234@zing.com");

} finally {
  session.close();
}
```
> *这里注意：参数只能传一个，如果需要传多个参数作为筛选条件，可使用一个实体类封装，或者使用List，Map封装，获取这些参数可以参考后面OGNL的介绍*

## 5.查询结果映射实体类
可以将实体类，配置到Mapper里面，colum表示列名，jdbcType可以参考jdbc包下面的Type类，property表示实体类中的属性名字
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.zing.account.dao.LoginAndSignUpDAO">
    <resultMap id="userEntity" type="com.zing.account.entity.UserEntity">
        <id column="id" jdbcType="INTEGER" property="id"/>
        <result column="name" jdbcType="VARCHAR" property="userName"/>
        ...
    </resultMap>
    <select id="isEmailAlreadyExists">
        SELECT count(0)
        FROM cmc_login_oauth
        WHERE login_email=#{email};
    </select>
</mapper>
```

## 6.OGNL表达式

参考下面列表

![6-1](http://upload-images.jianshu.io/upload_images/1112615-93adf63d365a7038.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![6-2](http://upload-images.jianshu.io/upload_images/1112615-839ad13f438bac6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 7.拼接SQL
首先，注意&&和`" `这类符号，要在XML转义
```
&& ：and
|| ：or
% ：mod

" ：&quot;
< ：&lt;
```
符号转义可以参考：http://www.w3school.com.cn/xml/xml_cdata.asp
更多的转义，可以百度
那么拼接SQL可以像下面一样
```xml
    <select id="isEmailAlreadyExists">
        SELECT count(0)
        FROM cmc_login_oauth
        WHERE login_email=#{email}
        <if text="userName!=null and "".equals(userName.trim())">and user_name=#{userName}</if>
    </select>
```
这样满足userName不为空的时候，就会拼接SQL预编译。
> 注意注入漏洞

## 8.参考
MyBatis官方文档：http://www.mybatis.org/mybatis-3/zh/getting-started.html
慕课网：http://www.imooc.com

> *转载请务必注明出处：https://micorochio.github.io/2017/02/07/my-batis-note/*