---
layout: 
title: Android Service
date: 2022-03-09 11:45 +0800
tags: android
---

Service 是一个没有用户界面的在后台长时间运行操作的应用组件。组件可通过绑定到服务与之交互，甚至是执行进程间通信。例如，服务可在后台处理网络事务、播放音乐、执行文件 I/O 或与 ContentProvider 进行交互。

Service 在其托管进程的主线程中运行，它即不创建自己的线程，也不在单独的进程中运行（除非另行指定）。

<!--more-->



## Service的类型

### 前台（Foreground Service）

前台服务执行一些用户能注意到的操作。例如，音频应用会使用前台服务来播放音频曲目。前台服务必须显示[通知](https://developer.android.com/guide/topics/ui/notifiers/notifications)。即使用户停止与应用的交互，前台服务仍会继续运行。

### 后台（Background Service）

后台服务执行用户不会直接注意到的操作。

### 绑定（Bound Service）

当应用组件通过调用 `bindService()` 绑定到服务时，服务即处于绑定状态。绑定服务会提供 C/S 接口，以便组件与服务进行交互、发送请求、接收结果，甚至是利用进程间通信 (IPC) 跨进程执行这些操作。仅当与另一个应用组件绑定时，绑定服务才会运行。多个组件可同时绑定到该服务，但全部取消绑定后，该服务即会被销毁。



## Service的生命周期

![android-service-lifecycle](https://s2.loli.net/2022/07/19/WcZYNstPu3SnlLM.png)



## Service的启动过程

当 App 通过调用 `startService()` 或 `bindService()` 来启动服务的过程，主要由 ActivityManagerService 来完成。

![android-start-service-process](https://s2.loli.net/2022/07/19/LeloFrOXV1MdpqA.jpg)



### startService

![android-start-service-lifeline](../../../Pictures/gallery/android/service/android-start-service-lifeline.jpeg)

### bindService

![android-bind-service](https://s2.loli.net/2022/07/19/IykZPRCB7JhmqvS.jpg)

图中蓝色代表的是 Client进程（发起端）, 红色代表的是 system_server 进程, 黄色代表的是 target 进程（service所在进程）



## Service的数据结构

### ServiceRecord

ServiceRecord 位于 system_server 进程，是 AMS 管理各个 app 中 service 的基本单位。ServiceRecord 继承于 Binder 对象，作为 Binder IPC 的 Bn 端，Binder 将其传递到 Service 进程的 Bp 端，保存在 Service.mToken，即 ServiceRecord 的代理对象。



## IntentService

IntentService 继承于 Service，**用来处理异步请求**。客户端可以通过 `startService(Intent)` 方法传递请求给 IntentService。IntentService 在 `onCreate()` 函数中通过 HandlerThread 单独开启一个线程来依次处理所有 Intent 请求对象所对应的任务。　
　　
这样以免事务处理阻塞主线程（ANR）。执行完一个 Intent 请求对象所对应的工作之后，如果没有新的 Intent 请求达到，则 **自动停止** Service；否则执行下一个 Intent 请求所对应的任务。　
　　
IntentService 在处理事务时，还是采用的 Handler 方式，创建一个名叫 ServiceHandler 的内部 Handler，并把它直接绑定到 HandlerThread 所对应的子线程。 ServiceHandler 把处理一个 Intent 所对应的事务都封装到 `onHandleIntent()` 虚函数；因此我们直接实现虚函数，根据 Intent 的不同进行不同的事务处理就可以了。
另外，IntentService 默认实现了 `OnBind()` 方法，返回值为 `null`。

使用 IntentService 需要实现的两个方法：

 - 构造函数　
   IntentService 的构造函数一定是**参数为空**的构造函数，然后再在其中调用 `super(name)` 这种形式的构造函数。因为Service的实例化是系统来完成的，而且系统是用参数为空的构造函数来实例化Service的

 - 实现虚函数 `onHandleIntent()`
   在里面根据 Intent 的不同进行不同的事务处理。　
   好处：处理异步请求的时候可以减少写代码的工作量，比较轻松地实现项目的需求。



### IntentService与Service的区别

Service 不是独立的进程，也不是独立的线程，它是依赖于应用程序的主线程的，不建议在Service 中编写耗时的逻辑和操作，否则会引起 ANR。

IntentService 它创建了一个独立的工作线程来处理所有的通过 `onStartCommand()` 传递给服务的 intents（把intent插入到工作队列中）。通过工作队列把 intent 逐个发送给`onHandleIntent()`。　

不需要主动调用 `stopSelft()` 来结束服务。因为，在所有的 Intent 被处理完后，系统会自动关闭服务。

