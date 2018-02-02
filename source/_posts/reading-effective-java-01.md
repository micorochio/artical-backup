---
title:
- Effective Java 读书笔记-第二章
date:
- 2017-07-27 23:15:36
tags:
- Java
- 笔记
- Java基础
---
Thinking in java 太厚了，我不想看，所以先拿EJ开坑。
Effective Java 和 Thinking in java都是java基础评分超高的书，所以有必要看一看。
Effective Java这本书被java之父推荐，所以特意买了本正版。

>我很希望10年前就拥有这本书。可能有人认为我不需要任何Java方面的书籍，但是我需要这本书。<br>
——Java 之父 James Gosling

![](http://upload-images.jianshu.io/upload_images/1112615-b9600d580327741e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当然，这本书是2009年出版的，已经过去N多个年头，所以带着“批判”的眼光来瞻仰这本巨作。

<!-- more -->

ps：中文版的翻译真是烂的可以，英文厉害的强烈推荐看英文原版，另外，新手还是先多搬砖，或者去看看java核心技术上下两卷，这本书不适合0基础。

## 0x01 静态工厂方法代替构造器
用静态工厂方法代替构造器有什么好处呢？
> 1. 静态工厂方法有方法名称，可以更确切的针对对象进行不同的构造如构造素数和正整数，可以在同一个类里放两个不同的工厂方法，起两个有意义的名字
> 2. 工厂方法不一定每次都会创建一个新对象，如果配合单例，会有更好的性能提升（这个视场景而定）
> 3. 可以返回子类型的对象，具有更高的灵活性。
> 4. 在创建参数化实例时，代码更加简洁（这个书中使用了泛型作为例子，现在java已经有了类型推断的能力，所以看情况咯）
```java
//以前
 Map<String,List<String>> m = new HashMap<String,List<String>>();
//现在
Map<String,List<String>> m = new HashMap<>();
 //工厂
public static <K,V> HashMap<K,V> newIncetance(){
    return new HashMap<K,V>();
}
```


当然工厂方法也有缺点
> 1. 如果类中没有public 或者protected的构造器，就不能子类化
> 2. 工厂方法和其他静态方法没有区别，所以不能作为特殊的方法对待

## 0x02 构造器
上面提到的工厂方法和类的构造函数对多个可选参数时，劣势就明显出来了，一个方法中包含多个参数对调用方是非常不友好的，这时候可以考虑使用构造器（Builder）
放上书中的实例
```java
// Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories     = 0;
        private int fat          = 0;
        private int sodium       = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val) 
            { calories = val;        return this; }
        public Builder fat(int val)
            { fat = val;             return this; }
        public Builder sodium(int val)
            { sodium = val;          return this; }
        public Builder carbohydrate(int val)
            { carbohydrate = val;    return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

这样调用方就可以非常方便的构造出带有不同属性的对象了,这些属性都是可选的，而且可读性非常好
```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
    .calories(100).sodium(35).carbohydrate(27).build();

```
但是Builder模式也不是完美的，创建对象前需要先创建Builder对象，而且Builder代码相对比较多，并不适合参数少的时候使用。但是一个类如果要考虑以后扩展属性，最好一开始就使用Builder模式，因为属性越多，静态工厂和构造函数就越难控制。

## 0x03 单例模式
单例模式一般面试被问到的可能性比较高，什么饱汉饿汉的区别，几种单例模式的写法，单例模式的好处自然是很多的，对只需要被实例化一次的类，最好使用单例模式
一般单例模式的类的构造器被私有化，并且加了一重或者两重判断来保障线程安全，并且有的写法还在构造器中添加防止反射强制实例化的代码。
关于单例模式的几种写法我就不写了，网上有。
Effective Java上介绍的单例模式的代码
```java
// 单例模式静态成员变量
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ... }
}
```

```java
// 静态工厂方法
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE }

    public void leaveTheBuilding() { ... }
}
```

```java
// 枚举单例模式
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```

另外为了防止反序列化出假冒的单利的对象，书上说要加上这么一句。具体的牵扯到后面内容，坑以后填。

```java
// readResolve method to preserve sigleton property
private Object readResolve() {
    // Return the one true Elvis and let the garbage collector
    // take care of the Elvis impersonator.
    return INSTANCE;
}
```

##0x04 私有构造器强化 禁止工具类被实例化
Math类，Arrays类这些工具类是不应该被实例化的，因为里面的方法都是静态的。并且没有实例化的意义，实例化反而会浪费内存。所以编写这类Java类的时候，最好将构造器私有化。
当实例化这些类的时候，应该有异常抛出。
```java
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
    ... // Remainder omitted
}
```

##0x05 不重复创建对象

书中建议相同功能的对象只需要创建一次就行了，不需要多次创建，另外要尽可能的重用对象，高效运用内存。
```java
String s = new String("stringette"); // 别这么干！
```
上面的代码会在常量池创建一个String，再在用构造器在堆里创建一个String，相当于两次创建，最好是这样的写法

```java
String s = "stringette";
```

另外不变的常亮最好事先加载，不要每次使用对象的时候重新创建。书中用了一个例子，判断一个人是不是1946年至1964年生的
```java
public class Person {
    private final Date birthDate;

    // 2B程序员的写法
    public boolean isBabyBoomer() {
        // 没有必要每次都创建Calendar对象
        Calendar gmtCal = 
            Calendar.getInstance(TimeZone.getTimeZone("GMT"));
        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
        Date boomStart = gmtCal.getTime();
        gmtCal.set(1964, Calendar.JANUARY, 1, 0, 0, 0);
        Date boomEnd = gmtCal.getTime();
        return birthDate.compareTo(boomStart) >= 0 && 
               birthdate.compareTo(boomEnd)   <  0;
    }
}
```
很明显，下面代码会好很多，如果你看不出来，请回炉重学java
```java
public class Person {
    private final Date birthDate;

    /**
     * 正经程序员的写法
     */
    private static final Date BOOM_START;
    private static final Date BOOM_END;

    static {
        Calendar gmtCal = 
            Calendar.getInstance(TimeZone.getTimeZone("GMT"));
        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_START = gmtCal.getTime();
        gmtCal.set(1964, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_END = gmtCal.getTime();
    }

    public boolean isBabyBoomer() {
        return birthDate.compareTo(boomStart) >= 0 && 
               birthdate.compareTo(boomEnd)   <  0;
    }
}
```

在这一节，还讲到了拆装箱对程序性能的影响

```java
public static void main(String[] args) {
    Long sum = 0;
    for (long i = 0; i < Integer.MAX_VALUE; i++) {
        sum += i;
    }
    System.out.println(sum);
}
```
因为将long的第一个字母大写了，导致程序慢了近40秒。
并不是所有创建对象的开销都很大，但是重复创建是很浪费的，内存就那么大，CPU速度也有上限，无意义的拆装箱浪费了性能。所以养成好习惯，节约内存。

另外书中说自己维护对象池是个费力不讨好的事情，如果能交给GC，请务必交给GC。否则代码后期维护将是一个很重量级的工作。

## 0x06 及时丢弃无用对象
首先看一段代码,是一种栈的实方式
```java
// 这里藏着一个内存泄漏的隐患
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 确保至少有一个元素的可用空间，每次到达临界时让容量增加一倍。
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copayOf(elements, 2 * size + 1);
    }
}
``` 

写C/C++的程序员在对象失去作用的时候，会把对象置空。这个是好办法，但是java里没有必要这样,所以程序员对释放资源松懈了.
栈弹出元素后，需要解除对这个元素的引用，否则有可能会导致内存泄漏。

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 置空引用
    return result;
}
```

java的缓存是一个内存泄漏的高发地段，因为缓存的对象会被遗忘，最后即使不用也放在缓存中，所以现在的带缓存的功能都有一个时间机智，一段时间不用后，自动回收。

另外回调函数也有可能造成内存泄漏。如果一个对象注册了回调，但是还没等到回调这个对象就被干掉了，这时候，回调的时候就出现了内存泄漏问题。所以注册回调的时候最好是弱引用。减少内存泄漏的概率

## 0x07 尽量别使用finalizer方法
书中说了很多，概括一下就是
+ 使用这个方法不知道什么时候会被调用，甚至不会执行，容易造成内存泄漏
+ 即使被调用了，不一定会让你得到想要的结果，比如打印异常日志。
+ 如果是多线程，分布式系统，容易导致系统崩溃。（线程被锁住，宕机）
+ 严重的性能损耗
所以能在对象回收前做完的，不要等到对象失去引用后再做！能不用finalize()方法就不用。

至于好处嘛，finalizer方法的确有两个优点
当对象的所有者忘记调用前面段落中建议的显式终止方法时，可以作为保险方法
另一种是对GC不知道的对象进行保险回收操作，比如Native Peer对象。

（`FileInputStream`、`FileOutputStream`、`Timer`和`Connection`），都具有终结方法，当它们的close方法未能被调用的情况下，终结方法提供了一层保障。
书上说：
> 总之，除非是作为安全网，或者是为了终止非关键的本地资源，否则请不要使用终结方法。在这些很少见的情况下，既然使用了终结方法，就要记住调用super.finalize。如果终结方法作为安全网，要记得记录终结方法的非法用法。最后，如果需要把终结方法与公有的非final类关联起来，请考虑使用终结方法守卫者，以确保即使子类的终结方法未能调用super.finalize，该终结方法也会被执行。


love & peace

转载请注明出处：https://micorochio.github.io/2017/07/28/reading-effective-java-01/

如若有误请帮忙指正，谢谢