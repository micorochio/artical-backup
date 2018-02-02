---
title:
- Java线程学习笔记
date:
- 2017-12-10 01:17:45
tags:
- Java
- 笔记
- Java基础
- 多线程
---
线程学习笔记，
转载请注明出处：https://micorochio.github.io/2017/12/10/learning-thread-note-01/

![](http://upload-images.jianshu.io/upload_images/1112615-a8dafc7ea73afce7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!-- more -->
## 线程的创建与销毁
+ 创建运行

一般情况，线程的创建有两种方式

1. 实现Runnable接口，放入Thread类中执行
2. 继承Thread类，覆写run方法。

启动：`start();`

> 注意点：thread对象虽然可以直接调用run方法，但是，直接调用run方法不会启动新线程，只是在当前线程执行run方法而已，只有使用start方法，才会启动新线程！

+ 停止和销毁

1. 自然停止，当run方法执行完成后，线程会自动停止。
2. stop方法强制停止。
3. interrupt中断线程。

自然停止的线程无须关系，因为执行结束就自动停止了。
然而对可以无限时间执行的线程，需要注意：
> **stop方法不能手动直接调用。** 
> stop方法会直接将线程置空，会抛出`ThreadDeath`错误，无法预料线程对数据的影响。

> **interrupt方法对处于阻塞中的线程，无法处理**
> 线程执行sleep或wait方法时，或是读取文件，执行interrupt会抛出异常，因为中断线程导致线程无法完成正在等待的任务。

提倡方法,run方法中使用isInterrupted()作为终止条件
```java
   public void run() {
        while(!isInterrupted()){
            // TODO 正常逻辑
        }
    }

    public synchronized void stopTheThread(){
        isInterrupted = false;
    }
```
这样调用interrupt方法时，逻辑会在执行完成后停止线程。

> *ps: interrupt不能中断正在进行的数据写入。*

## JVM中线程的内存分布工作方式，以及线程通信

[Java线程的内存模型基础知识](http://www.jianshu.com/p/4e0bfcb1711e)
##线程中的异常处理

所有线程都不允许抛出未捕获的checked exception，如果线程任务中有checked exception，需要自己catch住，并处理掉，如果是unchecked exception出现的话，这个线程就挂了，当然，不会直接影响其他线程，导致其他线程或宿主宕掉。

如果想处理`unchecked exception`，可以使用`public void uncaughtException(Thread t, Throwable e)` 方法处理,但是这个方法依旧在`Thread`内部。而且若是在线程池中，这个方法是不会被调用的。
╮(╯ ╰)╭

## 死锁与解决办法
什么是线程死锁，看一段代码

```java
        String lock1 = "l1";
        String lock2 = "l2";

        Thread t1 = new Thread(()->{
            synchronized (lock1){
                System.out.println("t1:"+lock1);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                synchronized (lock2){
                    System.out.println("t1:"+lock2);
                }
            }
        });


        Thread t2 = new Thread(()->{
            synchronized (lock2){
                System.out.println("t2:"+lock1);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                synchronized (lock1){
                    System.out.println("t2:"+lock1);
                }
            }
        });


        t1.start();
        t2.start();
```
这段代码运行后，无法退出，因为t1进入sleep后，t2占用了lock2，然后t2 没有释放lock2 就进入了sleep，切换回t1，这时候lock2被占用，无法进一步执行，等待线程时长结束后，t1没有释放lock1 导致t2也不能进入下一步执行,这就形成了死锁。

避免死锁可以从下面几个角度去修改代码:
> 1. 破坏互斥条件：不要出现一个资源只能被一个进程占用，直到该进程释放资源 
> 2. 取消请求和保持条件：当一个线程等不到请求的资源时，不要阻塞。 
> 3. 剥夺条件：尽量不要让一个变量只能被一个线程独占，
> 4. 打破循环等待：当发生死锁时，所等待的进程必定会形成一个环路，尽量不要让线程相互占用对方的独立资源，从而导致循环阻塞。

这四个满足一个就可以避免死锁的发生。

By：Zing，转载请注明出处：https://micorochio.github.io/2017/12/10/learning-thread-note-01/