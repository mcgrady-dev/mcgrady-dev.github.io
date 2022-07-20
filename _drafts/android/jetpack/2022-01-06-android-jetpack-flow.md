

## 概述

- Flow 是 Kotlin语言提供的功能，属于 Kotlin协程的一部分，以响应式编程模型为模型，被设计用来处理异步数据流。

- 挂起函数可以异步的返回单个值，但是该如何异步返回多个计算好的值呢？这正是 Flow 的用武之地。

- 数据流以协程为基础构建，可通过异步方式进行计算处理一组数据序列。所发出值的类型必须相同，例如：`Flow<Int>`

- StateFlow 和 SharedFlow ，允许数据流以最优方式发出状态更新并向多个使用方发出值。



## FlowBuilder（流构建器）

- **flow {...}**

  ```kotlin
  fun <T> flow(@BuilderInference block: suspend FlowCollector<T>.() -> Unit): Flow<T> = SafeFlow(block)
  ```

  构建一个SateFlow

- **flowOf()**

  ```kotlin
  fun <T> flowOf(vararg elements: T): Flow<T> = flow {
      for (element in elements) {
          emit(element)
      }
  }
  ```

  `flowOf()`定义了一个发射固定值集的流。

- **asFlow()**

  ```kotlin
  fun <T> (() -> T).asFlow(): Flow<T> = flow {
      emit(invoke())
  }
  ```

  使用`.asFlow()`扩展函数，可以将各种集合和序列转换为流。



## StateFlow

**StateFlow 是一个状态容器式可观察数据流，可以向其收集器发出当前状态更新和新状态更新**。

在Android中，StateFlow 非常适合需要让可变状态保持持续观察的类。



### StateFlow 与 LiveData 的差异

StateFlow 和 LiveData 具有相似之处，或者说它们定位类似。两者都是**可观察的数据容器类**，并且在应用架构中使用时，两者都遵循相似模式。但在行为上却有所差异。

#### StateFlow 与 LiveData 的相同点

- StateFlow 提供了「可读可写」和「仅可读」两个版本
  StateFlow 实现了 ShreadFlow，MutableStateFlow 实现了 MutableSharedFlow
- 值是唯一的
- 它允许被多个观察者共用（因此是共享的数据流）
- 它永远只会把最新的值重现给订阅者，这与活跃观察者的数量无关
- 支持 DataBinding

#### StateFlow 与 LiveData 的不同点

- StateFlow 需要将初始状态传递给构造函数，而 LiveData 不需要。
- 当 View 进入 `STOPPED` 状态时，`LiveData.observe()` 会自动取消注册使用方，而从 StateFlow 或任何其他数据流收集数据的操作并不会自动停止。如需实现相同的行为，需要从 `Lifecycle.repeatOnLifecycle` 块收集数据流。
- value 空安全
  MutableStateFlow 构造方法强制赋值一个非空的数据，而且 value 也是非空的。这意味着 StateFlow 永远有值
- 防抖
  StateFlow 默认是防抖的，在更新数据时，会判断当前与新值是否相同，如果相同则不更新数据。



## SharedFlow



### SharedFlow 与 StateFlow 的相同点

- 与 StateFlow 一样，SharedFlow 也提供了「可读可写」和「仅可读」两个版本

### SharedFlow 与 StateFlow 的不同点

- MutableSharedFlow 没出初始值
  意味着 MutableSharedFlow 没有默认值。

- SharedFlow 可以保留历史数据
  StateFlow 只保留最新值，即新的订阅者只会获得最新的和之后的数据，而 SharedFlow 根据配置可以保留历史数据，新的订阅者可以获取之前发射过的一系列数据。
- MutableSharedFlow 发射值需要调用 emit() / tryEmit() 方法，没有 setValue() 方法



shareIn 使冷数据流变为热数据流

`shareIn` 函数会返回一个热数据流 SharedFlow，此数据流会向从其中收集值的苏由于使用方发出数据。**SharedFlow 是 StateFlow 的可配置性极高的泛化数据流**。



## 协程设计来源

### 1. 协程设计模式

#### scope

`CoroutineScope`协程作用域：每个协程体都存在一个作用域，异步还是同步由该作用域决定

#### channel

`Channel`通道：数据如同一个通道进行发送和接收，可以在协程之间互相传递数据或者控制阻塞和继续

#### select

### 2. await模式（JavaScript异步任务解决方案）

### 3. 参照响应式框架（如：RxJava）创造 Flow

Flow 的主要目标是拥有尽可能简单的设计， 对 Kotlin 以及挂起友好且遵从结构化并发。没有响应式的先驱及他们大量的工作，就不可能实现这一目标。

### 4. 不同场景使用不同调度器，不需要考虑线程问题



 

## Sequences

Flow 跟 Sequences 之间的区别是 Flow 不会阻塞主线程的运行，而 Sequences 会阻塞主线程的运行。



## `Flow` VS `RxJava`

|    RxJava2    |     Coroutine      |
| :-----------: | :----------------: |
|   Single<T>   |    Deferred<T>     |
|   Maybe<T>    |   Deferred`<T?>`   |
|  Completable  |        Job         |
| Observable<T> | Channel<T>/Flow<T> |
| Flowanble<T>  | Channel<T>/Flow<T> |



## Cold Stream

flow 的代码块只有调用 collected() 才开始运行，正如 RxJava 创建的 Observables 只有调用 subscribe() 才开始运行一样。

## Hot Stream (StateFlow/SharedFlow)





## Flow 操作符



### 过渡流操作符

过渡流操作符应用于上游流，并返回与下游流。这些操作符也是冷操作符。

- **map**

  ```kotlin
  inline fun <T, R> Flow<T>.map(crossinline transform: suspend (T) -> R): Flow<R>
  ```

  转换流操作符，将 A 变成 B

- **filter**

  ```kotlin
  inline fun <T> Flow<T>.filter(crossinline predicate: suspend (T) -> Boolean): Flow<T>
  ```

  过滤操作符

- **onEach**

  ```kotlin
  fun <T> Flow<T>.onEach(action: suspend (T) -> Unit): Flow<T>
  ```

  用于每次元素发射后执行相关的操作，使该流更具说明性以及简洁。

### 转换操作符

- **transform**

  ```kotlin
  inline fun <T, R> Flow<T>.transform(crossinline transform: suspend FlowCollector<R>.(T) -> Unit): Flow<R>
  ```

  可以将流发射任意值任意次

### 限长操作符

限长过渡操作符在流触及相应限制时会将它的执行取消（协程中的取消操作总是通过抛出异常来执行的）。

- **take**

  ```kotlin
  fun <T> Flow<T>.take(count: Int): Flow<T>
  ```

  返回 N 个计数元素的流，当 N 个元素被消耗时，流将被取消。

### 末端流操作符

末端操作符是用于启动流收集的挂起函数。

- **collect**

  ```kotlin
  suspend fun Flow<*>.collect()
  ```

- **toList**

  ```kotlin
  suspend fun <T> Flow<T>.toList(destination: MutableList<T> = ArrayList()): List<T>
  ```

  转化为List集合

- **first**

  ```kotlin
  suspend fun <T> Flow<T>.first(): T
  suspend fun <T> Flow<T>.first(predicate: suspend (T) -> Boolean): T
  ```

  获取第一个值

- **single**

  ```kotlin
  suspend fun <T> Flow<T>.single(): T
  ```

  确保流发射单个值的操作符

- **reduce**

  ```kotlin
  suspend fun <S, T : S> Flow<T>.reduce(operation: suspend (S, T) -> S): S
  ```

  将流规约到单个值

- **fold**

  ```kotlin
  inline suspend fun <T, R> Flow<T>.fold(initial: R, crossinline operation: suspend (R, T) -> R): R
  ```

  从初始值开始将流规约到单个值

### 特殊操作符

- **buffer**
  缓冲操作符，并发运行流中发射元素的代码以及收集的代码，而不是顺序运行它们：

  ```kotlin
  fun <T> Flow<T>.buffer(capacity: Int = BUFFERED, onBufferOverflow: BufferOverflow = BufferOverflow.SUSPEND): Flow<T>
  ```

- **conflate**

  ```kotlin
  fun <T> Flow<T>.conflate(): Flow<T> = buffer(CONFLATED)
  ```

  用于跳过中间值。

- **collectLatest**

  ```kotlin
  suspend fun <T> Flow<T>.collectLatest(action: suspend (value: T) -> Unit)
  ```

  在新值产生的时候取消执行其块中的代码，用于发射租后一个值

- **flowOn**

  ```kotlin
  fun <T> Flow<T>.flowOn(context: CoroutineContext): Flow<T>
  ```

  创建了另一个协程，用于改变流发射的上下文。

### 组合操作符

- **zip**

  ```kotlin
  fun <T1, T2, R> Flow<T1>.zip(other: Flow<T2>, transform: suspend (T1, T2) -> R): Flow<R>
  ```

  用于组合两个流中的相关值

- **combine**

  ```kotlin
  fun <T1, T2, R> Flow<T1>.combine(flow: Flow<T2>, transform: suspend (a: T1, b: T2) -> R): Flow<R>
  ```

  依赖于相应流的最新值，并且每当上游流产生值的时候都需要重新计算。

### 展平流操作符

```kotlin
val startTime = System.currentTimeMillis() // 记录开始时间
(1..3).asFlow().onEach { delay(100) } // 每 100 毫秒发射一个数字 
    .展平流 { requestFlow(it) }                                                                           
    .collect { value -> // 收集并打印
        println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
    } 
```

上面得到了一个包含流的流（`Flow<Flow<String>>`）,需要将其进行展平为单个流以进行下一步处理。

- **flatMapConcat**

  ```kotlin
  fun <T, R> Flow<T>.flatMapConcat(transform: suspend (value: T) -> Flow<R>): Flow<R>
  ```

  在等待内部流完成之前开始收集下一个值

  ```
  1: First at 121 ms from start
  1: Second at 622 ms from start
  2: First at 727 ms from start
  2: Second at 1227 ms from start
  3: First at 1328 ms from start
  3: Second at 1829 ms from start
  ```

- **flatMapMerge**

  ```kotlin
  fun <T, R> Flow<T>.flatMapMerge(
      concurrency: Int = DEFAULT_CONCURRENCY,
      transform: suspend (value: T) -> Flow<R>
  ): Flow<R>
  ```

  并发收集所有传入的流，并将它们的值合并到一个单独的流

  ```
  1: First at 136 ms from start
  2: First at 231 ms from start
  3: First at 333 ms from start
  1: Second at 639 ms from start
  2: Second at 732 ms from start
  3: Second at 833 ms from start
  ```

- **flatMapLatest**

  ```kotlin
  public inline fun <T, R> Flow<T>.flatMapLatest(@BuilderInference crossinline transform: suspend (value: T) -> Flow<R>): Flow<R>
  ```

  在发出新流后立即取消先前流的收集

  ```
  1: First at 142 ms from start
  2: First at 322 ms from start
  3: First at 425 ms from start
  3: Second at 931 ms from start
  ```



## Channel

Channel 提供了一种在流中传输值的方法。



## Pipelines﻿（管道）

管道是一种一个协程在流汇总开始生产可能无穷多个元素的模式



## Fan-out（扇出）



## Fan-in（扇入）
