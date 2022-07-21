---
layout: 
title: Android Hotfix
date: 2016-09-27 11:46 +0800
tags: android

---

Android热修复（热更新）

<!--more-->



## 热修复实现方式一

通过上面的分析可以知道运行一个 Android 程序时使用到 PathClassLoader，即 BaseDexClassLoader，而 apk 中的 dex  相关文件都会存储在 BaseDexClassLoader 的 pathList 对象的 dexElements 属性中。

那么热修复的原理就是将改好 bug 的 dex  相关文件放进 dexElements 集合的头部，这样遍历时会首先比那里修复好的 dex 并找到修复好的类，因为类加载器的双亲委托模式，，旧 dex  中的存在 bug 的 class 没有机会上场。这样就能实现在没有发布新版本的情况下，修复现有的 bug class。