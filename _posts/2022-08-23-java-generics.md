---
layout: article
title: Java 泛型
date: 2022-08-23 18:06 +0800
tags: java

---

在了解Java 泛型前，先认识下什么是**泛型编程**：泛型程序设计是程序设计语言的一种风格或规范，泛型允许在强类型程序设计语言中编写代码时使用一些以后才指定的类型，在实例化时作为参数指明这些类型。

<!--more-->

## Java 泛型



## 泛型的协变逆变

Java中泛型是不支持协变的，也不能逆变。但是可以使用通配符实现类似的效果。

```java
List<? extends Food> foodList = new ArrayList();
List<Apple> appleList = new ArrayList();
foodList = appleList;	//协变

List<? super Fruit> fruitList = new ArrayList();
List<Food> foodList = new ArrayList();
fruitList = foodList;	//逆变
```

