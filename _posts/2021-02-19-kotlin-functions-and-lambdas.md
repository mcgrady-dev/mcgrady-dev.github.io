

---
layout: 
title: Kotlin Functions
date: 2021-02-19 10:31 +0800
tags: kotlin
---

<!--more-->

## infix method (中辍函数)

声明 `infix` 关键字的函数是中辍函数，调用中辍函数时可以省略圆点以及括号等程序符号，让语句更自然。

满足以下条件时，函数可以使用中辍符号：

1. 成员函数或扩展函数
2. 函数只有一个参数
3. 不能使用可变参数或默认参数

```kotlin
infix fun String.eat(fruit: String): String {
  return "${this}吃${fruit}"
}
"mcgrady" eat "Apple"
```



## 函数类型参数

在调用函数时可以以参数命名，但命名参数不能被用于Java函数中，因为Java的字节码不能确保方法参数命名的不变性

```kotlin
fun foo(
    bar: Int = 0,
    baz: Int = 1,
    qux: () -> Unit,
) { /*……*/ }
```



## 不带返回值的函数

如果函数不会返回任何有用值，默认返回的类型是 `Unit`，`Unit`是一种只有唯一值的类型，并不需要被直接返回。

```kotlin
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello $name")
    else
        println("Hi there!")
    // `return Unit` 或者 `return` 是可选的
}
```

`Unit` 返回类型声明也是可选的。上面的代码等同于：

```kotlin
fun printHello(name: String?) { …… }
```



## 可变数量的参数

函数的参数（通常最后一个参数）可以用 `vararg` 修饰符进行标记，标记后允许给函数传递可变长度的参数

```kotlin
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}
```



## 函数范围

### 局部函数

1. 局部函数可以访问外部函数的局部变量（比如闭包）
2. 局部函数可以返回到外部函数

### 内联函数

在函数体内访问外部变量，内存占用（函数对象和类都会占内存）以及虚方法调用带来运行时的消耗。

| 函数名  | 定义inline的结构                                             | 函数体内使用的对象       | 返回值       | 是否是扩展函数 | 适用的场景                                                   |
| ------- | ------------------------------------------------------------ | ------------------------ | ------------ | -------------- | ------------------------------------------------------------ |
| `let`   | `fun <T, R> T.let(block: (T) -> R): R = block(this)`         | it指代当前对象           | 闭包形式返回 | 是             | 适用于处理不为null的操作场景                                 |
| `with`  | `fun <T, R> with(receiver: T, block: T.() -> R): R = receiver.block()` | this指代当前对象或者省略 | 闭包形式返回 | 否             | 适用于调用同一个类的多个方法时，可以省去类名重复，<br />直接调用类的方法即可，经常用于Android中RecyclerView中onBinderViewHolder中，<br />数据model的属性映射到UI上 |
| `run`   | `fun <T, R> T.run(block: T.() -> R): R = block()`            | this指代当前对象或者省略 | 闭包形式返回 | 是             | 适用于let,with函数任何场景。                                 |
| `apply` | `fun T.apply(block: T.() -> Unit): T { block(); return this }` | this指代当前对象或者省略 | 返回this     | 是             | 1. 适用于run函数的任何场景，一般用于初始化一个对象实例的时候，操作对象属性，并最终返回这个对象。 <br />2. 动态inflate出一个XML的View的时候需要给View绑定数据也会用到。 <br />3. 一般可用于多个扩展函数链式调用 4、数据model多层级包裹判空处理的问题 |
| `also`  | `fun T.also(block: (T) -> Unit): T { block(this); return this }` | it指代当前对象           | 返回this     | 是             | 适用于let函数的任何场景，一般可用于多个扩展函数链式调用      |



## 高阶函数 和 lambda 表达式

**高阶函数可以接收参数作为函数或者返回一个函数的函数。**

在 Kotlin 中 函数并不能传递，传递的是对象，匿名函数和 Lambda 表达式其实都是对象。

- 在 Kotlin 中，有一类 Java 中不存在的类型，叫做「函数类型」，这类类型的对象在可以当函数来用的同时，还能作为函数的参数、函数的返回值以及赋值给变量
- 创建一个函数类型对象的三种方式：双冒号加函数名、匿名函数和 Lambda



## 尾递归函数

当函数被标记为 `tailrec` 时，编译器会优化递归，并用高效迅速的循环代替它。 

使用 `tailrec` 修饰符必须在最后一个操作调用自己。



## 构造函数

通过下面语法指定主构造函数的可见性（必须使用 `constructor` 关键字）

```kotlin
class C private constructor(a: Int) { ... }
```



## 函数可见性

`internal` 修饰符指成员的可见性只在同一个模块中才可见。



## 扩展函数

定义一个扩展，并咩偶遇修改它所扩展的类，只是让这个类的实例对象能够通过 `.` 调用新的函数。

扩展函数是静态分发的，意味着扩展函数的调用时由发起函数调用的表达式的类型决定，而不是运行时动态获得的表达式的类型决定。

### 指向扩展函数的引用

```kotlin
Int::toFloat
```

`:: 指向的并不是函数本身，而是和函数等价的一个对象。`

```kotlin
(Int::toFloat)(1)	//等价于1.toFloat()
Int::toFloat>invoke(1)	//等价于1.toFloat()
1.toFloat.invoke()	//报错
```

普通函数可以被指向，同样，扩展函数也可以被指向

```kotlin
fun String.method1(i: Int) {
  
}

String::method1
```



## field

`field` 在 Kotlin里，相当于每个 `var` 内部的一个变量。



## inline method (内联函数)

inline 关键字可以对函数进行内联优化，即调用的函数再编译期间编程代码内嵌的形式：

```kotlin
inline fun hello() {
  println("hello")
}

//调用处
fun main(args: Array<String>) {
  hello()
}
// 实际编译的代码
fun main(args: Array<String>) {
  println("hello")
}
```

`inline`关键字可以让函数内容插到调用处的方式来优化代码结构，从而减少函数类型的对象创建；实际上，`inline` 关键字不知可以内联自己的内部代码，还可以内联自己内部的内部代码，编译器在编译期间不仅会把函数内联过来，而且会把它内部的函数类型的参数（Lambda）也内联过来，例如：

```kotlin
fun hello(postAction: () -> Unit) {
  println("hello")
  postAction()
}

//调用处
fun main(args: Array<String>) {
  for (i in 1..100) {
    hello {
    println("bye bye")
    }
  }
}
//实际编译的代码
fun main(args: Array<String>) {
  for (i in 1..100) {
    println("hello")
    println("bye bye")
  }
}
```

经过这种优化，就避免了函数类型的参数所造成的临时对象的创建。

这就是 `inline` 关键字的用于：高阶函数（Higher-order Functions）有它们天然的性能缺陷，通过 `inline` 关键字让函数用内联的方式进行编译，来减少参数对象的创建，从而避免出现性能问题。

### inline 使用建议

如果写的是高阶函数，用到函数类型参数，就加上 `inline` 关键字。



## noinline method

`noinline` 关键字 是局部关掉 inline 优化，来摆脱 inline 带来的 「不能把函数类型参数当对象使用」的限制。

`noinline` 作用于函数的参数，对于一个标记了 inline 的内联函数，你可以对它的任何一个或多个函数类型的参数添加 `noinline` 关键字：

```kotlin
inline fun hello(preAction: () -> Unit, noinline postAction: () -> Unit) {
  preAction()
  println("hello")
  postAction()
  return postAction
}

//调用处
fun main(args: Array<String>) {
  hello({
    println("Hi")
  }, {
    println("bye bye")
  })
}
// 实际编译的代码
fun main(args: Array<String>) {
  println("Hi")
  println("hello")
  val postAction = ({
    println("bye bye")
  }).invoke()
  postAction
}
```

所以，noinline 的作用是用来局部的、指向性的关掉函数的内联优化，可能有些情况因为这种优化导致函数中的函数类型参数无法被当做对象使用。



## corssinline method

corssinline 关键字是局部加强内联优化，让内联函数里的函数类型参数可以被当做对象使用。

```kotlin
inline fun hello(crossinline postAction: () -> Unit) {
  println("hello")
  runOnUiThread {
    postAction()
  }
}

//调用处
fun main(args: Array<String>) {
  hello {
    println("bye bye")
    return
  }
}
```

Lambda 里的 `return`，结束的不是直接的外层函数，而是外层再外层的函数，但只有内联函数的 Lambda 参数可以使用 `return`。

