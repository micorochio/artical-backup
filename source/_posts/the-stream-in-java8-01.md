---
title:
- 学习Java8中的Stream （一）
date:
- 2018-01-21 23:32:52
tags:
- Java
- 笔记
- Java进阶
- Stream
- 流
---
不想再用for嵌套for操作了，java8 带来了新的API —— Stream，非常强大！
Stream中文翻译成流，是一个支持串行和并行操作元素的序列，也是Lambda表达式配合使用的强大工具。

 
源码在java.util.stream中，感兴趣可以阅读阅读。

![](http://upload-images.jianshu.io/upload_images/1112615-b50c59601c96c8df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!-- more -->
## 0x00 java8 中的流是什么
在编程中，对一串数据进行0个或者多个中间操作后，最后再获得结果。这个操作在没有流的情况下一般会涉及到多次循环，这是非常低效的。

流是为了处理一串数据（sequence），而不需要多次循环的一种方式。

流在操作序列的时候，会将数据放在一个叫Stream Pipeline的地方，这个地方会有三部分
+ 源（一般为集合）
+ 0或多个中间操作 （一般为惰性操作，不会直接操作数据）
+ 终止操作 （一般为求最终值，这时流的整个流程结束）

流支持并行操作，而迭代器，for循环都是串行操作，所以流在多核处理上有强大优势。 
![流处理模型](http://upload-images.jianshu.io/upload_images/1112615-cf1b08a3a58deb6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1x00 Stream的类型和创建方式
+  创建流
```java
// 从数组创建
        int [] source = {1,2,3,4,5,6};
        IntStream s = Arrays.stream(source);
// 从集合创建
        List list = Arrays.asList(1,2,3,4,5);
        Stream s2 = list.stream();
// 创建1到10的流
        IntStream s3 = IntStream.range(1,10);
//  直接创建
        Stream s4 = Stream.of("wo", "ai", "?")
```
+ 其他流
此外，流还提供了几种包装好的流：
```
// 支持串行并行操作的序列，元素只有double类型的流
DoubleStream 
```
```
// 支持串行并行操作的序列，元素只有int类型的流
IntStream
```
```
// 支持串行并行操作的序列，元素只有long类型的流
LongStream
```

## 2x00 常用的流方法
这里不过多提及，最好看一看API文档，我这里举一些常用的例子
```
// （惰性操作）中间操作，遍历
Stream<T> map(Function<? super T,? extends R> mapper)

// （及早求值操作）终止操作，遍历
void forEach(Consumer<? super T> action)
```
例子：
```
// 将元素的平方打印出来
int[] nums = {2,3,4,5,6};
Arrays.stream(nums)
    .map(i->i*i)
    .forEach(System.out::println);
```

![运行](http://upload-images.jianshu.io/upload_images/1112615-0fa91ce0a2b897d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
// 将元素中的所有偶数累加求和
int[] nums = {2, 3, 4, 5, 6};
System.out.println(
        Arrays.stream(nums)
                .map(i -> i % 2 == 0 ? i : 0)
                .reduce(0, Integer::sum)
);
```
![运行结果](http://upload-images.jianshu.io/upload_images/1112615-ea2c98cdf9090aa2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
// flatMap处理嵌套的list
        List<List<Integer>> ll =
                Arrays.asList(
                        Arrays.asList(1, 2, 3),
                        Arrays.asList(11, 22, 33),
                        Arrays.asList(0xF1, 0xF2, 0xF3)
                );

        ll.stream()
                .flatMap(list -> list.stream())
                .map(i -> 2 * i)
                .forEach(i -> System.out.println(i));
```
![运行结果](http://upload-images.jianshu.io/upload_images/1112615-114631a7eff0df74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

假设有N条营业数据，前5条是无关的测试数据，中间10条是要参加考核的，参与考核的需要知道其中超过50w（包括50）的数据的交易额平均值，其他不参与考核的忽略。
测试数据如下：
`{11, 9, 2, 13, 1, 2, 99, 54, 23, 66, 70, 23, 46, 50, 100, 10, 24, 18, 19, 2};`

```java
  Stream<Integer> trans = Stream.of(11, 9, 2, 13, 1, 2, 99, 54, 23, 66, 70, 23, 46, 50, 100, 10, 24, 18, 19, 2);

        IntSummaryStatistics all = trans
// 前5条跳过，2, 99, 54, 23, 66, 70, 23, 46, 50, 100, 10, 24, 18, 19, 2
                .skip(5)
// 取10条考核交易 2, 99, 54, 23, 66, 70, 23, 46, 50, 100
                .limit(10)
// 将50以下的交易剔除 99, 54, 66, 70, 50, 100
                .filter(i -> i >= 50)
// 转换成数字。如果是IntStream 则不需要转换
                .mapToInt(i->i)
// 将流的统计结果放入包装对象中
                .summaryStatistics();
// 交易总量 439w，平均值为439/6
        System.out.println(all.getAverage());
```
![运行结果](http://upload-images.jianshu.io/upload_images/1112615-925980d98317d48b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


以上为流的一些基础使用方法。后续会有一些详细的补充，容我后面填坑。

## 3x00 流的特性
+ 基于集合或者序列
+ 流不存储值，也不能重复使用，数据通过管道的方式进行操作
+ 每个操作都是函数式的，对流的操作不会影响源数据
+ 多数操作（排序，映射，过滤等），可以延迟实现

基于集合和序列就不写例子了，不存储值也是一个概念，下面验证一下流不能重复使用。
#####3x01 不能重复使用
```java
        Stream<Integer> trans = Stream.of(11, 9, 2);
        trans.forEach(i -> System.out.println(i));
        trans.reduce(0, Integer::sum);
```
当我第二次使用trans时，报错了。
![运行结果](http://upload-images.jianshu.io/upload_images/1112615-db48f4d73e9e34a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**流只能使用一次，无法重复使用**

##### 3x02 验证流延迟操作
流只要在终止操作(及早求值)时，才会对数据统一做操作，在没有遇到求值操作的时候，惰性操作代码不会被执行。
```java
        Stream<Integer> trans = Stream.of(11, 70, 23, 46, 50, 100, 10, 24, 18, 19, 2);
        trans.map(i->{
            System.out.println(i);
            return i;
        });
```
![运行结果，上面都没打印](http://upload-images.jianshu.io/upload_images/1112615-e44c8ecb04c55176.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 3x02 不影响源数据
可以创建一个List去实践，这里不写代码，当流执行完成之后，源List的数据是不会发生变化的
大家可以自己实践一下



by:Zing 
转载请注明出处：https://micorochio.github.io/2018/01/22/the-stream-in-java8-01/
