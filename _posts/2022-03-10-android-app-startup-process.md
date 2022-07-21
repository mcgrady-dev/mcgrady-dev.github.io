---
layout: 
title: Application starup process
date: 2022-03-10 14:36 +0800
tags: android
---

Android应用启动过程

<!--more-->

## Activity启动过程

1. 首先通过Binder调度AMS#startActivity()
2. AMS判断目标进程是否启动，否则通过Socket发送创建新进程请求，Zygote#fork() 新的进程
3. ActivityThread#main()方法开始执行
4. ActivityThread 实例初始化
5. 主线程Looper初始化
6. ActivityThread#attach() 方法通过Binder调度AMS发送创建Application消息
7. 主线程Looper进入循环
8. AMS判断Application是否已经创建，否则ApplicationThread通过Handler发送创建Application消息
9. ActivityThread#H 接收创建Application的消息，开始执行Application初始化（通过反射）和生命周期回调
10. ApplicationThread#scheduleLuaunchActivity() 通过Handler发送创建Activity 消息
11. ActivityThread#H 接收创建Activity的消息，开始执行Activity初始化（通过反射）和生命周期回调

![android-activity-startup-process](https://s2.loli.net/2022/07/19/FuJLyT8H2EY5jPp.jpg)

### ActivityThread

Application 启动入口类

#### main()

1. 初始化主线程Looper
2. 初始化ActivityThread
3. ActivityThread#attach()
   1. 通过Binder调度AMS#attachApplication()发送初始化Application消息
   2. ContextImpl#createAppContext()
   3. 初始化Instrumentation
   4. Instrumentation#callApplicationOnCreate()
4. Looper#loop()

#### H (extends Handler)

ActivityThread的内部类，主线程Handler，负责处理主线程的所有消息，尤其是Application、Activity生命周期相关的Message。

#### ApplicationThread (extends Binder)

ActivityThread的内部类，可以看做一个Binder，它的一个重要作用就是作为ActivityManagerService向当前进程发送消息（IPC）的桥梁。

### ActivityManager

作为与ActivityManagerService 通讯的工具（IPC），通过它实现当前进程向ActivityManagerService发送消息的功能。

### ActivityManagerService

ActivityManagerService 系统服务类，统一管理Android中的Application、Activity的创建、启动和生命周期。

### Instrumentation

Instrumentation会在应用程序的任何代码初运行之前被实例化，它能够允许你监控应用程序和系统的所有交互，且最终Application的创建、Activity的创建、以及生命周期都会经过这个对象去执行。

Instrumentation实例化的方法是反射，而反射的ClassName来自于ActivityManagerService传过来的IBinder。

### Binder

### Zygote

Zygote 是 Android系统创建新进程的核心进程，负责启动Dalvik虚拟机，加载一些必要的系统资源和系统类，启动system_service进程，随后进入等待处理App应用请求。

#### SystemService

SystemServer(进程名为system_server)是android服务的提供者，所有service运行在该进程中，通过Zygote#fork 一个新进程用于启动 com.android.server.SystemService。







