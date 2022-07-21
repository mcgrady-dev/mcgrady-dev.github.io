---
layout: 
title: Kotlin Coroutine
date: 2021-04-29 22:31 +0800
tags: [kotlin,coroutine]

---

**协程是一种非抢占式或者说协作式的计算机程序并发调度的实现，程序可以主动挂起或者恢复执行**。

我们在 Java 虚拟机上所认识到的线程大多数的实现是映射到内核的线程的，也就是说线程当中的代码逻辑在线程抢到 CPU 的时间片的时候才可以执行，否则就得歇着，当然这对于我们开发者来说是透明的；而经常听到所谓的协程更轻量的意思是，协程并不会映射成内核线程或者其他这么重的资源，它的调度在用户态就可以搞定，任务之间的调度并非抢占式，而是协作式的。

<!--more-->

## Introduction

- 协程是一种在程序中处理并发任务的方案，同时也是这种方案的一个组件
- 协程和线程属于一个层级的概念
- Kotlin对于JVM的协程，是一套线程框架

协程通过将复杂性方入库来简化异步编程。程序的逻辑可以在协程中顺序地表达，而底层库会为我们解决其异步性。该库可以将用户代码的相关部分波安装为回调、订阅相关事件、在不同线程（甚至不同机器）上调度执行，而代码则保持如同顺序执行一样简单。

**协程是一种并发设计模式（编程思想）**，您可以在 Android 平台上使用它来简化异步执行代码。简单的概括就是，以同步的方式去编写异步执行的代码。

协程依赖于线程，但协程挂起时不需要阻塞线程，几乎是无代价的。所以协程像是一种用户态的线程，非常轻量级，一个线程中可以创建N个协程。



## Coroutine 的分类

### 按调用栈分类

由于协程需要支持挂起、恢复，因此对于挂起点的状态保存就显得极其关键。类似地，线程会因为 CPU 调度权的切换而被中断，它的中断状态会保存在调用栈当中，因而协程的实现也按照是否开辟相应的调用栈存在以下两种类型：

- 有栈协程 Stackful Coroutine：每一个协程都会有自己的调用栈，有点儿类似于线程的调用栈，这种情况下的协程实现其实很大程度上接近线程，主要不同体现在调度上。
- 无栈协程 Stackless Coroutine：协程没有自己的调用栈，挂起点的状态通过状态机或者闭包等语法来实现。

Kotlin 的协程是一种无栈协程的实现，它的控制流转依靠对协程体本身编译生成的状态机的状态流传来实现，变量保存也是通过闭包语法来实现的，不过 Kotlin 的协程是可以通过 `suspend` 函数在任意调用层次挂起。

### 按调度方式分类

调度过程中，根据协程转移调度权的目标又将协程分为**对称协程**和**非对称协程**：

- 对称协程 Symmetric Coroutine：任何一个协程都是相互独立且平等的，调度权可以在任意协程之间转移。
- 非对称协程 Asmmetric Coroutine：协程出让调度权的目标只能是它的调用者，即协程之间存在调用和被调用关系。

Kotlin 中执行 `async`/`await`时，会将调度权转移到异步调用中，异步调用返回结果或抛出异常时总是将调度权转移回 `async`/`await` 的位置。

从实现的角度来讲，非对称协程的实现更加自然，不过只要对非对称协程稍作修改，即可实现对称协程的能力。在非对称协程的基础上添加一个重力的第三方作为协程调度权的分发中心，所有的协程在挂起时豆浆控制权转移给分发中心，分发中心根据参数来决定将调度权转移给目标线程，例如：Kotlin 中基于 Channel 的通信。





## Coroutine Builders

协程的创建是通过 CoroutineScope 创建，协程的启动方式有以下几种：

- **runBlocking**

  ```kotlin
  @Throws(InterruptedException::class)
  public fun <T> runBlocking(context: CoroutineContext = EmptyCoroutineContext, block: suspend CoroutineScope.() -> T): T
  ```

  runBlocking 是一个顶级函数，启动了一个新的协程并阻塞调用它的线程，直到里面的代码执行完毕，返回值是泛型 T。

  runBlocking 它设计的目的是将常规的阻塞代码连接到一起，主要用于 main() 函数和 Testing 中。

- **launch**

  ```kotlin
  public fun CoroutineScope.launch(
      context: CoroutineContext = EmptyCoroutineContext,
      start: CoroutineStart = CoroutineStart.DEFAULT,
      block: suspend CoroutineScope.() -> Unit
  ): Job
  ```

  启动一个协程但不会阻塞调用线程，必须在协程作用域（`CoroutineScope`）中才能调用，返回值是 Job。

- **async**

  ```kotlin
  public fun <T> CoroutineScope.async(
      context: CoroutineContext = EmptyCoroutineContext,
      start: CoroutineStart = CoroutineStart.DEFAULT,
      block: suspend CoroutineScope.() -> T
  ): Deferred<T>
  ```

  启动一个协程但不会阻塞调用线程，必须要在协程作用域 CoroutineScope 中才能调用，以 Deferred 对象的形式返回协程任务，返回值是 T。



### Job

**Job 是协程的句柄**。使用 `launch` 或 `async` 创建的每个协程都会返回一个 Job 实例，**该实例是相应协程的唯一标识并管理器生命周期**。您还可以将 Job 传递给 CoroutineScope 以进一步管理器生命周期。

我们可以通过下图大概了解协程作业创建到完成或取消的过程：

![kotlin-coroutine-job](https://s2.loli.net/2022/07/19/VFCJZ7QLSfKB8jk.png)

#### join





### Deferred

Deferred 继承自 Job，可以看看做一个带返回值的 Job。



## Coroutine Scope

协程的作用域是协程运行的作用范围。`launch`、`async` 都是 `CoroutineScope` 的扩展函数。

CoroutineScope 定义了新启动的协程作用范围，同时会继承他的 `coroutineContext` 自动传播其所有的 `elements` 和取消操作。换句话说，如果这个作用域销毁了，那么里面的协程也随之失效。

协程作用域分为以下三种：

### GlobalScope（顶级作用域）

没有父协程的协程所在的作用域称之为顶级作用域。

### 协同作用域

**子协程所在的作用域默认为协同作用域**。此时子协程抛出未捕获的异常时，会将异常传递给父协程处理，则所有子协程同时也会被取消。

### SupervisorScope（主从作用域）

与协同作用域一致，区别在于该作用域下的协程取消操作的单向传播性，子协程的异常不会导致其它子协程取消。但是如果父协程被取消，则所有子协程同时也会被取消。



### 协程作用域的使用建议

- 对于没有协程作用域，但需要启动协程的时候，适合用 GlobalScope
- 对于已经有协程作用域的情况（例如通过 GlobalScope 启动的协程体内），直接用协程启动器启动
- 对于明确要求子协程之间相互独立不干扰时，使用 supervisorScope
- 对于通过标准库 API 创建的协程，这样的协程比较底层，没有 Job、作用域等概念的支撑，例如我们前面提到过 suspend main 就是这种情况，对于这种情况优先考虑通过 coroutineScope 创建作用域；更进一步，大家尽量不要直接使用标准库 API，除非你对 Kotlin 的协程机制非常熟悉。



## suspend（挂起函数）

![kotlin-suspend](https://s2.loli.net/2022/07/19/Qy38cCaVYgb1dhU.gif)

`suspend` 是修饰函数的关键字，意思是当前的函数是可以挂起的，线程在执行到有 suspend 关键字的函数时，会被挂起，而所谓的挂起就是切换线程，挂起函数再执行完成之后，协程会重新切回原先的线程。

但是 suspend 它仅仅起着提醒的作用，比如，当我们的函数中没有需要挂起的操作的时候，编译器回给我们提醒 `Redudant suspend modifier`，意思是当前的 suspend 是没有必要的，可以把它删除。

**什么时候需要使用挂起函数呢？**

如果某个函数比较耗时，就要把它写成 `suspend` 函数；耗时操作一般分为两类：I/O 操作和 CPU 计算工作，比如：

- 文件读写
- 网络交互
- 图片处理



### Coroutine 的状态转移

- 协程的挂起函数本质上就是一个回调，回调类型就是 Continuation
- 协程体的执行就是一个状态机，每一次遇到挂起函数，都是一次状态转移



## Coroutine 的并发和同步

父协程的协程调度器是处于`Dispatchers.Main`情况下，它会将协程调度到主线程中执行，那么子协程将是同步执行。



## Coroutine Dispatcher

协程调度器确定了相关的协程在哪个线程或哪些线程上执行，协程调度器可以将协程限制在一个特定的线程执行，或将它分派到一个线程池，亦或让它不受限制地执行。

协程调度器是基于协程拦截器实现的，换句话说调度器就是拦截器的一种。

- **Dispatchers.Default**

  默认调度器，CPU密集型任务调度器，适合处理后台计算。通常处理一些单纯的计算任务，或者执行时间较短任务。比如：Json解析、数据计算等。

- **Dispatchers.IO**
  IO 调度器，IO 密集型任务调度器，适合执行 IO 相关操作。比如：网络处理、数据库操作、文件操作等。

- **Dispatchers.Main**
  UI 调度器，即在主线程上执行，通常用于 UI 交互，刷新等。如：MainScope`、`LifecycleScope`、`ViewModelScope`，都使用的 `Dispatcher.Main。

- **Dispatchers.Unconfined**
  非受限调度器，不要求协程执行在特定协程上，指定的线程可能会随着**挂起**的函数的发生变化。



## Coroutine Context

协程上下文包含了一些 Element 元素集，每个 `Element` 都是唯一的 `Key`，这些元素集定义了协程的行为，它们包含：

- **Job**：控制协程的生命周期
- **CoroutineDispatcher**：协程调度器可以将协程限制在一个特定的线程执行，或将它分派到一个线程池，亦或是让它不受限地运行
- **CoroutineName**：协程的名称，可用于调试
- **CoroutineExceptionHandler**：处理未捕获的异常
- **ContinuationInterceptor**：拦截

CoroutineContext 的操作符：

- **plus**

```kotlin
public operator fun plus(context: CoroutineContext): CoroutineContext = ...
```

通过重载运算符 plus 返回一个由原始的 Element 集合和通过 + 号 引入的 Element 产生新的 Element 集合。

- **get**

```kotlin
public operator fun <E : Element> get(key: Key<E>): E?
```

通过 Key 获取一个 Element

- **fold**

```kotlin
public fun <R> fold(initial: R, operation: (R, Element) -> R): R
```

用来遍历当前协程上下文中的 Element 集合

- **minusKey**

```kotlin
public fun minusKey(key: Key<*>): CoroutineContext
```

与 `plus` 的作用相反，相当于做减法，用来取出 `Key` 以外的当前协程上下文其他 `Element` ，返回不包含 `Key` 的写协程上下文。



## Continuation Interceptor



`suspendCoroutine`：它运行在协程当中并且帮我们渠道当前协程的 Continuation 实例，也就是拿到回调，方便后面调用它的 `resume`/`resumeWithException`来返回结果或者抛出异常。



## Courtine 启动模式

CoroutineStart 启动模式，是启动协程时需要传入的第二个参数。启动方式有四种：

#### DEFAULT

默认启动模式，协程创建后立即开始调度，但不是立即执行，有可能在执行前被取消。

#### LAZY

懒汉启动模式，启动后并不会有任何调度行为，知道执行它时才会产生调度。也就是说只有我们主动调用 Job 的 `start`、`join` 或 `await` 等函数时才开始调度。

#### ATOMIC

通过 `ATOMIC` 模式启动的协程执行到第一个挂起点之前是不响应 `cancel` 操作的，`ATOMIC` 一定要涉及到挂起点之后的执行，将取决于挂起点本身的逻辑和协程上下文中的调度器。

#### UNDISPATCHD

当以 UNDISPATCHD 模式启动时：

- 无论我们是否指定协程调度器，挂起前的执行都是在当前线程下执行。
- 如果所在的协程没有指定调度器，那么就会在 join 处恢复执行的线程里执行。
- 当指定了协程调度器时，遇到挂起点之后的执行将取决于挂起点本身的逻辑和协程上下文中的调度器。



## Coroutine 的异常处理

默认情况下，当协程因出现异常失败时，它会将异常传播到父级，父级会取消其余的子协程，同时取消自身的执行。最后将异常传播给它的父级。当异常到达当前层次结构的根，在当前协程作用域启动的所有协程都将被取消。

![kotlin-exception-dispatche](https://s2.loli.net/2022/07/19/HoCyWYdtr5xewQN.gif)



### 异常的传播

异常传播还涉及到协程作用域的概念。

- 通过 GlobeScope 启动的协程单独启动一个协程作用域，内部的子协程遵从默认的作用域规则。
- coroutineScope 是继承外部 Job 的上下文创建爱你作用域，在其内部的取消操作是双向传播的，子协程未捕获的异常也会向上传递给父协程。
  它跟适合一些列对等的谢恒并发的完成一项工作，任何一个协程异常退出，那么整体都将退出。这也是协程内部再启动子协程的默认作用域。
- supervisorScope 同样继承外部作用域的上下文，但其内部的取消操作是单向传播的，子协程出了异常并不会影响父协程以及其它兄弟协程。它更适合一些独立不相干的任务，任何一个任务出问题，并不会影响其它任务的工作。
  需要注意的是，supervisorScope 内部启动的子协程内部再启动子协程，如无明确指出，则遵守默认作用域规则，也即 supervisorScope 只作用于其直接子协程。

当构建器用于创建一个跟协程时（即该协程不是另一个协程的子协程），不同形式的协程构建器对异常传播有所不同：

- 自动传播异常（`launch` 与 `actor`）：这类构建器将异常视为未捕获异常，类似 Java 的 `Thread.uncaughtExceptionHandler`
- 向用户暴露异常（`async` 与 `produce`）：这类构建器始终会捕获所有异常并将其表示在结果 Deferred 对象中（因此它的 `CoroutineExceptionHandler` 也无效），而将依赖用户最终消费异常，例如通过 `await` 或 `receive`



### Couroutine Exception Handler

将**未捕获**异常打印到控制台的默认行为是可自定义的。*根*协程中的 CoroutineExceptionHandler 上下文元素可以被用于这个根协程通用的 `catch` 块，及其所有可能自定义了异常处理的子协程。 它类似于 `Thread.uncaughtExceptionHandler` 。你无法从 `CoroutineExceptionHandler` 的异常中恢复。当调用处理者的时候，协程已经完成并带有相应的异常。通常，该处理者用于记录异常，显示某种错误消息，终止和（或）重新启动应用程序。

通过 `async` 启动的协程出现未捕获的异常时会忽略 CoroutineExceptionHandler，这与 `launch` 的设计思路是不同的。



### 异常处理小结

1. **协程内部异常处理流程**：launch 会在内部出现未捕获的异常时尝试触发对父协程的取消，能否取消要看作用域的定义，如果取消成功，那么异常传递给父协程，否则传递给启动时上下文中配置的 CoroutineExceptionHandler 中，如果没有配置，会查找全局（JVM上）的 CoroutineExceptionHandler 进行处理，如果仍然没有，那么就将异常交给当前线程的 UncaughtExceptionHandler 处理；而 async 则在未捕获的异常出现时同样会尝试取消父协程，但不管是否能够取消成功都不会后其他后续的异常处理，直到用户主动调用 await 时将异常抛出。
2. **异常在作用域内的传播**：当协程出现异常时，会根据当前作用域触发异常传递，GlobalScope 会创建一个独立的作用域，所谓“自成一派”，而 在 coroutineScope 当中协程异常会触发父协程的取消，进而将整个协程作用域取消掉，如果对 coroutineScope 整体进行捕获，也可以捕获到该异常，所谓“一损俱损”；如果是 supervisorScope，那么子协程的异常不会向上传递，所谓“自作自受”。
3. **join 和 await 的不同**：join 只关心协程是否执行完，await 则关心运行的结果，因此 join 在协程出现异常时也不会抛出该异常，而 await 则会；考虑到作用域的问题，如果协程抛异常，可能会导致父协程的取消，因此调用 join 时尽管不会对协程本身的异常进行抛出，但如果 join 调用所在的协程被取消，那么它会抛出取消异常，这一点需要留意。



## Coroutine 的取消

### suspendCancellableCoroutin



## Sequence（序列）

除了集合之外，Kotlin 标准库还包含另一种容器类型——*序列*（Sequence）。 序列提供与 Iterable 相同的函数，但实现另一种方法来进行多步骤集合处理。

当 `Iterable` 的处理包含多个步骤时，它们会优先执行：每个处理步骤完成并返回其结果——中间集合。 在此集合上执行以下步骤。反过来，序列的多步处理在可能的情况下会延迟执行：仅当请求整个处理链的结果时才进行实际计算。

操作执行的顺序也不同：`Sequence` 对每个元素逐个执行所有处理步骤。 反过来，`Iterable` 完成整个集合的每个步骤，然后进行下一步。

因此，这些序列可避免生成中间步骤的结果，从而提高了整个集合处理链的性能。 但是，序列的延迟性质增加了一些开销，这些开销在处理较小的集合或进行更简单的计算时可能很重要。 因此，应该同时考虑使用 `Sequence` 与 `Iterable`，并确定在哪种情况更适合。



#### 序列的操作

关于序列操作，根据其状态要求可以分为一下几类：

- **无状态操作**不需要状态，并且可以独立处理每个元素，例如 `map()` 或 `filter()`。无状态操作还可能需要少量常数个状态处理元素，例如 `take()` 与`drop()`
- **有状态操作**需要大量状态，通常与序列中元素的数量成比例。





## Coroutine 的生命周期

| **State**                        | [isActive] | [isCompleted] | [isCancelled] |
| -------------------------------- | ---------- | ------------- | ------------- |
| *New* (optional initial state)   | `false`    | `false`       | `false`       |
| *Active* (default initial state) | `true`     | `false`       | `false`       |
| *Completing* (transient state)   | `true`     | `false`       | `false`       |
| *Cancelling* (transient state)   | `false`    | `false`       | `true`        |
| *Cancelled* (final state)        | `false`    | `true`        | `true`        |
| *Completed* (final state)        | `false`    | `true`        | `false`       |





**参考文献：**

[Courtines basics](https://kotlinlang.org/docs/coroutines-basics.html)

[Using Kotlin Coroutine builders in Android](https://flexiple.com/android/using-kotlin-coroutine-builders-in-android/)



