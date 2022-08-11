---
layout: article
title: 'Deep understanding of JVM: Part1'
date: 2022-08-04 15:49 +0800
tags: [notes,java,jvm,深入理解Java虚拟机系列]
---

深入理解Java虚拟机系列——第1部分：走近Java

进一步地了解和思考Java技术体系中的一些优秀的技术特性，以及过去、现在和未来的发展趋势。

<!--more-->

## 1. Java技术体系

![java-language-system](https://s2.loli.net/2022/08/04/agHwI26CSzWeiBJ.png)

- **JDK（Java Development Kit）**：用于支持Java程序开发的最小环境，包含Java程序设计语言、Java虚拟机、Java类库。
- **JAR（Java ARchiv）**：一种软件包文件格式，以ZIP格式构建，通常用于聚合大量的Java类文件、相关的元数据和资源（文本、图片等）文件到一个文件，以便分发Java平台应用软件或库。
- **JRE（Java Runtime Environment）**：Java运行环境
- **JSR（Java Specification Request）**：Java规范请求作为正式规范文档，描述被提议加入到Java体系中的规范和技术。
- **JCP（Java Community Process）**：一个由业界多家技术巨头组成的社区组织，使用JSR定义和发展Java的技术规范。



## 2. Java虚拟机家族

Java虚拟机家族的发展轨迹和历史变迁回顾。

### 2.1 虚拟机始祖：Sun Classic/Exact VM

- **Classic** 是*“世界上第一款商用Java虚拟机”*，只能使用纯解释器方式来执行Java代码。
- **Exact VM** 是Sun的虚拟机团队在JDK 1.2发布的虚拟机，为了解决Classic虚拟机所面临的各种问题，提升运行效率。特点：热点侦探、两级即时编译、编译器与解释器混合工作模式等。

### 2.2 武林盟主：HotSpot VM

HotSpot是目前使用范围最广的Java虚拟机，既继承了Sun之前两款商用虚拟机的优点（如前面提到的准确式内存管理），也有许多自己新的技术优势，如名称“HotSpot”意指热点代码探测技术。

### 2.3 小家碧玉：Mobile/Embedded VM

面向移动和嵌入式的Java虚拟机产品。

### 2.4 天下第二：BEA JRockit/IBM J9 VM

- **JRockit** 是BEA System公司在2002年从Appeal Virtual Machines公司收购获得的Java虚拟机，专注于服务端应用。
- **J9** 最初是由IBM Ottawa实验室的一个SmallTalk虚拟机项目扩展而来，它是一款在设计上全面考虑服务端、桌面应用，再到嵌入式的多用途虚拟机。

### 2.5 软硬合璧：BEA Liquid VM/Azul VM 

特定硬件平台绑定、软硬件配合工作的专有虚拟机：

- **Liquid VM** 是BEA公司开发的可以直接运行在自家Hypervisor系统上的JRockit虚拟机的虚拟化版本。
- **Azul VM** 是Azul Systems公司在HotSpot基础上进行大量改进，运行于Azul Systems公司的专有硬件Vega系统上的Java虚拟机。
- **Zing** 虚拟机是一个从HotSpot某旧版代码分支基础上独立出来重新开发的高性能Java虚拟机。

### 2.6 挑战者：Apache Harmony/Google Android Dalvik VM

- **Apache Harmony** 是一个Apache软件基金会旗下以Apache License协议开源的实际兼容于JDK 5和JDK 6的Java程序运行平台，它含有自己的虚拟机和Java类库API，用户可以在上面运行Eclipse、Tomcat、Maven等常用的Java程序。

  由于某些原因，Harmony没有真正被大规模商业运用过，但它的许多代码（主要是Java类库部分）被吸纳进IBM的JDK 7实现以及Google Android SDK之中，尤其**对Android的发展起了很大的推动作用**。

- **Dalvik** 虚拟机曾经是Android平台的核心组成部分之一，名字来源于冰岛一个名为Dalvik的小渔村。

  Dalvik不能直接执行Java的Class文件，使用寄存器架构而不是Java虚拟机中常见的栈架构，它执行的DEX（Dalvik Executable）文件可以通过Class文件转化而来，使用Java语法编写应用程序，可以直接使用绝大部分的Java API等。

  发展历程：

  1. Android 2.2：开始提供即时编译[^JIT]
  2. Android 4.4：支持提前编译[^AOT]的ART虚拟机崛起
  3. Android 5.0：ART全面代替Dalvik虚拟机

### 2.7 没有成功，但并非失败：Microsoft JVM/Other

除去前面介绍的被大规模商业应用过的Java虚拟机外， 还有许多虚拟机是不为人知地默默沉寂，或者曾经绚丽过但最终夭折湮灭；其中就包括 Microsoft公司的Java虚拟机。



## 3. 展望Java技术的未来

### 3.1 无语言倾向

![graalvm-logo-320x64](https://s2.loli.net/2022/07/22/g8jPbT1cJnhUl3k.png)

**Graal VM** 被官方称为“Universal VM”*和“Polyglot VM”*，这是一个在HotSpot虚拟机基础上增强而成的**跨语言全栈虚拟机**，可以作为*“任何语言”*[^任何语言]的运行平台使用。Graal VM可以无额外开销地混合使用这些编程语言，支持不同语言中混用对方的接口和对象，也能够支持这些语言使用已经编写好的本地库文件。

从严格的角度来看，Graal VM才是真正意义上与物理计算机相对应的高级语言虚拟机，因为它与物理硬件的指令集一样，做到了只与机器特性相关而不语某种高级语言特性相关。

对Java而言，Graal VM本是在HotSpot基础上诞生的，天生就可作为一套完整的符合Java SE 8标准的Java虚拟机来使用。如果Java语言或者HotSpot虚拟机真的有被取代的一天，那从现在看来Graal VM是希望最大的一个候选项。

### 3.2 新一代即时编译器

HotSpot虚拟机中含有两个即时编译器[^AOT]，通常它们会在分层编译机制下与解释器互相配合来共同构成HotSpot虚拟机的执行子系统：

- 编译耗时短但输出代码优化程度较低的**客户端编译器**（简称为C1[^C1]）
- 编译耗时长但输出代码优化质量也更高的**服务端编译器**（简称为C2[^C2]）

自JDK 10起，HotSpot中又加入了一个全新的以C2编译器替代者的身份登场的即时编译器：**Graal编译器**。

Graal编译器本身就是由Java语言写成，开发效率和扩展性上都要显著优于C2编译器，虽然尚且年幼，还未经过足够多的实践验证，但未来的前途可期，作为Java虚拟机执行代码的最新引擎，它的持续改进，会同时为HotSpot与Graal VM注入更快更强的驱动力。

### 3.3 向Native迈进

提前编译[^JIT]是相对于即时编译[^AOT]的概念，提前编译能带来的最大好处是Java虚拟机加载这些已经预编译成二进制库之后就能够直接调用，而无须再等待即时编译器在运行时将其编译成二进制机器码。

理论上，提前编译可以减少即时编译带来的预热时间，减少Java应用长期给人带来的*“第一次运行慢”*的不良体验，可以放心地进行很多全程序的分析行为，可以使用时间压力更大的优化措施；但坏处也很明显，它破坏了Java*“Write Once，Run Anywhere”*的承诺，必须为每个不同的硬件、操作系统去编译对应的发行包；也显著降低了Java链接过程的动态性，必须要求加载的代码在编译期就是全部已知的，而不能在运行期才确定。

直到在Grall VM 0.20版本里出现的**Substrate VM**一个极小型的运行时环境，目标是代替HotSpot用来支持提前编译后的程序执行，可以直接从目标程序开始运行，而无需重复进入Java虚拟机的初始化过程。

Substrate好处是能显著降低内存占用及启动时间，并有轻量化特性，使得它十分适合嵌入其他系统，从而成为了Graal VM*“Run Programs Faster Anywhere”*愿景蓝图里的最后一块拼图。

### 3.4 灵活的胖子

HotSpot的定位是**面向各种不同应用场景的全功能Java虚拟机**，这是一个极高的要求。

经过一系列的重构与开放，HotSpot虚拟机逐渐从时间的侵蚀中挣脱出来，虽然代码复杂度还在增长，体积仍在变大，但其架构并未老朽，而是拥有了越来越多的开放性和扩展性，使得HotSpot成为一个能够联动外部功能，能够应对各种场景，身手灵活敏捷的*“胖子”*。

### 3.5 语言语法持续增强

随着Java每半年更新一次的节奏，新版本的Java中会出现越来越多其他语言里已有的优秀特性，譬如：

- 本地类型变量推断
- `switch`语句的表达式支持
- Lambda





参考文献：

[深入理解Java虚拟机：JVM高级特性与最佳实践（第3版）—周志明](https://read.douban.com/ebook/128052544/)

> 「深入理解JVM」系列为阅读笔记，如有侵权请第一时间联系删除， 谢谢！
>
> <mcgrady911@foxmail.com>



[^JIT]: Just-In-Time Compiler（即时编译器），指运行期把字节码转变成本地机器码的过程。
[^AOT]: Ahead-of-Time Compilation（提前编译器），指直接把程序编译成目标机器指令集相关的二进制代码的过程。
[^任何语言]: 这里的“任何语言”包括Java、Scala、Groovy、Kotlin等基于Java虚拟机之上的语言，还包括 C/C++、Rust等基于LLVM的语言；同时支持其他像JavaScript、Ruby、Python、R语言等。
[^C1]: 编译耗时短但输出代码优化程度较低的**客户端编译器**。
[^C2]: 编译耗时长但输出代码优化质量也更高的**服务端编译器**。
