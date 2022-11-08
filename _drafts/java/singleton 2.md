---
{}
date: 2018-02-02 10:00:31
title: Singleton 单例模式
tags:
categories:
---

# Singleton 单例模式



## 简介

**单列模式**（singleton），是一种常用的软件设计模式，**单列对象的类必须只有一个实例存在，并且自行实例化向整个系统提供**。

## 构建方式
通常单列模式在Java中，有两种构建方式:
### 懒汉模式
指全局的单列实例在第一次被使用时构建。
```java
public class SingletonClass {
	private static SingletonClass INSTANCE = null;

	private SingletonClass() {}

	public static synchronized SingletonClass getInstance() {
		if (INSTANCE == null) {
			INSTANCE = new SingletonClass();
		}
		
		return INSTANCE;
	}
}
```

### 饿汉模式
指全局的单列实例在类装载时构建。
线程不安全，当多个线程调用`getInstance`方法时，可能会创建多个实例。
```java
public class SingletonClass {
	private final static SingletonClass INSTANCE = new SingletonClass();

	private SingletonClass() {}

	public static SingletonClass getInstance() {
		return INSTANCE;
	}
}
```

### 双重锁模式
当多线程调用时，如果多线程同时执行完了第一次检查，其中一个inrush同步代码块创建了实例，后面的线程因第二次检测不会创建新实例。
使用volatile 禁止指令重排序优化。在volatile 变量的赋值操作后面会有一个内存屏障，读操作不会被重排序到内存屏障之前。
```java
public class SingletonClass {
	private static volatile SingletonClass INSTANCE = null;

	private SingletonClass() {}

	public static SingletonClass getInstance() {
		if (INSTANCE == null) {
			synchronized(SingletonClass.class) {
				if (INSTANCE == null) {
					INSTANCE = new SingletonClass();
				}
			}
		}
		
		return INSTANCE;
	}
}
```

### 实现Serializable接口
需要重写`readResolve`方法，才能保证其反序依旧是单列：
```java
public class SingletonClass implement Serializable {
	private static SingletonClass INSTANCE = null;

	private SingletonClass() {}

	public static synchronized SingletonClass getInstance() {
		if (INSTANCE == null) {
			INSTANCE = new SingletonClass();
		}
		
		return INSTANCE;
	}

	// 如果实现了Serializable，必须重写这个方法
	private Object readResolve() throws ObjectStreamException {
		return INSTANCE;
	}
}
```

## 优缺点
### 优点
- 实例控制
  单列模式会阻止其他对象实例化其自己的单列对象副本，从而确保所有对象都访问唯一实例。
- 灵活性
  因为类控制了实例化过程，所以类可以灵活更改实例化过程。

### 缺点
- 开销
  每次对象请求引用时都要检查是否存在类的实例
- 可能的开发混淆
- 对象生存期