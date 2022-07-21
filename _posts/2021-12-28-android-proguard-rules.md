---
layout: 
title: Android ProGuard
date: 2021-12-28 17:25 +0800
tags: android
---

Android 混淆规则

<!--more-->

## 1. ProGuard 概述

ProGuard 是一个 **Java 类文件压缩器、优化器、混淆器和预验证器**。

### 1.1. ProGuard执行的流程

![android-proguard-rules2](https://s2.loli.net/2022/07/19/1BatdUFG4sSyDIT.png)

### 1.2. ProGuard的作用

- **代码缩减（摇树优化）**：从应用及其库依赖项中检测并安全地移除不使用的类、字段、方法和属性（这使其成为了一个对于规避 [64k 引用限制](https://developer.android.com/studio/build/multidex)非常有用的工具）。例如，如果您仅使用某个库依赖项的少数几个 API，那么缩减功能可以识别应用不使用的库代码并仅从应用中移除这部分代码。
- **资源缩减**：从封装应用中移除不使用的资源，包括应用库依赖项中不使用的资源。此功能可与代码缩减功能结合使用，这样一来，移除不使用的代码后，也可以安全地移除不再引用的所有资源。
- **混淆**：使用无意义的简短名称重命名剩余的类、字段和方法，从而减小 DEX 文件的大小。
- **优化**：检查并重写代码，以进一步减小应用的 DEX 文件的大小。例如，如果 R8 检测到从未采用过给定 `if/else` 语句的 `else {}` 分支，则会移除 `else {}` 分支的代码。



## 2. ProGuard 原理

### 2.1. 代码缩减（摇树优化）

是指移除由R8确定的在运行时不需要的代码的过程。此过程可以大大减小应用的大小，例如，当您的应用包含许多库依赖项，但只使用它们的一小部分功能时。

R8 首先会根据组合的[配置文件集](https://developer.android.com/studio/build/shrink-code#configuration-files)确定应用代码的所有入口点。这些入口点包括 Android 平台可用来打开应用的 Activity 或服务的所有类。从每个入口点开始，R8 会检查应用的代码来构建一张图表，列出应用在运行时可能会访问的所有方法、成员变量和其他类。系统会将与该图表没有关联的代码视为执行不到的代码，并可能会从应用中移除该代码。

![android-proguard-rules](https://s2.loli.net/2022/07/19/ynE4hpl8sBD1dZx.png)

### 2.2. 资源缩减

资源缩减只有在与代码缩减配合使用时才能发挥作用。在代码缩减器移除所有不使用的代码后资源缩减器开始执行。



### 2.3. 对代码进行混淆处理

由于混淆处理会对代码的不同部分进行重命名，因此在执行某些任务（如检查堆栈轨迹）时需要使用额外的工具。如果您的代码依赖于应用的方法和类的可预测命名（例如，使用反射时），您应该将相应签名视为入口点并为其指定保留规则。



### 2.4 代码优化

为了进一步缩减应用，R8 会在更深的层次上检查代码，以移除更多不使用的代码，或者在可能的情况下重写代码，以使其更简洁。下面是此类优化的几个示例：

- 如果您的代码从未采用过给定 `if/else` 语句的 `else {}` 分支，R8 可能会移除 `else {}` 分支的代码。
- 如果您的代码只在一个位置调用某个方法，R8 可能会移除该方法并将其内嵌在这一个调用点。
- 如果 R8 确定某个类只有一个唯一子类且该类本身未实例化（例如，一个仅由一个具体实现类使用的抽象基类），它就可以将这两个类组合在一起并从应用中移除一个类。
- 如需了解详情，请阅读 Jake Wharton 撰写的[关于 R8 优化的博文](https://jakewharton.com/blog/)。





### 2.1. R8 编译器

当您使用 Android Studio 3.4 或 Android Gradle 插件 3.4.0 及更高版本时，R8 是默认编译器，用于**将项目的 Java 字节码转换为在 Android 平台上运行的 DEX 格式**。

#### 2.1.1 R8 配置文件

R8 使用 ProGuard 规则文件来修改其默认行为并更好地了解应用的结构，比如充当应用代码入口点的类。虽然您可以修改其中一些规则文件，但某些规则可能由编译时工具（如 AAPT2）自动生成，或从应用的库依赖项继承而来。下表介绍了 R8 使用的 ProGuard 规则文件的来源：

| 来源                         | 位置                                                         | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Android Studio               | <module-sdk-dir>/proguard-rules.pro                          | Android Studio 会在创建模块的根目录中创建 `proguard-rules.pro` 文件。<br />默认情况下，此文件不会应用任何规则。 |
| Android Gradle Plugin        | 由AndroidGradle插件在编译时生成                              | AndroidGradle插件会生成 `proguard-android-optimize.txt`（其中包含了对大多数 Android 项目都有用的规则），并启用 [`@Keep*` 注解](https://developer.android.com/reference/androidx/annotation/Keep)。 |
| 库依赖项                     | AAR：<library-dir>/proguard.txt<br />JAR：<library-dir>/META-INF/proguard/ | 如果依赖项的AAR库使用自己的ProGuard规则发布的，R8在编译项目时会自动应用其规则。<br />不过由于ProGuard规则是累加的，因此AAR库依赖项包含的某些规则无法移除，并且可能会影响对应用其他部分的编译。例如，如果某个库包含停用代码优化功能的规则，该规则会针对整个项目停用优化功能。 |
| AAPT2 (Android 资源打包工具) | <module-dir>/build/intermediates/proguard-rules/debug/aapt_rules.txt | AAPT2会根据对应用清单中的类、布局及其他应用资源的引用，生成保留规则。例如，AAPT2会为您在应用清单中注册为入口点的每个Activity添加一个保留规则。 |
| 自定义配置文件               | <module-dir>/proguard-rules.pro                              | 您可以[添加其他配置](https://developer.android.com/studio/build/shrink-code#add-configuration)，R8会在编译时应用这些配置。 |



## 2. 混淆的规则

### 2.1 Input/Output

### 2.2 Keep



```properties
#混淆时不生成大小写混合的类名
-dontusemixedcaseclassnames
#不忽略非公共的类库
-dontskipnonpubliclibraryclasses
#混淆过程中打印详细信息
-verbose

#关闭优化
-dontoptimize
#不预校验
-dontpreverify

#Annotation注释不能混淆
-keepattributes *Annotation*
#对于NDK开发 本地的native方法不能被混淆
-keepclasseswithmembernames class * {
    native <methods>;
}
#保持View的子类里面的set、get方法不被混淆（*代替任意字符）
-keepclassmembers public class * extends android.view.View {
   void set*(***);
   *** get*();
}

#保持Activity子类里面的参数类型为View的方法不被混淆，如被XML里面应用的onClick方法
# We want to keep methods in Activity that could be used in the XML attribute onClick
-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}

#保持枚举类型values()、以及valueOf(java.lang.String)成员不被混淆
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

#保持实现Parcelable接口的类里面的Creator成员不被混淆
-keepclassmembers class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator CREATOR;
}

#保持R类静态成员不被混淆
-keepclassmembers class **.R$* {
    public static <fields>;
}

#不警告support包中不使用的引用
-dontwarn android.support.**
-keep class android.support.annotation.Keep
-keep @android.support.annotation.Keep class * {*;}
#保持使用了Keep注解的方法以及类不被混淆
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <methods>;
}
#保持使用了Keep注解的成员域以及类不被混淆
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <fields>;
}
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <init>(...);
}
```



## 4. 混淆后

### 4.1 dump.txt

混淆后类的内部结构说明

### 4.2 mapping.txt

混淆前与混淆后名称对应关系

### 4.3 seeds.txt

经过了一系列`keep`语句的保持，没有被混淆的类，成员的名称列表文件

### 4.4 usage.txt

经过压缩后被删除的没有使用的代码，方法`...`等的名称的列表文件

### 4.5 retrace 工具

混淆反推工具为`retrace.sh`(Mac平台)或者`retrace.bat`（Windows平台）
该工具在sdk根目录`\tools\proguard\bin\retrace.sh` (Windows平台类似)；
命令格式：

```bash
./retrace.sh [mapping.txt目录] [崩溃日历目录]
```



