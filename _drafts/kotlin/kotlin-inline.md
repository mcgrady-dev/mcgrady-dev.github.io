## inline-fun（内联函数）

`inline`修饰符修饰的函数，会把方法体内的代码放到调用的地方，其主要的目的提高性能，减少对象的创建。

内联函数：

```kotlin
inline fun printByteCode() {
  println("printByteCode")
}
```

编译前：

```kotlin
fun testInline() {
  printByteCode()
}
```

编译后：

```java
public static final void testInline() {
  String var1 = "printByteCode";
  System.out.println(var1);
}
```



内联函数适用的情况：

-  `inline` 修饰符适用于把函数作为另一个函数的参数，例如高阶函数`filter`、`map`、`joinToString`或者一些独立的函数`repeat`
- `inline` 操作符适合和`reified`操作符结合在一起使用
- 如果函数体很短，使用`inline`操作符提高效率



| 函数名  | 定义inline的结构                                             | 函数体内使用的对象       | 返回值       | 是否是扩展函数 | 适用的场景                                                   |
| ------- | ------------------------------------------------------------ | ------------------------ | ------------ | -------------- | ------------------------------------------------------------ |
| `let`   | `fun <T, R> T.let(block: (T) -> R): R = block(this)`         | it指代当前对象           | 闭包形式返回 | 是             | 适用于处理不为null的操作场景                                 |
| `with`  | `fun <T, R> with(receiver: T, block: T.() -> R): R = receiver.block()` | this指代当前对象或者省略 | 闭包形式返回 | 否             | 适用于调用同一个类的多个方法时，可以省去类名重复，<br />直接调用类的方法即可，经常用于Android中RecyclerView中onBinderViewHolder中，<br />数据model的属性映射到UI上 |
| `run`   | `fun <T, R> T.run(block: T.() -> R): R = block()`            | this指代当前对象或者省略 | 闭包形式返回 | 是             | 适用于let,with函数任何场景。                                 |
| `apply` | `fun T.apply(block: T.() -> Unit): T { block(); return this }` | this指代当前对象或者省略 | 返回this     | 是             | 1. 适用于run函数的任何场景，一般用于初始化一个对象实例的时候，操作对象属性，并最终返回这个对象。 <br />2. 动态inflate出一个XML的View的时候需要给View绑定数据也会用到。 <br />3. 一般可用于多个扩展函数链式调用 4、数据model多层级包裹判空处理的问题 |
| `also`  | `fun T.also(block: (T) -> Unit): T { block(this); return this }` | it指代当前对象           | 返回this     | 是             | 适用于let函数的任何场景，一般可用于多个扩展函数链式调用      |



## inline-classes（内联类）

内联类只能在构造函数中传入一个参数，参数需要用`val`声明，编译之后，会替换为传进去的值。

```kotlin
inline class User(val name: String)

fun testInline() {
  println(User("Hello"))
}
```

 编译之后：

```java
public static final void testInline() {
  System.out.println("Hello");
}
```



## value-classes

`inline-classes` 是 `value-classes` 的子集，`value-classes` 比 `inline-classes` 会得到更多优化，现阶段（Kotlin 1.5）`value-classes` 和 `inline-classes` 一样，将来可以在构造函数中添加多个参数。

**value-classes 不能被继承**

因为`value-classes`编译后将会添加`final`修饰符，因此不能被继承，同样也不能继承其它的类。

**value-classes 可以实现接口**

```kotlin
@JvmInline
value class User(val name: String) : LoginInterface
```

当`value-classes`实现接口时，失去内联效果。

**value-classes 禁止使用 `===`，但可以使用 `==`**

操作符 `===` 用于比较对象的引用是否相等，由于内联的原因，最终会替换为传进去的值。但操作符 `==` 不受影响。



