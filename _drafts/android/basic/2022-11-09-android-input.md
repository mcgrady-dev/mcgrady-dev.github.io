---
layout: article
title: Android Input System
date: 2022-03-09 15:22 +0800
tags: android

---



<!--more-->



硬件按键事件始终传递给目前处于焦点的View对象，从View层次结构的顶层开始分派，直到到达合适的目的地。



## InputManagerService

InputManagerService分为两部分，Java层负责与WMS通信，Native层由InputReader和InputDispatcher输入系统关键组件的运行容器。

Java层的InputManagerService主要工作是为ReaderPolicy与DispatcherPolicy提供实现，以及与Android其系统服务进行写作，其中主要的写作者是WMS。

NativeInputManager位于InputManagerService的JNI层，负责Native层的组件与Java层的InputManagerService互相通信。同时，它为InputReader和InputDispatcher提供策略请求的接口。策略请求被它转发给Java层的InputManagerService，由InputManagerService最终定夺。

InputManager是InputReader与InputDispatcher的运行容器它创建了两个线程分别承载InputReader和InputDispatcher的运行。

内核将原始事件写入设备节点，InputReader不断通过EventHub将原始事件取出并转换成Android输入事件，然后交给InputDispatcher，InputDispatcher根据WMS提供的窗口信息将事件交给合适的窗口，窗口的ViewRootImpl对象再沿着View树将事件分发给感兴趣的View，View收到时间做出响应，更新自己的画面、执行特定的动作。





### EventHub

### InputReader

### InputDispatcher

