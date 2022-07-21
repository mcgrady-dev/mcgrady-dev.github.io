---
layout: 
title: Java Serialization
date: 2017-02-07 08:49 +0800
tags: java
---

Java 提供了一种称为对象序列化的机制，其中对象可以表示为一个字节序列，其中包括对象的数据以及有关对象类型和对象中存储的数据类型的信息。

序列化是将对象的状态转换为字节流；反序列化则相反。 换句话说，序列化是将 Java 对象转换为静态字节流（序列），然后我们可以将其保存到数据库或通过网络传输。

<!--more-->



## JDK类库中的序列化API

- `java.io.ObjectOutputStream`代表对象输出流。
  - `.writeObject(Object obj)`方法可对参数指定obj对象进行序列化，然后把得到的字节序列写到一个目标输出流中。
- `java.io.ObjectInputStream`代表对象输入流，
  - `.readObject()`方法从一个源输入流中兑取字节序列, 再把它们反序列化为一个对象,并将其返回。

- 只有实现了Serializable和Externalizable接口的类的对象才能被序列化。
  - `Externalizable`接口继承自`Serializable`接口，实现`Externalizable`接口的类完全由自身来控制反序列化的行为。
  - 而仅实现`Serializable`接口的类可以采用默认的序列化方式。



## 对象序列化步骤

1. 创建一个对象输出流，它可以包装一个其他类型的目标输出流，如文件输出流。
2. 通过对象输出流的`.writeObject()`方法写对象。



## 对象反序列化步骤

1. 创建一个对象输入流，它可以包装一个其他类型的源输入流，如文件输入流。
2. 通过对象输入流的`.readObject()`方法读取对象。




## Transient关键字

用来表示一个域不是该对象串行化的一部分，当一个对象被串行化的时候，`transient`类型变量的值不包括在串行化的表示中。

