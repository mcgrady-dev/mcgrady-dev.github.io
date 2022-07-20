---
layout: 
title: Android Serialization
date: 2016-09-27 11:40 +0800
tags: android
---

Android序列化

<!--more-->



## Serializable

Serializable 是一种表示接口（marker interface），这意味着无需实现方法，Java便会对这个对象进行高效的序列化操作。
```java
public class SerializableDeveloper implements Serializable
    String name;
    int yearsOfExperience;
    List<Skill> skillSet;
    float favoriteFloat;
 
    static class Skill implements Serializable {
        String name;
        boolean programmingRelated;
    }
}
```



## Parcelable

### Parcelable 的实现

- 实现 `writeToParcel()` 指明序列化时要写出的数据
- 实现参数为 Parcel 的实例化方法，指明序列化时，要读入的数据
- 实现 `static final` 成员变量 Creator，已提供创阿金对象实例的入口

### Parcelable 中对象和集合的处理

- 集合一定要初始化



## Parcelable和Serializable的区别

**优点：**
Serializable 简单易用
Parceable 速度至上
**缺点：**
Serializable 使用了反射，序列化过程较慢。这种机制会在序列化的时候创建许多的临时对象，容易出发垃圾回收。
Parcelable 实现Parcelable接口需要大量的模板代码，这使得对象代码变得难以阅读和维护。



## 速度测试

### 测试方法
- 通过一个对象放到一个bundle里，然后调用Bundle#writeToParcel(Parcel, int)方法来模拟传递对象给一个activity的过程，然后再把这个对象取出来。
- 在一个循环里运行1000次。
- 两种方法分别运行10次减少内存管理，CPU被其他应用占用等情况的干扰。
- 参与测试的对象就是代码中的SerializableDeveloper和ParcelableDeveloper
- 在多种Android软硬环境上进行测试

### 结果
![serializable-vs-parcelable](https://s2.loli.net/2022/07/19/D5Tz1uFpE7nHjal.jpg)

**由此可以得出：**Parcelable 比 Serializable快了10多倍。有趣的是，即使在Nexus 10这样性能强悍的硬件上，一个相当简单的对象的序列化和反序列化的过程要花将近一毫秒。



## 总结

如果你想成为一个优秀的软件工程师，你需要花多点时间来实现Parcelable，因为这将会为你对象的序列化过程快10多倍，而且占用较少的资源。

但是大多情况下，Serializable的龟速不会太引人注目。你想偷点懒就用它把，不过要记得serialization是一个比较耗资源的操作，尽量少用。

如果你想传递一个包含多个对象的列表，那么整个序列化的过程的时间开销可能会超过一秒，这回让屏幕转向的时候变得很卡顿。
