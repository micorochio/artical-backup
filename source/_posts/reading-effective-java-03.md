---
title:
- Effective Java 读书笔记-第四章
date:
- 2017-08-07 00:00:41
tags:
- Java
- 笔记
- Java基础
---
第四章讲了类的设计，大部分应该遵守，书中也给出了遵守这些规则的理由, 然后书已经看得差不多了，这样记笔记实在没啥效率，后面就不这么写了。


![](http://upload-images.jianshu.io/upload_images/1112615-89717ee7e4af41a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!-- more -->

## 0x01 类的成员的可访问性最小化
可访问性最小化的好处：
+ 隐藏了实现，保护了信息
+ 封装，减少了耦合，减轻了维护负担


> + 私有的（private） —— 只有在声明该成员的顶层类内部才可以访问这个成员。
> + 包级私有的（package-private） —— 声明该成员的包内部的任何类都可以访问这个成员。从技术上讲，它被称为“缺省（default）访问级别”，如果没有为成员指定访问修饰符，就采用这个访问级别。
> + 受保护的（protected） —— 声明该成员的子类可以访问这个成员（但有一些限制[JLS，6.6.2]），并且，声明该成员的包内部的任何类也可以访问这个成员。
> + 公有的（public） —— 在任何地方都可以访问该成员。


一旦类的属性被公开，则你有责任负责兼容到底。
如果一个类中用一个对象实例作为属性，则这个属性一定不能是public的，最好是final的，可以保证线程安全。

```java
public class xxx{
  //潜在安全漏洞
  public static final Thing[] VALUES = { ... };
}

```

## 0x02 不直接公开属性修改权限，用方法操作属性
```java
class Point {
    public double x;
    public double y;
}
```

上面的类无法改变属性的表示方式，也不能对两个属性进行任何附加要求，比如限定上下限。
```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
      this.x = x;
      this.y = y;
    }

  public double getX() { return x; }
  public double getY() { return y; }

  public void setX(double x) { this.x = x; }
  public void setY(double y) { this.y = y; }
}
```
这段代码很好的封装了内部属性，并提供了操作方法，在方法内部可以对属性操作进行约束。
数据是抽象的，应最小化的缩小对外界的影响，所以可变的属性应该私有化，而不是直接暴露。只有不需要改变的属性，才可以暴露，比如：
```java
    private static final int HOURS_PER_DAT    = 24;
    private static final int MINUTES_PER_HOUR = 60;
```
## 0x03 可变性最小化
`String`是final的。`BigInteger`,`BigDecimal`有很多属性也是final的，这些final的属性让这些类的可变性变小了，为什么要这么设计？
+ 不可变的类比可变类更加易于设计、实现和使用。
+ 它们不容易出错，而且不可变对象本质上是线程安全的，它们不要求同步。
+ 不可变对象可以被自由地共享。
+ 不需要进行保护性拷贝。
+ 也可以共享它们的内部信息。
+ 不可变对象为其他对象提供了大量的构建（building blocks）
> 最后一条解释一下：不可变对象即使被放进集合set、或者map中，（一般设计集合的键-值映射是不希望发生变化的）也不用考虑对象值被修改。

**当然也有缺点：每个值都需要一个新对象**
因为这有，有些操作，每进行一次操作，都会产生一个新对象，例如String的拼接。
所以，许多不可变的类拥有一个或者多个非final的域，它们在第一次被请求执行这些计算的时候，把一些开销昂贵的计算结果缓存在这些域中。如果将来再次请求同样的计算，就直接返回这些缓存的值，从而节约了重新计算所需要的开销。

书中有了5个提议：
> 1. 不要提供任何会修改对象状态的方法（也成为mutator）。
> 2. 保证类不会被扩展。
> 3. 使所有的域都是final的。(其实不用特别严格执行这一条)
> 4. 使所有的域都成为私有的。
> 5. 确保对于任何可变组件的互斥访问。

另外对于不可变对象，构造器应该创建完全初始化的对象，并建立起所有的约束关系。

如果类不能被做成是不可变的，仍然应该尽可能地限制它的可变性。

## 0x04 复合优先继承
复合优先继承的原因很简单，为了保证对象的安全。为什么这么说？
当类有跨包继承的时候，有些域属性按道理是不可以使用的，但是继承后某些方法就有可能会操作到这些属性，如果精心设计了，那倒没什么。就怕粗心了，有些无法预知的安全问题。
所以当对象有跨包使用并且需要扩展的时候，可以选择将对象放进一个新类作为域属性。这样处理会比继承好一点。

学术一点就是：跨包继承打破了封装性。

当然这不是绝对的，有些类天生应该被继承，这是设计上决定的。
书中有代码介绍了为什么会破坏封装性
```java
// Broken - Inappropriate use of inheritance!
public class InstrumentedHashSet<E> extends HashSet<E> {
    // The number of attempted element insertions
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```
这个类看起来非常合理，但是它不能正常工作。假设我们创建了一个实例，并利用addAll方法添加了三个元素：
```java
InstrumentedHashSet<String> s =
    new InstrumentedHashSet<String>();
s.addAll(Arrays.asList("Snap", "Crackle", "Pop"));
```
实际上继承后，一个`super.addAll()`导致了getAddCount方法拿出的结果不是预期的，上面的代码应该返回3，但是返回的确实6.
因为HashSet的addAll方法已经计数了。

综上，复合优先于继承，
你只需要

```java
public class InstrumentedHashSet<E>{
  private HashSet<E> innerHashSet;
}
```
就可以拥有HashSet的一切。


但是！ 复合后很难使用回调模型！使用回调模型的时候，注意规避。
> 简而言之，继承的功能非常强大，但是也存在诸多问题，因为它违背了封装原则。只有当子类和超类之间确实存在子类型关系是，使用继承才是恰当的。即便如此，如果子类和超类处在不同的包中，并且超类并不是为了继承而设计的，那么就成将会导致脆弱性（fragility）。为了避免这种脆弱性，可以用复合和转发机制来代替继承，尤其是当存在适当的接口可以实现包装类的时候。包装类不仅比子类更加健壮，而且功能也更加强大。

## 0x05 要么为继承而设计，要么禁止继承
上一条已经说过，继承不是最佳的代码复用方式。所以当你设计一个类，并且希望这个类可以被继承的时候，一定要做好准备工作，设计好方法，提供详实的文档，保证继承者的使用。让使用者能规避一些安全问题，或者设计的时候直规避。
书上举了两个例子：
+ `java.util.AbstractCollection`的规范:
> public boolean remove(Object o)<br>
Removes a single instance of the specified element from this colletion, if it is present(optional operation). More formally, removes an element e such that (o==null ? e==nul : o.equals()), if the collection contains one or more such elements. Returns true if the collection contained the specified element (or equivalently, if the collection changed as a result of the call).<br>
This implementation iterates over the collecting looking for the specified element. If it finds the elements, it removes the element from the collection using the iterators's remove method. Note that this implementation throws an UnsupportedOperationException if the iterator returned by this collection's iterator method does not implement the remove method.<br><br>（如果这个集合中存在指定的元素，就从中删除该指定元素中的单个实例（这是项可选的操作）。更一般地，如果集合中包含一个或者多个这样的元素e，就从中删除这种元素，以便(o==null ? e==nul : o.equals())。如果集合中包含指定的元素就返回true（如果调用最终改变了集合，也一样）。<br>
该实现遍历整个集合来查找指定的元素。如果它找到该元素，将会利用迭代器的remove方法将之从集合中删除。注意，如果由该集合的iterator方法返回的迭代器没有实现remove方法，该实现就会抛出UnsupportedOperationException。）

+ `java.util.AbstractList`中的`removeRange`方法：
> protected void removeRange(int fromIndex, int toIndex)<br>
Removes from this list all of the elements whose index is between fromIndex, inclusive, and toIndex, exclusive. Shifts any elements to the left (reduces their index). This call shortens the ArrayList by (toIndex - 'fromIndex') elements. (If toIndex==fromIndex, this operation has no effect.)<br>This method is called by the clear operation on this list and its sublists. Overriding this method to take advantage of the internals of the list implementation can substantially imporve the performance of the clear operation on this list and its sublists.<br>
This implementation get a list iterator positioned before fromIndex and repeatedly calls ListIterator.next follows by ListIterator.remove, until the entire range has been removed. Note: If ListIterator.remove requires linear time, this implementation requires quadratic time.
Parameters:<br><br>
fromIndex index of first element to be removed.<br>
toIndex index after last element to be removed.<br><br>
（从列表中删除所有索引处于fromIndex（含）和toIndex（不含）之间的元素。将所有符合条件的元素移到左边（减小索引）。这一调用将从ArrayList中删除（toIndex - fromIndex）之间的元素。（如果toIndex == fromIndex，这项操作就无效。）<br>
这个方法是通过clear操作在这个列表及其自列表中调用的。覆盖这个方法来利用列表实现的内部信息，可以充分地改善这个列表及其子列表中的clear操作的性能。<br>
这项实现获得了一个处在fromIndex之前的列表迭代器，并一次地重复调用ListIterator.remove和ListIterator.next，直到整个范围都被移除为止。<br>注意：如果ListIterator.remove需要线性的时间，该实现就需要平方级的时间。<br>
参数：<br><br>
fromIndex 要移除的第一个元素的索引<br>
toIndex 要移除的最后一个元素之后的索引）<br>

后面讲述了如何设计保护域属性，对不能继承的类要用final修饰来禁止子类化。
## 0x06 接口和抽象类优先使用接口
对的，标题即方法。

你会问为什么？
+ 子类可以实现多个接口，却不能实现多个抽象类。
+ 实现多个接口意味着类是混合类，更加适合代码复用
+ 接口可以让程序员设计出非层次结构的类

> 层次结构：就是金字塔结构

![图片来自网络，侵删！](http://upload-images.jianshu.io/upload_images/1112615-f77f4dd18ff0fad7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这样设计，顶部就是父类，一层一层往下就是子类，如果使用抽象类，则就是金字塔结构的层次模型。如果使用接口，那就是树模型，一个类可以实现多个接口，实现接口的功能。

书中有些例子，这里不说明了。

## 0x07 接口只定义类型
你一定见过下面的代码
```java
package com.zing.nio_study;

import java.math.BigDecimal;

public interface DemoInterface {
    public final static BigDecimal PI = BigDecimal.valueOf(3.1415926);
    publi final static int DELETED = 1;
    public final static int NORMAL = 2;
}

```
> **这是对接口的不良使用**

接口按照JAVA的初衷来说，是不涉及代码逻辑细节的，这些常量是纯粹的实现细节。这样的API暴露了设计细节。而且一旦以后某些属性不使用了，子类依旧需要实现该接口，维护代价也会上升。如果是存粹的常量值接口，你还不如用枚举。

这里应该使用枚举类型`enum`
```java
enum state{
        DELETED,
        NORMAL,
        ABNORMAL
    }
```
如果担心枚举需要扩展，可以让枚举实现一个空接口。
```java
interface ADEMO{}

enum demo1 implements ADEMO{
    A,
    B,
}

enum demo2 implements ADEMO{
    C,
    D,
    E
}
```
这样调用处可以随时扩展新的枚举：
```java
    public void doSomthingByDemo(ADEMO A){
        // TODO 
    }
```
*以上是枚举扩展部分是个人观点，实际中请慎用，应该尽量把类设计的完善一点，有问题可以留言。*

## 0x08 类层次优于标签类
标签类：
```java
// Tagged class - vastly inferior to a class hierarchy
class Figure {
    enum Shape { RECTANGLE, CIRCLE};

    // Tag field - the shape of this figure
    final Shape shape;

    // These fields are used only if shape is RECTANGLE
    double length;
    double width;

    // This field is used only if shape is CIRCLE
    double radius;

    // Constructor for circle
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // Constructor for rectangle
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch (shape) {
          case RECTANGLE:
            return length * width;
          case CIRCLE:
            return Math.PI * (radius * radius);
          default:
            throw new AssertionError();
        }
    }
}
```
上面一段代码，大致的意思是一个类，里面有两个标签对象，圆和矩形。
看起来还好，但是标签一多分支代码就越复杂，难以维护；而且还有很多模板代码… 写这种代码的大部分是刨坑小能手，交接给别人后，别人在一堆分支结构中摸不着头脑，然后问候你家人也是常有的事情。

这时候可以设计成层次类。
```java
// Class hierarchy replacement for a tagged class
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length; 
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }

    double area() { return length * width; }
}
```
优点：
+ 条理清楚，每个子类去掉了不相关的属性
+ 不需要大量的样板代码，无须标签区分
+ 别人看地轻松一点
+ 方便检查代码层次关系，灵活性很高


## 0x09 用函数对象表示策略
java已经支持lambda表达式了，所以，这一条可以使用lambda表达式来代替。
因为lambda表达式自身是没有域属性的（即无状态），天生线程安全。

函数对象表示策略，首先知道策略是什么，这里举个例子：
```java
class StringLengthComparator {
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
}
```
在一个函数中，需要动态的根据两个参数的不同属性来执行不同的逻辑，这个逻辑就是策略。

上面代码的策略是根据字符的长短实现字符串的大小比较。
书上有些说明，但是请还是了解一下java 8 的lambda表达式，很有用！！！
这里就不仔细说了。


## 0x10 优先考虑静态成员类
> 静态成员类是最简单的一种嵌套类。最好把他看作是普通的类，只是碰巧被声明在另一个类的内部而已，它可以方位外围类的所有成员，包括那些声明为私有的成员。

这么做的目的跟最早说明的访问权限最小化类似
当你需要一个类，只供某一个类访问时，务必将其作为静态成员类，放在某个类的内部。

静态static可以减少类对外围对象的依赖，减少时间和空间的消耗。

个人观点：虽让书上这么说了，不过实际上，一般很少设计一个类，只供某个类使用。如果类的代码量很大，还是抽出来做独立类吧。


love & peace

转载请注明出处：https://micorochio.github.io/2017/08/06/reading-effective-java-03/

如若有误请帮忙指正，谢谢