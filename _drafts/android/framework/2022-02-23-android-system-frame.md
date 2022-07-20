![](../../images/android/android_framework_details.png)

## 1. Apps

应用层包含系统内置应用和第三方开发者提供的应用。



## 2. Framework

Java API 框架层，这些 API 包含形成创建 Android 应用所需的构建块，可简化核心模块化系统组件和服务的重复使用。

### ActivityManager

活动管理器，管理各个应用程序生命周期，以及常用的导航回退功能。

### WindowManager

窗口管理器，管理所有开启的窗口程序。

### ViewSystem

视图系统，构建应用程序的基本组件

### ContentProviders

内容提供者，使不同的应用程序之间可以共享数据。

### LocationManager

位置管理器，提供地理位置定位功能服务。

### PackageManager

包管理器，管理所有安装在 Android 系统中的应用程序。

### NotificationManager

通知管理器，使得应用程序可以在状态栏中显示自定义提示信息。

### ResourceManager

资源管理器，提供应用使用的各种非代码资源，如本地化字符串、图片、布局文件、颜色文件等。

### TelephonyManager

电话管理器，管理所有移动设备功能。



## 3.1 Native Libraries

原生 C/C++ 库，许多核心 Android 系统组件和服务（例如 ART 和 HAL）构建自原生代码，需要以 C/C++ 编写的原生库。

### SQLite

轻量的关系型数据库引擎

### WebKit

### MediaFramework

多媒体库，支持多种常用的音频、适配格式录制和回放。

### AudioManager

### SurfaceManager

### OpenGL ES

OpenGL for Embedded Systems 是 OpenGL 三维图形 [API](https://baike.baidu.com/item/API/10154) 的子集，针对[手机](https://baike.baidu.com/item/手机/6342)、[PDA](https://baike.baidu.com/item/PDA/111022)和游戏主机等[嵌入式设备](https://baike.baidu.com/item/嵌入式设备/10055189)而设计，用于[渲染](https://baike.baidu.com/item/渲染)[2D](https://baike.baidu.com/item/2D)、[3D](https://baike.baidu.com/item/3D)[矢量图形](https://baike.baidu.com/item/矢量图形)的跨[语言](https://baike.baidu.com/item/语言)、[跨平台](https://baike.baidu.com/item/跨平台)的[应用程序编程接口](https://baike.baidu.com/item/应用程序编程接口)（API）。

### SSL

Secure Sockets Layer 安全套接字协议，是一种为网络通信提供安全即数据完整性的安全协议，在[传输层](https://baike.baidu.com/item/传输层/4329536)与[应用层](https://baike.baidu.com/item/应用层/16412033)之间对网络连接进行加密。



## 3.2 Android Runtime

### ART

Android Runtime (ART) 是 Android 上的应用和部分系统服务使用的**托管式运行时**。

以下是 ART 的一些主要功能：

- 预先（AOT）编译
- 优化的垃圾回收
- 在 Android 9（API 级别 28）及更高版本的系统中，支持将应用软件包中的 Dalvik Executable 格式 (DEX) 文件转换为更紧凑的机器代码
- 更好的调试支持

在 Android 8.0 版本中，Android Runtime (ART) 有了极大改进，以下是主要增强的功能：

- 并发压缩式垃圾回收器
- 循环优化

### Dalvik

Dalvik虚拟机，是 Android 平台的核心组成部分之一。虚拟机可运行Java平台应用程序，这些应用程序被转换成紧凑的Dalvik可执行格式（.dex），该格式适合内存和处理器速度受限的系统。

与 大多数虚拟机和真正的Java虚拟机不同，前者是栈机（stack machine），而Dalvik VM是基于寄存器的架构。

为满足低内存要求而不断优化， Dalvik虚拟机有一些独特的、有别于其它标准虚拟机的特征：

- 虚拟机很小，使用的空间也小；
- Dalvik没有JIT编译器；
- 常量池已被修改为只使用32位的索引，以简化解释器；
- 它使用自己的字节码，而非Java字节码。

### CoreLibraries

核心运行时库，可提供 Java API 框架所使用的 Java 编程语言中的大部分功能，包括一些 [Java 8 语言功能](https://developer.android.google.cn/guide/platform/j8-jack)。



## 4. HAL (Hardware Abstraction Layer)

硬件抽象层，是位于操作系统内核与硬件电路之间的接口层，其目的在于将硬件抽象化，为了保护硬件厂商的知识产权，它隐藏了特定平台的硬件接口细节，为操作系统提供了虚拟硬件平台，使其具有硬件无关性，可在多种平台上进行移植。通俗来讲，就是将控制硬件的动作放在硬件抽象层处理。



## 5. Linux Kernel

Android 平台的基础是 Linux 内核，例如，Android Runtime (ART) 依靠 Linux 内核来执行底层功能。并且**系统的安全性、内存管理、进程管理、网络协议和驱动模型等都依赖于 Linux 内核**。