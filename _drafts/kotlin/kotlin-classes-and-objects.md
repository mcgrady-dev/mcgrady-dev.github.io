---
layout: 
title: Kotlin Classes and Objects
date: 2021-02-09 14:58 +0800
tags: kotlin

---

<!--more-->

### Kotlin 的空安全设计

- 变量需要手动初始化，如果不初始化会报错
- 变量默认非空，所以初始化赋值 null 会报错，之后再次赋值 null 也会报错
- 变量用 `?` 设置为可空时，使用时因为「可能为空」又报错。

## 类和继承

### 主构造函数

1. 主构造函数 `primary constructor`
2. 默认的无参构造函数
3. 主构造函数中的可变/只读属性

```kotlin
class User {
  val name: String
  constructor(name: String) {
    this.name = name
  }
}

//or
class User constructor(name: String) {
  val name: String = name
}
```

通常情况下，主构造器中的 `constructor` 关键字可以省略

```kotlin
class User(name: String) {
  var name: String = name
}
```

但有些场景，`constructor` 是不可以省略的，例如在主构造器上使用「可见性修饰符」或「注解」时。

```kotlin
class User private constructor(name: String) {
  //修饰主构造器为私有的，外部无法调用该构造器
}
```

主构造器里的属性声明，如果在主构造器的参数声明上加 `var` 或 `val` ，就等价于在类中创建了该名称的属性`Property`，并且初始化值就是主构造器中该参数的值。

```kotlin
class User(var name: String, val id: Int) {
  
}
```



主构造器中的参数处理可以在类的属性中使用，还可以在 `init` 代码块中使用

```kotlin
class User constructor(name: String) {
  var name: String = name
  init {
    this.name = name
  }
}
```

其中 `init` 代码块是紧跟在主构造器之后执行的，这是因为主构造器本身没有代码体，所以 `init` 就充当了主构造器代码体的功能。



### 二级构造函数

1. `constructor` 关键字
2. `this` 关键字，如果类有主构造函数，每个二级构造函数都要间接或直接通过另一个二级构造函数代理主构造函数。

```kotlin
class User constructor(var name: String) {
  constructor(name: String, id: Int): this(name) {
    
  }
  
  constructor(name: String, id: Int, age: Int): this(name, id) {
    
  }
}
```

在使用次构造器创建对象时，**`init` 代码块是先于次构造器执行的**。

### 继承

1. 复写属性
2. 复写规则
3. 伴生对象
4. 密封类



## 属性和字段

### 备用字段

`field` 关键字

### 备用属性

### 编译时常量

### 延时初始化

`lateinit` 的作用是，告诉编译器这个变量没发第一时间初始化，但可以保证使用它之前完成初始化。简单来说就是，让IDE 不要对这个变量检查初始化和报错。

`lateinit` 修饰符的作用域：

1. 只能用类的在 `var` 类型的可变属性定义中
2. 属性不能有自定义的 `getter` 和 `setter` 访问器
3. 属性的类型必须是非空
4. 不能为一个基本类型
5. 不能用在构造方法中

在 `lateinit` 修饰的属性初始化前

## 可见性修饰词

Kotlin 中有四种修饰词：`private` `protected` `internal` `public`

### 包

1. 如果没有指明任何可见性修饰词，默认使用 `public`， 即声明在任何地方可见
2. `private` 只在包含声明的文件中可见
3. `internal` 在同一模块的任何地方可见
4. `protected` 在 `top-level` 中不可用

### 类和接口

1. `private` 只在该类（以及它的成员）中可见
2. `protected` 和 `private` 一样，但在子类中也可见
3. `internal` 在本模块的所有课访问到盛行区域的均可访问该类的所有 `internal` 成员
4. `public` 任何地方可见

Java使用注意：外部类不可以访问内部类的 `private` 成员。

相比  Java 少了一个 `default`「包内可见」修饰符。



## 对象表达式和声明

### 对象表达式和声明的区别

1. **对象表达式**在我们使用的地方即初始化并执行
2. **对象声明**是懒加载的，是我们第一次访问时初始化
3. **伴随对象**是在对应的类加载时初始化，和Java静态初始化是对应的

## 代理模式

### 委托实现

### 委托属性

语法结构：`var/val <property name> : <Type> by <expression>`

委托属性不需要任何接口的实现， 但必须提供 `getValue()` 方法（var 还需要提供 `setValue()` ）

### 延迟

`lazy()` 是接受一个 lamda 并返回一个 `Lazy<T>` 实例的函数，返回的实例可以作为实现延迟属性的委托，第一次调用 `get()` 执行传递给 `lazy()` 并存储接口，以后每次调用 `get()` 只是简单返回之前储存的值。

默认情况下延迟属性的计算是同步的：该值的计算只在一个线程里，其他所有线程都将读取同样的值。如果委托不需要同步初始化，而且允许出现多线程同时执行该操作，可以传 `LazyThreadSafetyMode.PUBLICATION` 参数给 `lazy()` 。

如果你想要线程安全，使用 `blockingLazy()`: 它还是按照同样的方式工作，但保证了它的值只会在一个线程中计算，并且所有的线程都获取的同一个值。

### 可观察属性

`Delegates` 是个对象， `observable` 接收两个参数：一个初始值，赋给被委托的属性，一旦委托属性的值发生变化（即调用 `set()` 方法 ）时，就会回调 lamda表达式

如果想打断赋值并 `否决` 它，就使用 `vetoable()` 取代 `observeable()`。在属性被赋新值生效之前会调用传递给 `vetoable` 的处理程序。`observable` 的回调时机是在属性值修改之后，`vetoable` 的回调时机在属性值被修改之前。如果返回值为 `true`，属性值就会被修改成新值；如果返回值为 `false`，此次修改就会直接被丢弃。

> 对比 `lazy` 委托 和 `observable` 委托： `lazy`专注于 `getValue()`，`observable`专注于 `setValue()` 。



### object

```kotlin
object Sample {
  val name = "mcgrady"
}
```

object 是 Kotlin 中的一个关键字，主要的意思是：创建一个类，并且创建这个类的的一个对象。

在代码中如果要使用这个对象，直接通过它的类名就可以，同时也可以作单例来使用

```kotlin
Sample.name
```

> 通过 `object` 实现的单例是一个饿汉式的单例，并且实现了线程安全。

#### object 的其它作用

- 继承类和实现接口

```kotlin
open class A {
  open fun method() {
    ...
  }
}

interface B {
  fun interfaceMethod()
}

object C : A(), B {
  override fun method() {
    ...
  }
  
  override fun interfaceMethod() {
    ...
  }
}
```

为什么 `object` 可以实现接口？简单来讲 object 其实是把两部合成了一步，即有 `class` 关键字的功能，又实现了单例。

- 匿名类

  Kotlin 还可以创建 Java  中的匿名类，只是写法有点不同：

  ```java
  //Java
  ViewPager.SimpleOnPageChangeListener listener = new ViewPager.SimpleOnPageChangeListener() {
    @Override
    public void onPageSelected(int position) {
      
    }
  };
  ```

  ```kotlin
  //Kotlin
  val listener = object: ViewPager.SimpleOnPageChangeListener() {
    override fun onPageSelected(positon: Int) {
      
    }
  }
  ```



### companion object 伴生对象

用 `objec  修饰的对象中的变量和函数都是静态的，但有时只想让类中的一部分函数和变量设置静态时，用` `companion object` 来实现。

```kotlin
class Sample {
  compaion object B {
    var C: Int = 0
  }
}
```

### top-level property / function 声明

除了 `object` 和 compaion object 实现静态方法，Kotlin 还提供了更便捷的 `top-level declaration` 顶级声明。其实就是把属性和函数的声明不写在 `class` 里。

这样写的属性和函数，不属于任何 `class` ，而是直接属于 `package` ，它和静态变量、静态函数一样是全局的。

顶级函数或变量还有个好处是：在 Android Studio 中写代码时，IDE 很容易根据你写的函数前几个字母自动联想出相应的函数，有效提高代码编写的效率，且可以减少项目中的重复代码。

> 如果在不同文件中声明命名相同的顶级函数，IDE 会自动加上 `package` 前缀来区分。

### object 、compaion object、top-level 对比

- 如果想写工具类的功能，直接创建文件，写 `top-level` 顶级函数
- 如果需要继承别的类或实现接口，就用 `object` 或 `compaion object`

### 常量

首先来看下 Java 与 Kotlin 里类中的常量的声明变化

```java
//Java
public class Sample {
  public static final int COUNT_NUMBER = 1;
}
```

```kotlin
//Kotlin
class Sample {
  compaion object {
    const val CONST_NUMBER = 1
  }
}
```

Kotlin 中只有基本类型和String可以声明常量，原因在于 Kotlin 中的常量是指 `compile-time constant` 编译时常量，编译器在编译期间就知道这个常量在每个调用处的实际值。

