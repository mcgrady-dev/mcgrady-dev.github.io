---
layout: article
title: Android Binder IPC
date: 2022-03-08 13:44 +0800
tags: android

---

通过理解IPC相关的概念和原理，来认识Android中相关的IPC机制，我们知道Binder是Android系统的IPC的核心，从框架层的系统服务的实现，到应用层跨进程的调用，都离不开Binder，同时其它的IPC技术实现，本文将带你逐步的深入Android中IPC的方方面面。

<!--more-->



## 前言

由于Android内核基于Linux，在认识Android系统的IPC机制之前，先了解Linux IPC的相关概念和原理。

![linux-ipc](https://s2.loli.net/2022/10/26/JICW9tXFM25KuiY.png)



### 虚拟内存与进程隔离

要理解进程的隔离，需要明白虚拟内存的出现才实现了进程的隔离。

**虚拟内存是计算机系统内存管理的一种技术**，使得应用程序认为它拥有连续的可用内存，而实际上，物理内存通常有可能是被分隔成多个物理内存碎片的，还有部分暂存在外部磁盘存储器上，在需要时进行数据交换。

**虚拟内存的出现解决了什么问题？**

1. 我们知道程序的运行是需要经过内存（RAM）执行的，如果执行的程序占用内存很大，则会导致内存消耗殆尽，为了解决该问题，虚拟内存技术将划出一部分硬盘空间来充当内存使用，当内存消耗殆尽时，自动调用磁盘来充当内存，以缓解内存的紧张，并且也提高了物理内存（RAM）的使用效率。
2. 虚拟内存的机制为每个进程分配了线性连续的内存空间，操作系统将这种虚拟内存空间映射到物理内存空间，每个进程才有自己的虚拟内存空间，并且进程只能操作自己的虚拟内存空间，进而不能操作其他进程的内存空间。对于物理内存空间，只有操作系统才能有权限操作物理内存空间，所以**进程的隔离保证了每个进程的内存安全**。

由此可知，**每个进程都有自己的一部分独立的系统资源，彼此是隔离的**。但进程间的通信是不可避免的，为了使不同的进程互相访问资源并进行协调工作，所以才有了进程间通信（IPC）。



### IPC（Inter-Process Communication）

**进程间通信**指至少两个进程或线程间传送数据或信号的一些技术或方法。通常，使用进程间通信的两个应用可以被分为客户端和服务器，客户端进程请求数据，服务端响应客户端的数据请求。

**Linux中常见的IPC实现有：**

- **管道(pipes)**：在创建时分配一个`page`大小的内存，缓存去大小比较有限；
- **消息队列(message queues)**：信息复制两次，额外的CPU消耗，不适合频繁或信息量打的通信；
- **共享内存(shared memory)**：无需复制，共享缓冲区直接附加到进程虚拟地址空间，速度快，但进程间的同步问题操作系统无法实现，必须个进程利用同步工具解决；
- **套接字(socket)**：作为更通用的接口，传输效率低，主要用于不同机器或跨网络的通信；
- **信号量(semaphores)**：作为一种锁机制，防止某进程正在访问共享资源时，其它进程也访问该资源。因此，主要作为进程间以及统一进程内不同线程之间的**同步手段**；
- **信号(signals)**：不适用与信息交换，更适用于进程中断控制，比如非法内存访问，杀死某个进程等；



### RPC（Remote Procedure Call）

远程过程调用是一个计算机通信协议，该协议描述了允许运行于一台计算机的程序调用另一个**地址空间**（通常为网络中的一台计算机）的子程序，而这个远程调用过程就像本地程序的调用一样，无需额外地关注这个交互的细节就可以完成这个过程。准确来讲，**远程调用是指跨进程的调用，可以是同一台计算上的多个进程、多个虚拟机实例或者多态计算机之间的进程的调用**。

同时**RPC也是一种进程间通信的模式**，程序分布在不同的地址空间里，例如在同一主机里，RPC可以通过不同的虚拟地址空间（即便使用相同的物理内存地址）进行通讯；而在不同的主机间，通过不同的物理地址进行交互。

通常**RPC的实现总是采用C/S架构实现，由客户端向服务端发出一个执行若干个过程的请求，并用客户端提供的参数完成执行，将结果返回给客户端**。

通过RPC的定义，可以理解Android系统中的Binder是利用远程过程调用（RPC）协议提供的一种远程通信（IPC）技术。



### 进程空间划分

操作系统的核心是内核，独立于普通的应用程序，可以访问受保护的内存空间，也可以访问底层硬件设备的权限，为了保护用户进程不能直接操作系统内核，从逻辑上讲虚拟空间划分为**用户空间（User Space）**和**内核空间（Kernel Space）**。



### 系统调用

系统调用是用户空间访问内核空间的唯一方式，所以用户空间要突破进程间的隔离限制才能访问内核空间，同时还要保证所有的资源访问都是在内核的控制下进行的，为了避免用户程序对系统资源的越权访问，以提高系统安全性和稳定性。Linux使用了两级保护机制来实现：0级共系统内核使用，3级供用户程序使用。

**当一个任务（进程）执行系统调用而陷入内核代码中执行时，称进程处于内核运行态（内核态）**，此时处理器处于特权级最高（0级）的内核代码中执行，当进程处于内核态时，执行的内核代码会使用当前进程的内核栈（每个进程都有自己的内核栈）。

**当进程在执行用户自己的代码时，称其处于用户运行态（用户态）**，此时处理器子处于特权级最低的（3级）用户代码中执行。

![android-binder-syscall](/Users/mcgrady/Pictures/gallery/android/binder/android-binder-syscall.png)



### 动态内核可加载模块

跨进程通信时需要内核空间做支持的，Linux现存的如管道、Socket都是内核的一部分，而Binder并不是Linux系统内核的部分，是通过Linux的**动态内核可加载模块（Loadable Kernel Module，LKM）**机制添加的。模块是具有独立功能的程序，可以被单独编译，在运行时被链接到内核作为内核的一部分运行。



### ioctl

**ioctl(input/output control)设备输入输出控制**是一个独立的系统调用，对设备的I/O通道进行管理，通过它用户空间可以跟内核设备驱动沟通。



### 内核映射

内核映射就是将用户空间的一块内存区域映射到内核空间，映射关系建立后，用户对这块内存区域的修改可以直接反映到内核空间， 反之内核空间对这块区域的修改也能直接反映到用户空间。内存映射能减少数据拷贝的次数，实现用户空间与内核空间的高效互动。Binder正是基于内存映射（mmap）实现的。

通常`mmap()`是用在物理介质的文件系统上，比如进程中的用户区域不能直接和物理磁盘打交道的，如果想把磁盘的数据读取到进程的用户区域，通常需要两次拷贝：
```mermaid
graph LR
	Disk(Disk) -->|copy one| KS(Kernel Space) -->|copy two| UP(User Space)
```

在这种情况下，`mmap()`就可以通过在物理介质和用户空间之间建立映射，减少数据的拷贝次数，用内存读取来取代I/O读写，提高文件读取效率。



### Gather-scatter机制

**Gather-scatter即聚集-发散机制**，常用于需要传输的数据分开处理的场景。

**Gather聚集机制**，指在`write()`操作时将多个Buffer的数据写入同一个Channel，因此Channel将多个Buffer的数据聚集后发送。

**Scatter发散机制**，指从Channel中`read()`操作时将读取的数据写入多个Buffer中，因此Channel将读取的数据分散到多个Buffer中。



## Binder

Binder是Android特有的通过轻量级远程过程调用（RPC）机制实现的进程间（IPC）通信技术。

Binder使用C/S架构。Binder框架定义了四个角色：Server、Client、ServiceManager以及Binder Driver。其中Server、Client、ServiceManager运行于用户空间，Binder Driver运行于内核空间。Binder Driver提供设备文件`/dev/binder`与用户空间交互，Client、Server 和 Service Manager通过 `open` 和 `ioctl` 文件操作函数与Binder Driver进行通信。



### ServiceManager

ServiceManager是Binder IPC通信过程中的**守护进程**，集中管理系统内的所有Server并向Client提供查询Server远程接口的功能。

> 注意：这里指的是Native(C/C++) Framework层的ServiceManager，并非Java Framework层的ServiceManager。

**ServiceManager核心的两个功能**

- 注册服务：记录服务名和handle信息，保存到`svclist`列表；
- 查询服务：根据服务名查询相应的handle信息；

**ServiceManager启动过程**

1. `binder_open()`打开Binder驱动，并调用`mmap()`分配128k内存映射空间；
2. `binder_become_context_manager()`通知Binder驱动使其称为守护进程；
3. 验证`slinux`权限，判断进程是否有权注册或查看指定服务；
4. `binder_loop()`进入循环状态，等待Client端的请求，同时也是ServiceManager成为Binder IPC中守护进程的关键；
5. 注册服务的过程，根据服务名称，但同一个服务已注册，重新注册前会先移除之前的注册信息；
6. 死亡通知，当Binder所在进程死亡后，调用`binder_release()`和`binder_node_release()`这个过程便会发出死亡通知的回调；



### Binder Driver

Binder驱动是一种虚拟的字符设备，注册在`/dev/binder`中，定义了一套Binder的通信协议，负责建立进程间的Binder通信，提供数据包在进程之间传递的一些列底层支持。

**Binder驱动的4个核心方法：**

- **binder_init()**：初始化binder驱动，binder驱动以misc设备进行注册，作为虚拟字符设备；
- **binder_open()**：打开驱动设备；
- **binder_mmap()**：在内核虚拟地址空间申请一块与用户虚拟内存空间相同大小的内存，然后再申请1个页（page）大小的物理内存，再将同一块物理内存分别映射到内核和用户的虚拟内存空间，从而实现了用户空间和内核空间的同步；
  <img src="https://s2.loli.net/2022/10/27/GKtDiYMjA7VHQTN.jpg" alt="binder_physical_memory" style="zoom:70%;" />
- **binder_ioctl()**：负责在两个进程间发IPC数据和IPC`replay`数据；



### Binder通信

**Binder在系统层级上的通信流程**

Client进程通过RPC与Server通信，可以简单地划分为三层，驱动层、IPC层、应用层。

<img src="/Users/mcgrady/Pictures/gallery/android/binder/android-java-binder.png" alt="android-java-binder" style="zoom:85%;" />

**Binder在进程间通信的流程**

<img src="http://gityuan.com/images/binder/binder_dev/binder_memory_map.jpg" alt="binder_memory_map" style="zoom: 75%;" />

**Binder通信从ServiceManager到Binder Driver的过程**

<img src="https://s2.loli.net/2022/11/01/l1tMNIzfcO4VGsH.jpg" alt="android-binder-ipc-from-sm-to-driver" style="zoom:75%;" />



### Binder事务缓冲区大小

Binder事务缓冲区具有一个有限的固定大小，当前为1MB。这里的1MB空间并不是当前操作独享的，而是由当前进程所共享的。也就是说Intent在Activity间传输数据，本身也不适合传输太大的数据。



### Binder通信协议

BINDER_COMMAND_PROTOCOL

BINDER_RETURN_PROTOCOL



### Binder线程池

Binder通信实际上是位于不同进程中的线程之间的通信，对于Server端进程，可能会有许多Client端同时发起请求，为了提高效率往往开Binder线程池并发处理请求。

每当Zygote进程`fork`新进程的同时，都会为该进程分配一个Binder线程池，并向其中注册一个Binder线程（主线程），之后Server进程也可以向Binder线程池注册新的线程，或者Binder驱动在探测没有空闲Binder线程时会主动向Server进程注册新的Binder线程，对于Server进程有一个最大的Binder线程数（默认16）的限制，对于Client进程的Binder事务请求都是交由Server进程的Binder线程处理。



### Binder中的Gather-scatter机制

在Android 8之前，通过Binder传输数据要经过3次复制：

![Fig4](https://s2.loli.net/2022/10/26/3dzZtYWSC5ailNm.png)

1. 在序列化到Package期间；
2. 在内核驱动程序将数据复制到目标进程期间；
3. 在目标进程对Package进行反序列化期间；

**为什么选择Gather-scatter优化？**

在Android 8中的Binder框架中，引入了Gatter-scatter优化，使用这种方法，可以将数据的复制次数从三个减少到一个，这种优化主要**避免了序列化和反序列化期间的复制**。

翻译：数据的原始数据结构和内存布局将被保留，内核驱动程序会将其复制到目标进程。由于即使在目标进程内存中结构和内存布局也是相同的，所以无需另一个副本即可读取数据。所以在目标进程中，也不需要反序列化复制。

**Gather-scatter优化后的Binder通信**

引入Gather-scatter机制是为了避免原始内存缓冲区的序列化和反序列化复制。通过引入两个新的事务命令`BC_TRANSACTION_SG`和`BC_REPLY_SG`来支持Gather-scatter机制：

![Fig9](https://s2.loli.net/2022/10/26/jLR9Sp6FOolAehb.png)



### Android为什么采用Binder作为IPC机制？

Android可能采用Binder作为Android IPC机制的几个方面：

- **性能**：Binder数据拷贝只需要1次，对于管道、消息队列、Socket都需要2次，共享内存无需拷贝，从性能角度看，Binder仅次于共享内存。
- **稳定性**：Binder基于C/S架构，Server端与Client端相对独立，稳定性较好；而共享内存实现方式复杂，需要葱粉考虑访问临界资源的并发同步问题，否则可能出现死锁问题；从稳定性角度看，Binder优于共享内存。
- **安全性**：传统 Linux IPC的接收方无法获得对方进程可靠的 UID/PID，从而无法鉴别对方身份，而Android为每个安装的App分配了UID，故进程的UID是鉴别进程身份的重要标准；Android还对外置暴露Client端，Server端会根据权限控制策略，判断UID/PID是否满足访问权限。
  传统Linux IPC只能由用户在数据包里填入UID/PID，另外，可靠的身份标记只有由IPC机制本身在内核汇总添加。其次传统IPC访问接入点是开放的，无法简历私有通道。从安全角度看，Binder的安全性更高。
- **语言层面**：Android基于Java语言（面向对象），而Binder也符合面向对象的思想，将进程通信转化为通过对某个 Binder 对象的引用调用该对象的方法，而其独特之处在于 Binder对象是一个可以跨进程引用的对象，它的实体位于一个进程中，而它的引用却遍布于系统的各个进程之中。可以从一个进程传给其它进程，让大家都能访问同一 Server，就像将一个对象或引用复制给另一个引用一样。Binder模糊了进程边界，淡化子进程间通信过程，整个系统仿佛运行于同一个面向对象的程序之中。从语言层面，Binder更适合基于面向对象语言的系统。
- **公司战略**：Linux内核是开源的系统，所开放源代码许可协议GPL保护，受GPL保护的Linux Kernel是运行在内核空间，对于上层的任何类库、服务、应用等运行在用户空间，一旦进行SysCall（系统调用），调用到底层Kernel，那么也必须遵循GPL协议。Google巧妙地将GPL协议控制在内核空间，将用户空间的协议采用Apache-2.0协议（允许基于Android的开发商不向社区反馈源码），同时在GPL协议与Apache-2.0之间的Lib库中采用BSD证授权方法，有效隔断了GPL的传染性。

### 为何不直接让发送端和接收端直接映射到同一个物理空间，那样就连一次复制的操作都不需要了？

0次复制操作那就与Linux标准内核的共享内存的IPC机制没有区别了，对于共享内存虽然效率高，但是对于多进程的同步问题比较复杂，而管道/消息队列等IPC需要复制2两次，效率较低。





## AIDL

**Android接口定义语言（Android Interface Definition Language，AIDL）**是Android提供的进程间通信IPC工具Binder的具体使用方法。通过AIDL可以定义客户端和服务端之间均认可的编程接口，以便二者使用进程间通信（IPC）进行互相通信。

![android-aidl](/Users/mcgrady/Pictures/gallery/android/binder/android-aidl.png)



## Messenger



## ContentProvider

ContentProvider（内容提供者）是Android的四大组件之一，管理android以结构化方式存放的数据，以相对安全的方式封装数据（表）并且提供简易的处理机制和统一的访问接口供**其他程序**调用。　

Android的数据存储方式总共有五种，分别是：Shared Preferences、网络存储、文件存储、外储存储、SQLite。但一般这些存储都只是在单独的一个应用程序之中达到一个数据的共享，有时候我们需要操作其他应用程序的一些数据，就会用到ContentProvider。而且Android为常见的一些数据提供了默认的ContentProvider（包括音频、视频、图片和通讯录等）。

但注意ContentProvider它也只是一个中间人，真正操作的数据源可能是数据库，也可以是文件、xml或网络等其他存储方式。



## Socket

Socket通信方式也是C/S架构，比Binder简单很多。Socket方式更多用于在Framework层与Native层之间的通信，在 Android系统中采用Socket通信方式的主要有：

- **zygote**
  用于浮华进程，system_server 进程创建进程是通过 socket 向 zygote 进程发起请求；
- **instanlld**
  用于安装 app 的守护进程，上层 PackageManagerService 很多实现组中都交由它来完成；
- **lmkd**
  LowMemoryKiller 的守护进程，Java层的 LowMemoryKiller 最终都由 lmkd 来完成；
- **adbd**
  用于服务 adb 的守护进程；
- **logcatd**
  用于服务 logcat 的守护进程；
- **vold**
  Vokume Daemon 是存储类的守护进程，用于负责如 USB、Sdcard 等存储设备的事件处理；

