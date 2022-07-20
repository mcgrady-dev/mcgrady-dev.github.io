## IPC

IPC (Inter-Process Communication) 即进程间通信，指的是**两个进程之间进行数据交换的过程**。





## Binder



## AIDL

AIDL (Android Interface Definition Language) 即 Android 接口定义语言，是 Android 提供的 IPC 的一种独特实现。

![MyServer_java_binder](../../images/android/android_aidl.png)

### 什么情况下要使用AIDL

使用AIDL只有在你允许来自不同应用的客户端跨进程通信访问你的service，并且想要在你的service种处理**多线程**的时候才是必要的。 如果你不需要执行不同应用之间的IPC并发，你应该通过实现Binder建立你的接口，或者如果你想执行IPC，但是不需要处理多线程。那么使用Messenger实现你的接口。



## Messenger



## ContentProvider

ContentProvider（内容提供者）是Android的四大组件之一，管理android以结构化方式存放的数据，以相对安全的方式封装数据（表）并且提供简易的处理机制和统一的访问接口供**其他程序**调用。　

Android的数据存储方式总共有五种，分别是：Shared Preferences、网络存储、文件存储、外储存储、SQLite。但一般这些存储都只是在单独的一个应用程序之中达到一个数据的共享，有时候我们需要操作其他应用程序的一些数据，就会用到ContentProvider。而且Android为常见的一些数据提供了默认的ContentProvider（包括音频、视频、图片和通讯录等）。

但注意ContentProvider它也只是一个中间人，真正操作的数据源可能是数据库，也可以是文件、xml或网络等其他存储方式。



## Socket

Socket 通信方式也是 C/S 架构，比 Binder 简单很多。Socket 方式更多用于在 Framework 层与 Native 层之间的通信，在 Android 系统中采用 Socket 通信方式的主要有：

- **zygote**
  用于浮华进程，system_server 进程创建进程是通过 socket 向 zygote 进程发起请求
- **instanlld**
  用于安装 app 的守护进程，上层 PackageManagerService 很多实现组中都交由它来完成
- **lmkd**
  LowMemoryKiller 的守护进程，Java层的 LowMemoryKiller 最终都由 lmkd 来完成
- **adbd**
  用于服务 adb 的守护进程
- **logcatd**
  用于服务 logcat 的守护进程
- **vold**
  Vokume Daemon 是存储类的守护进程，用于负责如 USB、Sdcard 等存储设备的事件处理