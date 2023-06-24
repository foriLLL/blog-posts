---
description: "多线程能满足程序员编写高效率的程序来达到充分利用 CPU 的目的。一个进程是一个独立(self contained)的运行环境，它可以被看作一个程序或者一个应用。而线程是在进程中执行的一个任务。Java运行环境是一个包含了不同的类和程序的单一进程。线程可以被称为轻量级进程。线程需要较少的资源来创建和驻留在进程中，并且可以共享进程中的资源。"
time: 2021-12-10 18:33:03+08:00
---

多线程能满足程序员编写高效率的程序来达到充分利用 CPU 的目的。一个进程是一个独立(self contained)的运行环境，它可以被看作一个程序或者一个应用。而线程是在进程中执行的一个任务。Java运行环境是一个包含了不同的类和程序的单一进程。线程可以被称为轻量级进程。线程需要较少的资源来创建和驻留在进程中，并且可以共享进程中的资源。

## 多线程编程的好处

在多线程程序中，多个线程被**并发**地执行以**提高程序的效率**，CPU不会因为某个线程需要等待资源而进入空闲状态。多个线程**共享堆内存**(heap memory)，因此创建多个线程去执行一些任务会比创建多个进程更好。  
大体来说有以下优点：
1. 资源利用率更好
2. 程序设计在某些情况下更简单
3. 程序响应更快

## 线程的生命周期

一个线程从产生到死亡有一个完整的生命周期。

<img width=700 src="https://www.runoob.com/wp-content/uploads/2014/01/java-thread.jpg"/>

**新建状态**:  
使用 new 关键字和 Thread 类或其子类建立一个线程对象后，该线程对象就处于新建状态。它保持这个状态直到程序 `start()` 这个线程。

**就绪状态**:  
当线程对象调用了 `start()` 方法之后，该线程就进入就绪状态。就绪状态的线程处于就绪队列中，要等待JVM里线程调度器的调度。

**运行状态**:  
如果就绪状态的线程获取 CPU 资源，就可以执行 `run()`，此时线程便处于运行状态。处于运行状态的线程最为复杂，它可以变为阻塞状态、就绪状态和死亡状态。

**阻塞状态**:  
如果一个线程执行了 `sleep`（睡眠）、`suspend`（挂起）等方法，失去所占用资源之后，该线程就从运行状态进入阻塞状态。在睡眠时间已到或获得设备资源后可以重新进入就绪状态。可以分为三种：
* 等待阻塞：运行状态中的线程执行 wait() 方法，使线程进入到等待阻塞状态。
* 同步阻塞：线程在获取 synchronized 同步锁失败(因为同步锁被其他线程占用)。
* 其他阻塞：通过调用线程的 sleep() 或 join() 发出了 I/O 请求时，线程就会进入到阻塞状态。当sleep() 状态超时，join() 等待线程终止或超时，或者 I/O 处理完毕，线程重新转入就绪状态。

**死亡状态**:  
一个运行状态的线程完成任务或者其他终止条件发生时，该线程就切换到终止状态。

## 创建一个线程

Java 提供了三种创建线程的方法：
* 通过实现 Runnable 接口；
* 通过继承 Thread 类本身；
* 通过 Callable 和 Future 创建线程。

### 通过实现 Runnable 接口来创建线程

创建一个新的线程最简单的方法就是创建一个实现Runnable接口的类。  
Runnable是一个方法接口（所以可以使用lambda表达式），一个类只需要执行一个方法调用 `run()`。  

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

你可以重写该方法，重要的是理解的 run() 可以调用其他方法，使用其他类，并声明变量，就像主线程一样。  

在有了实现了Runnable接口的类后，你就可以在你的代码里实例化线程对象（Thread）了，Thread定义了几个构造方法，常用：

```java
Thread(Runnable threadOb,String threadName);    // threadName指定新线程的名字
```

### 通过继承Thread来创建线程

创建一个线程的第二种方法是创建一个新的类，该类继承 Thread 类，然后创建一个该类的实例。  
继承类必须重写 run() 方法，该方法是新线程的入口点。它也必须调用 start() 方法才能执行。  
该方法尽管被列为一种多线程实现方式，但是***本质上也是实现了 Runnable 接口的一个实例。***

Thread类有一些重要的方法和静态方法，可以参考这个链接：[Thread方法](https://www.runoob.com/java/java-multithreading.html.reference)。

>  **可以直接调用Thread类的run()方法么？**  
当然可以，但是如果我们调用了Thread的run()方法，它的行为就会和普通的方法一样，为了在新的线程中执行我们的代码，必须使用Thread.start()方法使该线程开始执行，JVM会调用该线程的 run() 方法

### 通过Callable和Future创建线程

1. 创建 Callable 接口的实现类，并实现 call() 方法，该 call() 方法将作为线程执行体，并且**有返回值**。

2. 创建 Callable 实现类的实例，使用 FutureTask 类来包装 Callable 对象，该 FutureTask 对象封装了该 Callable 对象的 call() 方法的返回值。

3. 使用 FutureTask 对象作为 Thread 对象的 target 创建并启动新线程。

4. 调用 FutureTask 对象的 get() 方法来获得子线程执行结束后的返回值。

<img src="https://img.foril.fun/FutureTask%E7%BB%A7%E6%89%BF%E7%BB%93%E6%9E%84.jpg"/>

上图展示了FutureTask的继承实现结构，实质上是对Runnable的一个封装。

```java
@Test
@DisplayName("thread test")
public void testTest() throws ExecutionException, InterruptedException {
    class IC implements Callable<Integer>{
        @Override
        public Integer call() throws Exception {
            return 1;
        }
    }
    IC ic = new IC();
    FutureTask<Integer> ft = new FutureTask<>(ic);
    new Thread(ft).start();
    System.out.println(ft.get());
}
```

### 使用Callable的好处

Runnable缺少的一项功能是，当线程终止时（即run()完成时），我们无法使线程返回结果。为了支持此功能，Java中提供了Callable接口。  
而且Callable及Future能抛出异常，Runnable则不行。

## 参考

* https://www.runoob.com/java/java-multithreading.html
* https://www.cnblogs.com/dolphin0520/p/3932934.html
* https://www.cnblogs.com/guanbin-529/p/11784914.html