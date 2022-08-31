---
layout: article
title: Harmony应用开发（Java篇）
date: 2022-08-26 16:50 +0800
tags: harmony
---



<!--more-->

## 前言

### HarmonyOS VS Android

| HarmonyOS            | Android                              | 说明                       |
| -------------------- | ------------------------------------ | -------------------------- |
| Ability              | Activity & Service & ContentProvider | FA/PA                      |
| AbilitySlice         | Fragment                             | 单个页面及其控制逻辑的总和 |
| CommonEventSubsciber | BroadcastReceiver                    | 公共事件                   |
| AbilityPackage       | Appliaction                          | 应用程序入口及管理         |
| Intent               | Intent                               | 消息传递对象               |
| Ability Form         | Widget                               | 桌面小部件                 |
| NotificationHelper   | NotificationManager                  | 通知管理器                 |
| SystemPasteboard     | ClipboardManager                     | 系统剪贴板服务             |

<br>

## Ability框架

Ability是应用所具备能力的抽象，也是应用程序的重要组成部分。一个应用可以具备多种能力（即可以包含多个Ability）。Ability分为两种类型：**FA**(Feature Ability)和**PA**(Particle Ability)。

<br>

### PageAbility

Page模板是FA唯一支持的模板，用于提供与用户交互的能力。一个Page可以由一个或多个AbilitySlice构成，AbilitySlice是指应用的单个页面及其控制逻辑的总和。

Ability类提供的回调机制能够让Page及时感知外界变化，从而正确地应对状态变化（比如释放资源），这有助于提升应用的性能和稳健性。<br>

<img src="https://s2.loli.net/2022/08/29/uUWH9xvXnb48BOj.png" alt="harmonyos-page-ability-lifecycle" style="zoom:75%;" />

- **onStart()**
  当系统首次创建Page实例时触发，此后Page进入**INACTIVE**状态。
- **onActivie()**
  Page来到前台时回调，此后Page进入**ACTIVE**状态。
- **onInactive()**
  当Page失去焦点时回调，此后Page回到**INACTIVE**状态。
- **onBackground()**
  当Page对用户不可见时回调，此后Page进入**BACKGROUND**状态，此时Page仍驻留在内存中。
- **onForeground()**
  当处于**BACKGROUND**状态的Page重新回到前台时回调，此后Page回到**INACTIVE**状态。
- **onStop()**
  当系统将要销毁Page时回调，此后Page被销毁。销毁Page的原因可能包括以下几个方面：
  - 用户通过系统管理能力关闭指定Page；
  - 用户行为触发Page的`terminateAbility()`方法，例如使用应用的退出功能；
  - 配置变更导致系统短暂销毁Page并重建；
  - 系统出于资源管理目的，自动触发对处于BACKGROUND状态的Page进行销毁；

<br>

### AbilitySlice

AbilitySlice作为Page的组成单元，承载了具体的页面，其生命周期是依托于其所属Page生命周期的，**AbilitySlice和Page具有相同的生命周期状态和同名的回调**，此外AbilitySlice还有独立于Page的生命周期变化，变化发生在同一Page中的AbilitySlice之间导航时。

系统为每个Page维护了一个AbilitySlice实例的栈，每个进入前台的AbilitySlice实例均会入栈。当指定的AbilitySlice实例已经存在栈中时，则位于此实例之上的AbilitySlice均会出栈并终止其生命周期。

<br>

### ServiceAbility

基于Service模板的Ability主要用于后台运行任务（如执行音乐播放、文件下载等），但不提供用户交互界面。Service是单实例的。在同一设备上相同的Service只会存在一个实例。如果多个Ability共用这个实例，只有当与Service绑定的所有Ability都退出后，Service才能够退出。由于Service是在主线程里执行的，如果在Service里面的操作时间过长，会导致主线程阻塞，应用程序无响应，必须创建新的线程来处理。

ServiceAbility的生命周期：<br>

<img src="https://s2.loli.net/2022/08/29/QtYmHBuCFPRJgLI.jpg" alt="harmonyos-service-ability" style="zoom:85%;" />

- **onStart()**
  当创建Service时回调，在整个生命周期只会调用一次，用于Service初始化。
- **onCommand()**
  在Service创建完成后，当客户端每次启动该Service时回调。
- **onConnect()**
  当Ability和Service连接时回调，该方法会返回IRemoteObject对象，用于生成Ability与Service之间的IPC通信通道。
- **onDisconnect()**
  当Ability与绑定的Service断开连接时回调。
- **onStop()**
  在Service销毁时回调，用于清理资源、关闭现成、注销监听器等。销毁Service的原因可能包括以下几个方面：
  - 必要时系统会停止或销毁Service以回收内存资源；
  - 在Service中调用`terminateAbility()`方法停止Service；
  - 在其他Ability调用`stopAbility()`来停止Service；

<br>

### DataAbility

Data模板的Ability用于应用管理其自身和其他应用存储数据的访问，并提供与其他应用共享数据的方法。DataAbility既可用于同设备不同应用的数据共享，也支持跨设备不同应用的数据共享。

DataAbility的提供方和使用方都通过URL(Uniform Resource Identifier)来标识一个具体的数据，格式如下：

![harmonyos-dataability-scheme](https://s2.loli.net/2022/08/29/1bcDzIBYG47a3OC.png)

<br>

## 公共事件与通知开发

**公共事件服务**（Common Event Service）提供订阅、发布、退订公共事件的能力。<br>

<img src="https://s2.loli.net/2022/08/30/f1enVGgZtMYFIi8.jpg" alt="harmonyos-ces" style="zoom: 67%;" />

<br>

**通知增强服务**（Advanced Notification Service）提供应用发布即时消息和通信消息的能力。常见的场景包括：

- 显示接收到短消息、即时消息等；
- 显示应用的推送消息；
- 显示当前正在进行的时间，如播放音乐、导航、下载等；

<br>

![harmonyos-ans](https://s2.loli.net/2022/08/30/KPinpfzQOJX2d5U.png)

<br>

**NotificationSlot**可以对提示音、振动、重要级别等进行设置。一个应用可以创建一个或多个NotificationSlot，在发布通知时，通过绑定不同的NotificationSlot，实现不同用途。

<br>

## 后台任务调度和管控

HarmonyOS将后台任务分为三种类型：

- 无后台业务：无任务处理；
- 短时任务：退到后台的应用有不可中断且短时间能完成的任务时使用，可申请延迟进入挂起（Suspend）状态；
- 长驻任务：退到后台的应用有长期运行的任务时使用，可避免进入挂起状态；

### 托管任务

托管任务是系统提供的一种后台代理机制。通过系统提供的代理API接口，用户可以把任务（如后台下载、定时提醒、后台非持续定位）交由系统托管。

<br>

## 线程管理

多线程开发可以使用TaskDispatcher来分发不同的任务。

TaskDispatcher是一个任务分发器，是Ability分发任务的基本接口，隐藏任务所在的线程细节。TaskDispatcher有多种不同任务的实现：

### GlobalTaskDispatcher

全局并发任务分发器，适用于任务之间没有联系的情况。一个应用只有一个GlobalTaskDispatcher。

### ParallelTaskDispatcher

并发任务分发器，与GlobalTaskDispatcher不同的是，ParallelTaskDispatcher不具备全局唯一性，可以创建多个实例。

### SerialTaskDispatcher

串行任务分发器，所有任务都按一定的顺序执行，但执行这些任务的线程并不是固定的。

### SpecTaskDispatcher

专有任务分发器，绑定到专有线程上的任务分发器。目前已知的专有线程为UI线程。

<br>

## 线程间通信

**EventHandler**是HarmonyOS用于处理线程间通信的一种机制。EventHandler的主要功能是将InnerEvent事件或者Runnable任务投递到其他的线程进行处理。<br>

<img src="https://s2.loli.net/2022/08/29/GEdbhXVqkCgHnfS.jpg" alt="harmonyos-eventhandler" style="zoom:80%;" />

EventRunner的工作模式可以分为**托管模式**和**手动模式**。两种模式是在调用EventRunner的`create()`方法时，通过选择不同的参数来实现的。

<br>

## 术语

- **FA**：Feature Ability，元服务，代表有界面的Ability，用于用户交互。
- **PA**：Particle Ability，元能力，代表无界面的Ability，主要为Feature Ability提供支持，；例如作为后台服务提供计算能力，或作为数据仓库提供数据访问能力。



<br><br>
