---
title:
- java基础之：JDBC
date:
- 2017-03-09 14:50:13
tags:
- Java
- 数据库
- JDBC
---
JDBC是Java连接数据库的标准，为了兼容大部分数据库，Java提出了JDBC标准，通过这个标准，让各个数据库提供实现支持，这样实现一处编码，处处运行的Java特性。

习惯了ORM框架，却忘记了原本的JDBC，所以我觉得有必要复习来夯实一下基础。

![](http://upload-images.jianshu.io/upload_images/1112615-719a927e4839a667.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!--more-->
## 0x00 JDBC 历史
JDBC是Sun公司为了能够让SQL访问统一的一套纯JAVA API设计的一套接口，这种接口是遵循了微软的ODBC API模式。其驱动实现是各家数据库供应商编写的，通过JDBC API可以通过驱动实现数据库通信。

## 0x01 链接数据库回顾
基本Web常用的数据库都是有供Java链接的驱动，

![三层结构](http://upload-images.jianshu.io/upload_images/1112615-d0884440199c9cc7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那么如何使用JDBC？
写个Demo
```java
import java.io.FileInputStream;
import java.io.IOException;
import java.sql.*;
import java.util.Properties;

/**
 * Created by zing on 2017/3/7.
 */
public class JDBCDemo {

  private  void testJDBC() throws ClassNotFoundException, IOException, SQLException {
        //注册驱动
        Class.forName("com.mysql.jdbc.Driver");
        //JDBC使用类似URL的数据源描述
        String url = "jdbc:mysql://localhost:3306/demo";//忽略
        //但是我们一般不会直接这样写死。而是使用配置来描述数据源，用户名，密码
        Properties props = new Properties();
        FileInputStream propertiesFile = new FileInputStream("JDBC.properties");
        props.load(propertiesFile);
        propertiesFile.close();

        String DriverStr = props.getProperty("jdbc.Driver");
        String urlStr = props.getProperty("jdbc.url");
        String userName = props.getProperty("jdbc.name");
        String passcode = props.getProperty("jdbc.passworld");

        //打开数据库链接
        Connection connection = DriverManager.getConnection(urlStr,userName,passcode);
        //执行SQL
        Statement sta = connection.createStatement();
        //executeUpdate可以返回数据库更新的行数
        int efactRow = sta.executeUpdate("UPDATE USER SET Permition = 'admin' WHERE username = 'Zing'");
        //executeQuery可以返回一个查询的结果集，这个集合的迭代器略有不同Iterator,没有hasNext方法，初始是，指针在数据前，必须调用next方法才能读取第一行数据
        ResultSet resultSet = sta.executeQuery("SELECT * FROM USER ;");
        while (resultSet.next()){
            //当前行获取第一栏的值，具体类型需要看数据库实现
            resultSet.getString(1);
        }
        //关闭语句
        sta.close();
        //关闭结果集
        resultSet.close();
        //关闭数据库连接
        connection.close();

        /*
        一般情况下，关闭的操作会放在catch语句的finally块中，catch处理数据库异常，finally来关闭连接
        */
    }
}
```

上面写的是大杂烩，一般会将获取连接抽取成一个方法，异常也会捕获，并在try/catch/finally中的finally块中，关闭数据库连接。

API用法可以看看`java.sql.Connection`，`java.sql.Statement`，`java.sql.ResultSet`，这样，基本的操作就可以了然了。

> *` boolean execute(String sql) throws SQLException;`这个方法可以执行任何SQL，返回执行是否成功*。慎用。

## 0x02 预编译SQL

PrepareStatement，一个可以让数据库预编译SQL的API。
并不是所有的SQL都是写死的，例如：
```sql
  SELECT * FROM UserAccount Where Name =
```
根据名称来查找用户，这里的名字自然是用户自己定义的，如果用Statement，则应该这么写
```java
public void findUserByName(String name){
  Statement  sta = connection.createStatement();
  String findByName = "SELECT * FROM USER WHERE Name=' "+name+" ';";
  ResultSet resultSet = sta.executeQuery(findByName);
}
```
如果将name交给普通用户来输入，则没什么问题，但是 如果交给黑客，name他会输入 `小明' OR '1' = '1`,这样语句拼接后就会变成
```sql
SELECT * FROM USER WHERE Name=' 小明' OR '1' = '1';
```
这一句就会把数据库所有的用户全部查出来了，很严重的注入漏洞，基本就会被脱库了。

所以Java JDBC定义的预编译SQL的API。
上例子：
```java

import java.io.FileInputStream;
import java.io.IOException;
import java.sql.*;
import java.util.Properties;

public class JDBCDemo {

    private void testJDBC() {

        Connection connection = null;
        Statement sta = null;
        ResultSet resultSet = null;
        PreparedStatement preSta = null;
        try {
            connection = getConnection();
            //执行SQL
            
            String findByName = "SELECT * FROM USER WHERE Name=?;";
            preSta = connection.prepareStatement(findByName);
            preSta.setString(1,"Zing");
            resultSet = preSta.executeQuery();


        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            /*
                一般情况下，关闭的操作会放在catch语句的finally块中，catch处理数据库异常，finally来关闭连接
            */
            //关闭语句
            try {
                preSta.close();
                //关闭结果集
                resultSet.close();
                //关闭数据库连接
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }


    }

    public Connection getConnection() throws ClassNotFoundException, IOException, SQLException {
        //注册驱动
        Class.forName("com.mysql.jdbc.Driver");
        //JDBC使用类似URL的数据源描述
        //但是我们一般不会直接这样写死。而是使用配置来描述数据源，用户名，密码
        Properties props = new Properties();
        FileInputStream propertiesFile = new FileInputStream("JDBC.properties");
        props.load(propertiesFile);
        propertiesFile.close();

        String DriverStr = props.getProperty("jdbc.Driver");
        String urlStr = props.getProperty("jdbc.url");
        String userName = props.getProperty("jdbc.name");
        String passcode = props.getProperty("jdbc.passworld");
        return DriverManager.getConnection(urlStr, userName, passcode);

    }
}

```
 顺便重构了之前的代码。
我们用 `?`占位，留下可变参数的位置，后来再用` setString(int parameterIndex, String x)`这个方法将数据填充进SQL，这样，如果参数含有SQL关键字时，就不能通过编译，查不到结果。可以避免SQL注入。

preSta.setString(1,"Zing");表示，在第一个`?`处设置参数为`Zing`
当然参数是数字，日期时，可以使用，
` void setDouble(int parameterIndex, double x) throws SQLException`
` setDate(int parameterIndex, java.sql.Date x)
            throws SQLException;`


等方法，根据不同类型设置参数。

## 0x03 数据库类型与转义
数据库类型和Java类型是有一点不一样的，但是JDBC定义了其中的大部分类型，这里不一一列举

![MySQL部分类型对照表，有兴趣可以查一查](http://upload-images.jianshu.io/upload_images/1112615-1b51a004c32f21c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

JDBC中的转义是为了让Java访问数据库时，得到普遍的支持。一般用于下列特性
+ 时间日期的字面常亮
+ 标量函数调用
+ 存储过程调用
+ 外连接查询
+ LIKE子句中转义字符

数据库的日期转换成Java的日期，是通过ISO8601标准衡定并相互转换的
> d表示DATE、t表示TIME、ts表示TIMESTANP
```
{d '2017-01-22'}
{t '19:30:29'}
{ts '2017-01-22 19:30:29.989'}
```

标量函数是获取一个数值的函数，一般调用时嵌入标准函数名和参数，这个很少见到有人使用的，就不举例了。

存储过程，是数据库自建的存储方式，不同的数据库存储过程基本不一样，要调用存储过程，需要用call来进行转义
```
{call PROC01(?,?)}
{call PROC02}
```
如果你不明白什么存储过程，可以看看数据库相关的资料。

外连接，就是Outter Join，借用核心卷II中的例子
```sql
SELECT * FROM {oj Books LEFT OUTER JOIN Publishers ON Books.Publish_ID = Publishers.Publish_ID }
```
这条语句表示查询找不到出版商的书，相反如果是`RIGHT OUTER JOIN`则会查询出没有出版书的出版商，如果需要查到全部，则用`FULL OUTER JOIN`
。这里用转义是因为有些数据库实现不太统一。

Like子句转义，是因为下划线和百分号在Like条件里是特殊的含义，需要用转义来表示
```sql
SELECT * FROM User WHERE Name LIKE %!_%ming {escape '!'}
```
{escape '!'}表示将！定义为转义符号，!_表示字面量下划线

## 0x04 事务
为了保证数据和业务逻辑的完整性，我们可以将一系列的SQL语句构建成一个事物，当所有语句都顺利执行的时候，事务可以被提交。但是如果中途被阻碍，则数据会被回滚，将数据恢复成执行前的样子。

首先需要关闭数据库自动提交
```java
connection.setAutoCommit(false);
```
然后根据实际业务执行多条UPDATE INSERT DELETE语句
```java
statement.executeUpdate("SQL1");
statement.executeUpdate("SQL2");
statement.executeUpdate("SQL3");
```
当所有语句顺利执行后，调用
```java
connection.commit();
```
如果遇到异常或错误，则可以调用
```java
connection.rollback();
```

其中JDBC支持事务保存点和批量更新
保存点：将事务的某一阶段设置为保存点后，可以控制回滚时，恢复到这个保存点的数据。从而更加精确的控制回滚操作
批量更新就是将大量数据一次性存入，或修改大量数据时使用的。两个🌰：
```java
statement.executeUpdate("SQL1");
Savepoint step1 = connection.setSavepoint();
statement.executeUpdate("SQL2");
if(something==false){
  connection.rollback(step1);
}
```

```java
String updateSQL = "……";
statement.addBatch(updateSQL);
while(needUpdate){
  command = "……"+"updateSQL2"
  statement.addBatch(updateSQL);
}
//批量执行
int effectRows = statement.executeBatch();
```
>__*批量执行中一定不能有查询语句，否则会抛出异常。*__

## 0x05 文件查询和存入数据库
不建议这么搞，数据库存入太多大文件会导致数据库庞大，备份和恢复的成本将增加。
在数据库中，二进制大对象称为Blob，字符型大对象为Clob
这里演示一下查询和存储
```java
       //读取
        PreparedStatement preparedStatement01 = connection.prepareStatement("SELECT picture FROM PictureTab WHERE picName=?;");
        preparedStatement01.setString(1,"superman");
        ResultSet rs = preparedStatement01.executeQuery();
        if(rs.next()){
            Blob picBlob = rs.getBlob(1);
            Image pic = ImageIO.read(picBlob.getBinaryStream());
        }

        //存储
        Blob pictureBlob = connection.createBlob();
        int offset = 0;
        OutputStream outStram = pictureBlob.setBinaryStream(offset);
        ImageIO.write(pictureBlob,"PNG",outStram);
        PreparedStatement preparedStatement02 = connection.prepareStatement("INSERT INTO PictureTab VALUE (?,?);");
        preparedStatement02.setString(1, "SuperMan");
        preparedStatement02.setBlob(2,pictureBlob);
        preparedStatement02.executeUpdate();

```


## 0x06 其他一些概念
+ 元数据：数据库的结构和表信息等描述数据库结构和组成部分的数据
+ 多结果集：一次查询，使用多个Select SQL语句是，会得到一个多结果集
+ 可滚动结果集：可以向前，向后查询的结果集，之前的只能用Next向后查询，使用
```java
Statement stat = Connection.createStatement(ResultSet.TYPE_SCROLL_INSENSTIVE , ResultSet.CONCUR_READ_ONLY )
```
在获取结果集的时候，会变成一个可滚动集。
+ 获取数据库生成键值`statemwnt.getGeneratedKeys();`
+ 行集 RowSet接口继承了ResultSet，但不需要长时间占用数据库链接。


____
love&peace
若有错误请不吝指出，谢谢。
我的博客：https://micorochio.github.io/
转载请注明出处:https://micorochio.github.io/2017/03/10/basic-of-java-JDBC/。