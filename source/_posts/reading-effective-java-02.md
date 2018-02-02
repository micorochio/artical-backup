---
title:
- Effective Java 读书笔记-第三章
date:
- 2017-08-05 23:15:36
tags:
- Java
- 笔记
- Java基础
---
第三章讲了一些通用的方法。看的时候很快，记笔记的时候慢了
![](http://upload-images.jianshu.io/upload_images/1112615-7341f62f8111e454.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2116504&auto=1&height=32"></iframe>

## 0x01 equals方法应该如何写
equals 方法是对象判断是否相等的方法，如果一个类具有多实例的需求时，可以重equals方法来判断两个对象是否相等。

除非：
> + 类的每个实例都是唯一的
> + 不关心实例是否逻辑上相等
> + 父类已经重写了equals 
> + 类是私有的，equals方法永远不会被调用

<!-- more -->

如果不是以上条件，那就需要考虑对象是不是逻辑相等。那么判断相等的方法就应该具有以下特性：
1. **reflexive** :  对象自己与自己对比永远是true，`x.equals(x)` 返回一定是true
2. **symmetric** : `x.equals(y)`为true时，`y.equals(x)`一定为true
3. **transitive** :   如果 `x.equals(y)` 为true，且 `y.equals(z)`也为true，那么`x.equals(z)` 也必须为true
3. **consistent** :   如果 `x.equals(y)` 为true，再x和y没有修改的情况下，无论何时再调用equals方法都会返回true。
5. 任何对象equals `null`时，必须返回false
> 以上对象全部都是非空对象

有了这几个标准，于是就有了写equals方法的公式：
1. 使用==操作符检查“参数是否为这个对象的引用”。
2. 使用instanceof操作符检查“参数是否为正确的类型”
3. 把参数转换成正确的类型。
4. 对于该类中每个“关键（significant）域，检查参数中的域是否与该对象中对应的域相匹配”
比较域（类属性参数） 的方法可以参照下下面的代码
```java
(field == o.field || (field != null && field.equals(o.field)))
```
5. 编写完成了equals方法之后，应该问自己三个问题：它是不是对称的、传递的、一致的


以上是写equals的公式

> 另外，equals只做比较对象的工作，不要让equals过于智能，且不要将equals声明中的Object对象替换为其他的类型.

## 0x02 hashCode方法为什么也要覆写
hashCode方法是需要和equals方法一起重新的，在使用HashMap，HashSet这样的散列集合时会大大提升效率

Object规范里有一段话
> + 在应用程序的执行期间，只要对象的equlas方法的比较操作所用到的信息没有被修改，那么对同一个对象调用多次，hashCode方法都必须始终如一地返回同一个整数。在同一个应用程序的多次执行过程中，每次执行所返回的整数可以不一致。
> + 如果两个对象根据equlas(Object)方法比较是相等的，那么调用这两个对象中任意一个对象的hashCode方法都必须产生同样的整数结果。
> + 如果两个对象根据equlas(Object)方法比较是不相等的，那么调用这两个对象中任意一个对象的hashCode方法，则不一定要产生不同的整数结果。但是程序员应该知道，给不相等的对象产生截然不同的整数结果，有可能提高散列表（hash table）的性能。

这段话第二条就明确了，重写equals方法之后就应该连同hashCode方法一起重写。

举个例子：
```java 
import java.util.HashMap;
import java.util.Map;


final class PhoneNumber {
    private final short areaCode;
    private final short prefix;
    private final short lineNumber;

    public PhoneNumber(int areaCode, int prefix,
                       int lineNumber) {
        rangeCheck(areaCode, 999, "area code");
        rangeCheck(prefix, 999, "prefix");
        rangeCheck(lineNumber, 9999, "line number");
        this.areaCode = (short) areaCode;
        this.prefix = (short) prefix;
        this.lineNumber = (short) lineNumber;
    }

    private static void rangeCheck(int arg, int max,
                                   String name) {
        if (arg < 0 || arg > max)
            throw new IllegalArgumentException(name + ": " + arg);
    }

    @Override
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNumber == lineNumber
                && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }
}

public class TestHashCode {

    public static void main(String[] args) {
        Map<String, String> m = new HashMap<>();
        m.put(new String("HAHA"), "Jenny");
        System.out.println(m.get(new String("HAHA")));

        Map<PhoneNumber, String> s = new HashMap<PhoneNumber, String>();
        s.put(new PhoneNumber(707, 867, 5309), "Jenny");
        System.out.println(s.get(new PhoneNumber(707, 867, 5309)));
    }
}

```

TestHashCode 运行结果如下

![](http://upload-images.jianshu.io/upload_images/1112615-c5188e903a5f7e47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

String对象重写了hashCode，所以没啥问题，但是后面就不一样了。
很明显两个PhoneNumber实例逻辑是相等的：第一个被用于插入到HashMap中，第二个实例用于（试图用于）获取。由于PhoneNumber类没有覆盖hashCode方法，从而导致两个相等的实例具有不相等的散列码，违反了hashCode的约定。这样，用第二个实例去取Value的时候，现在的hashCode不在HashMap的hash bucket中，没有办法找到存放的value，即使对象明明相等。

那么如何写hashCode方法呢？
最好根据类的关键域（参与equals方法比较的属性）来写，书中给出了这样的建议：
>1.  把某个非零的常数值，比如说17，保存在一个名为result的int类型的变量中。
> 2. 对于对象中每个关键域f（指equals方法中涉及的每个域），完成以下步骤：
> + a. 为该域计算int类型的散列码c:
>    + i. 如果该域是boolean类型，则计算(f ? 1 : 0).
>    + ii. 如果该域是byte、char、short或者int类型，则计算(int)f。
>    + iii. 如果该域是long类型，则计算(int)(f ^ (f >>> 32))。
>    + iv. 如果该域是float类型，则计算Float.floatToIntBits(f)。
>    + v. 如果该域是double类型，则计算Double.doubleToLongBits(f)，然后按照步骤2.a.iii，为得到的long类型值计算散列值。
>    + vi. 如果该域是一个对象引用，并且该域的equlas方法通过递归地调用equlas的方式来比较这个域，则同样为这个域递归地调用hashCode。如果需要更复杂的比较，则为这个域计算一个“范式（canonical representation）”，然后针对这个范式调用hashCode。如果这个域的值为null，则返回0（或者其他某个常数，但通常是0）。
>    + vii. 如果该域是一个数组，则要把每一个元素当做单独的域来处理。也就是说，递归地应用上述规则，对每个重要的元素计算一个散列码，然后根据步骤2.b中的做法把这些散列值组合起来。如果数组域中的每个元素都很重要，可以利用发行版本1.5中增加的其中一个Arrays.hashCode方法。
> + b. 按照下面的公式，把步骤2.a中计算得到的散列码c合并到result中：
result = 31 * result + c;
> 3. 返回result。

## 0x03 toString方法的好处
这条不多说了，这个方法可以让类在运行的时候具有更高的可读性，方便调试。

## 0x04 谨慎使用clone方法
Object的clone方法是受保护的。如果不借助于反射（reflection）（见第53条），就不能仅仅因为一个对象实现了Cloneable，就可以调用clone方法。
而且目前JDK中对clone的约束非常弱。
一般情况下，良好的clone方法可以调用构造器来创建对象，构造之后再复制内部数据。如果这个类是final的，clone甚至可能会返回一个由构造器创建的对象。

Cloneable接口并没有清楚地指明，一个类在实现这个接口时应该承担哪些责任。一个默认调用的是super.clone()，这样不能保证`x.clone().getClass() == x.getClass()` 返回true了。这不太合逻辑。

另外，如果对象中的属性是个引用对象，那么Clone出来的东西就可能出现各种问题。操作源对象的属性将会影响克隆对象的属性，结果将是灾难性的。

另外，也不可以在一个类中的构造函数中使用clone方法。
clone方法再线程安全的类中也需要谨慎使用，保证同步。

因为Cloneable缺陷很多，所以非必要一般不使用。

##0x05 考虑实现一下Comparable接口
实现Comparable就表示类的对象中存在大小先后关系，允许执行顺序比较。实现Comparable接口的对象数组进行排序
```java
Arrays.sort(a);
```
实现这个接口和重新equals方法有点类似：
> 符号sgn（表达式）表示数学中的signum函数，它根据表达式（expression）的值为负值、零和正值，分别返回-1、0或1。
> + 实现者必须确保所有的x和y都满足sgn(x.compareTo(y) == -sgn(y.compareTo(x)))。（这也暗示着，当且仅当y.compareTo(x)抛出异常时，x.compareTo(y)才必须抛出异常。）
> + 实现者还必须确保这个比较关系是可传递的：x.compareTo(y) > 0 && y.compareTo(z) > 0暗示着x.compareTo(z) > 0。
> + 最后，实现者必须确保x.compareTo(y) == 0暗示着所有的z都满足sgn(x.compareTo(z)) == sgn(y.compareTo(z))。
> + 强烈建议(x.compareTo(y) == 0) == (x.equals(y))，但这并非绝对必要。一般说来，任何实现了Comparable接口的类，若违反了这个条件，都应该明确予以说明。推荐使用这样的说法：“注意，该类具有内在的排序功能，但是与equals不一致。”

另外，注意compareTo方法中最好不要有可能溢出的计算，会导致compareTo方法返回错误的结果


love & peace

转载请注明出处：https://micorochio.github.io/2017/08/06/reading-effective-java-02/

如若有误请帮忙指正，谢谢