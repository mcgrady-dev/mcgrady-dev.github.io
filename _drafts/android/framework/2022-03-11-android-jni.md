## 概述

JNI（Java Native Interface），即 Java 本地调用。通过 JNI 可以做到一下两点：

- Java 程序中的函数可以调用 Native 语言写的函数，Native 一般指的是 C/C++ 编写的函数。
- Native 程序中的函数可以调用 Java 层的函数，也就是在 C/C++ 程序中可以调用 Java 函数。

