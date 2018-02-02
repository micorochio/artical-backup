---
title:
- 函数式接口功能笔记
date:
- 2017-01-23 14:29:36
tags:
- Java
- Lambda
- 函数式接口
- 函数式编程
---

## 0x00 Consumer
接受一个参数，不反回结果，所以是消费者
```Java
   /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);
```
内部还有一个默认方法
```Java
   /**
     * 返回一个组合的Consumer，依次执行前面Consumer的`accept()`， 接着是{@code after}后操作。
     * 如果执行任一操作抛出异常，则将异常抛出给操作的调用方。
     * 如果执行前面的`accept()`会引发异常，则不执行{@code after}后的操作。
     *
     * @param after the operation to perform after this operation
     * @return a composed {@code Consumer} that performs in sequence this
     * operation followed by the {@code after} operation
     * @throws NullPointerException if {@code after} is null
     */
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
```
<!--more-->
## 0x01 Function
接受一个参数，返回一个参数，是处理一个对象，所以是功能
```Java
/**
     * 请求执行，就是apply
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);

    /**
     * 返回一个组合Function，它首先执行{@code before}内的方法，
     * 将方法返回值作为下一个Function函数的输入参数，
     * 如果两个函数的求值都抛出异常，它将被传递给组合函数的调用者。
     */
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    /**
     * 返回一个组合Function，首先执行super中的方法，
     * 然后将{@code super}函数的返回值作为after内方法是输入参数，再执行after内的方法。
     * 如果两个函数的求值都抛出异常，它将被传递给组合函数的调用者。
     */
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    /**
     * 返回这个Function自己
     */
    static <T> Function<T, T> identity() {
        return t -> t;
    }
```

一般用来做行为传递，将行为交给编码者<br>
**eg：**
```Java
public void testLambda() {
    doSomeXX(12, num -> num + num + num, c -> c * c);
}

 public Function doSomeXX(int num, Function<Integer, Integer> fistDo,Function<Integer, Integer> andThen) {
    fistDo.andThen(andThen).apply(num);
    return fistDo;
}
```

## 0x02 Pradicate
输入一个值，判断是否符合要求，返回一个`boolean`值.
```Java
   /**
     * 测试参数t,是否符合要求
     */
    boolean test(T t);

    /**
     * 创建一个组合Predicate，同时测试t是否符合另一标准，并且返回这两个结果的`&&`值。
     */
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    /**
     * 返回测试结果的非运算值`!`,即不符合标注。
     */
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    /**
     * 返回一个组合的Predicate，将两个Predicate的执行结果进行`||`运算。
     */
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    /**
     * 差不多是equals的意思
     */
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
```

下面这个例子包含了stream，可以只看看思想.从组数据中筛选出质数的集合

代码丑了点，别介意😜
**eg：**
```Java
    List<Integer> natureNum = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20);
    List<Integer> primeNum = natureNum.stream()
            .filter(n -> {
                if (n <= 2) {
                    return true;
                }
                for (int a = 2; a <= Math.ceil(n / 2); a++) {
                    if (n % a == 0) {
                        return false;
                    }
                }
                return true;
            }).collect(Collectors.toList());
    primeNum.forEach(System.out::println);
```
## 0x03 Supplier
不用接受任何输入参数，获得一个固定的结果，所以他是提供者
```Java
public interface Supplier<T> {

    /**
     * 获得一个结果
     */
    T get();
}

```
一般在单例模式里面可以用这个，但是没想不出来模拟使用场景，所以没有例子。
> 注意，无输入参数！



## 0x04 UnaryOperator
这个，还没看出怎么用，他继承了`Function`接口，所以功能跟`Function`接口类似
```Java
public interface UnaryOperator<T> extends Function<T, T> {

    /**
     * 返回自己
     */
    static <T> UnaryOperator<T> identity() {
        return t -> t;
    }
}
```
## 0x05 BiFunction
接受两个参数，处理之后返回一个值
```Java
   /**
     * Applies this function to the given arguments.
     *
     * @param t the first function argument
     * @param u the second function argument
     * @return the function result
     */
    R apply(T t, U u);

    /**
     * 参考Function接口的andThen 
     */
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t, U u) -> after.apply(apply(t, u));
    }
```
BiFunction没有compose方法，因为BiFunction无法返回两个值，所以组合的时候会少一个参数，干脆就没有组合方法了
**eg：**
```Java
    BiFunction<Integer ,List<Person>,List<Person>> b = (age,allPerson)->{
        for(Person person:allPerson) {
            if(person.getAge() > age ){
                allPerson.remove(person);
            }
        }
        return allPerson;
    };
    List<Person> young =  b.apply(30, persons);

```

## 0x06 BinaryOperator
继承BiFunction，接受两个类型相同的参数，返回一个类型跟参数一样的结果
```Java
public interface BinaryOperator<T> extends BiFunction<T,T,T> {
    /**
     * 创建一个BinaryOperator，这个BinaryOperator可以根据comparator，返回输入参数中较小的值
     */
    public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
    }

    /**
     * * 创建一个BinaryOperator，这个BinaryOperator可以根据comparator，返回输入参数中较大的值
     */
    public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
    }
}
```
**eg：**
```Java
public String getBigString(){
    String a ="Hello", b = "world";
    BinaryOperator<String> m = BinaryOperator.maxBy((x,y)->x.length()-y.length());
    /* System.out.println(m.apply(a,b)); */
    return m.apply(a,b)
}
```