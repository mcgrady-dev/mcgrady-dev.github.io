---
layout: 
title: Android Handler
date: 2016-09-27 11:47 +0800
tags: android

---

<!--more-->

## Handler

![android-handler-flowchart-diagram](https://s2.loli.net/2022/07/19/HTeOLWhGBJyZoiC.webp)

- 事件驱动模型
- 切换线程
- 执行延时任务
- 保证多个任务执行时的有序性



## Looper

![android-handler-looper-flowchart-diagram](https://s2.loli.net/2022/07/19/Kg3t8Ma2CWde9GQ.png)



## MessageQueue

MessageQueue 是一个由单向链表构成的优先级队列

当next()还处于阻塞状态时，外部向消息队列插入了一个可以立即处理或者是阻塞等待时间较短的Message，此时就需要唤醒休眠的线程。

enququeMessage

```java
boolean enqueueMessage(Message msg, long when) {
  synchronized (this) {
    ...
  }
}
```

因为存在多个线程同时往一个 MessageQueue 发送消息的可能，所以 enqueueMessage 内部需要进行线程同步，此外 next() 方法内部也加上了同步锁，所以也保障了 Looper 分发 Message 的有序性



## Message

Message 结构

- what：唯一标识
- obj：数据
- when：时间戳
- next：下一个节点
- target：对应的进行消息分发的Handler



### Message 的缓存复用

为了减少 Message 频繁重复创建的情况，Message 提供了 MessagePool 用于实现 Message 的缓存复用，以此来优化内存使用。

当 Looper 消费了 Message 后会调用`recycleUnchecked()`方法将 Message 进行回收，在清除了各项资源后会缓存到 sPool 变量上，同时将之前缓存的 Message 置为下一个节点 next，通过这种链表结构来缓存最多 50 个Message。这里使用到的是**享元设计模式**。



## 同步屏障

同步屏障可以理解为拦截同步消息的执行，来保证某些优先级高的异步消息能尽快被执行，而采用的一种消息屏障（Barrier）机制。

其大致流程是：通过 `postSyncBarrier()` 发送一个屏障消息到 MessageQueue 中，当 MessageQueue 遍历到该屏障消息时，就会判断当前队列中是否存在异步消息，有的话则先跳过同步消息（开发者主动发送的都属于同步消息），优先执行异步消息。这种机制就会使得在异步消息被执行完之前，同步消息都不会得到处理。

屏障消息的作用就是可以确保高优先级的异步消息可以优先被处理，ViewRootImpl 就是通过该机制来使 View 的绘制流程可以优先被处理。



## IdleHandler

IdleHandler 是 MessageQueue 的一个内部接口，可以用于在 Loop 线程处于空闲状态的时候执行一些优先级不高的操作，通过 MessageQueue 的 `addIdleHandler` 方法来提交要执行的操作

MessageQueue 在执行 `next()` 方法时，如果发现当前队列是空的或者队头消息需要延迟处理的话，那么就会去尝试遍历 `mIdleHandlers`来依次执行 IdleHandler

例如，ActivityThread 就向主线程 MessageQueue 添加了一个 GcIdler，用于在主线程空闲时尝试去执行 GC 操作



## Handler中的设计模式

### 生产者/消费者模式

Handler发送消息到MessageQueue的过程

### 享元模式



## Handler的内存泄露

- 当退出 Activity 时，如果作为内部类的 Handler 中还保存着待处理的延时消息的话，那么就会导致内存泄漏，可以通过调用`Handler.removeCallbacksAndMessages(null)`来移除所有待处理的 Message

- 内部类导致的内存泄露改为静态内部类，并对上下文使用弱引用



## epoll机制

epoll 机制在 Handler 中的应用，主要在主线程的 MessageQueue 没有消息时，便阻塞在 looper 的 `queue.next()` 中的 `nativePollOnce()` 方法里，最终调用到 `epoll_wait()` 进行阻塞等待。此时主线程会释放 CPU 资源进入休眠状态，直到下个消息到达或者有事务发生，通过往 `pipe` 管道写端写入数据来唤醒主线程工作。

这里采用的 `epoll` 机制，是一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读/写就绪)，则立刻通知相应程序进行读/写操作，本质同步I/O，即读/写是阻塞的。 所以说，主线程大多数时候都是处于休眠状态，并不会消耗大量CPU资源。

在 Linux 新的内核中， `epoll` 用来替换 `select` ，`eopll` 是 Linux 内核为处理大批量文件扫描符而做了改进的 `poll` ，是Linux 下多路复用 I/O 接口 `select/poll` 的增强版，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统 CPU 利用率。`epoll` 内部用于保存事件的数据类型是红黑树，查找速度快，`select` 采用的数组保存信息，查找速度慢，只有当等待少量文件描述符时， `epoll` 和 `select` 的效率才会差不多。





