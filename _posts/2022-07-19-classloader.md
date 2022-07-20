---
layout: 
title: ClassLoader
date: 2022-07-19 15:20 +0800
tags: [java,jvm]
---

Java ClassLoader 是 JVM 的一个部件，负责动态加载 JavaClass 到 JVM的内存空间中，即每个 JavaClass 必须由某个 ClassLoader 装载到内存。JavaClass 通常是按需加载，即第一次使用该类时才加载。

<!--more-->

## ClassLoader 的类型

Java 中的 `ClassLoader` 可以加载 `jar` 文件 和 `class` 文件，这一点在 Android 中并不适用，因为无论是 `DVM` 还是 `ART` 它们加载的不再是 `class` 文件，而是 dex 文件。

Android 中的 `ClassLoader` 类型和 Java 中的 `ClassLoader` 类型类似，也分为两种类型，分别是`系统 ClassLoader` 和`自定义 ClassLoader`。

其中 Android `系统 ClassLoader` 包括三种分别是 `BootClassLoader`、`PathClassLoader`和 `DexClassLoader`，而 Java 系统类加载器也包括3种，分别是 `Bootstrap ClassLoader`、 `Extensions ClassLoader` 和 `App ClassLoader`。

### BootClassLoader

`BootClassLoader` 是 `ClassLoader` 的内部类，继承自 `ClassLoader`，并且是一个单例对象。

Android 系统启动时会使用 `BootClassLoader` 来预加载常用类，与 Java 中的 `BootClassLoader` 不同，它并不是由 C/C++ 代码实现，而是由 Java 实现的。

### PathClassLoader

`PathClassLoader`继承自 `BaseDexClassLoader`，很明显 `PathClassLoader` 的方法实现都在 `BaseDexClassLoader` 中。

Android 系统使用 `PathClassLoader` 来加载系统类和应用程序的类，如果是加载非系统应用程序类，则会加载 `data/app/$packagename`下的 dex 文件以及包含 dex 的 apk 文件或 jar 文件，不管是加载哪种文件，最终都是要加载 dex 文件。

### DexClassLoader

`DexClassLoader` 可以加载 dex 文件以及包含 dex 的 apk 文件或 jar 文件，也支持从 SD 卡进行加载，这也就意味着 `DexClassLoader` 可以在应用未安装的情况下加载 dex 相关文件。**因此，它是热修复和插件化技术的基础。**

```java
package dalvik.system;

public class DexClassLoader extends BaseDexClassLoader {
    public DexClassLoader(String dexPath, String optimizedDirectory,
            String librarySearchPath, ClassLoader parent) {
        super(dexPath, null, librarySearchPath, parent);
    }
}
```

`DexClassLoader` 构造方法的参数要比 `PathClassLoader` 多一个 `optimizedDirectory` 参数，参数 `optimizedDirectory` 代表什么呢？应用程序在第一次被加载的时候，为了提高以后的启动速度和执行效率，Android 系统会对 dex 相关文件做一定程度的优化，并生成一个 `ODEX` 文件，此后再运行这个应用程序的时候，只要加载优化过的 `ODEX` 文件就行了，省去了每次都要优化的时间，而参数 `optimizedDirectory` 就是代表存储 `ODEX` 文件的路径，这个路径必须是一个内部存储路径。



## ClasssLoader 的继承关系

- `ClassLoader` 是一个抽象类，其中定义了 `ClassLoader` 的主要功能。`BootClassLoader` 是它的内部类。
- `SecureClassLoader`类和 `JDK8` 中的 `SecureClassLoader` 类的代码是一样的，它继承了抽象类 `ClassLoader`。`SecureClassLoader` 并不是 `ClassLoader` 的实现类，而是拓展了 `ClassLoader` 类加入了权限方面的功能，加强了 `ClassLoader` 的安全性。
- `URLClassLoader` 类和 `JDK8` 中的 `URLClassLoader` 类的代码是一样的，它继承自 `SecureClassLoader`，用来通过URl路径从 jar 文件和文件夹中加载类和资源。
- `BaseDexClassLoader` 继承自 `ClassLoader`，是抽象类 `ClassLoader` 的具体实现类，`PathClassLoader`和 `DexClassLoader` 都继承它。
