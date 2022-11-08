## Thread

### 线程的上下文切换（Thread Context Switch）

发生线程切换的原因：

- 线程的CPU时间片用完；
- 垃圾回收；
- 有更高优先级的线程需要运行；
- 线程主动调用了 `sleep` `yield` `wait` `join` `park` `synchronized` `lock`等方法；

当ContextSwitch发生时，需要由操作系统保存当前线程状态，并恢复另一个线程的状态，Java中对应的概念就是程序计数器（Program Counter Register），它的作用是记住下一条JVM指令的执行地址，是线程私有的。

线程的状态包括：程序计数器、虚拟机栈中每个栈帧的信息。

ContextSwitch频繁发生会影响性能

### 线程的状态

![ace830df-9919-48ca-91b5-60b193f593d2](https://s2.loli.net/2022/09/16/vkaWAUPpFZzhsHX.png)

从操作系统层面的状态分类：

- 初始状态
- 可运行状态
- 阻塞状态
- 运行状态
- 终止状态

从 Java API 层面的状态分类：

- NEW
- RUNNABLE
- WAITING
- TIMED_WAITING
- TERMINATED
- BLOCKED

### 线程的常见方法

- `sleep()`

- `yield()`
  向调度程序提示当前线程愿意放弃其当前对处理器的时间片的使用。当前线程也会从运行状态变成就绪状态，在下一次被调度时重新竞争CPU时间片的使用权；

- `join()`
  等待线程运行结束

- `interrupt()`

  中断目标线程，给目标线程一个终端信号，线程被打上中断标记：

  - 如果线程处于 `sleep` `wait` `join` （阻塞状态），会导抛出 `InterruptedException`，并清空打断标记；
  - 如果线程正在运行，会设置打断标记但不会终止线程，可以通过 `isInterrupted()` 判断是否结束线程的执行；

- `isInterrupted()`
  Thread的成员方法，判断是否被打断，不会清除打断标记；

- `interrupted()`
  Thread的静态方法，判断是否被打断，会清除打断标记；

### 线程优先级

线程优先级会提示调度器优先调度该线程，但它仅仅是一个提示，调度器可以忽略它；如果 cpu 比较忙，那么优先级高的线程会得到更多的时间片，但 cpu 闲时，优先级几乎没作用。

- MIN_PRIORITY：1
- NORM_PRIORITY：5
- MAX_PRIORITY：10

### 线程同步与异步

- 同步：需要等待结果返回，才能继续运行；
- 异步：不需要等待结果返回，就能继续运行；



### 守护线程

Java的线程分为两种：用户线程和守护线程，守护线程（Deamon Thread）是一个服务线程，用于服务其他（用户）线程，在JVM中，所有用户线程执行完毕后，无论有无守护线程，JVM都会自动退出。守护线程最典型的就是垃圾回收线程。

### 线程阻塞

线程阻塞通常指一个线程在执行过程中暂停以等待某个条件的触发。在阻塞状态下CPU时间片使用权是否流转根据具体方法而定：

1. **sleep()**：当线程调用`sleep()`方法后，线程处于睡眠状态，此时其它线程需要执行会进入阻塞状态，且`sleep()`方法调用完后线程也不会释放锁对象；
2. **wait()**：当线程在运行时，调用`wait()`方法会让出该线程的CPU使用权，然后进入等待状态，与睡眠状态不一样的是，需要执行`notify()`或`notfityAll()`方法来唤醒，被唤醒后将进入可运行状态，此时是没有CPU使用权的，需要重新竞争锁；
3. **yield()**：当线程在运行时，调用yield()方法会通知系统让出该线程的CPU使用权，同等级或更高一级的线程优先执行，让出后的线程进入阻塞状态，但该线程随时可能被分配到CPU的使用权；
4. **join()**：当线程在运行时，调用join()方法会进入阻塞状态，另一个线程会执行，直到执行结束，原线程才会进入可运行状态；



## 线程间通信

线程间通信方式有两种：共享内存和消息传递。

### 共享内存



### 消息传递





## ThreadLocal

Thread的局部变量，通过为每个线程提供一个独立的变量副本解决变量并发访问的冲突问题。

### ThreadLocalMap

ThreadLocal是如何做到为每一个线程维护变量的副本的呢？在ThreadLocal类中定义了一个ThreadLocalMap类型的变量threadLocals，用于存储每个线程的变量副本，只能被当前线程读取和修改。

在Therad类中有一个threadLocals和inheritableThreadLocals，它们都是ThreadLocalMap类型的变量，而 ThreadLocalMap是一个定制化的HashMap。ThreadLocal类通过操作每一个线程特有的ThreadLocalMap副本，从而实现了变量访问在不同线程中的隔离。因为每个线程的变量都是自己特有的，完全不会有并发错误。

### ThreadLocal与同步机制的比较

ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。

在同步机制中，通过对象的锁机制保证同一时间只有一个线程访问变量。这时该变量是多个线程共享的，使用同步机制要求程序慎密地分析什么时候对变量进行读写，什么时候需要锁定某个对象，什么时候释放对象锁等繁杂的问题，程序设计和编写难度相对较大。

而ThreadLocal则从另一个角度来解决多线程的并发访问。ThreadLocal会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。**因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了**。ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进ThreadLocal。

概括起来说，对于多线程资源共享的问题，同步机制采用了*“以时间换空间”*的方式，而ThreadLocal采用了*“以空间换时间”*的方式。前者仅提供一份变量，让不同的线程排队访问，而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响。



## ThreadPool

线程池是一种基于池化思想管理线程的工具，由于线程过多会带来调度开销，线程池维护着多个线程，并对线程进行统一的分配和监管，避免了在处理短时间任务时创建与销毁线程的代价，保证内核的充分利用，还能防止过分调度。

### ThreadPoolExecutor

#### BlockingQueue

#### Executors

Executors工具可以创建几种类型的线程池：

- **FixedThreadPool** 被称为可重用固定线程数的线程池；
- **SingleThreadExecutor** 只有一个线程的线程池；
- **CacheThreadPool** 是一个根据需要而创建新线程的线程池；

### ScheduledThreadPoolExecutor





参考文献：
[Java线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
