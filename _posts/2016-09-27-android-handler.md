---
layout: 
title: Android Handler
date: 2016-09-27 11:47 +0800
tags: android

---

<!--more-->



![android-handler-flowchart-diagram](https://s2.loli.net/2022/07/19/HTeOLWhGBJyZoiC.webp)

- 事件驱动模型
- 切换线程
- 执行延时任务
- 保证多个任务执行时的有序性



## Looper

![android-handler-looper-flowchart-diagram](https://s2.loli.net/2022/07/19/Kg3t8Ma2CWde9GQ.png)



## MessageQueue

MessageQueue是一个由单向链表构成的优先级队列。

当`next()`还处于阻塞状态时，外部向消息队列插入了一个可以立即处理或者是阻塞等待时间较短的Message，此时就需要唤醒休眠的线程。

```java
boolean enqueueMessage(Message msg, long when) {
  synchronized (this) {
    ...
  }
}
```

因为存在多个线程同时往一个MessageQueue发送消息的可能，所以`enqueueMessage()`内部需要进行线程同步，此外 `next()`方法内部也加上了同步锁，所以也保障了Looper分发Message的有序性。





## Message

Message 结构

- what：唯一标识
- obj：数据
- when：时间戳
- next：下一个节点
- target：对应的进行消息分发的Handler

### Message的复用原理

Message采用单向链表结构实现Message对象的管理，在`recyclerUnlock()`和`obtain()`里对Message进行回收，采用了栈的先进后出（FILO）思想，最新回收的对象会被优先再次使用。



### Message 的缓存复用

为了减少 Message 频繁重复创建的情况，Message 提供了 MessagePool 用于实现 Message 的缓存复用，以此来优化内存使用。

当 Looper 消费了 Message 后会调用`recycleUnchecked()`方法将 Message 进行回收，在清除了各项资源后会缓存到 sPool 变量上，同时将之前缓存的 Message 置为下一个节点 next，通过这种链表结构来缓存最多 50 个Message。这里使用到的是**享元设计模式**。



## 同步屏障

同步屏障可以理解为拦截同步消息的执行，来保证某些优先级高的异步消息能尽快被执行，而采用的一种消息屏障（Barrier）机制，ViewRootImpl就是通过该机制来使View的绘制流程可以优先被处理。

其大致流程是通过`postSyncBarrier()`发送一个屏障消息到MessageQueue中，当遍历到该屏障消息时，就会判断当前队列中是否存在异步消息，有的话则先跳过同步消息（开发者主动发送的都属于同步消息），优先执行异步消息。这种机制就会使得在异步消息被执行完之前，同步消息都不会得到处理。



## IdleHandler

IdleHandler是MessageQueue的一个内部接口，可以用于在Looper线程处于空闲状态的时候执行一些优先级不高的操作，通过MessageQueue的 `addIdleHandler` 方法来提交要执行的操作。

MessageQueue在执行 `next()`方法时，如果发现当前队列是空的或者队头消息需要延迟处理的话，那么就会去尝试遍历 `mIdleHandlers`来依次执行 IdleHandler。例如ActivityThread就向主线程MessageQueue添加了一个GcIdler，用于在主线程空闲时尝试去执行GC操作。



## view.post()







## Handler中的设计模式

### 生产者/消费者模式

Handler发送消息到MessageQueue的过程

### 享元模式



## Handler的内存泄露

- 当退出Activity时，如果作为内部类的 Handler 中还保存着待处理的延时消息的话，那么就会导致内存泄漏，可以通过调用`Handler.removeCallbacksAndMessages(null)`来移除所有待处理的Message。

- 内部类导致的内存泄露改为静态内部类，并对上下文使用弱引用



### epoll机制在Handler中的运用

epoll机制在Handler中的应用，主要在主线程的MessageQueue没有消息时，便阻塞在Looper的 `queue.next()`中的 `nativePollOnce()`方法里，最终调用到`epoll_wait()`进行阻塞等待。此时主线程会释放CPU资源进入休眠状态，直到下个消息到达或者有事务发生，通过往`pipe`管道写端写入数据来唤醒主线程工作。

通过pipe管道写端写入数据来唤醒主线程工作，原理类似于I/O，读写是阻塞的，不占用CPU资源。

epoll机制：在没有消息时阻塞在pipe管道的读端。

没有消息时会调用到epoll_wait()使得线程被挂起休眠，且不耗费CPU；有消息时，通过epoll机制唤醒主线程。

epoll机制是一种I/O多路复用机制，可以同时监听多个描述符，当某个描述符就绪（读或写就绪），则立即通知相应程序进行读或写操作，本质同步I/O，即读写是阻塞的。





参考资料：
[I/O多路复用技术（multiplexing）是什么？](https://www.zhihu.com/question/28594409)
