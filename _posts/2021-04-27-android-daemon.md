---
layout: 
title: Android Deamon
date: 2021-04-27 22:29 +0800
tags: android
---



<!--more-->

## Android 进程管理策略

Android 是基于 Linux 实现的系统，继承了 Linux 的内存管理策略（lowmemorykiller），即进程退出并不会立即杀死，而是在系统内存吃紧的时候再按照优先级杀掉进程，不同于微软的Windows，Windows 在进程退出的同时一并杀掉进程，将内存释放。（Linux 这么做的主要原因，尽可能的充分利用内存，来提高再次启动进程的速度。）

## Android 进程保活思路



### 1. 添加系统广播监听事件

通过静态注册允许的系统广播事件，在 onReceive 里检查进程服务是否还存在，否则重新拉起。

常见的系统广播有：

- android.intent.action.USER_PRESENT
- android.net.conn.CONNECTIVITY_CHANGE
- android.intent.action.MEDIA_MOUNTED
- android.intent.action.MEDIA_UNMOUNTED
- android.net.wifi.RSSI_CHANGED
- android.net.wifi.STATE_CHANGE
- android.net.wifi.WIFI_STATE_CHANGED

> Android 7 取消了 WiFi 静态广播的注册方式



### 2. Native 层保活

开启一个 Native 子进程定期发送 Intent 检查主进程是否存活，如果主进程挂掉了，重新拉起。
内部原理是一个没有主动权的消息轮询器，必须要简历在 Native 子进程存活的基础上，目前来看之后Android 5.0 以下非国产机型才有这样的漏洞，也就是在 force close 的时候，系统忽略了 Native 子进程的存在；在 Android 5.0 以上 force close 的时候，会连同 Native 子进程一并清理掉；而且这个方案不算守护方案，因为它是单向的，只能 A保B，B保不了A。



### 3. 双向进程守护

通过父进程和子进程之间建立双管道，来监听进程是否存在，然后互相拉起。（无耗电问题、无内存消耗问题）

仅在 Android 5.0 以下系统起作用，原因是 Android 5.0 的源码中系统强杀时会连同 Group 中的所有进程一起清理掉。

**什么是双进程守护？**

首先 fork() 这个方法会创建一个子进程，然后在父进程中调用 waitpid() 阻塞函数，它会一直 wait 到子进程挂掉，才继续向下执行，利用这个机制，可以在主进程 fork 一个子进程，然后父进程就可以监听到子进程的死亡，然后去执行重启。



### 4. JobScheduler

JobScheduler 是 Android 5.0 之后引入的，本质是系统定时任务，如果进程被杀，任务仍然会被执行，但在 Android 7.0 之后 JobScheduler 添加了限制，最低间隔15分钟，但是又概率出现存在进程死亡后不在触发的情况。



### 5. AlarmManager

本质上也是通过设置定时任务，如果进程被杀，任务也仍然会被执行，此时可以拉活进程。Doze 模式会影响 AlrmManager 不被触发，此时要用 setAlarmClock 来设置。同样有概率出现存在进程死亡后，不触发的情况。
而且 Android 9.0 原生系统多了一个功能，就是显示手机下一个闹钟是几点，如果用了这种保活方式，用户也注意到了这个功能，那么闹钟上的时间会暴露有应用在执行保活策略。