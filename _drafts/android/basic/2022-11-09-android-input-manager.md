---
layout: article
title: Android Input System
date: 2022-03-09 15:22 +0800
tags: android

---



<!--more-->

## 基础概念

### inotify

inotify是Linux内核所提供的一种文件系统变化通知机制。它可以为应用程序监控文件系统的变化，如文件的新建、删除、读写等，分别为inotify对象与watch对象，都使用文件描述符表示。

相较于主动轮询文件系统，透过操作系统主动告知文件异动的方式，不必每个一段时间扫描整个文件系统，甚至可以在文件变更最后一秒内通知更新。这也使得inotifty成为`select()`和`poll()`得以实现的基础。

但inotify也有缺点，它无法监控软链接型的子目录。





![android-input-manager-process.drawio](https://s2.loli.net/2022/11/14/lZALCd97jzohSiJ.png)

硬件按键事件始终传递给目前处于焦点的View对象，从View层次结构的顶层开始分派，直到到达合适的目的地。



## InputManagerService

InputManagerService分为两部分，Java层负责与WMS通信，Native层由InputReader和InputDispatcher输入系统关键组件的运行容器。

![android-input-manager-service](/Users/mcgrady/Pictures/gallery/android/input/android-input-manager-service.png)

### IMS的Java层

Java层的InputManagerService主要工作是为ReaderPolicy与DispatcherPolicy提供实现，以及与Android其系统服务进行写作，其中主要的写作者是WMS。

### IMS的Native层

NativeInputManager位于InputManagerService的JNI层，负责Native层的组件与Java层的InputManagerService互相通信。同时，它为InputReader和InputDispatcher提供策略请求的接口。策略请求被它转发给Java层的InputManagerService，由InputManagerService最终定夺。



### InputManager

InputManager是InputReader与InputDispatcher的运行容器它创建了两个线程分别承载InputReader和InputDispatcher的运行。

内核将原始事件写入设备节点，InputReader不断通过EventHub将原始事件取出并转换成Android输入事件，然后交给InputDispatcher，InputDispatcher根据WMS提供的窗口信息将事件交给合适的窗口，窗口的ViewRootImpl对象再沿着View树将事件分发给感兴趣的View，View收到时间做出响应，更新自己的画面、执行特定的动作。



### InputReader

InputReader运行在InputReaderThread中，通过线程循环从EventHub中抽取未处理的事件列表，这些事件分为两类，通过`processEventLocked()`对事件分别进行处理：

- 从设备节点中读取的原始输入事件，简称**原始输入事件**：对根据设备的可用性加载或移除设备对应的配置信息；
- 输入设备可用性变化事件，简称为**设备事件**：进行转译、封装与加工后将结果暂存到`mQueuedListener`中；

通所有的事件处理完成后，调用`mQueueListener.flush()`将暂存的输入事件一次性地交给InputDispatcher处理。



### EventHub

EventHub即事件集线器，它将所有的输入事件通过一个接口`getEvent()`把多个输入设备节点中读取的事件交给InputReader，是输入系统中最底层的一个组件。

EventHub内部是通过epoll监控原始输入事件和inotifty监控设别节点增删事件的。



### InputDispatcher

InputDispatcher运行在独立的InputDispatcherThread线程中，通过线程循环`dispatchOnce()`函数来完成输入事件的派发动作。派发线程的一次循环中包含以下工作内容：

- 进行一次事件派发：当命令队列汇总没有命令时才会进行；
- 执行命令列表中的命令：通过`postCommandLocked()`将其追加到命令列表中；
- 陷入休眠状态：当派发对垒为空，派发线程陷入无限期休眠状态，由于派发线程循环的本质是epoll，因此派发线程在三种情况下可能被唤醒：
  1. 调用`Looper.wake()`主动唤醒；
  2. 达到`nextWakeupTime`的事件点时唤醒；
  3. `epoll_wait()`监听的fd有epoll_event发生时唤醒；

当事件进入InputDispatcher时，原始输入事件经过InputReader加工已经形成了以下三种类型：Key事件、Motion事件、Switch事件。InputDispatcher根据不同的事件类型选择事件派发策略（InputDispatcherPolicy）处理事件派发流程。

InputDispatcher选择好输入事件的目标窗口后，便准备将事件发送给它。由于InputDispatcher位于system_server进程中，而窗口运行与其所在的应用进程中，所以它们之间需要跨进程通信，核心在于InputChannel。

### InputChannel

InputChannel本质是一对SocketPair（非网络套接字），用于实现在本机进程间的通信。SocketPair将两个可以双通的文件描述符分别分配给InputDispatcher和Window，InputDispatcher向InputChannel中写入输入事件，可以由Window从自己的InputChannel中读取。

在InputDispatcher中，InputChannel被封装为一个Connection对象，描述了从InputDispatcher到目标窗口中的一个连接，随后通过`registerInputChannel()`将InputChannel的可读性事件注册到mLooper中（本质是epoll机制），当来自窗口的反馈时，派发线程的`mLooper.pollOnce()`被唤醒，并回调`handleReceiveCallbak()`进行处理。因此窗口反馈的到来也会导致派发线程进入下一次派发循环。

![android-input-window-connect](https://s2.loli.net/2022/11/14/NOK12drz8nwoDeF.png)



 
