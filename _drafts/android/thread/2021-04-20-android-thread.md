---
layout: 
title: Android Thread
date: 2022-04-20 23:05 +0800
tags: android
---



按照操作系统中的描述，线程是CPI调度的最小单元，同时线程是一种有限的系统资源。

而进程一般值一个执行单元，在PC和移动设备上指一个程序或者一个应用。一个进程可以包含多个线程。

<!--more-->



## Android 多线程操作方式

### 1. Handler + Thread

![android-handler-thread](https://s2.loli.net/2022/07/19/7sh8NZvelqE3MTF.webp)

### 2. ThreadPoolExecutor

ThreadPoolExecutor 提供了一组线程池，可以管理多个线程并行执行。这样一方面减少了每个并行任务肚子简历线程的开销，另一方面可以管理多个并发线程的公共资源，从而提高多线程的效率。

所以ThreadPoolExecutor 比较适合一组任务的执行。

![thread-pool-executor](https://s2.loli.net/2022/07/19/Ef3yQumXzZ4BIT7.webp)



#### 线程池的结构和原理

一个完整的线程池应该有以下几个部分组成：

- 核心线程
  核心进程即线程池汇总长期存活的线程，即使闲置下来也不会被销毁，需要的时候可以直接拿来用。

- 任务队列

- 非核心进程

  普通线程则有一定的寿命，如果闲置时间超过寿命，则这个线程就会被销毁。

当我们通过线程池执行异步任务的时候，其实是依次进行了下面的流程：

1. 检查核心线程数是否达到最大值，否则创建新的核心线程执行任务，是则进行下一步
2. 检查任务队列是否已满，否则将任务添加到任务队列中，是则进行下一步
3. 检查非核心线程数是否达到最大值，否则创建新的非核心线程执行任务，是则说明这时线程池已经饱和，执行饱和策略，默认的饱和策略是抛出 RejectedExecutionException 异常



#### Executors 提供的四种创建 ThreadPoolExecutor 的方法

```java
// 创建一个定长的线程池，每提交一个任务就创建一个线程，直至达到池的最大长度，这时线程池保持长度不变。
// 使用场景：明确同时执行任务数量时
Executors.newFixedThreadPool();

// 创建一个可缓存的线程池，如果当前线程池的长度超过了处理的需要时，它可以灵活的回收空闲的线程，当需要增加时，它可以灵活的添加新的进程，而不会对线程池的长度作任何限制。
// 使用场景：处理大量耗时较短的任务
Executors.newCachedThreadPool();

// 创建一个定长的线程池，而且支持定时的以及周期性的任务执行，类似于Timer
// 使用场景：处理延时任务
Executors.newScheduledThreadPool();

// 创建一个单线程化的executor，它只创建唯一的worker线程来执行任务。
// 使用场景：多个任务需要访问同一个资源时
Executors.newSingleThreadExecutor();
```



#### 线程池中的一些重要的概念



#### 线程池的优点

- 重用线程池中的线程，避免因为线程的创建和销毁所带来的性能开销
- 能有效控制线程池的最大并发数，避免大量的线程之间因互相抢占系统资源而导致的阻塞现象
- 能够对线程进行简单的管理，并提供定时执行以及制定间隔循环执行等功能



### 3. IntentService

IntentService 继承自 Service，是一个经过包装的轻量级的 Service，**用来接收并处理通过 Intent 传递的异步请求**，客户端通过调用 startService(Intent) 启动一个 IntentService，利用一个work线程依次处理顺序过来的请求，处理完成后自动结束Service。

IntentService 内部实现是封装了HandlerThread 和 Handler ，使用的话要遵循 Service 的使用规则。



## RxJava

## Kotlin 协程

根据官方的定义，协程本质上是轻量级的线程，说明协程的资源消耗和性能等方面和线程比起来应该是有优势的。

## AsyncTask

