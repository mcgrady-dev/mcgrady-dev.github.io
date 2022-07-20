---
layout:
title: Java Memory Manager
date: 2017-02-07 08:49 +0800
tags: [java,jvm]
---

引言: JAVA和C++之间由一堵由内存分配和垃圾收集技术所围成的高墙，墙外的人想将来，墙里面的人想出来。

对于从事C和C++ 程序开发的开发人员来说，在内存管理领域，他们既是拥有最高权力的皇帝，又是从事最基础工作的劳动人民，既拥有每一个对象的所有权又担负着每一个对象生命开始到终结的维护责任。 

对于Java程序员来说，在虚拟机的自动内存管理机制的帮助下，不再需要为每一个new 操作去写配对的`delete/free`代码，而且不容易出现内存泄漏和内存溢出问题，看起来由虚拟机管理内存一切都很美好。不过也正是因为Java 程序员把内存控制的权力交给了Java 虚拟机，一旦出现内存泄漏和溢>出方面的问题，如果不了解虚拟机是怎样使用内存的那排查错误将会成为一项异常艰难的工作。 [ ——周志明 ][深入理解Java虚拟机]

<!--more-->



## Java 内存模型

Java 内存模型（Java Memory Model，JMM）是一种虚拟机规范，定义了 JVM 在计算机内存（RAM）中的工作方式，屏蔽了各种硬件和操作系统的内存访问差异，实现让 Java 程序在各平台上都能达到一致的并发效果。

## Java 内存区域

而 Java 内存区域指的是 JVM 的运行时数据分区，与 Java 内存模型是不同的两个概念。



## 内存分配策略

一般内存粗糙的可以分为两块：
>*Java内存远比这个复杂，只是堆和栈是最重要的两块。*
### 堆（Heap）

**堆是内存中最大的一块，为线程共享，JVM启动时创建**。**主要用于存放`new`出来的对象以及数组**。而Java的垃圾回收机制管理的往往也就是这个堆内存，因此也成为GC（Grabage Collection Heap）堆。

> 此外还有方法区和常量池等内存区域。

- **Java Heap**
  在Android中，Java Heap 一般的大小上限是 16M 24M 48M 76M ，具体视手机而定。
- **Native Heap**
  Native Heap 是对于 C/C++ 直接操纵的系统堆内存，所以它的上限一般是具体的RAM的2/3左右。

### 栈（Stack）
栈，成为局部变量表，主要存储的是在非`static`的自动变量、函数参数、表达式的临时结果和函数返回值。**栈是线程私有的，生命周期与线程相同**。



## 自动垃圾回收机制

Java的垃圾回收机制最为Java语言的一大特性，**将Java堆空间内存的释放交给JVM自动处理**，无需开发人员在程序中显示调用，从而避免了因为开发人员忘记释放内存而造成的内存溢出。

其实Java自动垃圾回收主要做的事两件事：
- 内存回收
- 碎片清理

对于堆的管理，不同的语言实现的方式不同。
- **C**语言是通过库函数`malloc()`和`free()`来实现的。
- **C++**直接将对堆空间中对象的操作和分配释放到语言层次，使用`new`和`delete`语句。
- **Java**只需要开发人员在需要的时候创建就可以了，合适释放都由JVM来控制。而在Java的`Object`类中有一个`finalize()`方法，在垃圾回收期真正回收之前调用。



## java的内存分配策略

### 静态存储分配

指在编译时究竟能确定每个数据目标在运行时刻的存储空间需求，因而在编译时就可以给它们分配固定的内存空间空间。

### 栈式存储分配

在栈式存储方案中，程序对数据区的需求在编译时是完全未知的，只有到运行的时候才能够知道，但规定在运行中进入一个程序模块时，



## 堆式存储分配







**参考文献：**

[ 深入理解Java虚拟机 ](https://book.douban.com/subject/34907497/)