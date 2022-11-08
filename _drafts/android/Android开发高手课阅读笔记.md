## 01 如何打造高质量应用

### 应用交付流程

一个应用至少会经过开发、编译 CI、测试、灰度和发布这几个阶段：

1. **开发阶段**
   - TraceView
2. **编译 CI 阶段**
   - d8/ReDex
   - Civerity
   - Infer
3. **测试阶段**
   - 自动化测试
4. **灰度和发布阶段**
   - 动态部署

### 移动 APM 质量平台

APM（Application Performance Manager） 即应用性能管理



## 02 崩溃优化(上) : 关于「崩溃」那些事儿

### Android 的两种崩溃

Android 崩溃分为：

- **Java 崩溃**：简单来说 Java 崩溃是在 Java 代码中，出现了未捕获异常，导致程序异常退出。
- **Native 崩溃**：Native 崩溃则是在 Native 代码中访问非法地址，也可能是地址对齐出现了问题，或者发生了程序主动 `abort`，这些都会产生相应的 `signal` 信号，导致程序异常退出。



### Native 代码的崩溃捕获机制及实现

#### 信号机制

信号机制是进程之间相互传递消息的一种方法，信号全称为软终端信号。

函数运行在用户态，当遇到系统调用、中断或是异常的情况时，程序会进入内核态。信号涉及到了两种状态之间的转换。

![](../../blog/images/android/信号机制.png)





## 02 崩溃优化(下) : 应用崩溃了，你应该如何去分析？

### 崩溃现场

来看看再崩溃现场应该采集哪些信息：

#### 1. 崩溃信息

- **进程名、线程名**
  崩溃的进程是前台还是后台进程，崩溃是不是发生在UI线程。

- **崩溃堆栈和类型**
  崩溃是属于 Java 崩溃、Native 崩溃，还会 ANR，对于不同类型的崩溃我们关注的点也不太一样。特别需要看崩溃堆栈的栈顶，看具体崩溃在系统的代码，还是自己写的代码里。

  ```
  Process Name: 'com.sample.crash'
  Thread Name: 'MyThread'
   
  java.lang.NullPointerException
      at ...TestsActivity.crashInJava(TestsActivity.java:275)
  ```

#### 2. 系统信息

- **Logcat**
  这里包括应用、系统的运行日志。由于系统权限问题，获取到的 Logcat 只能包含与当前 App 相关的。其中系统的 event logcat 会记录 App 运行的一些基本情况，记录在文件 `/system/etc/event-log-tags` 中。

  ```
  system logcat:
  10-25 17:13:47.788 21430 21430 D dalvikvm: Trying to load lib ... 
  event logcat:
  10-25 17:13:47.788 21430 21430 I am_on_resume_called: 生命周期
  10-25 17:13:47.788 21430 21430 I am_low_memory: 系统内存不足
  10-25 17:13:47.788 21430 21430 I am_destroy_activity: 销毁 Activty
  10-25 17:13:47.888 21430 21430 I am_anr: ANR 以及原因
  10-25 17:13:47.888 21430 21430 I am_kill: APP 被杀以及原因
  ```

- **机型、系统、厂商、CPU、ABI、Linux 版本等**

- **设备状态**
  是否 root、是否模拟器。一些问题是由 Xposed 或多开软件造成的，对这部分问题要区别对待。

#### 3. 内存信息

OOM、ANR、虚拟内存耗尽等，很多崩溃都跟内存有直接关系。

- **系统剩余内存**
  当系统可用内存很小（低于 MemTotal 的10%）时，OOM、大量GC、系统频繁自杀拉起等问题较容易出现。可以通过读取 `/proc/meminfo` 文件获取系统内存状态。
- **应用使用内存**
  包括 Java 内存、RSS（Resident Set Size）、PSS（Proportional Set Size），PSS 和 RSS 通过 `/proc/self/smap` 计算，进一步得到例如 apk、dex、so 等更加详细的分类统计。
- **虚拟内存**
  虚拟内存可以通过 /proc/self/status 得到，通过 `/proc/self/maps` 文件可以得到具体的分步情况。

#### 4. 资源信息

有的时候我们会发现应用堆内存和设备内存都非常充足，还是会出现内存分配失败的情况，这跟资源泄漏可能有比较大的关系。

- **文件句柄 id**
  文件句柄的限制可以通过 `/proc/self/limits` 获得，一般单个进程允许打开的最大文件句柄个数为 1024.但如果文件句柄超出 800 个就比较危险，需要将所有的 fd 以及对应的文件名输出到日志中，进一步排查是否出现了又文件或线程的泄露。

  ```
  opened files count 812:
  0 -> /dev/null
  1 -> /dev/log/main4 
  2 -> /dev/binder
  3 -> /data/data/com.crash.sample/files/test.config
  ...
  ```

- **线程数**
  当前线程数大小可以通过上面的 status 文件得到，一个线程可能就占 2MB 的虚拟内存，过多的线程会对虚拟内存和文件句柄带来压力。根据我的经验来说，如果线程数超过 400 个就比较危险。需要将所有的线程 id 以及对应的线程名输出到日志中，进一步排查是否出现了线程相关的问题。

  ```
   threads count 412:               
   1820 com.sample.crashsdk                         
   1844 ReferenceQueueD                                             
   1869 FinalizerDaemon   
   ...  
  ```

- **JNI**
  使用 JNI 时，如果不注意很容易出现引用失效、引用爆表等一些崩溃。我们可以通过 DumpReferenceTables 统计 JNI 的引用表，进一步分析是否出现了 JNI 泄漏等问题。

#### 5. 应用信息

- **崩溃场景**
  崩溃发生在哪个 Activity 和 Fragment，发生在哪个业务中。
- **关键操作路径**
  不同于开发过程的详细打点日志，我们可以记录关键的用户操作路径，这对复现崩溃会有比较大的帮助。
- **其它自定义信息**



### 崩溃分析

#### 第一步 确认和分析重点

- **Java 崩溃**
  Java 崩溃类型比较明显，根据不同的异常执行，比如 OutOfMemeoryError 是资源不足，这个时候需要进一步查看日志中的“内存信息”和“资源信息”。
- **Native 崩溃**
  需要观察 signal、code、fault addr 等内容，以及崩溃时 Java 的堆栈。
- **ANR**
  先看主线程的堆栈，是否因为锁等待导致。接着看 ANR 日志中 iowait、CPU、GC、system server 等信息，进一步确定是 I/O 问题，或是 CPU 竞争问题，还是由于大量 GC 导致卡死。
- **Logcat**
  locat 一般会存在一些有价值的线索，日志级别是 Warning、Error 的需要特别注意。
- **各个资源情况**
  结合崩溃的基本信息，接着看是不是”内存信息“先关，是不是跟 ”资源信息“相关。比如物理内存不足、虚拟内存不足，还是文件句柄 fd 泄露了。

无论是资源文件还是 Logcat，内存与线程相关的信息都需要特别注意，很多崩溃都是由于它们使用不当造成的。



## 02 番外篇: 滴滴出行安卓端 finalize time out 的解决方案

GC `finalize()` 方法超时出现 `java.util.concurrent.TimeoutException` 异常，如 `android.content.AssetManager.finalize() timed out after 10 seconds` 等。

### 原因分析

#### 对象 finalize() 方法耗时较长

当 `finalize()` 方法中有耗时操作时，可能会出现方法执行超时。耗时操作一般有两种情况：

1. 方法内部确实有比较耗时的操作，比如 IO 操作，线程休眠等。
2. 有种线程同步耗时的情况也需要注意：有的对象在执行 `finalize()` 方法时需要线程同步操作，如果长时间拿不到锁，可能会导致超时，如 `android.content.res.AssetManager$AssetInputStream` 类。

#### 5.0版本以下机型 GC 过程中 CPU 休眠导致

开始之前，先了解下 Dalvik GC 的基本工作方式：在 GC 循环汇总，收集器有一个要销毁的对象列表，基本的循环处理的流程可以简述为：

1. 记录开始的时间戳 `starting_timestamp`
2. 从要释放的对象类表汇总把对象移除掉
3. 释放对象 - 有必要的话会调用 `finalize()` 和本地 `destroy()` 方法
4. 记录结束的时间错 `end_timestamp`
5. 计算（end_timestamp - starting_timestamp），并将其与硬编码的超时时间10秒进行比较
6. 如果超时了，抛出 TimeoutException 异常，并杀死进程




所以系统有可能会在执行 `finalize()` 方法时进入休眠， 然后被唤醒恢复运行后，会使用现在的时间戳和执行 `finalize()` 之前的时间戳计算耗时，如果休眠时间比较长，就会出现 `TimeoutException`。

下面通过一个场景来分析：有一个后台运行的进程，在运行过程中，对象被创建、使用并且需要被手机以释放内存，这意味着不时的执行 GC 动作。通常情况下，GC 动作会正常执行完而不会被挂起，但有些时候操作系统会在 GC 运行的过程中进入休眠。

这样的情况下，在设备开始进行GC，记录开始时间戳，然后在系统对象调用 `destroy()` 的过程中进入了休眠。当被唤醒的时候，GC 恢复运行，这时候 `destroy()` 方法将要结束，记录结束时间戳，也就是说这次 GC 话费的时间等于 （`destroy()` 方法执行时长 + 休眠时长）。如果超过10s，就会抛出 TimeoutException 异常。

#### IO 负载过高

许多类的 `finalize()` 都需要释放 IO 资源，当 APP 打开的文件数目过多，或者在多进程或多线程并发读取磁盘的情况下，随着并发数的增加，磁盘 IO 效率将大大下降，导致 `finalize()` 方法中的 IO 操作运行缓慢导致超时。

#### FinalizerDeamon 中线程优先级过低

FinalizerDaemon 中运行的线程是一个守护线程，该线程优先级一般为默认级别 (`nice=0`)，其他高优先级线程获得了更多的 CPU 时间，在一些极端情况下高优先级线程抢占了大部分 CPU 时间，FinalizerDaemon 线程只能在 CPU 空闲时运行，这种情况也可能会导致超时情况的发生。

### 解决方案

- 减少对 `finalize()` 方法的依赖，尽量不依靠 `finalize()` 方法释放资源，手动处理资源释放逻辑。

- 减少 finalizable 对象个数，即减少有 `finalize()` 方法的对象创建，降低 finalizeable 对象 GC 次数。

- `finalize()` 方法内尽量减少耗时以及线程同步时间。

- 减少高优先级线程的创建和使用，降低高优先级线程的 CPU 使用率。

- 手动修改 `finalize()` 方法超时时间

  ```java
  try {
    Class<?> c = Class.forName(“java.lang.Daemons”);
    Field maxField = c.getDeclaredField(“MAX_FINALIZE_NANOS”);
    maxField.setAccessible(true);
    maxField.set(null, Long.MAX_VALUE);
    catch (Exception e) {
      ...
    }
  ```

  这种方案思路是有效的，但是这种方法却是无效的。Daemons 类中 的 MAX_FINALIZE_NANOS 是个  long 型的静态常量，代码中出现的 MAX_FINALIZE_NANOS 字段在编译期就会被编译器替换成常量，因此运行期修改是不起作用的。MAX_FINALIZE_NANOS 默认值是 10s，国内厂商常常会修改这个值，一般有 15s，30s，60s，120s，我们可以推测厂商修改这个值也是为了加大超时的阙值，从而缓解此类 Crash。

- 手动停掉 FinalizerWatchdogDaemon 线程

  ```java
  try {
    Class clazz = Class.forName("java.lang.Daemons$FinalizerWatchdogDaemon");
    Method method = clazz.getSuperclass().getDeclaredMethod("stop");
    method.setAccessible(true);
    Field field = clazz.getDeclaredField("INSTANCE");
    field.setAccessible(true);
    method.invoke(field.get(null));
  } catch (Throwable e) {
    e.printStackTrace();
  }
  ```

  利用反射调用 FinzlizerWatchdogDaemon 的 `stop()` 方法，以使 FinalizerWatchdogDaemon 计时器功能永远停止。当 `finalize()` 方法出现超时，FinalizerWatchdogDaemon 因为已经停止而不会抛出异常。这种方案页存在明显的缺点：

  - 在 Android 5.1 版本以下系统中，当 FinalizerDaemon 正在执行对象的 `finalize()` 方法时，调用 FinalizerWatchdogDaemon 的 `stop()` 方法，将导致 `run()` 方法正常逻辑被打断，错误判断为 `finalize()` 超时，直接抛出 TimeoutException。
  - Android 9.0 版本开始限制 Private API 调用，不能使用反射调用 Daemons 以及 FinalizerWatchdogDaemon 类方法。

### 终极方案

以上的方案都是阻止 FinalizerWatchdogDaemon 的正常运行，避免出现 Crash，从原理上还是具有可行性的：`finalize()` 方法虽然超时，但是当 CPU 资源充裕时，FinalizerDaemon 线程还是可以获得充足的 CPU 时间，从而获得了继续运行的机会，最大可能的延长了 App 存活时间。但这些方案或多或少都有些缺陷，下面我们来介绍终极的方案：

方案的核心就是忽略掉这个 Crash，首先来梳理一下 Crash 的出现过程：

- FinalizerDaemon 执行对象 `finalize()` 超时
- FinalizerWatchdogDaemon 检测到超时后，构造异常交给 Thread 的 `defaultUncaughtExceptionHandler` 调用 `uncaughtException()` 方法处理
- App 停止运行

可以看出，App Crash 时弹出的停止运行对话框以及退出进程操作都是在 UncaughtExceptionHandler 中处理的，那么只要不让这个代码继续执行就可以阻止 App 停止运行了。基于这个思路可以将这个方案表示为：

```java
final Thread.UncaughtExceptionHandler defaultUncaughtExceptionHandler = Thread.getDefaultUncaughtExceptionHandler();
Thread.setDefaultUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler() {
  @Override
  public void uncaughtException(Thread t, Throwable e) {
    if (t.getName().equals("FinalizerWatchdogDaemon") && e instanceof TimeoutException) {
    } else {
      defaultUncaughtExceptionHandler.uncaughtException(t, e);
    }
  }
});
```

### 总结

对于这类问题来说，虽然人为阻止了 Crash，避免了 APP 停止，APP 能够继续运行，但是 `finalize()` 超时还是客观存在的，如果 `finalize()` 一直超时的状况得不到缓解，将会导致 FinalizerDaemon 中 FinalizerReference 队列不断增长，最终出现 OOM 。因此还需要从一点一滴做起，优化代码结构，培养良好的代码习惯，从而彻底解决这个问题。



## 03 番外篇: Android 内存优化杂谈

Android 内存优化是性能优化工作中比较重要的一环，这里的工作主要包括两个方面：

1. **优化 RAM**，即降低运行时内存。这里的目的是防止程序发生 OOM 异常，以及降低程序由于内粗南国大被 LowMemoryKiller 杀死的概率。另一方面，不合理的内存使用会使 GC 大大 增多，从而导致程序变卡。
2. **优化 ROM**，降低程序占用 ROM 的体积。这里主要是为了降低程序占用的空间，防止由于 ROM 空间不足导致程序无法安装。

### 内存泄露的检测和修改

内存泄露，简单来说由于编码错误或系统原因，仍然存在着对其直接或间接的引用，导致系统无法进行回收。内存泄露，容易留下逻辑隐患，同时增加了应用内存峰值与发生 OOM 的概率。

下面试造成内存泄露的一些常见原因，但是如何建立一套发现内存泄露、解决内存泄露的闭环方案，才是我们工作的重点。

- 静态对象：监听器、广播、WebView
- this$0：线程、定时器、Handler
- 系统：TextLine、输入法、音频、Dialog

#### 一. 内存泄露的监控方案

- **leakcanry**
  通过弱引用方式侦查 Activity 或对象的生命周期，若发现内存泄露自动dump Hprof文件，通过 HAHA 库得到泄露的最短路径，最后通过 Notification 展示。

#### 二. 对系统内存泄露的Hack Fix

AndroidExcludedRefs 列出了一些由于系统原因导致引用无法释放的例子，同时对于大多数的例子，都会提供建议如何通过hack的建议去修复。例如：TextLine、InputMethodManager、AudioManger、android.os.Message。

#### 三. 通过兜底回收内存

兜底回收是指对于已泄露的Activity，尝试回收其持有的无法释放的资源，例如：Bitmap、DrawingCache 等，泄露的仅仅是一个 Activity 空壳，从而降低对内存的压力。

方案：在 Activity onDestory 时，从 rootView 开始递归释放所有子view涉及的图片、背景、DrawingCache，监听器等资源，让 Activity 称为一个不占资源的空壳，泄露了也不会导致图片资源被持有。

```java
Drawable d = iv.getDrawable();
if (d != null) {
  d.setCallback(null);
}
iv.setImageDrawable(null);
```



### 降低运行时内存的一些方法

降低运行时内存，更多时候是希望降低应用发生 OOM 的概率。

#### 一. 减少bitmap占用的内存

1. 防止 bitmap 占用资源过多导致 OOM
   - Android 2.x 系统 BitmapFactory.Options 里面隐藏的 inNativeAlloc 反射打开后，申请的 bitmap 就不会算在 external 中。
   - Android 4.x 系统，可采用 facebook的fresco 库，即可把图片资源放于 native 中。
2. 图片按需加载
   即图片的大小不应该超过 view 的大小。图片载入内存之前，我们需要计算出一个合适的 inSampleSize 缩放比例，避免不必要的图片载入。对此，可以重载 Drawable 与 ImageView ，例如在 Activity onDestroy 时，检测图片大小与View的大小，若超过，可以上报或提示。
3. 统一的 bitmap 加载器
   Glide、Picasso、Fresco 或自定义的 ImageLoader。有了统一的bitmap加载器，我们可以在加载bitmap时，若发生OOM(try catch方式)，可以通过清除 cache，降低 bitmap format(ARGB8888/RBG565/ARGB4444/ALPHA8) 等方式，重新尝试。
4. 图片存在像素浪费
   对于 .9png ，美工可能在出图时在拉伸与非拉伸区域都有大量的像素重复。通过获取图片的像素 ARGB 值，计算连续相同的像素区域，自定义算法判定这些区域是否可以缩放。关键也是需要将这些工作做到系统化，可及时发现问题，解决问题。

#### 二. 自身内存占用监控

定期（例如前台每隔3分钟）获取当前整整使用的 dalvik 内存，当值达到危险值时（例如80%），我们应该去释放各种 cache 资源（bitmap 的 cache 为大头），同时显示的去 Trim 应用的 memory，加速内存收集。

```java
Runtime.getRuntime().maxMemory();  
Runtime.getRuntime().totalMemory() - Runtime.getRuntime().freeMemory()；
```

```java
WindowManagerGlobal.getInstance().startTrimMemory(TRIM_MEMORY_COMPLETE);
```

#### 三. 使用多进程

对于 webview、图库等，由于存在内存系统泄露或者占用内存过多的问题，可以考虑采用单独的进程。

#### 四. 上报 OOM 详细信息



### GC 优化

Java拥有GC的机制，不同的系统版本GC的实现可能有比较大的差异。但是无论哪种版本，大量的GC操作则会显著占用帧间隔时间(16ms)。如果在帧间隔时间里面做了过多的GC操作，那么自然其他类似计算，渲染等操作的可用时间就变得少了。

![](../../blog/images/java/large_gc.png)

#### 一. GC的类型

GC 的类型有以下几种，其中 GC_FOR_ALLOC 是同步方式进行，对因公因帧率的影响最大。

1. **GC_FOR_ALLOC**
   当堆内存不够的时候容易被触发，尤其是 new 对象的时候，很容易被触发到，提高 dalvik.vm.heapstartsize 的值，这样在启动过程中可以减少 GC_FOR_ALLOC 的次数。注意这个触发是以同步的方式进行的。如果 GC 后仍然没有空间，则堆进行扩张。

2. **GC_EXPLICIT**
   这个 gc 是可以被调用的，比如`system.gc`， 一般 gc 线程的优先级比较低，所以这个垃圾回收的过程不一定会马上触发， 千万不要认为调用了`system.gc`，内存的情况就能有所好转。

3. **GC_CONCURRENT**
   当分配的对象大小超过**384K**时触发，注意这是以异步的方式进行回收的。如果发现大量反复的 `Concurrent GC` 出现，说明系统中可能一直有大于384K的对象被分配，而这些往往是一些临时对象，被反复触发了。给到我们的暗示是：对象的复用不够。

4. **GC_EXTERNAL_ALLOC**（3.0 系统之后被废弃了）

   Native层的内存分配失败了，这类GC就会被触发。如果GPU的纹理、bitmap、或者 `java.nio.ByteBuffers` 的使用没有释放，这种类型的GC往往会被频繁触发。



#### 二. 内存抖动现象

内存抖动（Memory Churn）是因为在段时间内大量的对象被创建又马上被释放。瞬间产生大量的对象会验证占用内存区域，当达到阈值，剩余空间不够的时候，会触发 GC 从而导致刚产生的对象又很快被回收。几时每次分配的对象占用了很少的内存，但是他们叠加在一起回增加 Heap 的压力，从而触发更多其它类型的 GC。这个操作可能会影响到帧率，并使得用户感知到性能问题。

![](../../blog/images/java/java_memory_churn.png)

通过 Memory Monitor 可以跟踪整个 app 的内存变化情况。若段时间发生了很多次内存的跌涨，这意味着很可能发生了内存抖动。

#### 三. GC 优化

通过 Heap Viewer，我们可以查看当前内存快照，便于对比分析哪些对象有可能发生了泄漏。更重要的工具是 Allocation Tracker，追踪内存对象的类型、堆栈、大小等。

内存抖动一般需要注意的几个方面：

1. 字符串拼接优化
   减少字符串使用**+**号拼接，改为使用 StringBuilder。减少 StringBuilder.enlarge，初始化时设置 capacity；这里需要注意的是，若打开 Looper 中 Printer 回调，也会存在较多的字符串拼接。
2. 文件读取优化
   读文件使用 ByteArrayPool，初始设置 capacity，以减少 expand
3. 资源重用
   建立全局缓存池，对频繁申请、释放的对象类型重用
4. 减少不必要或不合理的对象
   例如在`onDraw()`、`getView()` 中应减少对象申请，尽量重用。更多是一些逻辑上的东西，例如循环中不断申请局部变量等。
5. 选用合理的数据格式
   如使用SparseArray、SparseBooleanArray、LongSparseArray 来代替 HashMap





## 03 内存优化(上) : 4GB内存时代，再谈内存优化

### 内存问题

![java_memory_question](../../blog/images/java/java_memory_question.png)

1. 内存造成的第一个问题是异常，包括 OOM、内存分配失败这些崩溃，也包括因为整体内存不足导致应用被杀死、设备重启等问题。

2. 内存造成的第二个问题是卡顿，Java 内存不足会导致频繁 GC，这个问题在 Dalvik 虚拟机会更加明显。而 ART 虚拟机再内存管理和回收策略上都做了大量优化，内存分配和 GC 效率比提升了 5~10 倍。
   如果想具体测试 GC 性能，例如暂停挂起时间、总耗时、GC 吞吐量，我们可以通过发送 SIGQUIT 信号获得 ANR 日志。另外还可以使用 systrace 来观察 GC 的性能耗时。

3. 除了内存引起的异常和卡顿，在日常做内存优化和架构设计时，容易陷入的两个误区：

   - **误区一：内存占用越少越好**
     应用是否暂用了过多的内存，跟设备、系统和当时情况有关，而不是 300MB、400MB 这样一个绝对的数值。
     应该在系统内存充足时，多用一些获得更好的性能，当内存不足时，希望可以做到“**用时分配，及时释放**”，当系统内存出现压力时，能够迅速释放各种缓存来减少系统压力。
     ![java_memory_leak](../../blog/images/java/java_memory_leak.png)

   - **误区二：Native 内存不用管**

     虽然 Android 8.0 重新将 Bitmap 内存放回到 Native 中，如果 Android 3.0 ~7.0 系统向达到做到相同的效果，可能需要比较复杂的一些设计，例如：Fresco 图片库在 Dalvik 会把图片放到 Native 内存中。不过这个“黑科技”有两个主要问题，一个是兼容性问题，另外一个是频繁申请释放 Java Bitmap 容易导致内存抖动。



### 测量方法

#### Java 内存分配

最常用的通过 Allocation Tracker 和 MAT 工具跟踪 Java 堆内存的使用情况。

还可以自定义 “Allocation Tracker” 来监控 Java 内存，也可以拓展成实时监控 Java 内存泄露。

#### Native 内存分配

AddressSanitize 内存泄露检测只支持 x86_64 Linux / OS X 系统。

关于 Native 内存调试，有两种方法：

- **Malloc 调试**
  可以帮助我们去调试 Native 内存的一些使用问题，例如对破坏、内存泄露、非法地址等。Android 8.0 之后支持在非 root 的设备做 Native 内存调试，不过跟 AddressSanitize 一样，需要通过 [wrap.sh](http://developer.android.com/ndk/guides/wrap-script.html) 做包装。
- **Malloc 钩子**
  出现在 Android P 之后，Android 的 libc 支持拦截在程序执行期间发生的所有「分配/释放」调用，这样就可以构建出自定义的内存检测工具。



## 04 内存优化(下) : 内存优化这件事，应该从哪里着手？

要进行内存优化，通常会从设备分级、Bitmap 优化和内存泄露这三方面入手：

### 设备分级

### 缓存管理

### 进程模型

### 安装包大小

### Bitmap 优化

- 统一图片库
- 统一监控

### 内存泄露

- Java 内存泄露
- OOM 监控
- Native 内存泄露监控
- 针对无法重编 so 的情况
- 针对可重编 so 的情况



## 05 卡顿优化(上) : 你要掌握的卡顿分析方法

### 基础知识

造成卡顿的晕啊因可能有千百种，不过最终都会反映到 CPU 时间上。可以把 CPU 时间分为两种：

- **用户态时间**：指执行用户态应用程序代码所消耗的时间。
- **内核态时间**：指执行内核态系统调用所消耗的时间，包括 I/O、锁、中断以及其它系统调用的时间。

### 卡顿问题分析指标

- **CPU 使用率**：大于 80%处于繁忙状态，普通应用高于30%需进一步检查
- **CPU 饱和度（负载）**：反应线程排队等待 CPU 情况，跟线程数有关，启动线程过多，容易导致系统不断切换执行的线程，把大量的时间浪费在上下文切换（每次 CPU 上下文切换都需要刷新寄存器和计数器，至少需要几十纳秒）；另一个影响 CPU 饱和度的是线程优先级，它会影响系统的调度策略，主要有 nice 和 cgroup 类型共同决定。当 CPU 繁忙时，线程调度会对执行效率有非常大的影响。

### 卡顿排查工具

首先从工具的实现来看，分为两个流派：

- **instrument**：获取一段时间内所有函数的调用过程，通过分析这段时间内的函数调用过程，再进一步分析待优化的点。
- **sample**：有选择性或者采用抽样的方式观察某些函数调用过程，可以通过这些有限的信息推测出流程中的可以点，然后在继续细化分析。

#### 1. Traceview

Traceview 利用 Android Runtime 函数调用的 event 事件，将函数运行的耗时和调用关系写入 trace 文件中。由此可见，Traceview 属于 instrument 类型，它可以用来**查看整个过程有哪些函数调用**，但工具本身带来的性能开销过大，有时无法反映真实的情况。比如一个函数本身的耗时是 1 秒，开启 Traceview 后可能会变成 5 秒，而且这些函数的耗时变化并不是成比例放大。

#### 2. Nanoscope

Uber 开源，在 instrument 类型的性能分析工具里，性能损耗比较小的一类，原理是直接修改 Android 虚拟机源码，在`ArtMethod`执行入口和执行结束位置增加埋点代码，将所有的信息先写到内存，等到 trace 结束后才统一生成结果文件。

Uber 写了一系列自动化脚本协助整个流程，使用起来还算简单。Nanoscope 作为基本没有性能损耗的 instrument 工具，它非常**适合做启动耗时的自动化分析**。

#### 3. systrace

systrace 是 Android 4.1 新增的性能分析工具，通常用来跟踪系统的 I/O 操作、CPU 负载、Surface 渲染、GC 等事件。

systrace 利用 Linux 的 ftrace 调试工具，在系统各个关键位置都添加了一些性能探针，也就是在代码里加了一些性能监控的埋点。

systrace 工具只能监控特定系统调用的耗时情况，所以它是属于 sample 类型，而且性能开销非常低。但是它不支持应用程序代码的耗时分析，所以在使用时有一些局限性。

#### 4. systrace 插桩版本

由于系统预留了`Trace.beginSection`接口来监听应用程序的调用耗时，可以通过**编译时给每个函数插桩**的方式来实现，也就是在重要函数的入口和出口分别增加`Trace.beginSection`和`Trace.endSection`。这样就实现了在 systrace 基础上增加**应用程序耗时**的监控。

#### 5. Simpleperf

Android 5.0 新增了 Simpleperf 性能分析工具，利用 CPU 的性能监控单元（PMU）提供的硬件 perf 事件可以看到所有的 Native 代码的耗时。

#### 6. Android Profiler

随着 Android 版本的演进，在 Android 3.2 的 Profiler 中继承了几种性能分析工具，大大降低了开发者的使用门槛，其中：

- Sample Java Methods 的功能类似于 Traceview 的 sample 类型。
- Trace Java Methods 的功能类似于 Traceview 的 instrument 类型。
- Trace System Calls 的功能类似于 systrace。
- SampleNative（API 26）的功能类似于 Simpleperf。



以上这些工具都支持了 Call Chart 和 Flame Chart 两种展示方式。下面来介绍它们适合的场景：

#### 1. Call Chart

Call Chart 显示了应用内所有函数的调用关系以及运行时间，开发者快速查找某个时间点应用内函数的调用和耗时情况。比如：是否存在线程间的锁、主线程是否存在长时间的 I/O 操作、是否存在空闲等。

#### 2. Flame Chart（火焰图）

Flame Chart 更方便开发者查询哪些函数消耗的时间最多。如果看顶层的哪个函数占据的宽度比较大，就表示该函数可能存在性能问题。



## 06 卡顿优化(补充篇) : 卡顿现场与卡顿分析

### 卡顿现场：线程状态以及堆栈收集分析

- **获取 Java 线程状态**：通过 `Thread.getState()` 获取线程状态。
- **获取所有线程的堆栈**：通过 `Thread.getAllStackTraces()` 获取所有线程的堆栈。需要注意在 Android 7.0 该方法不会返回主线程的堆栈。

#### 卡顿现场：ANR 日志获取

- **读取 traces.txt 文件**：当监控到主线程卡顿时，主动向系统发送 SIGQUIT 信号，等待 `/data/anr/traces.txt` 文件生成，文件生成后进行上报。用 SIGQUIT 信号获取 ANR 日志，存在着几个问题：
  - 可行性：高版本系统已经没有全新啊读取 `/data/anr/traces.txt` 文件。
  - 性能：获取所以线程堆栈以及各种信息非常耗时，对卡顿线程不一定合适，可能会进一步加剧用户的卡顿。
- **Hook 实现**：模拟系统打印 ANR 日志的流程，有可能会造成线上崩溃，为了兼容性考虑，可以通过 fork 子线程方式实现，这样进程崩溃也不会影响主进程运行，且不会卡主线程。
  - 通过 `libart.so`、`dlsym` 调用 `ThreadList::ForEach` 拿到所有 Native 线程对象。
  - 遍历线程对象列表，调用 `Thread::DumpState`。



## 06 卡顿优化(下) : 如何监控应用卡顿？

### 卡顿监控

#### 1. 消息队列

通过替换 Looper 的 Printer 实现

#### 2. 插桩

#### 3. Profilo

实现类似 Native 崩溃捕捉的方式快速获取 Java 堆栈，通过间隔发送 SIGPROF 信号，Signal Handler 捕获到信号后，拿取到当前正在执行的 Thread，通过 Thread 对象可以获取当前线程的 ManagedStack，ManagedStack 是一个单链表，它保存了当前的 ShadowFrame 或者 QuickFrame 栈指针，先依次遍历 ManagedStack 链表，然后遍历其内部的 ShadowFrame 或者 QuickFrame 还原一个可读的调用栈，从而 unwind 出当前的 Java 堆栈。

### 其它监控

#### 1. 帧率

#### 2. 生命周期监控

对于生命周期的监控实现，我们可以利用插件化技术 Hook 的方式。但是 Android P 之后，我还是不太推荐你使用这种方式。

#### 3. 线程监控



## 07 启动优化(上) : 从启动过程看启动速度优化

### 启动优化

#### 1. 优化工具

systrace + 插桩

#### 2. 优化方案

- **闪屏优化**
  把预览窗口实现成闪屏效果，这样用户只需很短的时间就可以看到“预览闪屏”，但对于中低端机，会把总得闪屏时间变得更长。推荐Android 6.0 或 7.0 以上才启用 “预览闪屏”方案。
- **业务梳理**
- **线程优化**
  - 控制线程数量，要有统一的线程池
  - 检查线程间的锁，排查等待事件是否可以优化，特别是防止主线程出现长时间的空转。
  - Pipeline 机制，根据业务优先级规定业务初始化时机。
- **GC优化**
  - 减少 GC 次数，避免造成主线程长时间的卡顿。
  - 启动过程避免进行大量的字符串操作，特别是序列化跟反序列化过程。
- **系统调用优化**
  - 在启动过程，尽量不要做系统调用，例如 PackageManagerService 操作、Binder 调用等待。
  - 启动过程也不要过早地拉起应用的其他进程，System Server 和新的进程都会竞争 CPU 资源，特别是系统内存不足时，有可能会触发系统的 LMK 机制，导致系统杀死和拉起（保活）大量的进程，从而影响前台进程的CPU。



## 08 启动优化(下) : 优化启动速度的进阶方法

### 启动进阶方法

#### 1. I/O优化

#### 2. 数据重排

- **类重排**：启动过程加载顺序可以通过复写 ClassLoader 得到。然后通过 [ReDex](https://github.com/facebook/redex) 的 [Interdex](https://github.com/facebook/redex/blob/master/docs/Interdex.md) 调整类在 Dex 中的排列顺序，最后可以利用 010  Editor 查看修改后的效果。
- **资源文件重排**：通过修改 Kernel 源码，单独编译了一个特殊的 ROM，用来统计应用启动过程加载了安装包中哪些资源文件，比如 assest、drawables、layout 等，可以得到一个资源加载的顺序列表，在完成资源顺序重排后，需要确定是否真正有效，当然通过定制 ROM 的一些买点和配合工具，可以将他们放到自动化流程中。

#### 3. 类的加载

通过 Hook 去掉 verify class 步骤

#### 4. 黑科技

- 保活
- 插件化和热修复
- 应用加固：应用加固对启动速度来说简直是灾难

总的来说，对于黑科技需要谨慎，当足够了解它们内部的机制后，可以选择性的使用。

### 启动监控



## 09 I/O 优化(上) : 开发工程师必备的 I/O 优化知识

### I/O 的基础知识

#### 1. 文件系统

应用程序调用 read() 方法，系统会通过中断从用户空间进入内核处理流程，然后经过虚拟文件系统、具体文件系统、页缓存，下面是 Linux 一个通用的 I/O 框架模型：

![fb11cbe604eb6c0fc2ba5825275f104b](../../blog/images/linux/io.png)

- **虚拟文件系统（VFS）**：它主要用于实现屏蔽具体的文件系统，为应用程序的操作提供一个统一的接口。
- **具体文件系统（File System）**：ext4、F2FS 都是具体文件系统实现，文件元数据如何组织、目录和索引结构如何设计、怎么分配和清理数据，简单来说就是**存储和组织数据的方式**，这些都是设计一个文件系统必须要考虑的。
- **页缓存（Page Cache）**：
  - Page Cache：是**文件系统对数据的缓存**，目的是提升内存命中率。
    在读取文件时，先看它是不是已经在 Page Cache 中，如果命中就不会去读取磁盘。
  - Buffer Cache：是**磁盘对数据的缓存**，目的是合并部分文件系统的 I/O 请求、降低磁盘 I/O 的次数。
    在 Linux 2.4.10 之前还有一个单独的 Buffer Cache，后来合并到了 Page Cache 中的 Buffer Page 了。

#### 2. 磁盘

磁盘指的是系统的存储设备，比如 CD、机械硬盘、SSD固态硬盘。应用程序向磁盘发起 I/O 请求的过程要先经过内核的通用块层、I/O 调度层、设备驱动层，最后才会交给具体的硬件设备处理。

![13c06810c88632db1050ab3e56139a18](../../blog/images/linux/disk.png)

- **通用块层**：通用块层主要的作用是接收上层发出的磁盘请求，并最终发出I/O请求，让上层不需要关心底层硬件设备的具体实现。
- **I/O调度层**：I/O 调度层会根据设置的调度算法对请求合并和排序，避免收到磁盘请求就立刻交给驱动层处理，从而降低真正的磁盘 I/O，因为磁盘 I/O 相对来说是比较慢的。
- **块设备驱动层**：块设备驱动层根据具体的物理设备，选择对应的驱动程序通过操控硬件设备完成最终的 I/O 请求。例如光盘是靠激光在表面烧录存储、闪存是靠电子擦写存储数据。



### Android 闪存

手机使用闪存作为存储设备，也就是我们常说的 ROM。

#### 疑问：I/O 有时候为什么会突然很慢？

在一些低端机上面，发生 I/O 卡顿，突然变慢可能有以下几个原因：

- **内存不足**：当手机内存不足时，系统会回收 Page Cache 和 Buffer Cache 的内存，大部分的写操作会直接落盘，导致性能低下。
- **写入放大**：闪存重复写入需要进行擦除操作，但这个擦除操作的基本单元是 block 块，一个 page 页的写入操作将会引起整个块数据的迁移，这就是典型的写入放大现象。低端机或使用较久的设备，由于磁盘碎片多、剩余空间少，非常容易出现写入放大现象。
- **由于低端的 CPU 和闪存性能相对较差**：



### I/O 性能评估

#### 1. I/O 性能指标

I/O 性能评估汇总最为核心的指标是：

- **吞吐量 (Throughput)**：即连续读写速度
-  **IOPS (Input/Outpu Per Second)**：指的是每秒可以读写的次数，对于随机读写频繁的应用，例如大量的小文件存储。

#### 2. I/O 测量

- proc：
  ```
  proc/self/schedstat:
  se.statistics.iowait_count：IO 等待的次数
  se.statistics.iowait_sum：  IO 等待的时间
  ```

- strace
  ```
  strace -ttT -f -p [pid]
  
  read(53, "*****************"\.\.\., 1024) = 1024       <0.000447>
  read(53, "*****************"\.\.\., 1024) = 1024       <0.000084>
  read(53, "*****************"\.\.\., 1024) = 1024       <0.000059>
  ```

  必须是 root 的机器

- vmstat
  ```
  //清除Buffer和Cache内存缓存
  echo 3 > /proc/sys/vm/drop_caches
  //每隔1秒输出1组vmstat数据
  vmstat 1
  
  //测试写入速度，写入文件/data/data/test，buffer大小为4K，次数为1000次
  dd if=/dev/zero of=/data/data/test bs=4k count=1000
  ```



## 10 I/O优化(中) : 不同I/O方式的使用场景是什么？

### I/O 的三种方式

![three_ways_of_io](../../blog/linux/three_ways_of_io.png)

- **标准 I/O**：我们应用程序平时用到 read/write 操作都属于标准 I/O，也叫缓存 I/O。它的关键特性有：

  - 对于读操作，当读取的块数据已存在页缓存中时，可以立即返回给应用程序，而不需要经过实际的物理读盘操作。
  - 对于写操作，数据首先写到缓存中去，数据是否呗立即写入磁盘取决于应用程序所采用的写操作的机制。默认采用延迟写机制，系统会定期将页缓存中的数据刷到磁盘上。

- **mmap**：通过把文件映射到进程的地址空间，可以带来的好处有：

  - 减少系统调用：只需要一次 mmap() 调用，后续所有的调用像操作内存一样，绕过文件系统的大量的 read/write 调用。
  - 减少数据拷贝：普通的 read() 数据需要经过两次拷贝，而 mmap() 只需要从磁盘拷贝一次就可以，并且由于做过内存映射，也不需要再拷贝回用户空间。
  - 可靠性高：数据写入页缓存后，依靠内核线程定期写回磁盘。

  从上面看 mmap 在调用过程上由于标准 I/O，为什么不全用 mmap 呢？事实上它也存在着一些缺点：

  - 虚拟内存增大：mmap 会导致虚拟内存增大，32位的设备上，一版可以使用的虚拟内存空间只有 3GB 左右，假设 mmap 一个 1GB 的文件，很容易出现虚拟内存不足导致的 OOM。
  - 磁盘延迟：mmap 通过缺页中断向磁盘发起真正的磁盘 I/O，所以如果我们当前的问题是在于磁盘 I/O 的高延迟，那么用 mmap() 消除小小的系统调用开销是杯水车薪的。

  mmap 使用案例：

  - APK、Dex、so 都是通过 mmap 读取
  - 微信的 mars 框架的 xlog 也是采用 mmap 来保证性能和可靠性
  - Android Binder 机制，内部也是使用 mmap 实现

- **直接 I/O**：直接 I/O 访问方式减少了一次数据拷贝和一些系统调用的耗时，很大程度降低了 CPU 的使用率以及内存的占用。不过有时也会产生负面影响：
  - 对于读操作，会造成磁盘的同步读，导致进程需要较长的时间才能执行完。
  - 对于写操作，与磁盘需要同步执行，需要较长时间。

### 多线程阻塞 I/O 和 NIO

#### 1. 多线程阻塞 I/O

#### 2. NIO（Non-Blocking I/O）

非阻塞的 NIO 将 I/O 以事件的方式通知（I/O复用模型），最大化提升应用整体的 CPU 使用率，可以减少线程切换的开销。但 NIO 的缺点也非常明显，应用程序的实现会变得更复杂。



## 11 I/O优化 (下) : 如何监控线上I/O操作？

### I/O 跟踪

- Java Hook
- Native Hook

### I/O 线上监控

- **主线程I/O**
- **读写 Buffer 过小**
  读写单位，对于文件系统是以 block 为单位读写，对于磁盘是以 page 为单位读写。
- **重复读**
  虽然重复读时数据可以都来自 Page Cache，不会发生真正的磁盘操作，但是它依然需要消耗系统调用和内存拷贝的时间，而且 Page Cache 的内存也很有可能被替换或释放。
- **资源泄露**

### I/O 与启动优化

- 对大文件使用 mmap 或 NIO 方式
  - MappedByteBuffer 就是 Java NIO 中的 mmap 封装
- 安装包不压缩：对启动过程需要的文件，可以指定在安装包中不压缩，这样可以加快启动速度，但带来的影响是apk体积增大。
- Buffer 复用：
  - Okio，内部的 ByteString 和 Buffer 通过重用等急求，很大程度上减少 CPU 和内存的消耗
- 存储结构和算法的优化



## 12 存储优化 (上) : 常见的数据存储方式有哪些？

### Android 的存储基础

#### 1. Android 分区

分区简单来说就是将设备中的存储划分为一些互不重叠的部分，每个部分都可以单独格式化，用作不同的目的。且不同的分区可以使用不同的文件系统。其中比较重要的有：

- **/system** 分区：存放 Google 提供的 Android 组件的地方。这个分区只能以只读方式 mount，保证 `/system` 分区的内容不会受到破坏和篡改。
- **/data** 分区：所有用户数据存放的地方。主要为了实现数据隔离，即系统升级和恢复的时候会擦除整个 `/system` 分区，但是却不会影响 `/data` 的用户数据。而恢复出厂设置，只会擦除 /data 的数据。
- /**vendor** 分区：存放厂商特殊系统修改的地方。特别是在 Android 8.0 以后，隆重推出了[Treble](https://source.android.com/devices/architecture)项目。厂商 OTA 时可以只更新自己的 `/vendor` 分区即可，让厂商能够以更低的成本，更轻松、更快速地将设备更新到新版 Android 系统。
- **/cache** 分区：系统升级过程使用的分区或者recovery
- **/storge** 分区：外置或者内置sdcard

#### 2. Android 存储安全

- **权限控制**：在 Android 4.3 之前的版本中，这些沙盒使用了标准 Linux 的保护机制，通过为每个应用创建独一无二的 Linux UID 来定义。每个应用都在自己的应用沙盒内运行，保证应用内数据访问的权限。
  在 Android 4.3 引入了SELinux（Security Enhanced Linux）机制进一步定义 Android 应用沙盒的边界。作用是即使进程有 root 权限也不能为所欲为，想在 SELinux 系统中干任何事情，都必须在专门的安全策略文件中赋予权限。
- **数据加密**：Android 有两种**设备加密**方法：
  - 全盘加密：Android 4.4 中引入的，并在 Android 5.0 中默认打开。它会将 `/data` 分区的用户数据操作加/解密，对性能会有一定的影响，但是新版本的芯片都会在硬件中提供直接支持。
  - 文件级加密：基于文件的加密模式，会给文件都分配一个必须用用户 passcode 推导的密钥，特定的文件被屏幕锁屏之后，知道用户下一次解锁屏幕期间都不能访问。



### 常见的数据存储方法

如何在合适的场景选择合适的存储方法是存储优化的关键，应该学会通过六大关键要素分解某个存储方法：

- **正确性**：存储方案设计是否完备；有没支持多线程或跨进程同步操作；有没考虑异常情况下数据的校验和恢复，比如采用双写或备份文件策略，即使文件因为系统底层导致损坏，也可以一定程度上恢复大部分数据。
- **时间开销**：CPU时间、I/O时间
- **空间开销**：相同的数据如果使用不同的编码方式，最后占用的存储空间也会有所不同；除了编码方式的差异，在一些场景可能还需要引入压缩策略来进一步减少存储空间；数据存储的空间开销还需要考虑内存空间的占用量，整个存储过程会不会导致出现大量GC、OOM等。
- **安全**：应用中的一些敏感数据，即使存储在 `/data/data` 中，依然必须将它们加密，例如微信的聊天数据是存储在加密的数据库中，一些账号相关的数据也要单独加密落地。根据加密强度不同，可以选择 RSA、AES、chacha20、TEA 等方式。
- **开发成本**：开发过程越简单越好，替换成本越低越好
- **兼容性**：兼容性首先要考虑向前、向后的兼容性，老数据再升级时能否迁移过来，新数据再老版本能否降级使用，另外一个需要考虑的可能是多语言的问题。



#### SharedPreferences

SharedPreferences 作为 Android 中用来存储一些比较小的键值对集合，虽然使用非常简便，但性能问题比较多：

- **跨进程不安全**：由于没有使用跨进程的锁，就算使用MODE_MULTI_PROCESS，在跨进程频繁读写有可能导致数据全部丢失。
- **加载缓慢**：文件的加载使用了异步线程，而且加载线程并没有设置线程优先级，如果这个时候主线程读取数据就需要等待文件加载线程的结束。这就导致出现主线程等待低优先级线程锁的问题，比如一个 100KB 的 SP 文件读取等待时间大约需要 50~100ms，我建议提前用异步线程预加载启动过程用到的 SP 文件。
- **全量写入**：无论是调用 commit() 还是 apply()，即使我们只改动其中的一个条目，都会把整个内容全部写到文件。而且即使我们多次写入同一个文件，SP 也没有将多次修改合并为一次，这也是性能差的重要原因之一。
- **卡顿**：由于提供了异步落盘的 apply 机制，在崩溃或者其他一些异常情况可能会导致数据丢失。所以当应用收到系统广播，或者被调用 onPause 等一些时机，系统会强制把所有的 SharedPreferences 对象数据落地到磁盘。如果没有落地完成，这时候主线程会被一直阻塞。这样非常容易造成卡顿，甚至是 ANR，从线上数据来看 SP 卡顿占比一般会超过 5%。



#### ContentProvider

ContentProvider 提供了不同进程甚至不同应用程序之间共享数据的机制。在使用过程有以下几点需要注意：

- **启动性能**：ContentProvider 的生命周期默认在 `Application onCreate()` 之前，ContentProvider 初始化工作尽量不要做耗时操作，会拖慢启动速度。
- **稳定性**： 批量插入很多数据，有可能出现数据超大异常
- **安全性**：ContentProvider 是 exported，当支持执行 SQL 语句时就需要注意 SQL 注入的问题。另外如果我们传入的参数是一个文件路径，然后返回文件的内容，这个时候也要校验合法性，不然整个应用的私有数据都有可能被别人拿到，在 intent 传递参数的时候可能经常会犯这个错误。



### 知识点

- Android Binder 传输大小限制一般是 1~2MB
- 匿名共享内存机制：通过 Binder 传递 CursorWindow 对象内部的匿名共享内存的文件描述符，在跨进程中，结果数据并不需要跨进程传输，而是在不同进程中通过传输的匿名共享内存文件描述符来操作同一块匿名内存，这样来实现不同进程访问相同数据的目的。



## 13 存储优化 (中) : 如何优化数据存储？

宽泛的来讲，数据存储不限于将数据存放到磁盘中，比如放到内存、通过网络传输也算是存储的一种形式。或者也可以把这个过程叫作对象或数据的序列化。

### 对象的序列化

对象序列化是把一个 Object 对象所有的信息表示成一个字节序列，包括 Class 信息、继承关系信息、访问权限、变量类型以及数值信息等。

#### 1. Serializable

Serializable 原理是通过 ObjectInputSteam 和 ObjectOutputStream 来实现，整个过程使用了大量的反射和临时变量，而且在序列化对象的时候，不仅会序列化当前对象本身，还要递归序列化对象引用的其他对象。

整个过程计算非常复杂，而且因为存在大量反射和GC的影响，序列化的性能会比较差。另外一方面，序列化文件需要包含非常多的信息，导致序列化后的大小比 Class 本身要大很多，可能会导致 I/O 读写上的性能问题。

**Serializable 注意事项**

- 不被序列化的字段：static 变量、声明为 transient 的字段
- serialVersionUID：建议显式声明
- 构造方法：反序列化默认不会执行构造函数，可以通过进阶方法做自定义反序列化修改



#### 2. Parcelable

Parcelable 可以理解为系统实现的一套以Parcel为基础实现的序列化方案。Parcelable 只会在内存中进行序列化操作，并不会将数据存储到磁盘里。

**Parcelable 注意事项**

- 系统版本的兼容性：Parcel.cpp 在不同版本实现有所差异，且厂商也可二次修改，可能存在兼容问题。
- 数据前后兼容性：Parcelable 没有版本管理的设计，如果类的版本出现升级，写入的顺序及字段类型的兼容都需要格外注意，同时也带来很大的维护成本。



#### 3. Serial

原理上 Serial 像是把 Parcelable 和 Serializable 的优点集合在一起的方案：

- 不使用反射
- 序列化过程可控
- 序列化过程可 debug
- 版本管理



### 数据的序列化

对象的序列化在操作比较频繁时，对应用的影响还是不小，这个时候可以选择使用数据的序列化。

#### 1. JSON

JSON 是一种轻量级的数据交互格式，它被广泛使用在网络传输中，很多应用与服务端的通信都是使用 JSON 格式进行交互。JSON 的优势有：

- 相比对象序列化方案，速度更快，体积更小
- 相比二进制序列化方案，结果可读，易于排查问题
- 使用方便，支持快平台、跨语言、支持嵌套引用

#### 2. Protocol Buffers

Protocol Buffers 适用于数据量非常大，或者对性能有更高的要求，主要有以下几个优缺点：

- 性能：使用二进制编码压缩，相比 JSON 体积更小，编解码速度也更快
- 兼容性：跨语言和前后兼容性都不错，也支持基本类型的自动转换，但是不支持继承与引用类型。
- 使用成本：Protocol Buffers 的开发成本很高，需要定义.proto 文件，并用工具生成对应的辅助类



### 存储监控

#### 1. 性能监控

- 正确性
- 时间开销
- 空间开销

#### 2. ROM 监控

ROM 监控的两个核心：

- 文件总大小
- 总文件数



### 知识点

- 文件遍历在 API 26 之后建议使用 FileVisitor 代替 ListFiles，性能会好很多



## 14 | 存储优化（下）：数据库SQLite的使用和优化

### SQLite 优化

#### 1. ORM

ORM（Object Relational Mapping）对象关系映射，用面向对象的概念把数据库中表和对象关联起来，可以让我们不关心数据库底层的实现。

#### 2. 进程与线程并发

- 多进程并发：SQLite 默认是支持多进程并发操作，它通过文件锁来控制多进程的并发。仅针对整个 DB 文件。
- 多线程并发：SQLite 支持多线程并发模式，系统默认开启（Multi-thread）

两种并发都是采用锁机制，锁的力度是 DB 文件级别，并没有实现表级或行级的锁。

优化实践：

- **Busy Retry 方案**：即发生阻塞时会触发 Busy Handler， 可以让线程休眠一段时间后重新尝试操作。
- **WAL（Write-Ahead-Logging）模式**：WAL 模式会将修改的数据单独写到一个 WAL 文件汇总，同时也会引入 WAL 日志文件锁，通过 WAL 模式读和写可以完全地并发执行，不会互相阻塞。可以进一步提高并发性能。

需要注意的是，**写之间仍然不能并发**。否则会抛出 SQLiteDatabaseLockedException 异常，但可以通过捕捉异常尝试一段时间后重试。

#### 3. 查询优化

- 索引优化
- 页大小与缓存大小
- 其他优化
  - 慎用`select*`，需要用多少列就取多少列
  - 正确使用事务
  - 预编译与参数绑定，缓存被编译后的 SQL 语句
  - 对于 blob 或超大的 Text 列，可能会超出一个页的大小，导致出现超大页。建议将这些列单独拆表，或者放到表字段的后面
  - 定期整理或者清理无用或可删除的数据，如果用户访问到这些数据，从网络重新拉取即可

### SQLite 的其他特性

#### 1. 损坏与恢复

#### 2. 加密与安全

数据库的安全主要有两个方面：

- 防注入：防注入可以通过静态安全扫描的方式
- 加密：一般会使用 SQLCipher 支持



## 15 | 网络优化（上）：移动开发工程师必备的网络优化知识



### 网络基础知识

- 千兆级 LTE：指蜂窝网络在理论上的速度可以达到光纤级别的 1Gbps（125MB/s）
- 802.11ac 无线网络：指使用 802.11ac标准的 WiFi，它的理想速率可以达到 866.7Mbps。WiFi 由 IEEE 定义和进行标准化规范
- 所有的无线网络都通过基带芯片支持，目前高通在基带芯片领域占据了比较大的优势
- Link Turbo：网络聚合加速技术，在使用 WiFi 的同时使用移动网络加速

### 网络性能评估

- 延迟：数据从信息源发送到目的地所需要的时间
  - 系统调用发送/接收延时
  - 连接延时
  - 首包延时
  - 网络往返时间
- 带宽：逻辑与物理通信路径最大的吞吐量
- 吞吐量：网络接口接收的传输的每秒字节数
- 连接数：每秒的连接数
- 错误：丢包计数、超时等



## 16 | 网络优化（中）：复杂多变的移动网络该如何优化？

### 网络优化

移动端网络优化的三个核心点：

- 速度：网络正常或良好时，如何更好的利用带宽，进一步提升网络请求速度
- 弱网络：移动端网络复杂多变，网络不稳定时，如何最大程度保证网络的连通性。
- 安全：如何防止被第三方劫持、窃听甚至篡改

### 网络请求过程

![8d6e833f2a6c6d1e8beeb92d4579c38b](../../blog/images/network/netowrk_request_process.png)

- **DNS 解析**：通过 DNS 服务器，拿到对应的域名的 IP 地址，这里比较关注是解析的耗时情况、运营商 LocalDNS 的劫持、DNS 调度等问题。
- **创建连接**：包括 TCP 三次握手、TLS 密匙协商等工作。
- **发送/接收数据**：如何根据网络情况将带宽利用好；怎么快速侦测网络延时；弱网下如何调整包大小。
- **关闭连接**：主动关闭、被动关闭

### 大网络平台

#### 1. HTTPDNS

首先 DNS 解析是网络请求的第一项工作，使用的是 UDP 无状态协议，容易出现：域名劫持、调度不准确、延时等问题。为了解决这些问题，就有了 HTTPDNS，简单来说就是自己做域名解析，通过 HTTP 请求后台去拿到域名对应的 IP 地址。

#### 2. 连接复用

创建连接需要经过 TCP 三次握手、TLS 密钥协商，建立连接的代价是非常大的，这里主要的优化思路是：复用连接，不用每次都重新建立连接。

这里可以利用 HTTP 协议里的 keep-alive ，而 HTTP/2.0 的多路复用则可以进一步提升连接复用率，它复用的这条连接支持同时处理多条请求，所有请求都可以并发在这条连接上进行。



![8215799e2bb66c6668e9b73e4130f0ac](../../blog/images/network/http2_multiplexing.png)



#### 3. 压缩与加密

**压缩**

首先来看看 HTTP 请求数据的主要部分：

- Request URL：对于请求URL来说，一般会带些公共参数，这些参数大部分都是不变的，这样的参数客户端只需要上传一次即可，其他请求我们可以在接入层进行参数扩展。
- Request header：HTTP/2.0 连接本身使用了头部压缩技术
- Request body：对于请求body来说，分为两个方面：
  - 数据通信协议的选择：JSON 和 Protocol Buffers
  - 压缩算法的选择：gzip、Brotli、Z-standard
  - 当然针对特定数据还有其他的压缩方法，例如图片可以使用：webp、hevc、SharpP 等

**安全**

HTTPS 的优化有以下几个思路：

- 连接复用率：通过多个域名共用一个 HTTP/2 连接、长链接等方式
- 减少握手次数：TLS 1.3 可以实现 0-RTT 协商
- 性能提升：使用 ecc 证书代替 RSA，服务器签名性能可以提升 4~10 倍，但客户端校验性能降低了 20 倍，另一方面可以通过 Session Ticket 回话复用，节省一个 RTT 耗时。
- 证书锁定（Certificate Pinning）：客户端设置代理，TLS 加密的数据可以被解开并可能被利用，可以将证书锁定，为了前后兼容性和证书灵活性，建议锁定根证书

#### 4. 其他优化

- 部署跨国专线
- 加速点
- 多 IDC 就近接入
- CDN 服务
- P2P 技术



### QUIC 与 IPv6

#### QUIC

QUIC（Quick UDP Internet Connection）是谷歌制定的一种基于UDP的低时延的互联网传输层协议。QUIC 融合了 HTTP/2.0、TLS 1.3、UDP等协议的特性。

在 2018 年基于 QUIC 协议的 HTTP 更被确认为 HTTP/3。

![7d97cb60db49e280ac346b85bf943bb1](../../blog/images/network/http3.png)

HTTP/3 带来的优势：

- 灵活控制拥塞协议：内部的拥塞控制算法等模块进行优化和升级
- “真”连接复用：不仅解决了队首阻塞的问题，在客户端网络切换的时候也不需要重连
- 创建连接成功率：
- 运营商支持：

#### IPv6

推行 IPv6 后，无穷无尽的 IP 地址意味着可以告别各种 NAT，P2P、QUIC 的连接也不再是问题。



## 17 | 网络优化（下）：大数据下网络该如何监控？

### 监控网络

- 插桩
  - ArgusAPM
  - TraceNetTrafficMonitor
  - OkHttp3Aspect
- Native Hook
  - 连接相关：connect
  - 发送数据相关：send 和 sendto
  - 接收数据相关：recv 和 recvfrom

- 统一网络库（跨平台）
  - 延时监控
  - 纬度监控
  - 错误监控
  - 流量监控
  - 代理WebView网络请求


### 监控流量

- TrafficStats
- Facebook：network-connection-class



## 18 | 耗电优化（上）：从电量优化的演进看耗电分析

`power_profiler.xml`文件定义了不同模块的电流消耗值以及该模块在一段时间内大概消耗的电量，Android 系统的电量计算PowerProfile也是通过读取`power_profile.xml`的数值。

BatteryStatsService是对外的电量统计服务，但具体的统计工作是由BatteryStatsImpl来完成的，而 BatteryStatsImpl 内部使用的就是 PowerProfile。

### Android 耗电的演进历程

1. **野蛮生长 Android 5.0**：在 Android 5.0 之前，系统并不是那么完善，对于电量优化相对还是比较少的。特别没有对应用的后台做严格的限制，多进程、fork native 进程以及广播拉起等各种保活流行了起来。

2. **逐步收紧：Android 5.0～Android 8.0**

   ![](../../blog/images/android/android_volta_project.webp)

3. **最严限制：Android 9.0**：对电源管理引入了更加严格的限制
   ![13697353748c1637643a6970db22808e](../../blog/images/android/android_power_manager.png)



## 19 | 耗电优化（下）：耗电的优化方法与线上监控

### 电量监控方案与规则

- Alarm Manager wakeup 唤醒过多
- 频繁使用局部唤醒锁
- 后台网络使用量过高
- 后台 WiFi scans 过多

### 耗电监控

- Android Vitals
  - 后台耗电监控
  - 现场信息
  - 提炼规则
    ![d48b7e4d3fdceb101fa7716b5892b0be](../../blog/images/android/android_power_consumption_rules.png)



## 20 | UI 优化（上）：UI 渲染的几个关键概念

### UI 渲染的背景知识

#### 2. CPU 与 GPU

UI 组件在绘制到屏幕前，都需要经过 Rasterization（栅格化）操作，而栅格化操作又是一个非常耗时的操作。GPU 在这一环节发挥了它的作用，主要用于处理图形运算，加快栅格化操作。

#### 3. OpenGL 与 Vulkan



## 21 | UI 优化（下）：如何优化 UI 渲染？

UI 优化要解决的核心是由于渲染性能本身造成用户感知的卡顿，可以认为它是卡顿优化的一个子集。

### UI 渲染测量

- **测试工具**：Profile GPU Rendering 和 Show GPU Overdraw
- **问题定位工具**：Systrace 和 Tracer for OpenGL ES，在 AndroidStudio 3.1 后，推荐用Graphics API Debugger 代替 Tracer for OpenGL ES。
- **自动化测量工具**：
  - gfxinfo：可以输出包含各阶段发生的动画以及帧相关的性能信息，除了渲染的性能之外，gfxinfo 还可以拿到渲染相关的内存和 View hierarchy 信息。
  - SurfaceFlinger：查看 Graphic Buffer 占用的内存

### UI 优化的常用手段

- **尽量使用硬件加速**：SVG 也是一个非常典型的例子，SVG 有很多指令硬件加速都不支持，但可以提前将这些 SVG 转成 Bitmap 缓存起来，这样系统可以更好的使用硬件加速。同理，对于其他圆角、渐变等场景，也可以改为 Bitmap 实现。
- **Create View 优化**：View 的创建是在 UI线程里完成，包括各种 XML 的随机读的 I/O 时间、解析 XML 的时间、生成对象的时间（Framework 大量使用到反射）。
  - 通过 X2C 将 XML 转换为 Java代码
  - 异步创建，在使用线程创建UI时，通过反射把线程 Looper 的 MessageQueue 替换成 UI 线程 Looper 的 MessageQueue。注意：创建完成后需恢复原来的 MessageQueue。
  - View 重用：参考 RecyclerView 思想，在 不同的 Activity 和 Fragment 使用 View 缓存机制。
- **measure / layout 优化**：
  - 减少 UI 布局层次，尽量扁平化，使用 <ViewStub> <merge> 等优化
  - 优化 layout 开销：推荐 LinearLaout、ConstraintLayout
  - 背景优化：去除重复背景设置
  - 异步 measure 和 layout：PrecomputedText

### UI 优化的进阶手段

- **Litho**：异步布局
  - 异步 measure 和 layout
  - 界面扁平化
  - 优化 RecyclerView 的缓存和回收方法，原生基于 viewType 进行缓存和回收，而 Litho 基于 text、image、video 独立回收
  - 使用单独组件优化，RecyclerViewCollectionComponent 和 Sections 优化 RecyclerView 性能
- **Flutter**
- **RenderThread**：使用 RenderThread 实现动画的异步渲染
- **RenderScript**：提高图像处理



## 22 | 包体积优化（上）：如何减少安装包大小？

![apk_file_module](../../blog/images/android/apk_file_module.png)

### 包体积对应用性能的影响

- **安装时间**：文件拷贝、Library 解压、编译 ODEX、签名校验
- **运行内存**：Resource 资源、Library 以及 Dex 类加载这些都会占用不少的内存
- **ROM 空间**：例如 100MB 的安装包，启动解压之后很有可能就超过 200MB 了。如果闪存空间不足，非常容易出现写入放大的情况。

### 包体积优化

- ProGuard
  - 可以把**非 exported**的四大组件以及 View 混淆，同时进行 XML 和代码 的替换
  - 使用新的 Dex 编译器 D8 与新混淆工具 R8
- Dex 分包
  - ReDex
- Dex 压缩
- Library 压缩

### 包体积监控

- 大小监控：每个版本跟上一个版本包体积的对比情况。如果某个版本体积增长过大，需要分析具体原因，是否有优化空间。
- 依赖监控：包括新增 JAR 以及 AAR 依赖
- 规则监控：是将包体积的监控抽象为规则，例如无用资源、大文件、重复文件、R 文件等。例如 ApkChecker



## 23 | 包体积优化（下）：资源优化的进阶实践

### AndroidResGuard

- 资源混淆
  - Shrink 裁剪
  - Optimize 优化
  - Obfuscate 混淆
- 极限压缩
  - 利用 7-Zip 的大字典优化，提高压缩率
  - 压缩更多的文件
    - resources.arsc
    - PNG、JPG、GIF

### 进阶的优化方法

- 资源合并
- 无用资源

### 无用资源优化方案的演进

- **第一阶段：Lint 静态代码扫描工具**，可以轻松删除所有的无用资源，它最大的问题在于没有考虑到 ProGuard 的代码裁剪。在 ProGuard 过程我们会 shrink 掉大量的无用代码，但是 Lint 工具并不能检查出这些无用代码所引用的无用资源。
- **第二阶段：shrinkResources 资源压缩功能**，如果 ProGuard 把部分无用代码移除，这些代码所引用的资源也会被标记为无用资源，然后通过资源压缩功能将它们移除。但是目前的 shrinkResources 实现起来还有几个缺陷：
  - 没有处理 resources.arsc 文件，导致大量无用的 String、ID、Attr、Dimen 等资源并没有被删除。
  - **没有真正删除资源文件**。对于 Drawable、Layout 这些无用资源，shrinkResources 也没有真正把它们删掉，而是仅仅替换为一个空文件。为什么不能删除呢？主要还是因为 resources.arsc 里面还有这些文件的路径。
- **第三阶段：realShrinkResources**，怎么样才能真正实现无用资源的删除功能呢？ResourceUsageAnalyzer 的注释中就提供了一个思路，我们可以利用 resources.arsc 中 Public ID 的机制，实现非连续的资源 ID。



## 27 | 编译插桩的三种方法：AspectJ、ASM、ReDex

编译插桩：在代码编译期间修改已有的代码或生成新的代码。

![apt](../../blog/images/android/apt.png)

### 编译插桩的基础知识

从技术实现上看，可以把编译插桩分为两类：

- **Java 文件**：类似 APT、AndroidAnnotation 他们生成的都是 Java 文件，是在编译的最开始接入。
- **字节码（Bytecode）**：对于 代码监控、代码修改以及代码分析这三个场景，一般采用操作字节码的方式。可以操作 Java 字节码，也可以操作 Dalvik 字节码，取决于使用的插桩方法。
  相对于 Java 文件方式，字节码操作方式功能更加强大，应用场景也更广，但是它的使用复杂度更高。

#### 1. 应用场景

- 代码生成：Dagger、ButterKnife、Protocol Buffers、数据库 ORM 框架
- 代码监控：网络监控、耗电监控、各种的性能监控。
- 代码修改：例如修改某些第三方 SDK 的源码。
- 代码分析：自定义代码检查，例如检查代码中 new Thread() 的调用，检查代码汇总一些敏感权限的使用等。

#### 2. 字节码

对于 Java 平台，Java 虚拟机运行的是 Class 文件，内部对应的是 Java 字节码。而针对 Android 这种嵌入式平台，为了优化性能，Android 虚拟机运行的是[Dex 文件](https://source.android.com/devices/tech/dalvik/dex-format)，Google 专门为其设计了一种 Dalvik 字节码，虽然增加了指令长度但却缩减了指令的数量，执行也更为快速。

Java字节码和Dalvik字节码对比

![Java字节码和Dalvik字节码对比](../../blog/images/android/class_vs_dex.png)

它们的主要区别有：

- **体系结构**：Java 虚拟机是基于栈实现，而 Android 虚拟机是基于寄存器实现。在 ARM 平台，寄存器实现性能会高于栈实现。

  > 为什么 Java 虚拟机是基于栈实现的呢？
  > 每一条 Java 虚拟机线程都有自己私有的 Java 虚拟机栈，这个栈与线程同时创建，用于存储栈帧（Stack Frame）。——《Java 虚拟机规范》

- **格式结构**：对于 Class 文件，每个文件都会有自己单独的常量池以及其他一些公共字段。对于 Dex 文件，整个 Dex 中的所有 Class 共用同一个常量池和公共字段，所以整体结构更加紧凑，因此也大大减少了体积。

- **指令优化**：Dalvik 字节码对大量的指令专门做了精简和优化，如下图所示，相同的代码 Java 字节码需要 100 多条，而 Dalvik 字节码只需要几条。

### 编译插桩的三种方法

AspectJ 和 ASM 框架的输入和输出都是 Class 文件，它们是我们最常用的 Java 字节码处理框架。

#### 1. AspectJ

[AspectJ](https://www.eclipse.org/aspectj/)是 Java 中流行的 AOP（aspect-oriented programming）编程扩展框架，内部是通过字节码处理技术实现的代码注入。

AspectJ 的一些劣势：

- 切入点固定
- 正则表达式
- 性能较低



#### 2. ASM

ASM 操作字节码主要的特点有：

- **操作灵活**：操作起来很灵活，可以根据需求自定义修改、插入、删除。
- **上手难：上手比较难，需要对 Java 字节码有比较深入的了解。

相比于 BCEL 框架，ASM 的优势是提供了一个 Visitor 模式的访问接口（Core API），使用者可以不用关心字节码的格式，只需要在每个 Visitor 的位置关心自己所修改的结构即可。但是这种模式的缺点是，一般只能在一些简单场景里实现字节码的处理。



#### 3. ReDex

ReDex 不仅只是作为一款 Dex 优化工具，这个工具可以在所有方法或者指定方法前面插入一段跟踪代码。









## 33 | 做一名有高度的移动开发工程师

- 深度：充分研究浏览器的渲染原理和缓存机制
- 广度：跳出客户端范畴来思考优化策略，清楚不同端该做和能做的事情，不局限于客户端技术栈；
  客户端技术并不是唯一选择
- 组件化：组件化是客户端技术最基本的抽象的体现
- 平台化：持续交付平台、APM平台、测试平台、发布平台、数据平台、网络平台、日志平台，等等。
- 中台：可以简单理解为是把上面分散的平台又统一为一个超大的平台。



## 35 | Native Hook 技术，天使还是魔鬼？

Hook 直译过来是 “钩子” 的意思，是截获进程对某个 API 函数的调用，使得 API 的执行流程转向我们实现的代码片段。



### Java Hook

- 反射：是 Java 查看、检查、修改自身的一种行为
- 动态代理：动态代理相对静态代理而言，是动态的，通过反射对被代理对象的放啊发，代理成代理对象对应的方法。



### Native Hook

- GOT/PLT Hook：主要用于替换某个 so 的外部调用，通过将外部函数调用跳转成我们的目标函数。
- Trap Hook：原理是在需要 Hook 的地方想办法触发断点，并捕获异常。兼容性好，可以在生产环境中使用，单最大的问题是效率低，不适合 Hook 非常频繁调用的函数。
- Inline Hook：一般会使用在自动化测试或者线上疑难问题的定位





































