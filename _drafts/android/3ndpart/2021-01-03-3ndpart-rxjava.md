---
title: RxJava笔记
tags:
  - rxjava
categories: rxjava
date: 2018-01-23 21:30:50
updated: 2018-03-27 09:30:50
---



## 摘要

1. **观察者模式**

![onClick](./images/rxjava/onclick.jpg)
![Subscribe](./images/rxjava/subscribe.jpg)

| 被观察者       | 观察者             | 订阅                   | 事件        |
| ---------- | --------------- | -------------------- | --------- |
| Button     | OnClickListener | setOnClickListener() | onClick() |
| Observable | Observer        | subscribe()          | onEvent() |



### Observables

#### Single

#### Completable

从 CompletableEmitter 可以看出，Completable 在创建后，不会发射任何数据。

Completable 只有 `onComplete` `onError` 事件，同时 Completable 没有 map flatMap 等操作符，它的操作符比起 Observable Flowable 要少得多




















## Flowable
**情景分析：**在RxJava中会经常遇到一种情况就是被观察者发送消息十分迅速以至于观察者不能及时的响应这些消息。
```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                while (true){
                    e.onNext(1);
                }
            }
        })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Thread.sleep(2000);
                System.out.println(integer);
            }
        });
```
上述例子中可以看出生产者无限生成事件，而消费者每2秒才能消费一个事件，这会造成事件无限堆积，最后造成OOM。

Flowable就是由此产生，专门处理这些事件堆积问题。


### Backpressure
关于上述的问题，有个专有的名词来形容上述现象，**Backpressure(背压)**，即**生产者的速度大于消费者的速度带来的问题**，比如在Android中常见的点击事件，点击过快则会造成两次事件触发。

因此得知，**Flowable是为了应对Backpressure而产生的，Flowable是一个被观察者，与Subscriber（被观察者）配合使用，解决Backpressure问题**。



### 处理Backpressure的策略

#### ERROR
这种方式会在产生Backpressure问题的时候直接抛出一个**MissingBackpressureException**异常。

#### BUFFER
把默认的只能存**128**个事件的缓存池换成一个大的缓存池，这样，消费者通过**request()**即使传入一个很大的数字，生产者也会生产事件，并将处理不了的事件缓存。
但是这种方式任然比较消耗内存，除非是我们比较了解消费者的消费能力，能够把握具体情况，不会产生OOM，总之BUFFER要慎用。

#### DROP
消费者通过**request()**传入其需求n，然后生产者把n个事件传递给消费者供其消费。其他消费不掉的事件就丢掉。

#### LATEST
与DROP功能基本一致。
唯一的区别就是LATEST总能使消费者能够接收到生产者产生的最后一个事件。

#### 其他策略
如果Flowable对象不是自己创建的，可以采用**onBackpressureBuffer()**、**onBackpressureDrop()**、**onBackpressureLatest()**的方式指定。



## Scheduler简介

在不指定线程的情况下， RxJava 遵循的是线程不变的原则，即：在哪个线程调用 subscribe()，就在哪个线程生产事件；在哪个线程生产事件，就在哪个线程消费事件。如果需要切换线程，就需要用到 Scheduler （调度器）。
在RxJava 中，Scheduler，相当于线程控制器，RxJava 通过它来指定每一段代码应该运行在什么样的线程。

### Scheduler 的 API
RxJava已经内置了几个 Scheduler ，它们已经适合大多数的使用场景：

1. **Schedulers.immediate()：**直接在当前线程运行，相当于不指定线程。
2. **Schedulers.newThread()：**启用新线程执行操作。
3. **Schedulers.io()：**用于I/O操作（读写文件、读写数据库、网络信息交互等）。行为模式和`newThread()`差不多，区别在于`io()`的内部实现使用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下`io()`比`newThrad()`更效率。（不建议用于计算工作，避免创建不必要的线程）
4. **Schedulers.computation()：**用于计算（CPU密集型计算），既不会被I/O等操作限制性能的操作，例如图形计算。`computation()`内部使用了固定的线程池，大小为CPU核数。（不建议用于I/O操作，避免等待时间会浪费CPU）
5. Android 还有一个专用的 **AndroidSchedulers.mainThread()**，它指定的操作将在 Android 主线程运行。

## Disposable简介
Disposable的作用是，将Observer切断，不再接来自被观察者的事件，而被观察者的事件却仍在继续运行。

#### Disposable对象的获取
Disposable的对象通过观察者获得，具体分为两种方式：

1. **Observer接口**
```java
Observer<String> observer = new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onNext(String s) {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        };
```

2. **Consumer等其他函数式接口**
```java
Disposable disposable = Observable.just("你好").subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {

            }
        });
```

获取到Disposable对象后，可以调用`dispose()`方法断开连接，停止接收Observable发送的事件。



## 操作符

### 过滤操作
**filter()**
观测序列中只有通过的数据才会被发射
```java
Observable.just(4, 2, 6, 1, 7, 5)
                .filter(new Predicate<Integer>() {
                    @Override
                    public boolean test(@NonNull Integer integer) throws Exception {
                      // 过滤小于3的值  
                      return integer > 3;
                    }
                }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                Logger.d("value is " + integer);
            }
        });
```


**take()** 
只发射前N个元素

**takeLast()** 
只发射后N个元素

**first()** 
只发射第一个元素

**last()** 
只发射最后一个元素

**skip()** 
不发射前N个元素

**skipLast()**
不发射后N个元素

**distinct()** 
去除重复数据（仅一次）

**elementAt()** 
从序列中发射第N个元素然后完成操作

**sample()** 
定期发射`Observable`最近发射的数据项

**distinct()**
去重操作

**Single.just()**
```java
Single.just(new Random().nextInt())
                .subscribe(new SingleObserver<Integer>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                    }

                    @Override
                    public void onSuccess(@NonNull Integer integer) {
                        LogUtil.d("hello " + integer);
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {

                    }
                });
```
Single 只会接收一个参数，而SingleObserver 只会调用onError 或者onSuccess

**debounce()**
去除发送频率过快的项

### 合并操作
**merge()**
合并多Observable，接受可变参数、接受迭代器集合。和concat的区别在于，不用等发射器A发送完所有事件再进行发射器B的发送。

**zip()**

```java
Observable.zip(Observable.just(1, 2, 3, 4), Observable.just(5, 6, 7, 8),
                new BiFunction<Integer, Integer, String>() {
                    @Override
                    public String apply(@NonNull Integer integer1, @NonNull Integer integer2) throws Exception {
                      LogUtil.d(integer1 + "+" + integer2);
                        return String.valueOf(integer1 + integer2);
                    }
                }).subscribe(new Consumer<String>() {
            @Override
            public void accept(@NonNull String s) throws Exception {
                LogUtil.d(" = " + s);
            }
        });
```

zip 组合事件的过程是分别从发射器A和B中各取一个事件来组合，并且一个事件只能被使用一次，组合的顺序是严格按照事件发送的顺序来进行。



**concat()**



![rxjava_concat](./images/rxjava/rxjava_concat.png)

对于单一的两个发射器有序的组合成一个发射器，`concat`只接受相同泛型参数

- *示例一*
  下面是用`concat`操作符来实现多个数据源的例子，比如一个商品详情需要展示商品的信息、仓库信息、推荐信息，可以需要访问三个接口：

```java
// 商品信息
Observable<Object> goodInfo = Observable.create(new ObservableOnSubscribe<Object>() {
            @Override
            public void subscribe(ObservableEmitter<Object> e) throws Exception {
                GoodInfo goodInfoApi = goodsApi.getGoodInfo();
                if (goodInfoApi != null) {
                    e.onNext(goodInfoApi);
                }
                
                e.onComplete();
            }
        }).subscribeOn(Schedulers.io());

// 仓库信息
Observable<Object> stockInfo = Observable.create(new ObservableOnSubscribe<Object>() {
            @Override
            public void subscribe(ObservableEmitter<Object> e) throws Exception {
                Seller sotckInfoApi = goodApi.getStock();
                if (sotckInfoApi != null) {
                    e.onNext(sellerApi);
                }
                
                e.onComplete();
            }
        }).subscribeOn(Schedulers.io());

// 推荐信息
Observable<Object> relateGoods = Observable.create(new ObservableOnSubscribe<Object>() {
            @Override
            public void subscribe(ObservableEmitter<Object> e) throws Exception {
                Relate relateInfoApi = goodApi.getRelate();
                if (relateInfoApi != null) {
                    e.onNext(sellerApi);
                }
                
                e.onComplete();
            }
        }).subscribeOn(Schedulers.io());

// 组合
Observable.concat(goodInfo, stockInfo, realteGoods)
  				.observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Object>() {
                    @Override
                    public void accept(@NonNull Object objs) throws Exception {
                        LogUtil.d("" + objs);
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(Throwable throwable) throws Exception {
                      throwable.printStackTrace();
                      LogUtil.e(throwable.getMessage());
                    }
                });
```



- *示例二*
  使用`concat`操作符对缓存进行检查，如：内存缓存、本能地缓存、网络，那一层有数据立即返回（可以配合`first()`操作符来实现这个的效果）：
  ​
```java
final String[] data = {"memory", "disk", "network"};


// 内存缓存
        Observable<String> memoryCache = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> e) throws Exception {
                String cache = data[1];
                if (cache != null) {
                    e.onNext(cache);
                }

                e.onComplete();
            }
        }).subscribeOn(Schedulers.io());

// 本地缓存
        Observable<String> diskCache = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> e) throws Exception {
                String cache = data[2];
                if (cache != null) {
                    e.onNext(cache);
                }

                e.onComplete();
            }
        }).subscribeOn(Schedulers.io());

// 网络缓存
        Observable<String> networkCache = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> e) throws Exception {
                String cache = data[3];
                if (cache != null) {
                    e.onNext(cache);
                }

                e.onComplete();
            }
        }).subscribeOn(Schedulers.io());

// 组合
Observable.concat(memoryCache, diskCache, networkCache)
  				.takeFirst()
  				.first()
  				.observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(@NonNull String cache) throws Exception {
                        LogUtil.d("" + cache);
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(Throwable throwable) throws Exception {
                      throwable.printStackTrace();
                      LogUtil.e(throwable.getMessage());
                    }
                });
```

**map()**
用于转换一组数据


**flatMap()**

**concatMap()**

**join()**

### 阻塞操作
**toList()**

### 其他操作

**doOnNext()**
在每次输出一个元素之前做一些额外的事情

**subscribeOn()**
指定Observable(被观察者)所在的线程，或者叫做**事件产生的线程**。 

**observeOn()**
指定 Observer(观察者)所运行在的线程，或者叫做**事件消费的线程**。


##### 同步 异步

1. 当观察者和被观察者工作在**同一线程**时，是一个**同步**的订阅关系，被观察者每产生一个事件必须等观察者消费完才能接着生产下一个事件。
2. 当观察者和被观察者工作在**不同线程**时，是一个**异步**的订阅关系，被观察者产生事件不需要等待观察者消费，**因为两个线程斌不能直接进行通信**。



**发射规则**：

- Observable可发送无限个`onNext`，subscribe可以接受无限个`onNext`
- Observable发送`onComplete`或`onError`之后的事件将会继续发送，
- Observer收到`onComplete`或`onError`事件后将不再继续接收事件
- Observable可以不发送`onComplete`、`onError`
  `onComplete`和`onError`必须唯一并且互斥, 即不能发多个`onComplete`, 也不能发多个`onError`, 也不能先发一个`onComplete`, 然后再发一个`onError`, 反之亦然
  ​



## 版本差异

### RxJava 1.x

### RxJava 2.x

#### `Single` `Completable` `Maybe`

在 RxJava 2.x 版本中，除了 Observable 和 Flowable 之外，还有三种类型的Obsers

| 类型          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| Observable<T> | 能够发射0或n个数据，兵役成功或失败时间终止                   |
| Flowable<T>   | 能够发射0或n个数据，并以成功或错误事件终止；<br />支持 Backpressure（被压），可以控制数据源发射的速度。 |
| Single<T>     | 只发射单个数据或错误事件                                     |
| Maybe<T>      | 能发射0或1个数据，要么成功，要么失败，有点类似于 Optional    |



### RxJava 3.x



### 版本差异

#### 1.x 与 2.x 的区别

##### 



1. **Nulls**
   1.x是允许在发射事件传入null值的，从2.x开始不支持了。
2. **Flowable**
   2.x中Observable不再支持背压，将用一个全新的Flowable处理背压问题。
3. **Single/Completable/Maybe**
   ​
4. **线程调度相关**
   2.x中新增了Schedule.immediate()、Schedule.test()两个线程环境。
5. **Function相关**
   2.x替换了Func1为Function，BiFunction替换了Func2...FuncN，并且它们都增加了throwsException，意味着不用添加try-catch操作了。
6. **其他操作符相关**
   如Func1...FuncN的变化，Consumer和BiConsumer对Action1...Action2进行了替换，只保留了ActionN。

#### 2.x 与 3.x 的区别



