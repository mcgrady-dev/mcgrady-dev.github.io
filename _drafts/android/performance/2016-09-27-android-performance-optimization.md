---
layout: 
title: Android 性能优化
date: 2016-09-27 11:42 +0800
tags: android
---

Android性能优化

<!--more-->

<img src="https://developer.android.google.cn/topic/performance/vitals/images/android-vitals.png" alt="android-vitals" style="zoom: 50%;" />



## 性能优化原则

- 坚持性能测试：不要凭感觉检测性能问题、评估性能优化的效果，应该保持足够多的测量，用数据说话；使用各种性能工具测试及定位问题。
- 使用低配置的设备：同样的程序，在低端配置的设备中，相同的问题会暴露得更为明显。
- 权衡利弊：在能够保证产品稳定、按时完成需求的前提下去做优化。

## 性能优化方法

- 了解问题（分为可感知和不可感知的性能问题）：对于性能问题，只适用于某些明显的性能问题，很多无法感知的性能问题需要通过工具定位。例如：内存泄露、层级冗杂、过渡绘制等无法感知；滑动卡顿可以感知。
- 定位问题：通过工具检查、分析数据，定位什么地方存在性能问题。
- 分析问题：找到问题后，分析针对这个问题该如何解决，确定解决方案。
- 解决问题：根据分析结果寻找解决方案。
- 验证问题：保证优化有效，没有产生新的问题，以及产品稳定性。

## 性能优化工具

### 设备开发者模式

- 调试 GPU 过渡绘制
- 启用严苛模式
- 显示 CPU 使用情况
- GPU 呈现模式分析
- 显示所有 ”应用程序无响应“

### IDE 内置检查工具

- Lint
- Memory Monitor
- CPU Monitor
- NetWork Monitor
- GPU Monitor
- Layout Inspector
- Analyze APK

### SDK 内置检查工具

- DDMS
- HierarchyViewer
- TraceView

### 第三方工具

- MAT
- LeakCanary
- GT

## 性能优化指标

### 渲染

- 滑动流畅度：FPS（Frame per Second），一秒内的刷新帧数，越接近60帧越好
- 过渡绘制：单页面的3x（粉红色区域）`Overdraw` 小于 25%
- 启动时间：这要是 Activity 界面启动时间，一般低于 300ms，需要用高频摄像机计算时间。

### 内存

- 内存大小：峰值越低越好，需要优化前后做对比
- 内存泄露：需要用工具检查对比优化前后

### 功耗

- 单位时间内的耗电量，掉电量越少越好，没有固定标准。



## 渲染问题

### CPU、GPU 的职责

- CPU 负责包括 Measure、Layout 等计算操作
- GPU 负责 栅格化（Reasterization）操作，所谓栅格化是将矢量图形转换为位图的过程，再简单一点就是将 View 等组件拆分到一个个像素上去显示。

UI 渲染优化的目的就是减轻CPU、GPU的压力，除去不必要的操作，保证每一帧在16ms内处理完所有CPU、GPU的计算。

![](../../images/android/android-GPU-overdraw.png)





## 内存问题



## 耗电问题







## 性能检查项

### 启动速度
这里的启动速度指的是冷启动的速度，即杀掉应用后重新启动的速度，此项主要是和你的竞品。

不应在Application以及Activity的生命周期回调中做任何费时操作，具体指标大概是你在`onCreate`，`onResume`，`onStart`等回调中所花费的总时间最好不要超过***400ms***，否则用户在桌面点击你的应用图标后，将感觉到明显的卡顿。





### 界面切换
应用操作时，界面和动画不应有明显卡顿。

可通过在手机上打开 `设置->开发者选项->调试GPU过度绘制`，然后操作应用查看gpu是否超线进行初步判断：
![Alt text](./forumImage20160307112323643.png)

### 内存泄露
`BACK`退出不应存在内存泄露，简单的检查办法是在退出应用后，用命令`adb shell dumpsys meminfo 应用包名`查看 `Activities Views` 是否为零。

多次进入退出后的占用内存`TOTAL`不应变化太大：
![Alt text](./forumImage20160307112438814.png)

### onTrimMemory回调
应用响应此回调释放非必须内存；

2）验证可通过命令`adb shell dumpsys gfxinfo 应用包名-cmd trim 5`后，再）用命令`adb shell dumpsys meminfo 应用包名`查看内存大小。

### 过度绘制
打开设置中的GPU过度绘制开关，各界面过度绘制不应超过2.5x；也就是打开此调试开关后，界面整体呈现浅色，特别复杂的界面，红色区域也不应该超过全屏幕的四分之一。

### lint检查
通过Android Studio中的 `Analyze->Inspect Code` 对工程代码做静态扫描，找出潜在的问题代码并修改。

`0 error & 0 warning`，如果确实不能解决，需给出原因。

### 反射优化
在代码中减少反射调用

对频繁调用的返回值进行Cache

### 稳定性
连续48小时monkey不应出现闪退，anr问题。

如果应用接入了数据埋点的sdk，比如百度统计sdk等，这些sdk都会将应用的崩溃信息上报回来，开发者应每天关注这些统计到的崩溃日志，严格控制应用的崩溃率。

### 耗电
应用进入后台后不应异常消耗电量；

操作应用后，退出应用，让应用处于后台，一段时间后通过`adb shell dumpsys batterystats`查看电量消耗日志看是否存在异常。


## OOM问题优化

### OOM问题分析
#### OOM的必然性与解决性
#### OOM的绝大部分发生在图片

### 强引用、软引用意义

### 优化OOM问题的方法



## 性能问题常见原因

**性能问题一般归结为三类**：

1. UI卡顿和稳定性:这类问题用户可直接感知，最为重要；

2. 内存问题：内存问题主要表现为内存泄露，或者内存使用不当导致的内存抖动。如果存在内存泄露，应用会不断消耗内存，易导致频繁gc使系统出现卡顿，或者出现OOM报错；内存抖动也会导致UI卡顿。

3. 耗电问题：会影响续航，表现为不必要的自启动，不恰当持锁导致系统无法正常休眠，系统休眠后频繁唤醒系统等；



## UI卡顿常见原因和分析方法

*下面分别介绍出现这些问题的常见原因以及分析这些问题的一般步骤*。
### 卡顿常见原因
1. 人为在UI线程中做轻微耗时操作，导致UI线程卡顿；
2. 布局Layout过于复杂，无法在16ms内完成渲染；
3. 同一时间动画执行的次数过多，导致CPU或GPU负载过重；
4. View过度绘制，导致某些像素在同一帧时间内被绘制多次，从而使CPU或GPU负载过重；
5. View频繁的触发measure、layout，导致measure、layout累计耗时过多及整个View频繁的重新渲染；
6. 内存频繁触发GC过多（同一帧中频繁创建内存），导致暂时阻塞渲染操作；
7. 冗余资源及逻辑等导致加载和执行缓慢；
8. 工作线程优先级未设置为
```
Process.THREAD_PRIORITY_BACKGROUND
// 导致后台线程抢占UI线程cpu时间片，阻塞渲染操作。
```
9. ANR

### 卡顿分析解决的一般步骤：

#### 解决过度绘制问题
- 在`设置->开发者选项->调试GPU过度绘制`中打开调试，看对应界面是否有过度绘制，如果有先解决掉：
- 定位过渡绘制区域
- 利用Android提供的工具进行位置确认以及修改(HierarchyView , Tracer for OpenGL ES)
- 定位到具体的视图(xml文件或者View)
- 通过代码和xml文件分析过渡绘制的原因
- 结合具体情况进行优化
- 使用Lint工具进一步优化

#### 检查是否有主线程做了耗时操作：
严苛模式（StrictMode），是Android提供的一种运行时检测机制，用于检测代码运行时的一些不规范的操作，最常见的场景是用于发现主线程的IO操作。应用程序可以利用StrictMode尽可能的发现一些编码的疏漏。

**开启 StrictMode**
对于应用程序而言，Android 提供了一个最佳使用实践：尽可能早的在android.app.Application 或 android.app.Activity 的生命周期使能 StrictMode，onCreate()方法就是一个最佳的时机，越早开启就能在更多的代码执行路径上发现违规操作。

```java
public void onCreate() {
 if (DEVELOPER_MODE) {
 StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
 .detectAll() .penaltyLog() .build());
     StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
 .detectAll() .penaltyLog() .build());
 }
     super.onCreate();
}
```
如果主线程有网络或磁盘读写等操作，在logcat中会有"D/StrictMode"tag的日志输出，从而定位到耗时操作的代码。 

#### 若主线程无耗时操作
如果主线程无耗时操作，还存在卡顿，有很大可能是必须在UI线程操作的一些逻辑有问题，比如控件measure、layout耗时过多等，此时可通过Traceview以及systrace来进行分析。

#### Traceview
Traceview主要用做热点分析，找出最需要优化的点。

打开DDMS然后选择一个进程，接着点击上面的“Start Method Profiling”按钮（红色小点变为黑色即开始运行），然后操作我们的卡顿UI,然后点击"Stop Method Profiling",会打开如下界面：
![Alt text](./forumImage20160307112900105.png)
图中展示了Trace期间各方法调用关系，调用次数以及耗时比例。通过分析可以找出可疑的耗时函数并进行优化。

#### systrace：抓取trace：
*执行如下命令*：
```vim
$ cd android-sdk/platform-tools/systrace
$ python systrace.py --time=10 -o mynewtrace.html sched gfx view wm
```
操作APP，然后会生成一个mynewtrace.html 文件，用Chrome打开。
图示如下：
![Alt text](./forumImage20160307113057157.png)
通过分析上面的图，可以找出明显存在的layout，measure，draw的超时问题。

#### 导入如下插件
可通过在方法上添加@DebugLog来打印方法的耗时：
```gradle
build.gradle:
buildscript {
        dependencies {
```
用于方便调试性能问题的打印插件。给访法加上@DebugLog，就能输出该方法的调用参数，以及执行时间。
```gradle
 classpath 'com.jakewharton.hugo:hugo-plugin:1.2.1'
    }
}
```
用于方便调试性能问题的打印插件。给访法加上@DebugLog，就能输出该方法的调用参数，以及执行时间。
```gradle
apply plugin: 'com.jakewharton.hugo'
java：
@DebugLog
public void test( int a ){
int b=a*a;
}
```



### 卡顿的原理

卡顿主要由于主线程有耗时操作，导致View绘制掉帧，屏幕没16毫秒刷新一次，也就每秒刷新60次，人眼能感觉到卡顿的频率是每秒24帧。

#### 屏幕刷新机制





## 内存性能分析优化

### 内存泄露
该问题目前在项目中一般用leakcanary基本就能搞定，配置起来也相当简单：
```gradle
build.gradle:
dependencies {
   debugCompile 'com.squareup.leakcanary:leakcanary-android:1.3.1' // or 1.4-beta1
   releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.3.1' // or 1.4-beta1
   testCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.3.1' // or 1.4-beta1
 }
```
```java
public class ExampleApplication extends Application {
 
  @Override public void onCreate() {
    super.onCreate();
    LeakCanary.install(this);
  }
}
```
一旦有内存泄露，将会在通知栏生成一条通知，点开可看到泄露的对象以及引用路径：
![Alt text](./forumImage20160307113450862.png)

#### 内存泄露会导致剩余可用Heap越来越少，频繁出发GC

#### 尤其Activity泄露

#### 用Application Context 而不是Activity Context

#### 注意Cursor对象是否及时关闭

### 内存抖动
如果代码中存在在onDraw或者for循环等多次执行的代码中分配对象的行为，会导致运行过程中gc次数增多，影响UI流畅度。一般这些问题都可通过lint工具检测出来。

### 数据结构优化

#### 频繁字符串拼接用StringBuilder

#### ArrayMap、SparseArray 替换 HashMap

### 对象复用

#### 复用系统自带资源

#### ListView/GridView的ConvertView复用

#### 避免在onDraw方法里执行对象的创建



### LeakCanary

#### LeakCanary的工作原理

##### 1. Detecting retained objects (检测保留的对象)

##### 2. Dumping the heap (倾倒堆)

##### 3. Analyzing the heap (分析堆)

##### 4. Categorizing leaks (泄露分类)



## 耗电量优化建议
***电量优化主要是注意尽量不要影响手机进入休眠***，也就是正确申请和释放WakeLock，另外就是不要频繁唤醒手机，主要就是正确使用Alarm。



## App瘦身

[App瘦身最佳实践](https://juejin.cn/post/6844903457049051143)



## 一些好的代码实践
1. 节制地使用Service
2. 当界面不可见时释放内存
3. 当内存紧张时释放内存
4. 避免在Bitmap上浪费内存

对大图片，先获取图片的大小信息，根据实际需要展示大小计算inSampleSize，最后decode：
```java
public static Bitmap decodeSampledBitmapFromFile(String filename,
int reqWidth, int reqHeight) {
// First decode with inJustDecodeBounds=true to check dimensions
final BitmapFactory.Options options = new BitmapFactory.Options();
options.inJustDecodeBounds = true;
BitmapFactory.decodeFile(filename, options);
// Calculate inSampleSize
options.inSampleSize =
reqHeight);
calculateInSampleSize(options,
reqWidth,
// Decode bitmap with inSampleSize set
options.inJustDecodeBounds = false;
return BitmapFactory.decodeFile(filename, options);
}
public static int calculateInSampleSize(BitmapFactory.Options options,
int reqWidth, int reqHeight) {
// Raw height and width of image
final int height = options.outHeight;
final int width = options.outWidth;
int inSampleSize = 1;
if (height > reqHeight || width > reqWidth) {
if (width > height) {
inSampleSize = Math.round((float) height / (float) reqHeight);
} else {
inSampleSize = Math.round((float) width / (float) reqWidth);
}
}
return inSampleSize;
}
```
5. 使用优化过的数据集合
6. 谨慎使用抽象编程
7. 尽量避免使用依赖注入框架
   很多依赖注入框架是基于反射的原理，虽然可以让代码看起来简洁，但是是有碍性能的。
8. 谨慎使用external libraries
9. 优化整体性能
10. 使用ProGuard来剔除不需要的代码
``` gradle
android {
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'src/main/proguard-project.txt'
            signingConfig signingConfigs.debug
        }
}
```
11. 慎用异常,异常对性能不利
    抛出异常首先要创建一个新的对象。Throwable 接口的构造函数用名为fillInStackTrace() 的本地方法,fillInStackTrace() 方法检查栈,收集调用跟踪信息。只要有异常被抛出,VM 就必要调整调用栈,因为在处理过程中创建了一个新对象。
    异常只能用于错误处理,不应该用来控制程序流程。
    以下例子不好：
```java
try {
startActivity(intentA);
} catch () {
startActivity(intentB);
}
```
应该用下面的语句判断：
```java
if (getPackageManager().resolveActivity(intentA, 0) != null)
```
不要再循环中使用 try/catch 语句,应把其放在最外层，使用System.arraycopy()代替 for 循环复制。





## 启动速度优化

### Application 启动速度优化



### Activity 启动速度优化

#### 分析方法

activity 启动时间统计方法主要有以下几种：

- **logcat: Displayed**
  在启动 activity 之后抓取 logcat 里的 Displayed 关键字，会打印一个启动时间：

  ```properties
  ActivityTaskManager: Displayed com.android.calculator2/.Calculator: +710ms
  ```

- **am start -W**

  ```bash
  adb shell "am start -W com.android.calculator2/.Calculator"
  Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.android.calculator2/.Calculator }
  Status: ok
  LaunchState: WARM
  Activity: com.android.calculator2/.Calculator
  TotalTime: 710
  WaitTime: 731
  ```

- **reportFullyDrawn()**

  让应用决定什么时候界面准备好，调用这个方法通知系统，然后 logcat 会打印下面内容：

  ```properties
  ActivityTaskManager: Fully drawn com.android.settings/.Settings: +836ms
  ```

- **systrace: launching**
