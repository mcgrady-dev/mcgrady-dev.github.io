## 应用待机存储分区

应用待机存储分区，是 Android 9 引入的电池管理功能，用于系统根据应用的使用时间新近度和使用频率对应用资源请求确定优先级。根据应用使用模式，每个因公因都会被防止在**五**个优先**级存储分区**之一中。系统会根据应用所在的分区限制每个应用可用的设备资源。

### 优先级存储分区

系统会动态地将每个应用分配到优先级存储分区，并根据需要重新分配应用。

这些存储分区为：

- 活跃：应用目前正在使用，或者最近刚刚使用过
- 工作集：应用会定期使用
- 常用：应用会经常使用，但不会每天使用
- 极少使用：应用不经常使用
- 从未使用：已安装但从未运行的应用



## 通过线程提升性能

### 主线程

主线程唯一的工作就是**从线程安全工作队列获取工作块并执行，直到应用被终止**。

当有动画或屏幕更新正在进行时，系统每隔 16ms 足有尝试执行一个工作块（负责绘制屏幕），从而以每秒 60 帧的速度进行渲染。如果主线程无法在 16ms 内执行完工作块，则用户可能会察觉到卡顿、延迟或界面对输入无响应。如果主线程阻塞大约 5 秒，系统会显示 ANR 对话框，允许用户直接关闭应用。

**将大量或冗长的任务从主线程中移出，使其不影响流畅渲染和快速响应用户输入，这是开发者在应用中采用线程处理的最大原因**。

### 线程和界面对象引用

由于 Android 视图对象不是线程安全的，如果尝试在主线程以外的其它线程中修改甚至引用界面对象，则可能导致异常、无法提示故障、崩溃及其它未定义的异常行为。引用方面的问题分为两类：显式引用和隐式引用

#### 显式引用

如果工作线程访问视图层次结构中的某个对象并更改该对象的属性，与此同时有任何其他线程正在引用该对象，则结果无法确定。

例如，某个应用在工作线程上直接引用了界面对象。工作线程上的该对象可能包含对 `View` 的引用；但在工作完成之前，`View` 已从视图层次结构中移除。当这两个操作同时发生时，该引用会将 `View` 对象保留在内存中，并对其设置属性。 但是，用户从不会看到此对象，而且应用会在对象引用消失后删除该对象。

再举一个例子，假设 `View` 对象包含对其所属的 Activity 的引用。如果该 Activity 被销毁，但仍有直接或间接引用它的线程处理工作块，则垃圾回收器会等到该工作块执行完毕再收集该 Activity。

所以，在任何情况下，应用都只应在主线程上更新界面对象。这意味着您应制定允许多个线程将工作传回主线程的协商政策，让最顶层的 Activity 或 Fragment 负责更新实际界面对象。

##### 隐式引用

```java
public class MainActivity extends Activity {
  // ...
  public class MyAsyncTask extends AsyncTask<Void, Void, String>   {
    @Override protected String doInBackground(Void... params) {...}
    @Override protected void onPostExecute(String result) {...}
  }
}
```

以上代码中的 MyAsyncTask 非静态内部类会创建对封装 Activity 实例的隐式引用。因此，在线程处理工作完成之前，该对象一直包含对相应 Activity 的引用，导致所引用 Activity 的销毁出现延迟。 这种延迟进而会给内存带来更多压力。

解决方法是将内部类定义为静态内部类，因为静态内部类与内部类有所不同：内部类的实例要求对外部类的实例进行实例化，并且可直接访问封装实例的方法和字段。相反，静态内部类不需要引用封装类的实例，因此它不包含对外部类成员的引用。



## 系统跟踪

Android平台提供了多种不同的跟踪信息获取途径：

- **Android Studio CPU 性能剖析器**
- **“系统跟踪”应用**
- **Android System Trace 命令行工具**
- **Perfetto 命令行工具**





## 优化电池续航时间

### 针对低电耗模式和应用待机模式进行优化

从 Android 6.0 开始引入了两项省电功能，通过管理应用在设备未连接至电源时的行为方式：

- 当用户长时间未使用设备时，低耗电模式会延迟应用的后台 CPU 和 网络活动，从而降低耗电量。
- 当设备处于低电量模式时，应用对某些高耗电量资源的访问会延迟到维护期。[电源管理限制](https://developer.android.google.cn/topic/performance/power/power-details)中就列出了具体的限制。

#### 了解低电耗模式

如果用户未插接设备的电源，在屏幕关闭的情况下，让设备在一段时间内保持不活动状态，那么设备就会进入低电耗模式。

在低电耗模式下，**系统会尝试通过限制应用访问占用大量网络和 CPU 资源的服务来节省电量**。**它还会阻止应用访问网络，并延迟其作业、同步和标准闹钟**。

#### 低电耗模式限制

在低电耗模式下，您的应用会受到一下限制：

- 暂停访问网络
- 系统忽略唤醒锁定
- 标准 AlarmManager 闹钟推迟到下一个维护期
  - 如果您需要在设备处于低电耗模式时触发的闹钟，需使用 `setAndAllowWhileIdle()` 或 `setExactAndAllowWhileIdle()`
  - 使用 `setAlarmClock()` 设置的闹钟将继续正常触发，系统会在这些闹钟出发前不久退出低电耗模式。
- 系统不执行 WLAN 扫描
- 系统不允许运行同步适配器（AbstractThreadedSyncAdapter）
- 系统不允许运行 JobScheduler

#### 了解应用待机模式

应用待机模式允许系统判定应用在用户未主动使用它时是否处于闲置状态。当用户有一段时间未触摸应用时，系统便会作出此判定，以下条件均不适用：

- 用户明确启动应用
- 因公因当前有一个进程在前台运行
- 应用生成用户可在锁定屏幕或通知栏看到的通知
- 应用正在使用设备管理应用

当用户将设备插入电源时，系统会从待机状态释放应用，运行它们自由访问网络并执行任何待处理的作业和同步。如果设备长时间处于闲置状态，系统将允许闲置应用访问网络，频率大约每天一次。



### 电量分析工具

Android 提供了一些耗电量优化工具，通过这些工具确定要优化哪些方面，从而延长电池续航时间。

- **Profile GPU Rendering**

- **Batterystats**
  Batterystats 是包含在 Android 框架中的一种工具，用于收集设备上的电池数据。您可以使用 [adb](https://developer.android.google.cn/studio/command-line/adb) 将收集的电池数据转储到开发机器，并生成可使用 Battery Historian 分析的报告。
  适合场景：

  - 显示进程从什么位置以及通过何种方式消耗电磁用量

  - 识别系统为了延长电池续航时间可能延迟甚至移除应用中的哪些任务

- **Battery Historian**
  通过 Battery Historian 工具了解设备随时间的耗电情况。在系统级别，该工具以 HTML 的形式可视化来自系统日志的电源相关事件。在具体应用级别，该工具可提供各种数据，帮助您识别耗电的应用行为。

  除了系统级数据视图提供的宏观数据外，Battery Historian 还提供特定于您设备上运行的每个应用的数据表格和部分可视化内容。表格数据包括：

  - 应用在设备上的估计耗电量
  - 网络信息
  - 唤醒锁定次数
  - 服务
  - 进程信息



## 缩减应用大小

### 使用 AAB（Android App Bundle）上传应用

Google Play 的新应用服务模式“Dynamic Delivery”会使用您的 App Bundle 针对每位用户的设备配置生成并提供经过优化的 APK，因此他们只需下载运行您的应用所需的代码和资源。您无需再编译、签署和管理多个 APK 以支持不同的设备，而用户也可以获得更小、更优化的下载文件包。

### 使用 Android Size Analyzer 分析应用大小



### 缩减资源数量和大小

#### 移除未使用资源

- 通过 lint 工具 检查未被代码引用的 `res/` 资源

- `build.gradle` 中启用 shrinkResource

  ```groovy
  android {
    buildTypes {
      release {
        minifyEnabled true
        shrinkResources true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
      }
    }
  }
  ```

  使用 `shrinkResources` 还需启用代码压缩功能，在编译过程中 `R8` 首先会移除未使用的代码，然后 AGP 会移除未使用的资源。

#### 使用可绘制对象

框架可以在运行时动态绘制图片，如 Drawable，且 Drawable 对象占用更少的空间。

#### 重用资源

使用 `android:tint` 和 `tintMode` 属性来更改资源的颜色，对于较低版本的平台，则使用 `ColorFilter` 类。

#### 压缩 PNG 和 JPEG 文件

- aapt 工具可以在编译过程中通过无损压缩来优化放置在 `res/drawable/` 中的图片资源。但 appt 具有以下限制：

  - `aapt` 不会缩减 `asset/` 下包含 PNG 文件

  - 图片文件需要使用 256 种或更少颜色才可供 `aapt` 进行优化

  - `aapt` 可能会扩充已压缩的 PNG 文件，为防止这种情况，可以在 Gradle 中使用 `cruncherEnabled` 标记为 PNG 文件停用此过程。

    ```groovy
    aaptOptions {
      cruncherEnabled = false
    }
    ```

- 可以使用 [pngcrush](http://pmt.sourceforge.net/pngcrush/)、[pngquant](https://pngquant.org/) 或 [zopflipng](https://github.com/google/zopfli) 等工具缩减 PNG 文件的大小，同时不损失画质。所有这些工具都可以缩减 PNG 文件的大小，同时保持肉眼感知的画质不变。

- 要压缩 JPEG 文件，您可以使用 [packJPG](http://www.elektronik.htw-aalen.de/packjpg/) 和 [guetzli](https://github.com/google/guetzli) 等工具。

#### 使用 WebP 格式资源

WebP 格式提供有损压缩（如 JPEG）以及透明度（如 PNG），不过与 JPEG 或 PNG 相比，这种格式可以提供更好的压缩效果（需在 API 13 及更高版本为目标中使用）。

#### 使用 VectorDrawable 矢量图形

使用矢量图形创建与分辨率无关的图标和其他可伸缩媒体，可以极大地减少 APK 占用的空间。



### 减少原生和 Java 代码

#### 避免使用枚举



## 管理内存

### 内存管理概述

ART 和 Dalvik 虚拟机使用[内存分页](http://en.wikipedia.org/wiki/Paging)和[内存映射](http://en.wikipedia.org/wiki/Memory-mapped_files)来管理内存。这意味着应用修改的任何内存，都会一直驻留在 RAM 中，并且无法换出。要从应用中释放内存，只能释放应用保留的对象引用，使内存可供垃圾回收器回收。

#### 垃圾回收

ART 或 Dalvik 虚拟机之类的受管内存环境会跟踪每次内存分配。**一旦确定程序不再使用某块内存，它就会将该内存重新释放到堆中，无需程序员进行任何干预**。这种回收受管内存环境中的未使用内存的机制称为“垃圾回收”。垃圾回收有两个目标：

- 在程序中查找将来无法访问的数据对象
- 回收这些对象使用的资源

Android 的内存堆会根据分配对象的**预期寿命**和**大小**跟踪不同的分配存储分区。例如，最近分配的对象属于“新生代”。当某个对象保持活动状态达足够长的时间时，可将其提升为较老代，然后是永久代。

堆的每一代对相应对象可占用的内存量都有其自身的专用上限。每当一代开始填满时，系统便会执行垃圾回收事件以释放内存。垃圾回收的持续时间取决于它回收的是哪一代对象以及每一代有多少个活动对象。

#### 限制应用内存

为了维持多任务环境的正常运行，Android 会为每应用堆内存大小设置硬性上限。不同设备的确切堆大小上线取决于设备总体可用 RAM 大小。如果应用在达到堆容量上线后尝试分配更多内存，则可能会收到 `OutOfMemoryError` 。

#### 切换应用

当用户在应用之间切换时，Android 会将非前台应用保留在缓存中。非前台应用就是指用户看不到或未运行前台服务（如音乐播放）的应用。

如果您的应用具有缓存的进程且保留了目前不需要的资源，那么即使用户未使用您的应用，它也会影响系统的整体性能。当系统资源（如内存）不足时，它将会终止缓存中的进程。系统还会考虑终止占用最多内存的进程以释放 RAM。

### 进程间的内存分配

Android 平台在运行时不浪费可用的内存，会一直尝试利用所有可用内存。例如，系统会在应用关闭后将其保留在内存中，以便用户快速切回到这些应用。因此，通常情况下，Android 设备在运行时几乎没有可用的内存。要在重要系统进程和许多用户应用之间正确分配内存，内存管理至关重要。

#### 内存类型

![android-memory-types](../../images/android/android-memory-types.svg)

Andorid 设备包含三种不同类型的内存：

- **RAM**
  RAM 是最快的内存类型，但其大小通常有限。
- **zRAM**
  zRAM 是用于交换空间的 RAM 分区。**所有数据在放入 zRAM 时都会进行压缩，然后在从 zRAM 向外复制时进行解压缩**。这部分 RAM 会随着页面进出 zRAM 而增大或缩小。设备制造商可以设置 zRAM 大小上限。
- **存储器**
  存储器中包含所有持久性数据（例如文件系统等），以及为所有应用、库和平台添加的对象代码。存储器比另外两种内存的容量大得多。在 Android 上，存储器不像在其他 Linux 实现上那样用于交换空间，因为频繁写入会导致这种内存出现损坏，并缩短存储媒介的使用寿命。

#### 内存页面

RAM 分为多个“页面”。通常，每个页面为 4KB 的内存。

系统会将页面视为“可用”或“已使用”。可用页面是未使用的 RAM。已使用的页面是系统目前正在使用的 RAM，并分为以下类别：

- **缓存页**：有存储器中的文件（例如代码或内存映射文件）支持的内存。缓存内存有两种类型：

  - **私有页**：由一个进程拥有且未共享
    - 干净页：存储器中未经修改的文件副本，可由 [`kswapd`](https://developer.android.google.cn/topic/performance/memory-management#kswapd) 删除以增加可用内存
    - 脏页：存储器中经过修改的文件副本；可由 `kswapd` 移动到 zRAM 或在 zRAM 中进行压缩以增加可用内存
  - **共享页**：由多个进程使用
    - 干净页：存储器中未经修改的文件副本，可由 `kswapd` 删除以增加可用内存
    - 脏页：存储器中经过修改的文件副本；允许通过 `kswapd` 或者通过明确使用 [`msync()`](https://developer.android.google.cn/reference/android/system/Os#msync(long,%20long,%20int)) 或 [`munmap()`](https://developer.android.google.cn/reference/android/system/Os#munmap(long,%20long)) 将更改写回存储器中的文件，以增加可用空间

- **匿名页**：

  没有存储器中的文件支持的内存（例如，由设置了 `MAP_ANONYMOUS` 标记的 `mmap()` 进行分配）

  - 脏页：可由 `kswapd` 移动到 zRAM/在 zRAM 中进行压缩以增加可用内存



#### 内存不足管理

Android 有两种处理内存不足情况的主要机制：

- **内核交换守护进程**

- 内核交换守护进程 (`kswapd`) 是 Linux 内核的一部分，用于将已使用内存转换为可用内存。当设备上的可用内存不足时，该守护进程将变为活动状态。Linux 内核设有可用内存上下限阈值。当可用内存降至下限阈值以下时，`kswapd` 开始回收内存。当可用内存达到上限阈值时，`kswapd` 停止回收内存。

  `kswapd` 可以删除干净页来回收它们，因为这些页受到存储器的支持且未经修改。如果某个进程尝试处理已删除的干净页，则系统会将该页面从存储器复制到 RAM。此操作称为“请求分页”。
  ![android-memory-delete-clean-page](../../images/android/android-memory-delete-clean-page.svg)

  由存储器支持的干净页已删除。
  `kswapd` 可以将缓存的私有脏页和匿名脏页移动到 zRAM 进行压缩。这样可以释放 RAM 中的可用内存（可用页面)。如果某个进程尝试处理 zRAM 中的脏页，该页将被解压缩并移回到 RAM。如果与压缩页面关联的进程被终止，则该页面将从 zRAM 中删除。

  如果可用内存量低于特定阈值，系统会开始终止进程。
  ![android-memory-compressed-dirty-page](../../images/android/android-memory-compressed-dirty-page.svg)
  脏页被移至 zRAM 并进行压缩。

- **低内存终止守护进程**
  很多时候，`kswapd` 不能为系统释放足够的内存。在这种情况下，系统会使用 [`onTrimMemory()`](https://developer.android.google.cn/reference/android/content/ComponentCallbacks2#onTrimMemory(int)) 通知应用内存不足，应该减少其分配量。如果这还不够，内核会开始终止进程以释放内存。它会使用低内存终止守护进程 (Lower Memory Killer) 来执行此操作。
  LMK 使用一个名为 [`oom_adj_score`](https://android.googlesource.com/platform/system/core/+/master/lmkd/README.md) 的“内存不足”分值来确定正在运行的进程的优先级，以此决定要终止的进程。最高得分的进程最先被终止。后台应用最先被终止，系统进程最后被终止。下表列出了从高到低的 LMK 评分类别。评分最高的类别，即第一行中的项目将最先被终止 （Android 进程，高分在上，低分在下）：

  ![android-lmk-process-order](../../images/android/android-lmk-process-order.png)

  另外，设备制造商可以更改 LMK 的行为。



### 管理应用内存

#### 监控可用内存和内存使用量

使用 Memory Profile（内存剖析器）查找和诊断内存问题。

#### 释放内存以响应事件

可以在 `Activity` 类中实现 `ComponentCallbacks2` 接口。借助所提供的 `onTrimMemory()` 回调方法，可以在处于前台或后台时监听与内存相关的事件，然后释放对象以响应指示系统需要回收内存的应用生命周期事件或系统事件。

例如，可以根据回调等级响应不同的与内存相关的事件：

```kotlin
    import android.content.ComponentCallbacks2
    // Other import statements ...

    class MainActivity : AppCompatActivity(), ComponentCallbacks2 {

        // Other activity code ...

        /**
         * Release memory when the UI becomes hidden or when system resources become low.
         * @param level the memory-related event that was raised.
         */
        override fun onTrimMemory(level: Int) {

            // Determine which lifecycle or system event was raised.
            when (level) {

                ComponentCallbacks2.TRIM_MEMORY_UI_HIDDEN -> {
                    /*
                       Release any UI objects that currently hold memory.

                       The user interface has moved to the background.
                    */
                }

                ComponentCallbacks2.TRIM_MEMORY_RUNNING_MODERATE,
                ComponentCallbacks2.TRIM_MEMORY_RUNNING_LOW,
                ComponentCallbacks2.TRIM_MEMORY_RUNNING_CRITICAL -> {
                    /*
                       Release any memory that your app doesn't need to run.

                       The device is running low on memory while the app is running.
                       The event raised indicates the severity of the memory-related event.
                       If the event is TRIM_MEMORY_RUNNING_CRITICAL, then the system will
                       begin killing background processes.
                    */
                }

                ComponentCallbacks2.TRIM_MEMORY_BACKGROUND,
                ComponentCallbacks2.TRIM_MEMORY_MODERATE,
                ComponentCallbacks2.TRIM_MEMORY_COMPLETE -> {
                    /*
                       Release as much memory as the process can.

                       The app is on the LRU list and the system is running low on memory.
                       The event raised indicates where the app sits within the LRU list.
                       If the event is TRIM_MEMORY_COMPLETE, the process will be one of
                       the first to be terminated.
                    */
                }

                else -> {
                    /*
                      Release any non-critical data structures.

                      The app received an unrecognized memory level value
                      from the system. Treat this as a generic low-memory message.
                    */
                }
            }
        }
    }
    
```



#### 使用内存效率更高的代码结构

- **谨慎使用服务**
  在您启动某项服务后，系统更倾向于让此服务的进程始终保持运行状态。这种行为会导致服务进程代价十分高昂，因为一旦服务使用了某部分 RAM，那么这部分 RAM 就不再可供其他进程使用。这会减少系统可以在 LRU 缓存中保留的缓存进程数量，从而降低应用切换效率。
  通常应该避免使用持久性服务，因为它们会对可用内存提出持续性的要求。建议采用 WorkManager 等替代实现方式。
  如果必须使用某项服务，则限制此服务的生命周期最佳方式是使用 IntentService ，它会在处理完启动它的 `intent` 后立即自行结束。

- **使用经过优化的数据容器**

  编程语言所提供的部分类并未针对移动设备做出优化。例如，常规 `HashMap` 实现的内存效率可能十分低下，因为每个映射都需要分别对应一个单独的条目对象。
  Android 框架包含几个经过优化的数据容器，包括 `SparseArray`、`SparseBooleanArray` 和 `LongSparseArray`。

- **谨慎对待代码抽象**
  虽然抽象的代码可以提高代码灵活性和维护性，但抽象的代价很高：通常它们需要更多的代码才能执行，需要更多的时间和更多的 RAM 才能将代码映射到内存中。因此，如果抽象没有带来显著的好处，您就应该避免使用抽象。

- **针对序列化数据使用精简版 Protobuf**

- **避免内存抖动**
  垃圾回收事件通常不会影响应用的性能。不过，如果在短时间内发生许多垃圾回收事件，就可能会快速耗尽帧时间。系统花在垃圾回收上的时间越多，能够花在呈现或流式传输音频等其他任务上的时间就越少。

  例如，您可以在 `for` 循环中分配多个临时对象。或者，您也可以在视图的 `onDraw()` 函数中创建新的 `Paint` 或 `Bitmap` 对象。在这两种情况下，应用都会快速创建大量对象。这些操作可以快速消耗新生代 (young generation) 区域中的所有可用内存，从而迫使垃圾回收事件发生。



#### 移除会占用导量内存的资源和库

#### 谨慎使用第三方库



## 检查 GPU 渲染

检查 GPU 渲染速度和过渡绘制可帮助您直观地查看您的应用可能在何处遇到界面渲染问题，如果执行不必要的渲染工作，或执行长时间的线程和 GPU 操作。



## 使应用能迅速响应

### 什么会触发 ANR

在 Android 中，应用响应性由 ActivityManager 和 WindowManager 系统服务监控。当 Android 检测到一下某项条件时，便会对特定应用显示 ANR 对话框：

- 5 秒内对输入事件（例如按键或屏幕轻触事件）无响应
- BroadcastReceiver 在 10 秒内尚未执行完毕

### 如何避免 ANR

在UI线程中运行的所有方法都应该尽可能减少在此线程中的操作。特别是在 `onCreate()` 和 `onResume()` 等关键生命周期方法中，Activity 应尽量减少进行设置所需的操作。可能会长时间运行的操作（例如网络或数据库操作）或计算成本高昂的计算（例如调整位图大小）应在工作线程中完成（如果是数据库操作，则应通过异步请求完成）。

### 加强响应能力

通常，100 到 200 毫秒是一个阈值，一旦超出此阈值，用户便能够感受到应用速度缓慢。因此，除了采取措施以避免显示 ANR 之外，还有一些提示可以让用户感觉您的应用响应迅速：

- 如果应用在后台执行操作以响应用户输入，则显示正在进行该操作（例如在界面中使用 `ProgressBar`）。
- 特别是游戏，在工作线程中计算走法。
- 如果应用具有耗时较长的初始设置阶段，考虑显示启动画面或尽快呈现主视图，表明正在加载，并异步填充信息。在任何一种情况下，您都应以某种方式表明操作正在进行，以免用户认为应用已卡住。
- 使用 [Systrace](https://developer.android.google.cn/tools/help/systrace) 和 [Traceview](https://developer.android.google.cn/tools/help/traceview) 等性能工具确定应用响应能力方面的瓶颈。
