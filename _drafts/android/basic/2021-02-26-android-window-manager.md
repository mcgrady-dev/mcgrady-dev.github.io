---
layout: article
title: Android Window Manager
date: 2022-03-08 13:44 +0800
tags: android
---

Window、WindowManager和WindowManagerService的关系

<!--more-->

提供了标准的UI策略，例如背景、标题区域、默认键处理等。



## Window

Window是用于承载View对象的容器，Window对象始终由Surface对象提供支持。

每个Activity都有一个Window，用于在屏幕上显示其内容。当执行`setContentView()`方法时，WindowManager会将该视图附加到Activity的默认Window。默认Window会填满屏幕，因此窗口会隐藏任何其他Activity。



### Window的属性

- Application Window：应用程序窗口类型，Activity就是一个典型的应用程序窗口；
- Sub Window：子窗口类型，PopupWindow就属于子窗口；
- System Window：系统窗口类型，例如：Toast、输入法窗口、系统音量条窗口、系统错误窗口等；

当一个进程向WindowManagerService申请一个窗口时，会根据窗口的属性确定窗口显示的次序，这个次序叫做**Z-Order**，指屏幕垂直方向从内到外的一个次序。

### Window Flag

Window标记拥有控制Window的显示，常见的类型有：

- FLAG_FULLSCREEN：全屏显示，隐藏其余所有窗口；
- FLAG_KEEP_SCREEN_ON：只要窗口可见，屏幕一直处于常亮状态；
- FLAG_NOT_FOCUSABLE：窗口不能获取输入焦点；



### PhoneWindow

PhoneWindow是Window的具体实现类，它对View进行管理

Activity的启动过程中会调用ActivityThread的`performLaunchActivity()`方法，其中又会调用`Activity.attach()`方法，PhoneWindow就是在这里创建的。



### Window的删除过程

1. 检查删除线程的正确性，不满足则抛出异常；
2. 从ViewRootImpl列表、布局参数列表和View列表中删除View对应的元素；
3. 判断是否可以直接执行删除操作，不满足则推迟执行；
4. 执行删除操作，清理和释放与View相关的一切资源；



## WindowManager

WindowManager用于管理Window对角，并会监督生命周期、用户输入和聚焦事件、屏幕方向、旋转、位置布局、Z轴次序（Z-Order）等。WindowManager还会降所有的Window的窗口元数据（Window Metadata）发送到SurfaceFlinger，以便SurfaceFlinger使用这些数据在屏幕上合成Surface。

### WindowManagerImpl

WindowManagerImpl虽然是WindowManager的实现类，但没实现什么功能，而是委托给了WindowManagerGlobal，这里用到的是桥接模式。

### WindowManagerGlobal

WindowManagerGlobal作为与WindowManager的底层通信，为WindowManager提供Window操作相关的主要方法：添加、更新和删除。



## WindowManagerService

WIndowManagerService即窗口管理服务，负责Window的启动、添加和删除，另外Window的大小和层级也由WindowManagerService来管理。核心的成员有DisplayContent、WindowToken和WindowState。

下面来了解下WMS的核心组件：

### Session

Seesion主要用于进程间通信，其它应用程序进程想要和WMS交互就需要经过Session。



### DisplayContent

DisplayContent是Android 4.2为支持多屏幕输出锁引入的一个概念，一个DisplayContent对象用于描述一块屏幕。

### WindowToken

WindowToken窗口令牌主要有两个作用：

- 当应用程序向WMS申请创建一个Window时，则需要向WMS出示有效的WindowToken；
- 将同一类型组件的WindowState集合在一起管理，比如：Application Window、Sub Window、System Window；

### WindowState

WindowState用于保存窗口的信息，在WMS中WindowState用于描述一个窗口的所有属性。

### WindowAnimator

用于管理窗口的动画

### InputManagerService

输入系统管理服务，用于管理每个窗口的输入事件通道（InputChannel），而WMS仅是作为IMS的中转。



### WindowManagerService启动过程

1. **SystemServer.startOtherServices()**：开始启动WindowManagerService等服务；
2. **WindowManagerService.main()**：在`android.display`线程中初始化WindowManagerService实例，同时还创建了DisplayContent（用于多屏幕显示）和窗口管理策略PhoneWindowManager；
3. **PhoneWindowManager.init()**：在`android.ui`线程中运行；
4. **WindowManagerService.displayReady()**：显示就绪；
5. **WindowManagerService.systemReady()**：调用`PhoneWindowManager.systemReady()`；



### WindowManagerService添加Window的过程

1. **PhoneWindowManager.checkAddPermission()**：对所要添加的窗口进行检查，条件满足才得以继续；
2. 获取WindowToken，并对特定窗口类型进行WindowToken的检查，条件满足才得以继续；
3. 创建WindowState存储窗口的信息；
4. 创建和配置DisplayContent，完成窗口添加到系统前的准备工作；



### Window的布局

`WMS.relayoutWindow()`方法修改指定窗口的布局参数，然后`WMS.performLayoutAndPlaceSurfacesLocked()`遍历所有窗口并对它们进行重新布局
