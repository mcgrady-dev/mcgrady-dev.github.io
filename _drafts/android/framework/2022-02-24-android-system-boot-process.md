## Android 系统启动流程

![android_startup_process](../../images/android/android_startup_process.png)



### 1. Power On and System Startup

启动电源及系统启动，当电源按下时引导芯片代码从预定义的放（固化在ROM）开始执行。加载引导程序 BootLoader 到 RAM 中，然后执行。



### 2. BootLoader

引导程序 BootLoader 是在 Android 系统开始运行前的一个引导程序，主要作用是把操作系统拉起并运行。



### 3. Linux Kernel

Android 底层基于Linux Kernel，当内核启动时，设置缓存、被保护存储器、计划列表、加载驱动。在内核完成系统设置后，内核启动过程会在系统文件中寻找 `init.rc` 文件，并且创建和启动 `init` 进程。



### 4. init process

init 进程是 Android 系统中用户空间的第一进程（pid=1），可以说是根进程，主要用来初始化和启动属性服务，也用来启动 Zygote 进程。

init 进程主要做了一下几个事情：

#### 4.1. 创建和挂载系统运行时目录

运行时目录，只在系统运行时才会存在，系统停止时会消失，如：`/sys` `/dev` `/proc`

#### 4.2. 启动属性服务

Windows 平台上有一个注册表管理器，注册表的容采用键值对的形式来记录用户、软件的一些使用信息。即使系统或者软件重启，其还是能够根据之前注册表中的记录，进行相应的初始化工作。 Android 也提供了一个类似的机制，叫作属性服务。

init 进程启动时会启动属性服务，并为其分配内存，用来存储这些属性，如果需要直接读取就可以。

##### 系统属性

系统属性分为两种类型：

- **普通属性**
- **控制属性**
  控制属性以 `ctl.` 开头，用来执行一些命令，比如开机动画。



#### 4.3. 解析 `init.rc` 脚本，并启动 Zygote 进程

`init.rc` 是由 Android 初始化语言（Android Init Language ）编写的脚本，主要包含**5**种类型语句：`Action`、`Command`、`Service`、`Option`、`Import` 。

`init.zygoteXX.rc` 中定义了 Zygote 的启动配置，通过执行 `/system/bin/app_processXX` 来完成。

需要注意的是，在 Android 8.0 中对 init.rc 文件进行了拆分，每个服务对应一个 rc 文件。例如：64位处理器下的 Zygote 启动脚本则在 `init.zygote64.rc`。



### 5. Zygote process

#### 5.1. Zygote 概述

在 Android 系统中，Dalvik/ART 以及 App/SystemServer 的进程都是由 Zygote 进程来创建的，所以 Zygote 也称孵化器。它通过 fock（复制进程）的形式来创建应用程序进程和 SystemServer 进程，由于 Zygote 进程在启动时会创建 Dalvik/ART，因此 `fock` 出来的进程内部可以获取一个 Dalvik/ART 的实例副本。

#### 5.2. Zygote 进程启动过程

![android-zygote-init-process](../../images/android/android-zygote-init-process.png)



##### ZygoteInit

- **`registerZygoteSocket`**
  registerZygoteSocketf 方法，主要创建了名为 LocalServerSocket 服务器端的 Socket，在 Zygote 进程将 SystemServer 进程启动后，在服务器端的 Socket 上等待 AMS 请求 Zygote 进程来创建新的应用程序进程。
- **`startSystemServer`**
  最终会通过 fork 函数再当前进程创建一个子进程，也就是 SystemService 进程。
- **`runSelectLoop`**
  启动 SystemServer 进程后，会执行 `ZygoteServer.runSelectLoop` 方法，通过无线循环来等待 AMS 请求 Zygote 进程创建新的应用程序进程，收到 AMS 发过来的请求并创建一个新的应用程序进程后，将 ZygoteConnect 从 Sokcet 连接列表中清除。

`ZygoteInit.main()` 方法主要做了 4 件事：

1. 创建一个 Server 端的 Socket
2. 预加载类和资源
3. 启动 SystemServer 进程
4. 等待 AMS 请求创建新的应用程序进程



#### 5.3 Zygote 进程启动总结

Zygote 进程启动做了一下几件事：

1. 创建 AppRuntime 并调用其 `start` 方法，启动 Zygote 进程
2. 创建 Java 虚拟机并为 Java 虚拟机注册 JNI 方法
3. 通过 JNI 调用 ZygoteInit 的 `main` 函数进入 Zygote 的 Java 框架层
4. 通过 `registerZygoteSocket` 方法创建服务器端 Socket，并通过 `runSelectLoop` 方法等待 AMS 的请求来创建新的应用程序进程。
5. 启动 SystemServer 进程



### 6. SystemServer process

SystemServer 由 Zygote fork 生成，进程名为 `system_server` ，主要用于创建系统服务，例如我们熟知的 AMS、WMS 和 PMS 都有它来创建。

![android_systemserver](../../images/android/android_systemserver.png)

#### 6.1 ZygoteInit.nativeZygoteInit

`nativeZygoteInit` 是一个 Native 方法，具体的指向是 AndroidRuntime 的子类 AppRuntime 中的 `onZygoteInit` 方法，这里启动了一个 Binder 线程池，这样 SystemServer 进程就可以使用 Binder 与其它进程进行通信了。所以，`RuntimeInit.java` 的 `nativeZygoteInit` 方法主要是用来启动 Binder 线程池的。

#### 6.2 SystemServer.main

`SystemServer.main` 方法主要做了几件事：

1. 创建消息 Looper
2. 加载动态库 `libandroid.servers.so`
3. 创建系统 Context
4. 启动引导服务
5. 启动核心服务
6. 启动其它服务

#### 6.3 SystemServer 进程总结

SystemServer 进程被创建后，主要做了如下工作：

1. 启动 Binder 线程池，这样就可以与其他进程进行通信
2. 创建 SystemServiceManager ，其它用于对系统的服务进行创建】启动和生命周期管理
3. 启动各种系统服务



### 7. Launcher

#### 7.1 Launcher 概述

系统启动的最后一步是启动一个应用程序用来显示系统中已经安装的应用程序，这个应用程序就叫做 Launcher。Launcher 在启动过程中会请求 PackageManagerService 返回系统中已经安装的应用程序的信息，并将这些信息封装成一个快捷图标列表显示在系统屏幕上，这样用户可以点击这些快捷图标来启动相应的应用程序。

#### 7.2 Launcher 启动过程

SystemServer 进程在启动的过程中会启动 PackageManagerService，PackageManagerService 启动后会将系统中的应用程序安装完成。在此之前启动的 AMS 会将 Launcher 启动起来。

![android_launcher_process](../../images/android/android_launcher_process.png)



### 进程 main 方法

| 进程              | 主方法                |
| :---------------- | :-------------------- |
| init进程          | Init.main()           |
| zygote进程        | ZygoteInit.main()     |
| app_process进程   | RuntimeInit.main()    |
| system_server进程 | SystemServer.main()   |
| app进程           | ActivityThread.main() |

>注意`app_process`进程是指通过`/system/bin/app_process`启动的进程，且后面跟的参数不带`–zygote`，即并非启动zygote进程。 比如常见的有通过`adb shell`方式来执行`am`,`pm`等命令，便是这种方式。



## 应用程序进程启动过程

### 概述

AMS 在启动应用程序时会检查这个应用程序需要的应用程序进程是否存在，不存在就会请求 Zygote 进程启动需要的应用程序进程。Zygote 进程通过 fock 自身创建应用程序进程，应用程序进程在创建过程中获取虚拟机实例外，还创建了 Binder 线程池和消息循环。

### 1. AMS 发送启动应用程序进程请求

![android_app_startup_process](../../images/android/android_app_startup_process.png)

AMS 通过 startProcessLocked 方法向 Zygote 进程发送创建应用程序进程请求。



### 2. Zygote 接收请求并创建应用程序进程

![android_app_startup_process](../../images/android/android_app_startup_process_zygote.png)

`RuntimeInit.invokeStaticMain` 通过反射获取 `ActivityThread.main` 方法，传入 `Zygote.MethodAndArgsCaller` 类的构造方法中，执行 Method 的 `invoke` 方法，此时应用程序进程就创建完成并且运行了主线程的管理类 `ActivityThread.main` 方法。

### 3. Binder 线程池启动过程

在 Zygote 接收请求并创建应用程序进程过程中会调用 `ZygoteInit.nativeZygoteInit` 方法启动 Binder 线程池，并将当前线程注册到 Binder 驱动程序中，这样我们创建的线程就加入了 Binder 线程池。

### 4. 消息循环创建过程

ActivityThread 类用于管理当前应用程序进程的主线程，在 `main` 方法主要执行的内容有：

1. 创建主线程 Looper
2. 创建 ActivityThread 并初始化
3. 创建主线程 Handler
4. 开启 Looper 消息循环



## Activity 启动过程

### 1. Launcher 请求 AMS 过程

当我们点击某个应用程序的快捷图标时，就会通过 Launcher 请求 AMS 来启动该应用程序。

![android_activity_startup_process1](../../images/android/android_activity_startup_process1.png)

#### Instrumentation

Instrumentation 主要用来监控应用程序和系统的交互。

##### execStartActivity

`execStartActivity` 方法调用  `ActivityManager.getService()` 通过 AIDL 获取的 AMS 本地代理 IActivitManager，所以最终调用的是 AMS 的 startActivity 方法。

### 2. AMS 到 ApplicationThread 的调用过程

![android_activity_startup_process2](../../images/android/android_activity_startup_process2.png)

#### ActivityStarter

ActivityStarter 是 Android 7.0 中新加入的类，它是加载 Activity 的控制类，会收集所有的逻辑来决定如何将 Intent 和 Flags 转换为 Activity，并将 Activity 和 Task 以及 Stack 相关联。

##### startActivityUnchecked

`startActivityUnchecked` 方法主要处理与栈管理相关的逻辑。内部调用 `setTaskFromReuseOrCreateNewTask` 创建一个新的 Activity 任务栈。

#### ApplicationThread

ApplicationThread 是 ActivityThread 的内部类，是 AMS 所在进程（SystemServer 进程）和应用程序进程的通信桥梁。

![android_application_thread](../../images/android/android_application_thread.png)

### 3. ActivityThread 启动 Activity 的过程

![android_activity_startup_process3](../../images/android/android_activity_startup_process3.png)

#### H

H 继承自 Handler，是 ActivityThread 的内部类，作为应用程序进程中主线程的消息管理类。因为 ApplicationThread 是一个 Binder ，它的调用逻辑运行在 Binder 线程池中，所以这里需要用 H 将代码的逻辑切换到主线程中。

### 根 Activity 启动过程中设计的进程

![SCR-20220315-n7y-1](../../../../Pictures/Screenshot/SCR-20220315-n7y-1.png)

![SCR-20220315-n7y](../../../../Pictures/Screenshot/SCR-20220315-n7y.png)
