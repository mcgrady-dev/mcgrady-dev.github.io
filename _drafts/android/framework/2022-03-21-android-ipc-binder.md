## 概述

Binder 是 Android 提供的一种 IPC 机制，整个 Android 系统架构中，大量采用了 Binder 机制作为 IPC 方案，如四大组件。

![java_binder](../../images/android/android_java_binder.png)



## Binder 原理

每个 Android 进程只能运行在自己进程所拥有的的虚拟地址空间。对应一个 4GB 的虚拟地址空间，其中 3GB 是用户空间，1GB是内核空间。对于用户空间，不同进程之间批次是不能共享的，而内核空间却可以共享。而 Binder 恰恰是利用进程间可共享的内核内存空间来完成底层通信工作的，而方法往往是采用 ioctl 等跟内核空间的驱动进行交互。

Binder 使用 C/S 通信方式。Binder 框架定义了四个角色：Server，Client，ServiceManager 以及 Binder 驱动。其中Server，Client，ServiceManager 运行于用户空间，驱动运行于内核空间。Binder 驱动程序提供设备文件`/dev/binder`与用户空间交互，Client、Server 和 Service Manager 通过 `open` 和 `ioctl` 文件操作函数与 Binder 驱动程序进行通信。



## Binder Driver

Binder 驱动的底层架构与 Linux驱动一样。Binder 驱动在以 misc 设备进行注册，作为虚拟字符设配，没有直接操作硬件，只是对设备内存的处理。  

![android_binder_driver](../../images/android/android_binder_driver.png)

### 系统调用

从用户态的程序调用 kernel 层驱动需要陷入内核态，这都依赖于系统调用过程。

![binder_syscall](../../images/android/android_binder_syscall.png)

### 核心方法

- **binder_init**
  初始化字符设备，主要为了注册 misc 设备
- **misc_register**
  注册 misc 设备
- **binder_open**
  打开 binder 驱动设备
- **binder_mmap**
  申请内存空间，实现用户空间的 Buffer 和内核空间的 Buffer 同步操作功能。
  主要实现逻辑：首先在内核虚拟地址空间，申请一块与用户虚拟内存相同大小的内存；然后再申请1个page大小的物理内存，再将统一块物理内存分别映射到内存虚拟地址空间和用户虚拟内存空间。
- **binder_update_page_range**
  分配物理空间，将物理空间映射到内核空间，将物理空间映射到进程空间。另外，不同参数下该方法也可以释放物理页面。
- **binder_alloc_buf**
  `binder_alloc_buf` 是内存分配函数，主要分配 `binder_buf` 结构体
- **binder_ioctl**
  执行相应的 ioctl 操作，负责在两个进程间收发 IPC 数据和 IPC reply 数据
- **binder_get_thread**
  从 `binder_proc` 中查找 `binder_thread`
- **binder_ioctl_write_read**
  1. 首先，把用户空间数据ubuf拷贝到内核空间bwr；
  2. 当bwr写缓存有数据，则执行binder_thread_write；当写失败则将bwr数据写回用户空间并退出；
  3. 当bwr读缓存有数据，则执行binder_thread_read；当读失败则再将bwr数据写回用户空间并退出；
  4. 最后，把内核数据bwr拷贝到用户空间ubuf。

### Binder通信

Client 进程通过 RPC（Remote Procedure Call Protocol）与 Server 通信，可以简单地划分为三层，驱动层、IPC层、应用层。

![binder_ipc](../../images/android/android_binder_ipc.png)



## ServiceManager

ServiceManager 是 Binder IPC 通信过程中的守护进程，注意此处是指 Native 层的 ServiceManager(C/C++)，并非 framework 层的 ServiceManager(Java)。

ServiceManager 本身也是一个 Binder 服务，通过自行编写 `binder.c` 和 Binder 驱动通信，并且通过一个循环的 `binder_loop` 来进行读取和处理事务。



![create_servicemanager](http://gityuan.com/images/binder/create_servicemanager/create_servicemanager.jpg)

ServiceManager 最核心的两个功能：

- 注册服务：记录服务名和 handle 信息，保存到 `svclist` 列表
- 查询服务：根据服务名查询相应的 handle 信息



### ServiceManager 启动过程

1. `binder_open()` 打开 Binder 驱动，并调用 mmap() 分配 128k 内存映射空间
2. `binder_become_context_manager()` 通知 Binder 驱动 使其称为守护进程
3. 验证 `slinux` 权限，判断进程是否有权注册或查看指定服务
4. `binder_loop()` 进入循环状态，等待 Client 端的请求
5. 注册服务的过程，根据服务名称，但同一个服务已注册，重新注册前会先移除之前的注册信息
6. 死亡通知，当 binder 所在进程死亡后，调用 `binder_release()` 和 `binder_node_release()` 这个过程便会发出死亡通知的回调。



## 为什么 Android 要采用 Binder 作为 IPC 机制？

由于 Android 内核基于 Linux，开始前先简单概括下 Linux 现存多种进程间通信方式：

- **管道**
  在创建时分配一个 page 大小的内存，缓存去大小比较有限。
- **消息队列**
  信息复制两次，额外的CPU消耗，不适合频繁或信息量打的通信。
- **共享内存**
  无需复制，共享缓冲区直接附加到进程虚拟地址空间，速度快，但进程间的同步问题操作系统无法实现，必须个进程利用同步工具解决。
- **套接字**
  作为更通用的接口，传输效率低，主要用于不同机器或跨网络的通信。
- **信号量**
  作为一种锁机制，防止某进程正在访问共享资源时，其它进程也访问该资源。因此，主要作为进程间以及统一进程内不同线程之间的**同步手段**。
- **信号**
  不适用与信息交换，更适用于进程中断控制，比如非法内存访问，杀死某个进程等。



采用 Binder 作为 Android  IPC 机制的几个方面

### 性能

Binder 数据拷贝只需要1次，对于管道、消息队列、Socket 都需要2次，共享内存无需拷贝，从性能角度看，Binder 仅次于共享内存。

### 稳定性

Binder 基于 C/S 架构，Server 端与 Client 端相对独立，稳定性较好；而共享内存实现方式复杂，需要葱粉考虑访问临界资源的并发同步问题，否则可能出现死锁问题；从稳定性角度看，Binder 优于 共享内存。

### 安全性

传统 Linux IPC 的接收方无法获得对方进程可靠的 UID/PID，从而无法鉴别对方身份，而 Android 为每个安装的App分配了 UID，故进程的 UID 是鉴别进程身份的重要标准；Android 还对外置暴露 Client 端，Server 端会根据权限控制策略，判断 UID/PID 是否满足访问权限。

传统 Linux IPC 只能由用户在数据包里填入 UID/PID ，另外，可靠的身份标记只有由 IPC 机制本身在内核汇总添加。其次传统 IPC 访问接入点是开放的，无法简历私有通道。从安全角度看，Binder 的安全性更高。

### 语言层面                                                                                                                                                                                                

Android 基于 Java语言（面向对象），而 Binder 也符合面向对象的思想，将进程通信转化为通过对某个 Binder 对象的引用调用该对象的方法，而其独特之处在于 Binder 对象是一个可以跨进程引用的对象，它的实体位于一个进程中，而它的引用却遍布于系统的各个进程之中。可以从一个进程传给其它进程，让大家都能访问同一 Server，就像将一个对象或引用复制给另一个引用一样。Binder 模糊了进程边界，淡化子进程间通信过程，整个系统仿佛运行于同一个面向对象的程序之中。从语言层面，Binder 更适合基于面向对象语言的系统。

### 公司战略

Linux内核是开源的系统，所开放源代码许可协议GPL保护，受GPL保护的Linux Kernel是运行在内核空间，对于上层的任何类库、服务、应用等运行在用户空间，一旦进行SysCall（系统调用），调用到底层Kernel，那么也必须遵循GPL协议。Google巧妙地将GPL协议控制在内核空间，将用户空间的协议采用Apache-2.0协议（允许基于Android的开发商不向社区反馈源码），同时在GPL协议与Apache-2.0之间的Lib库中采用BSD证授权方法，有效隔断了GPL的传染性。

### 为什么 Binder 只进行了一次数据拷贝？


Linux内核实际上没有从一个用户空间到另一个用户空间直接拷贝的函数，需要先用`copy_from_user()`拷贝到内核空间，再用`copy_to_user()`拷贝到另一个用户空间。为了实现用户空间到用户空间的拷贝，`mmap()`分配的内存除了映射进了接收方进程里，还映射进了内核空间。所以调用`copy_from_user()`将数据拷贝进内核空间也相当于拷贝进了接收方的用户空间，这就是Binder只需一次拷贝的‘秘密’。


最底层的是Android的`ashmen(Anonymous shared memory)`机制，它负责辅助实现内存的分配，以及跨进程所需要的内存共享。AIDL(android interface definition language)对Binder的使用进行了封装，可以让开发者方便的进行方法的远程调用，后面会详细介绍。Intent是最高一层的抽象，方便开发者进行常用的跨进程调用。

从英文字面上意思看，Binder具有粘结剂的意思那么它是把什么东西粘接在一起呢？在Android系统的Binder机制中，由一系统组件组成，分别是Client、Server、Service Manager和Binder驱动，其中Client、Server、Service Manager运行在用户空间，Binder驱动程序运行内核空间。Binder就是一种把这四个组件粘合在一起的粘连剂了，其中，核心组件便是Binder驱动程序了，ServiceManager提供了辅助管理的功能，Client和Server正是Binder驱动和ServiceManager提供的基础设施上，进行Client-Server之间的通信。

1. Client、Server和ServiceManager实现在用户空间中，Binder驱动实现在内核空间中
2. Binder驱动程序和ServiceManager在Android平台中已经实现，开发者只需要在用户空间实现自己的Client和Server
3. Binder驱动程序提供设备文件/dev/binder与用户空间交互，Client、Server和ServiceManager通过open和ioctl文件操作函数与Binder驱动程序进行通信
4. Client和Server之间的进程间通信通过Binder驱动程序间接实现
5. ServiceManager是一个守护进程，用来管理Server，并向Client提供查询Server接口的能力

服务器端：一个Binder服务器就是一个Binder类的对象。当创建一个Binder对象后，内部就会开启一个线程，这个线程用户接收binder驱动发送的消息，收到消息后，会执行相关的服务代码。

Binder驱动：当服务端成功创建一个Binder对象后，Binder驱动也会相应创建一个mRemote对象，该对象的类型也是Binder类，客户就可以借助这个mRemote对象来访问远程服务。

客户端：客户端要想访问Binder的远程服务，就必须获取远程服务的Binder对象在binder驱动层对应的binder驱动层对应的mRemote引用。当获取到mRemote对象的引用后，就可以调用相应Binde对象的服务了。

在这里我们可以看到，客户是通过Binder驱动来调用服务端的相关服务。首先，在服务端创建一个Binder对象，接着客户端通过获取Binder驱动中Binder对象的引用来调用服务端的服务。在Binder机制中正是借着Binder驱动将不同进程间的组件bind(粘连)在一起，实现通信。

mmap将一个文件或者其他对象映射进内存。文件被映射进内存。文件被映射到多个页上，如果文件的大小不是所有页的大小之和，最后一个页不被使用的空间将会凋零。munmap执行相反的操作，删除特定地址区域的对象映射。

当使用mmap映射文件到进程后，就可以直接操作这段虚拟地址进行文件的读写等操作，不必再调用read,write等系统调用。但需注意，直接对该段内存写时不会写入超过当前文件大小的内容。

使用共享内存通信的一个显而易见的好处是效率高，因为进程可以直接读写内存，而不需要任何数据的拷贝。对于像管道和消息队列等通信方式，则需要在内核和用户空间进行四次的数据拷贝，而共享内存则只拷贝两次内存数据：一次从输入文件到共享内存区，另一次从共享内存到输出文件。实际上，进程之间在共享内存时，并不总是读写少量数据后就解除映射，有新的通信时，再重新建立共享内存区域，而是保持共享区域，直到通信完成为止，这样，数据内容一直保存在共享内存中，并没有写回文件。共享内存中的内容往往是在解除内存映射时才写回文件的。因此，采用共享内存的通信方式效率是非常高的。





### Binder 事务缓冲区大小

Binder 事务缓冲区具有一个有限的固定大小，当前为 1MB。这里的 1MB 空间并不是当前操作独享的，而是由当前进程所共享的。也就是说 Intent 在 Activity 间传输数据，本身也不适合传输太大的数据。



