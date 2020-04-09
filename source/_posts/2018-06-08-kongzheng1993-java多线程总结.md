---
layout: post
title: "java多线程总结"
date: 2018-06-08
excerpt: "java 多线程"
tags: [java,多线程,安全]
categories: [java]
comments: true
---

最近面试两场，都是银行保险类的项目，面试官都问了我多线程的一些问题。今天抽空总结一下。

## 相关概念

1. 进程：一段程序的执行过程。是操作系统动态执行的基本单元，也是资源分配的基本单元。进程可以理解为是“进行中的程序”，处理器赋予时，程序才变成活动的，也就是进程。
2. 程序：指令和数据的有序集合。本身是静态的概念。
3. 线程：通常在一个进程中可以包含若干个线程，当然一个进程中至少有一个线程，不然没有存在的意义。线程可以利用进程所拥有的资源，在引入线程的操作系统中，通常都是把进程作为分配资源的基本单位，而把线程作为独立运行和独立调度的基本单位，由于线程比进程更小，基本上不拥有系统资源，故对它的调度所付出的开销就会小得多，能更高效的提高系统多个程序间并发执行的程度。
4. 多线程：在一个程序中，这些独立运行的程序片段叫作“线程”（Thread），利用它编程的概念就叫作“多线程处理”。多线程是为了同步完成多项任务，$\color{red}{不是为了提高运行效率，而是为了提高资源使用效率来提高系统的效率}$。线程是在同一时间需要完成多项任务的时候实现的。
5. 进程和线程的相同与不同：
    - 不同点：
        进程：每个进程都有独立的代码和数据空间（进程上下文），进程间的切换会有较大的开销，一个进程包含1--n个线程。（进程是资源分配的最小单位）。多进程是指操作系统能同时运行多个任务（程序）。
        线程：同一类线程共享代码和数据空间，每个线程有独立的运行栈和程序计数器(PC)，线程切换开销小。（线程是cpu调度的最小单位）。多线程是指在同一程序中有多个顺序流在执行。
    - 相同点：
        线程和进程一样分为五个阶段：创建、就绪、运行、阻塞、终止。

## 实现多线程

在java中要想实现多线程，有两种手段，一种是继续Thread类，另外一种是实现Runable接口。(其实准确来讲，应该有三种，还有一种是实现Callable接口，并与Future、线程池结合使用）。

### 实现Runnable接口比继承Thread类所具有的优势：

1. 适合多个相同的程序代码的线程去处理同一个资源
2. 可以避免java中的单继承的限制
3. 增加程序的健壮性，代码可以被多个线程共享，代码和数据独立
4. 线程池只能放入实现Runable或callable类线程，不能直接放入继承Thread的类

## 线程状态转换

<img src='20150309140927553.jpeg'>

1. 新建状态（New）：新创建了一个线程对象。
2. 就绪状态（Runnable）：线程对象创建后，其他线程调用了该对象的start()方法。该状态的线程位于可运行线程池中，变得可运行，等待获取CPU的使用权。
3. 运行状态（Running）：就绪状态的线程获取了CPU，执行程序代码。
4. 阻塞状态（Blocked）：阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。阻塞的情况分三种：

    1. 等待阻塞：运行的线程执行wait()方法，JVM会把该线程放入等待池中。(wait会释放持有的锁)
    2. 同步阻塞：运行的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池中。
    3. 其他阻塞：运行的线程执行sleep()或join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。（注意,sleep是不会释放持有的锁）
5. 死亡状态（Dead）：线程执行完了或者因异常退出了run()方法，该线程结束生命周期。

## 线程池

阿里的java开发手册中指出，线程资源必须通过线程池提供，不允许在应用中自行显示的创建线程，这样更加规范，而且可以合理控制开辟线程的数量；另一方面线程细节管理交给线程池处理，优化了资源的开销。而且手册指出，线程池不允许使用Executors去创建，要通过ThreadPoolExecutor方式，这一方面是由于jdk中Executor框架虽然提供了如newFixedThreadPool()、newSingleThreadExecutor()、newCachedThreadPool()等创建线程池的方法，但都有其局限性，不够灵活；另外由于前面几种方法内部也是通过ThreadPoolExecutor方式实现，使用ThreadPoolExecutor有助于大家明确线程池的运行规则，创建符合自己的业务场景需要的线程池，避免资源耗尽的风险。

```java

public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }

```

上面是ThreadPoolExecutor的构造函数：

- corePoolSize:指定了线程池中的线程数量，它的数量决定了添加的任务是开辟新的线程去执行，还是放到workQueue任务队列中去；

- maximumPoolSize:指定了线程池中的最大线程数量，这个参数会根据你使用的workQueue任务队列的类型，决定线程池会开辟的最大线程数量；

- keepAliveTime:当线程池中空闲线程数量超过corePoolSize时，多余的线程会在多长时间内被销毁；

- unit:keepAliveTime的单位

- workQueue:任务队列，被添加到线程池中，但尚未被执行的任务；它一般分为直接提交队列、有界任务队列、无界任务队列、优先任务队列几种；

- threadFactory:线程工厂，用于创建线程，一般用默认即可；

- handler:拒绝策略；当任务太多来不及处理时，如何拒绝任务；



