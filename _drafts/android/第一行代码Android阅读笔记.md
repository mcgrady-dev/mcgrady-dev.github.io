## 第1章

### 1.4 Logcat

#### 1.4.2 为什么使用Log而不是println()

- `System.out.println()`
  不能控制日志开关，不能添加日志标签，没有级别区分
- Logcat
  可以添加过滤器，支持日志级别



## 第2章

### 2.1 Kotlin语言简介

### Java语言运行机制

编译语言大致分为两类：**编译型语言**和**解释型语言**

#### 编译型语言

编译型语言的特点是编译器会为我们编写的源代码一次性的编译成计算机可识别的二进制文件，然后计算机再执行，像C/C++都属于编译型语言。

#### 解释型语言

解释型语言含有解释器，在运行时解释器会一行一行地读取我们编写的源代码，然后实时将这些源代码解释成计算机可识别的二进制数据后再执行。因此解释型语言通常效率会差一些，像Python和JavaScript都属于解释型语言。

那么Java属于那种类型的语言呢？

Java属于解释型语言，因为Java代码编译之后的并不是计算机可识别的二进制文件，而是只有Java虚拟机（Android中叫ART）才能识别的class文件，而Java虚拟机担当的其实就是解释器的角色，它会在程序运行时将编译后的class文件解释成计算机可识别的二进制数据后再执行，所以Java属于解释型语言。



### 2.3 变量和函数

#### 2.3.1 变量

`val (value)` 用来声明一个不可变的变量，这种变量在初始赋值之后就再也不能重新赋值，对应Java的`final`变量。
Kotlin之所以设计`val` ，是为了解决Java中`final`关键字没有被合理使用的问题。

`var (variable)` 用来声明一个可变的变量，这种变量在初始赋值之后仍然可以再被重新复制，对应Java中的非`final`变量。

Kotlin拥有出色的类型推导机制，仅使用`val`或`var`声明的变量，编译器可以推导出这个变量是什么类型。



### 2.4 程序的逻辑控制

顺序语句、条件语句、循环语句

#### 2.4.2 when条件语句

Java中switch只能传入整型或短于整型的变量作为条件，JDK1.7之后增加了对字符串变量的支持。

#### 2.4.3 循环语句

Java中最常用的`for-i`循环在Kotlin中被舍弃了，而Java中另一种`for-each`循环则被Kotlin进行了大幅度的加强，变成了`for-in`循环。

```kotlin
//[0,10] 两端都为闭区间，表示0到10这两个端点都包含在区间中
val range = 0..10

//[0,10) 左闭右开区间，表示0到10，包含0但不包含10的区间
val rnage = 0 until 10
```



### 2.5 面向对象编程

#### 2.5.2 继承与构造函数

在Kotlin中任何以个非抽象类默认都是不可以被继承的，相当于Java中给类声明了`final`关键字。之所以这么设计，其实和`val`关键字的原因差不多。

> Effective Java：如果一个类不是专门为继承而设计的，那么久应该主动将它加上`final`声明，禁止它可以被继承。

在类前面添加`open`关键字，即可允许被继承

```kotlin
open class Person {
  ...
}
```

Kotlin中继承的写法：

```kotlin
class Student : Person() {
  ...
}
```

##### 为什么被继承的类后面要加上括号？

任何一个面向对象的编程语言都会有构造函数的概念，Kotlin中也有，但是Kotlin将构造函数分成了两种：**主构造函数**和**次构造函数**。

主构造函数将会是你最常用的构造函数，每个类默认都会有一个不带参数的主构造函数，当然你也可以显式地给它指明参数。主构造函数的特点是没有函数体，直接定义在类名的后面即可。比如下面这种写法：

```kotlin
class Student(val sno: String, val grade: Int) : Person() {
}
```

由于构造函数中的参数是创建实例的时候传入的，不像之前的写法那样还得重新赋值，因此我们可以将参数全部声明成`val`。

如果想在主构造函数中编写一些逻辑，可以使用`init`结构体处理：

```kotlin
class Student(val sno: String, val grade: Int) : Person() {
  init {
    ...
  }
}
```

所以，被继承的类后面的括号，涉及了Java继承特性中的一个规定，**子类中的构造函数必须调用父类中的构造函数**，这个规定在Kotlin中也要遵守。

#### 2.5.3 接口

接口是用于实现多态编程额重要组成部分。

```kotlin
fun main() {
  val student = Student("Jack", 19)
  doStudy(student)
}

fun doStudy(study: IStudy) {
  study.readBooks()
  study.doHomework()
}
```

以上演示了多态编程的特性，也叫做面向接口编程。

##### 函数的可见性修饰符

在Kotlin中`public`修饰符是默认项，而在Java中`default`才是默认项。

| 修饰符    | Java                               | Kotlin             |
| --------- | ---------------------------------- | ------------------ |
| public    | 所有类可见                         | 所有类可见（默认） |
| private   | 当前类可见                         | 当前类可见         |
| protected | 当前类、子类、同一包路径下的类可见 | 当前类、子类可见   |
| default   | 同一包路径下的类可见（默认）       | 无                 |
| internal  | 无                                 | 同一模块中的类可见 |



#### 2.5.4 数据类和单例类

数据类通常需要重写`equals()` `hashCode()` `toString()` 这几个方法：

- equals(): 用于判断两个数据类是否相等
- hashCode(): 作为equals()的配套方法，用于HashMap、HashSet等hash相关的系统类内部操作
- toString(): 用于提供清晰的输入日志



### 2.6 Lambda基础

#### 2.6.1 集合的创建于遍历

Kotlin中**不可变集合**指的是该集合只能用于读取，无法对集合进行添加、修改或删除。`listOf()`函数创建的就是一个不可变的集合。这么设计的理由和`val`关键字、类默认不可继承的设计初衷类似。**可见Kotlin在不可变性方面控制得极其严格**。

来看下如何在mapOf()函数中初始化键值对集合：

```kotlin
val map = mapOf("Apple" to 1, "Banana" to 2, "Orange" to 3)
```

这里的`to`并不是关键字，而是一个`infix`函数。

来看下如何通过`for-in`遍历Map集合中的数据：

```kotlin
for ((fruit, number) in map) {
  println("fruit is $fruit , number is $number")
}
```



#### 2.6.2 集合的函数式API

Lambda表达式的语法结构：

```kotlin
{参数名1: 参数类型，参数名2: 参数类型 -> 函数体}
```

首先最外层是一对大括号，如果有参数传入到Lambda表达式的话，还需要声明参数列表，参数列表的结尾使用`->`符号，表示参数列表的结束以及函数体的开始，函数体最后一行代码会自动作为Lambda表达式的返回值。

##### 2.6.2.1 maxBy

下面通过`maxBy`函数来一步步推导演化的方式，来展示这些Lambda简化版的写法。

首先，`maxBy`函数接收了一个Lambda类型的参数，并且在遍历集合时将每次遍历的值作为参数传递给Lambda表达式。`maxBy`函数的工作原理是根据我们传入的条件来遍历集合，从而找到该条件下的最大值。

现在我们开始套用Lambda表达式的语法结构，并将它传入maxBy函数中：

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape", "Watermelon")
val lambda = { fruit: String -> fruit.length }
val maxLengthFruit = list.maxBy(lambda)
```

下面开始对`maxBy`的Lambda表达式进行简化

第一步，将lambda表达式传入`maxBy`函数中，简化如下：

```kotlin
val maxLengthFruit = list.maxBy({ fruit: String -> fruit.length})
```

第二步，Kotlin中Lambda参数是最后的一个参数时，可以将Lambda表达式移到函数括号的外面，简化如下：

```kotlin
val maxLengthFruit = list.maxBy() { fruit: String -> fruit.length }
```

第三步，如果Lambda参数是函数的唯一的参数，还可以将函数的括号省略，简化如下：

```kotlin
val maxLengthFruit = list.maxBy { fruit: String -> fruit.length }
```

最后，当Lambda表达式的参数中只有一个参数时，也不必声明参数名，可以使用`it`关键字来代替，简化如下：

```kotlin
val maxLengthFruit = list.maxBy { it.length }
```

##### 2.6.2.2 map

集合中`mp`函数是最常见的一种函数式API，它用于将集合中的每个元素都映射成一个另外的值，映射的规则在Lambda表达式中指定，最终生成一个新的集合。

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape", "Watermelon")
val newList = list.map { it.toUpperCase() }
for (fruit in newList) {
  println(fruit)
}
```

##### 2.6.2.2 filter

`filter`函数是用来过滤集合中的数据的，它可以单独使用，也可以配合其它API函数一起使用。

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape", "Watermelon")
val newList = list.filter { it.length <= 5 }
								.map { it.toUpperCase() }
for (fruit in newList) {
  println(fruit)
}
```

##### 2.6.2.3 any 和 all

`any`函数用于判断集合中是否至少存在一个元素满足条件。

`all`函数用于判断集合中是否所有元素都满足条件。

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape", "Watermelon")
val anyResult = list.any { it.lenght <= 5 }
val allResult = list.all { it.lenght <= 5 }
println("anyResult is $anyResult , allResult is $allResult")
```



#### 2.6.3 Java函数式API的使用

Kotlin中调用Java方法时也可以使用函数式API，不过有一定的条件限制，Kotlin中调用的Java方法接收了一个Java单抽象方法接口参数时，就可以使用函数式API。**Java单抽象方法接口指的是接口中只有一个待实现方法**，如果接口中有多个待实现方法，则无法使用函数式API。

举例：

```java
public interface Runnable {
  void run();
}

new Thread(new Runnable() {
  @Override
	public void run() {
    println("Thread is running");
  }
}).start();
```

这里创建了一个Runnable接口的匿名类实例，并将它传给了Thread类的构造方法。

而直接翻译成Kotlin版本，写法如下：

```kotlin
Thread(object : Runnable {
  override fun run() {
    println("Thread is running")
  }
}).start()
```

Kotlin中匿名类的写法和Java有一点区别，由于Kotlin完全舍弃了`new`关键字，因此创建匿名类实例的时候就不能再使用`new`了，而是改用了`object`关键字。相比于Java的匿名类写法，并没有什么简化之处。

第一步，目前Thead类的构造方法是符合Java函数式API的使用条件的，下面来看看如何进行简化：

```kotlin
Thread(Runnable {
  println("Thread is running")
}).start()
```

Runnable类中只有一个待实现的方法，即使没有显式的重写`run()`方法，Kotlin也能明白Runnable后面的Lambda表达式就是在`run()`方法中实现的内容。

第二步，**如果一个Java方法的参数列表有且仅有一个Java单抽象方法接口参数，还可以将接口名进行省略**，简化如下：

```kotlin
Thread({
  println("Thread is running")
}).start()
```

第三步，**当Lambda表达式时方法的最后一个参数时，可以将Lambda表达式移到方法括号的外面**，同时，**如果Lambda表达式还是方法的唯一一个参数，还可以将方法的括号忽略**，简化如下：

```kotlin
Thread {
  println("Thread is running")
}.start()
```

最后对比下Java下的Lambda的写法：

```java
new Thread(() -> {
  println("Thread is running");
}).start();
```



### 2.7 空指针检查

#### 2.7.1 可空类型系统

Kotlin利用编译时判空检查的机制几乎杜绝了在Java中需要通过主动逻辑判断避免的空指针异常。

**Kotlin默认所有的参数和变量都不可为空，并且将空指针异常的检查提前到了编译期**，如果我们的程序存在空指针异常的风险，那么在编译的时候会直接报错，这样就可以尽可能的保证运行期不会出现空指针异常。

#### 2.7.2 判空的辅助工具

##### 2.7.2.1 `?.` 操作符

当对象不为空时正常调用相应的方法，当对象为空时则什么都不做。

```kotlin
a?.doSomething()
```

##### 2.7.2.2 `?:` 操作符

`?:` 操作符的左右两边都接收一个表达式，如果左边表达式的结果不为空则返回左边的表达式结果，否则返回右边表达式的结果。

```kotlin
fun getTextLength(text: String?) = text?.length ?: 0
```

##### 2.7.2.3 `!!`非空断言操作符

这是一种有风险的写法，确信了对象不会为空，可以告诉编译器不用做空指针检查

```kotlin
fun getTextLength(text: String?) = text!!.length
```

##### 2.7.2.4 `let` 函数

`let`既不是操作符，也不是关键字，而是一个函数，它提供了函数式API的编程接口，并将原始调用对象作为参数传递到Lambda表达式中。

```kotlin
text?.let { it ->
  ...
}
```

`let`函数属于Kotlin中的标准函数，`let`函数的特性通过与`?.`操作符的配合，可以在空指针检查的时候起到很大的作用。



## 第3章

### 3.3 使用Intent在Activity之间穿梭

#### 3.3.1 使用显式Intent

Intent是Android程序中各组件之间进行交互的一种重要方式，它不仅可以指明当前组件想要执行的动作，还可以在不同组件之间传递数据。Intent一般可用于启动Activity、Service或发送广播等场景。

Intent大致可分为两种：显式Intent和隐式Intent。

```kotlin
startActivity(Intent(this, SampleActivity::class.java))
```

#### 3.3.2 使用隐式Intent

相对与显式Intent，隐式Intent并不明确指出想要启动哪个Activity，而是指定了一些列更为抽象的`action`和`category`等信息，然后交由系统去分析这个Intent，并找出合适的Activity去启动。

**什么是合适的Activity？**

通过<activity>标签下配置<intent-filter>内容，可以指定当前Activity能够响应的`action`和`category`。每个Intent中可以指定一个`action`，但能指定多个`category`。

```xml
<activity android:name=".SampleActivity">
  <intent-filter>
    <action android:name="com.example.activity.ACTION_START" />

    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="com.example.activity.MY_CATEGORY" />
  </intent-filter>
</activity>
```

```kotlin
val intent = Intent("com.example.activity.ACTION_START")
intent.addCategory("com.example.activity.MY_CATEGORY")
startActivity()
```

#### 3.3.3 更多隐式Intent的用法

<intent-filter>标签中还可以配置一个<data>标签，用于更精确的指定当前Activity能够响应的数据。<data>标签可以配置以下内容：

- android:scheme
  用于指定数据的协议部分，例如`https`部分
- android:host
  用于指定数据的主机部分，例如www.baidu.com部分
- android:port
  用于指定主机和端口之后的部分
- android:path
  用于指定主机名和端口之后的部分，例如一段网址中跟在域名后面的内容
- android:mimeType
  用于指定可以处理的数据类型，允许使用通配符的方式进行指定

当<data>标签中指定的内容和Intent中携带的`data`完全一致时，当前Activity才能够响应该Intent。



### 3.4 Activity生命周期

#### 3.4.1 返回栈

Android是使用任务（`task`）来管理Activity的，一个任务就是一组存放在栈里的Activity的集合，这个栈也被称作返回栈（`back stack`）。栈是一种后进先出的数据结构，在默认情况下，每当我们启动了一个新的Activity，它就会在返回栈中入栈，并处于栈顶的位置。而每当我们按下Back键或调用`finish()`方法去销毁一个Activity时，处于栈顶的Activity就会出栈，前一个入栈的Activity就会重新处于栈顶的位置。系统总是会显示处于栈顶的Activity给用户。

#### 3.4.2 Activity状态

每个Activity在其生命周期中最多可能会有4种状态。

##### 运行状态

当一个Activity位于返回栈的栈顶时，Activity就处于运行状态。

##### 暂停状态

当一个Activity不再处于栈顶位置，但仍然可见时，Activity就进入了暂停状态。并不是每一个Activity都会占满整个屏幕，比如Dialog形式的Activity只会占用屏幕中间的部分区域。处于暂停状态的Activity仍然是完全存活着的，系统也不愿意回收这种Activity（因为它还是可见的，回收可见的东西都会在用户体验方面有不好的影响），只有在内存极低的情况下，系统才会去考虑回收这种Activity。

##### 停止状态

当一个Activity不再处于栈顶位置，并且完全不可见的时候，就进入了停止状态。系统仍然会为这种Activity保存相应的状态和成员变量，但是这并不是完全可靠的，当其他地方需要内存时，处于停止状态的Activity有可能会被系统回收。

##### 销毁状态

一个Activity从返回栈中移除后就变成了销毁状态。系统最倾向于回收处于这种状态的Activity，以保证手机的内存充足。

#### 3.4.3 Activity的生存期

##### onCreate()

这个方法会在Activity第一次被创建的时候调用。

##### onStart()

这个方法在Activity由不可见变为可见的时候调用。

##### onResume()

这个方法在Activity准备好和用户进行交互的时候调用。此时的Activity一定位于返回栈的栈顶，并且处于运行状态。

##### onPause()

这个方法在系统准备去启动或者恢复另一个Activity的时候调用。我们通常会在这个方法中将一些消耗CPU的资源释放掉，以及保存一些关键数据，但**这个方法的执行速度一定要快，不然会影响到新的栈顶Activity的使用**。

##### onStop()

这个方法在Activity完全不可见的时候调用。它和`onPause()`方法的主要区别在于，**如果启动的新Activity是一个对话框式的Activity，那么`onPause()`方法会得到执行，而`onStop()`方法并不会执行**。

##### onDestroy()

这个方法在Activity被销毁之前调用，之后Activity的状态将变为销毁状态。

##### onRestart()

**这个方法在Activity由停止状态变为运行状态之前调用**，也就是Activity被重新启动了。

##### 完整生存期

Activity在onCreate()方法和`onDestroy()`方法之间所经历的就是完整生存期。一般情况下，一个Activity会在`onCreate()`方法中完成各种初始化操作，而在onDestroy()方法中完成释放内存的操作。

##### 可见生存期

Activity在`onStart()`方法和`onStop()`方法之间所经历的就是可见生存期。**在可见生存期内，Activity对于用户总是可见的，即便有可能无法和用户进行交互**。我们可以通过这两个方法合理地管理那些对用户可见的资源。比如在`onStart()`方法中对资源进行加载，而在`onStop()`方法中对资源进行释放，从而保证处于停止状态的Activity不会占用过多内存。

##### 前台生存期

Activity在`onResume()`方法和`onPause()`方法之间所经历的就是前台生存期。在前台生存期内，Activity总是处于运行状态，**此时的Activity是可以和用户进行交互的**，我们平时看到和接触最多的就是这个状态下的Activity。



### 3.6 Activity的最佳实现

#### 3.6.1 知晓当前是哪个Activity

```kotlin
open class BaseActivity : AppCompatActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    Log.d("TAG", javaClass.simpleName)
  }
}
```

Kotlin中的`javaClass`表示获取当前实例的Class对象，相当于在Java中调用`getClass()`方法；而Kotlin中的`BaseActivity::class.java`表示获取BaseActivity类的Class对象，相当于在Java中调用`BaseActivity.class`。



### 3.7 Kotlin课堂： 标准函数和静态方法

#### 3.7.1 标准函数 with、run和apply

Kotlin的标准函数指的是`Standard.kt`中定义的函数，任何Kotlin代码都可以调用。

##### with函数

```kotlin
public inline fun <T, R> with(receiver: T, block: T.() -> R): R
```

`with`函数接收两个参数：第一个参数可以是一个任意类型的对象，第二个参数是一个Lambda表达式。`with`函数会在Lambda表达式中提供第一个参数对象的上下文，并使用Lambda表达式中的最后一行代码作为返回值返回。作用：**可以在连续调用同一个对象的多个方法时让代码变得更加精简**，看下面的例子：

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grap")
val result = with(StringBuilder()) {
  append("Start eating fruits.\n")
  for (fruits in list) {
    append(fruits).append("\n")
  } 
  append("Ate all fruits.")
  toString()
}
println(result)
```

##### run函数

```kotlin
public inline fun <R> run(block: () -> R): R
public inline fun <T, R> T.run(block: T.() -> R): R
```

`run`函数的用法和使用场景其实和`with`函数是非常类似。首先**`run`函数通常不会直接调用，而是要在某个对象的基础上调用**；其次`run`函数只接收一个Lambda参数，并且会在Lambda表达式中提供调用对象的上下文。其他方面和`with`函数是一样的，包括也会使用Lambda表达式中的最后一行代码作为返回值返回。那么现在使用`run`函数来改造上面`with`函数示例的代码，如下所示：

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grap")
val result = StringBuilder().run {
  append("Start eating fruits.\n")
  for (fruits in list) {
    append(fruits).append("\n")
  } 
  append("Ate all fruits.")
  toString()
}
println(result)
```

##### apply函数

```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T
```

`apply`函数和`run`函数也是极其类似的，都要**在某个对象上调用，并且只接收一个Lambda参数**，也会在Lambda表达式中提供调用对象的上下文，但是**`apply`函数无法指定返回值，而是会自动返回调用对象本身**。示例代码如下：

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grap")
val result = StringBuilder().apply {
  append("Start eating fruits.\n")
  for (fruits in list) {
    append(fruits).append("\n")
  } 
  append("Ate all fruits.")
  toString()
}
//result is StringBuilder
println(result.toString())
```

#### 3.7.2 定义静态方法

`companion object`关键字实际上会在类的内部创建一个伴生类，而`companion object`中定义的方法就是这个半生类里的实例方法，Kotlin会保证类始终只存在一个半生类对象。

由此可以看出，Kotlin中没有直接定义静态方法的关键字，极度弱化了静态方法这个概念，却提供了一些语法特性来支持类似于静态方法调用的写法。

Kotlin仍然提供了两种实现方式：

- 注解：给单例类或`companion object`中的方法加上`@JvmStatic`注解，那么Kotlin编译器就会将这些方法编译成真正的静态方法，如下所示：

  ```kotlin
  class Util {
    companion object {
      @JvmStatic
      fun doSomething() {
        ...
      }
    }
  }
  ```

  由于`doSomething()`方法已经成为了真正的静态方法，那么现在不管是在Kotlin中还是在Java中，都可以使用`Util.doSomething()`的写法来调用了。

- 顶层方法：指的是那些没有定义在任何类中的方法，首先创建一个`Hello.kt`的Kotlin文件（注意创建类型选择File），在文件中定义任何方法都会是顶层方法，如下所示：

  ```kotlin
  //Hello.kt
  fun doSomething() {
   ...
  }
  ```

  在Kotlin中，所有顶层方法都可以在任何位置被直接调用，不用管报名路径，也不用创建实例，直接键入方法名即可。但由于Java中没有顶层方法这个概念（所有的方法必须定义在类中），所以在Java中不能直接访问顶层方法，而是通过Kotlin编译器对包含顶层方法的文件自动创建的`<FileName>Kt.java`类，其中顶层方法就是以静态方法的形式定义在里面的。

  ```java
  public class JavaTest {
    HelloKt.doSomething();
  }
  ```

  

## 第4章

#### 4.3.1 LinearLayout

`android:layout_gravity` 与 `android:gravity` 的区别：

- `android:gravity` 用于指定文字在控件中的对齐方式
- `android:layout_gravity` 用于指定控件在布局中的对齐方式

### 4.4 创建自定义控件

所有的控件都是直接或间接的继承自View，所有的布局都是直接或间接的继承自ViewGroup。View是Android中最基本的一种UI组件，它可以在屏幕上绘制一块矩形区域，并能响应这块区域的各种事件，因此，我们使用的各种控件其实就是在View的基础上又添加了各自特有的功能。而ViewGroup则是一种特殊的View，它可以包含很多子View和子ViewGroup，是一个用于放置控件和布局的容器。



### 4.8 Kotlin课堂：延迟初始化和密封类

#### 4.8.1 对变量延迟初始化

当时用`lateinit`关键字声明的变量在没有初始化情况下使用它，程序会抛出`UninitializedPropertyAccessException`异常，但是我们可以通过代码来判断一个全局变量是否完成了初始化，这样在某些时候能够有效的避免重复对某个变量进行初始化操作，示例代码如下：

```kotlin
private lateinit var adapter: Adapter

override fun onCreate(savedInstanceState: Bundle?) {
  if (!::adapter.isInitialzed) {
    adapter = Adapter()
  }
}
```

#### 4.8.2 使用密封类优化代码

首先理解密封类具体作用之前，我们来看一个简单的例子：

```kotlin
interface Result
class Success(val msg: String): Result
class Failure(val error: Exception): Result

fun getResultMsg(result: Result) = when (reslut) {
  is Success -> result.msg
  is Failure -> result.error.message
  else -> thorw IllegalArgumentException()
}
```

如果不写`else`条件，Kotlin编译器会认为这里缺少条件分支，代码将无法编译通过，但实际上Result的执行结果只可能是Success或者Failure，`else`条件是永远不走的，所以这里抛出了一个异常只是为了满足Kotlin编译器的语法检查而已。

另外，编写`else`条件还有一个潜在的风险。如果现在新增了一个Unknown类并实现Result接口，用于表示未知的执行结果，getResultMsg()`方法中没有添加相应的条件分支时，编译器是不会提醒我们的，而是会在运行的时候进入`else`条件抛出异常并导致程序崩溃。

当然，这种为了满足编译器的要求而编写无用条件分支的情况不仅在Kotlin当中存在，在Java或者是其他编程语言当中也普遍存在。

下面我们来看看Kotlin的密封类是怎么解决这个问题的

密封类的关键字是 `scaled class`，通过改造实例如下：

```kotlin
scaled class Result
class Success(val msg: String): Result()
class Failure(val error: Exception): Result()

fun getResultMsg(result: Result) = when (reslut) {
  is Success -> result.msg
  is Failure -> result.error.message
}
```

这里去掉了`else`条件仍能编译通过的原因是，当`when`语句中传入一个密封类变量作为条件时，Kotlin编译器会自动检查该密封类有哪些子类，并强制要求你将每一个子类所对应的条件全部处理，这样就可以保证即使没有编写`else`条件，也不会出现漏写条件分支的情况。这就是密封类主要的作用和使用方法。

另外，密封类及其所有子类只能定义在同一个文件的顶层位置，不能嵌套在其他类中，这是被密封类底层的实现机制所限制的。



## 第5章

### 5.3 Fragment的声明周期

#### 5.3.1 Fragment的状态和回调

##### 运行状态

当一个Fragment所关联的Activity正处于运行状态时，该Fragment也处于运行状态。

##### 暂停状态

当一个Activity进入暂停状态时（由于另一个未占满屏幕的Activity被添加到了栈顶），与它相关联的Fragment就会进入暂停状态。

##### 停止状态

当一个Activity进入停止状态时，与它相关联的Fragment就会进入停止状态，或者通过调用FragmentTransaction的remove()、`replace()`方法将Fragment从Activity中移除，但在事务提交之前调用了`addToBackStack()`方法，这时的Fragment也会进入停止状态。总的来说，进入停止状态的Fragment对用户来说是完全不可见的，有可能会被系统回收。

##### 销毁状态

Fragment总是依附于Activity而存在，因此当Activity被销毁时，与它相关联的Fragment就会进入销毁状态。或者通过调用FragmentTransaction的`remove()`、`replace()`方法将Fragment从Activity中移除，但在事务提交之前并没有调用`addToBackStack()`方法，这时的Fragment也会进入销毁状态。



### 5.6 Kotlin学堂：扩展函数和运算符重载

#### 5.6.1 扩展函数

```kotlin
object StringUtil {
  fun lettersCount(str: String): Int {
    var count = 0
    for (char in str) {
      if (char.isLetter) {
        count++
      }
    }
    return count
  }
}
```

```kotlin
val str = "ABC123xyz!@#"
val count = StringUtil.lettersCount(str)
```

这中写法是Java编程标准的实现思维，但有了扩展函数之后就不一样了，我们可以使用一种**更加面向对象的思维**来实现这个功能，比如将**lettersCount()**函数添加到String类中。

现在在`String.kt`文件中编写如下代码：

```kotlin
fun String.lettersCount(): Int {
  var count = 0
  for (char in this) {
    if (char.isLetter) {
      count++
    }
  }
  return count
}
```

```kotlin
val count = "ABC123xyz!@#".lettersCount()
```

扩展函数在很多情况下可以让API变得更加简洁、丰富，更加面向对象。

#### 5.6.2 有趣的运算符重载

运算符重载是Kotlin提供的一个比较有趣的语法糖。我们知道，Java中有许多语言内置的运算符关键字，如`+ - * / % ++ --`。而Kotlin允许我们将所有的运算符甚至其他的关键字进行重载，从而拓展这些运算符和关键字的用法。

Kotlin的运算符重载允许我们让**任意两个对象进行相加，或者是进行更多其他的运算操作**，这颠覆了之前在Java中，比如两个数字相加表示求这两个数字之和，两个字符串相加表示对这两个字符串进行拼接，的这种基本用法。

运算符重载使用的是`operator`关键字，在指定函数的前面加上该关键字，就可以实现运算符重载的功能。

![](../../blog/images/kotlin/kotlin-operator.png)



## 第6章

### 6.1 广播机制简介

**标准广播**：是一种完全异步执行的广播，广播发出后，所有的BroadcastReceiver几乎同一时刻收到这条广播消息，因此它们没有任何顺序可言。

```java
public void sendBroadcast(Intent intent)
```

**有序广播**：是一种同步执行的广播，同一时刻只会有一个BroadcastReceiver能够收到这条广播消息，当这个BroadcastReceiver中的逻辑执行完毕后，广播才会继续传递。所以此时的BroadcastReceiver是有先后顺序的，优先级高的BroadcastReceiver就可以先收到广播消息，并且前面的BroadcastReceiver还可以截断正在传递的广播，这样后面的BroadcastReceiver就无法收到广播消息了。

```java
public void sendOrderedBroadcast(Intent intent, @Nullable String receiverPermission)
```



#### 6.2.2 静态注册广播

由于大量恶意的应用程序利用这个机制在程序未启动的情况下监听系统广播，从而使任何应用都可以频繁地从后台被唤醒，严重影响了用户手机的电量和性能，因此Android系统几乎每个版本都在削减静态注册BroadcastReceiver的功能。

在Android 8.0系统之后，所有隐式广播都不允许使用静态注册的方式来接收了。隐式广播指的是那些没有具体指定发送给哪个应用程序的广播，大多数系统广播属于隐式广播，但是少数特殊的系统广播目前仍然允许使用静态注册的方式来接收。这些特殊的系统广播列表详见https://developer.android.google.cn/guide/components/broadcast-exceptions.html。

Android 系统为了保护用户设备的安全和隐私，做了严格的规定：**如果程序需要进行一些对用户来说比较敏感的操作，必须在`AndroidManifest.xml`中进行权限声明，否则程序将会直接崩溃**。

以接收系统开机广播`android.intent.action.BOOT_COMPLETED`为例，需要声明`android.permission.RECEIVE_BOOT_COMPLETED`权限。



### 6.5 Kotlin课堂：高阶函数详解

#### 6.5.1 定义高阶函数

##### 定义

如果一个函数接收另一个函数作为参数，或者返回值的类型是另一个函数，那么该函数就称为高阶函数。

##### 作用

高阶函数允许让函数类型的参数来决定函数的执行逻辑。即使是同一个高阶函数，只要传入不同的函数类型参数，那么它的执行逻辑和最终的返回结果就可能是完全不同的。

#### 6.5.2 内联函数的作用

Kotlin编译器会将这些高阶函数的愈发转换成Java支持的语法结构。

```kotlin
fun num1Andnum2(num1: Int, num2: Int, operation: (Int, Int) -> Int): Int = operation(num1, num2)

fun main() {
  val result = num1Andnum2(10, 10) { n1, n2 ->
    n1 + n2
	}
}
```

上述Kotlin代码大致会被转换成如下Java代码：

```java
public static int num1Andnum2(int num1, int num2, Function operation) {
  int result = (int) operation.invoke(num1, num2);
  return result;
}

public static void main() {
  int num1 = 100, num2 = 80;
  //Function是Kotlin内置的接口，里面有一个待实现的invoke()函数
  int result = num1Andnum2(num1, num2, new Function() {
    @Override
    public Integer invoke(Integer n1, Integer n2) {
      return n1 + n2;
    }
  });
}
```

之前的Lambda表达式在这里变成了Function接口的匿名类实现，然后在`invoke()`中实现了Lambda的逻辑，并将结果返回。

这就是Kotlin高阶函数背后的实现原理。这表明，我们每调用一次Lambda表达式，都会创建一个新的匿名类实例，当然也会造成额外的内存和性能开销。

为了解决这个问题，Kotlin提供了内联函数的功能，他可以将使用Lambda表达式带来的运行时开销完全消除。

下面来展示下内联函数的代码替换过程：

```kotlin
inline fun num1Andnum2(num1: Int, num2: Int, operation: (Int, Int) -> Int): Int = operation(num1, num2)

fun main() {
  val num1 = 10
  val num2 = 10
  val result = num1Andnum2(num1, num2) { n1, n2 ->
    n1 + n2
	}
}
```

首先，Kotlin编译器会将Lambda表达式中的代码替换到函数类型参数调用的地方：

```kotlin
inline fun num1Andnum2(num1: Int, num2: Int, operation: (Int, Int) -> Int): Int {
  return n1 + n2
}
```

接下来，内联函数中的全部代码替换到函数调用的地方：

```kotlin
fun main() {
  val num1 = 10
  val num2 = 10
  val result = num1 + num2
}
```

所以，内联函数就完全消除了Lambda表达式代开的运行时开销。



#### 6.5.3 noinlinet 与 crossinline

`inline`修饰的函数中，使用`noinline`关键字修饰的参数（Lambda表达式）会忽略内联功能。

那么为什么Kotlin还要提供一个`noinline`来排除内联功能呢？

这是因为内联的函数类型参数在编译的时候会被进行代码替换，因此它没有真正的参数属性。非内联的函数类型参数可以自由地传递给其他任何函数，因为它就是一个真实的参数，而内联的函数类型参数只允许传递给另外一个内联函数，这也是它最大的局限性。

事实上，绝大多数高阶函数是可以直接声明成内联函数的，但是也有少部分例外的情况。观察下面的代码示例：

```kotlin
inline fun runRunnable(block: () -> Unit) {
  val runnable = Runnable {
    block()
  }
  runnable.run()
}
```

编译后出现报错：

```bash
Can't inline 'block' here: it may contain non-local returns. Add 'crossinline' modifier to parameter declaration 'block'
```

 在高阶函数中创建了另外的**Lambda**或者**匿名类的实现**，并且在这些实现中调用函数类型参数，此时再将高阶函数声明成内联函数，就一定会报错。

这种情况可以借助`crossinline`关键字解决：

```kotlin
inline fun runRunnable(crossinline block: () -> Unit) {
  val runnable = Runnable {
    block()
  }
  runnable.run()
}
```

因为**内联函数的Lambda表达式中允许使用`return`关键字**，和高**阶函数的匿名类实现中不允许使用`return`关键字**之间造成了冲突。而`crossinline`关键字就像一个契约，它用于保证在内联函数的Lambda表达式中一定不会使用`return`关键字，这样冲突就不存在了，问题也就巧妙地解决了。



## 第7章

### 7.2 文件存储

#### 7.2.1 将数据存储到文件中

```kotlin
fun save(inputText: String) {
  try {
    val output = openFileOutput("data", Context.MODE_PRIVATE)
    val writeer = BuffredWriter(OutputStreamWriter(output))
    writer.use {
      it.write(inputText)
    }
  } catch (e: IOException) {
    e.printStackTrace()
  }
}
```

这里还使用了一个`use`函数，这是Kotlin提供的一个内置扩展函数。它会保证在Lambda表达式中的代码全部执行完之后自动将外层的流关闭，这样就不需要我们再编写一个`finally`语句，手动去关闭流了。

Kotlin是没有异常检查机制（checked exception）的。这意味着使用Kotlin编写的所有代码都不会强制要求你进行异常捕获或异常抛出。上述代码中的`try catch`代码块是参照Java的编程规范添加的，即使你不写`try catch`代码块，在Kotlin中依然可以编译通过。

#### 7.2.2 从文件中读取

```kotlin
fun load(): String {
  val content = StringBuilder()
  try {
    val input = openFileInout("data")
    val reader = BufferedReader(InputStreamReader(input))
    reader.use {
      reader.forEachLine {
        content.append(it)
      }
    }
  } catch (e: IOException) {
    e.printStackTrace()
  }
  return content.toString()
}
```

这里从文件中读取数据使用了一个`forEachLine`函数，这也是Kotlin提供的一个内置扩展函数，它会将读到的每行内容都回调到Lambda表达式中，我们在Lambda表达式中完成拼接逻辑即可。



### 7.6 Kotlin学堂：高阶函数的应用

代码示例一，优化前：

```kotlin
val editor = getSharedPreferences("data", Context.MODE_PRIVATE).edit()
editor.putString("name", "Tom")
editor.putInt("age", 28)
editor.putBoolean("married", false)
editor.apply()
```

代码示例一，定义高阶函数：

```kotlin
fun SharedPreferences.open(block: SharedPreferences.Editor.() -> Unit) {
  val editor = edit()
  editor.block()
  editor.apply()
}
```

代码示例一，优化后：

```kotlin
getSharedPreferences("data", Context.MODE_PRIVATE).open {
  putString("name", "Tom")
  putInt("age", 28)
  putBoolean("married", false)
}
```

代码示例二，优化前：

```kotlin
val values = ContentValues()
values.put("name", "mcgrady")
values.put("author", "George Martin")
values.put("pages", 720)
values.put("price", 20.85)
db.insert("Book", null, values)
```

代码示例二，定义高阶函数：

```kotlin
fun cvOf(vararg pairs: Pair<String, Any?>) : ContentValues = ContentValues(pairs.size).apply {
    for ((key, value) in pairs) {
        when (value) {
            null -> putNull(key)
            is String -> put(key, value)
            is Int -> put(key, value)
            is Long -> put(key, value)
            is Boolean -> put(key, value)
            is Float -> put(key, value)
            is Double -> put(key, value)
            is ByteArray -> put(key, value)
            is Byte -> put(key, value)
            is Short -> put(key, value)
            else -> {
                val valueType = value.javaClass.canonicalName
                throw IllegalArgumentException("Illegal value type $valueType for key \"$key\"")
            }
        }
    }
}
```

`vararg`关键字对应的是Java中的可变参数列表，这些参数会被赋值到使用`vararg`声明的变量上，然后使用`for-in`循环可以将传入的所有参数遍历出来。

Pair类型是一种键值对的数据结构。

Any是Kotlin中所有类的共同基类，相当于Java中的Object。



## 第8章

### 8.1 ContentProvider简介

ContentProvider主要用于在不同的应用程序之间实现数据共享的功能，它提供了一套完整的机制，允许一个程序访问另一个程序中的数据，同时还能保证被访问数据的安全性。目前，使用ContentProvider是Android实现跨程序共享数据的标准方式。

ContentProvider的用法一般有两种：一种是使用现有的ContentProvider读取和操作相应程序中的数据；另一种是创建自己的ContentProvider，给程序的数据提供外部访问接口。

#### 8.3.1 ContentResolver的基本用法

对于每一个应用程序来说，如果想要访问ContentProvider中共享的数据，就一定要借助ContentResolver类。

ContentResolver中的增删改查方法都是使用一个Uri参数，这个参数被称为内容URI。

内容URI给ContentProvider中的数据建立了唯一标识符，它主要由两部分组成：authority和path。

- `authority`是用于对不同的应用程序做区分的，一般为了避免冲突，会采用应用包名的方式进行命名。比如com.example.app.provider。
- `path`则是用于对同一应用程序中不同的表做区分的，通常会添加到`authority`的后面。比如`com.example.app.provider/table1`和`com.example.app.provider/table2`。不过，目前还很难辨认出这两个字符串就是两个内容URI，我们还需要在字符串的头部加上协议声明。

内容URI标准格式和用法如下：

```kotlin
val uri = Uri.parse("content://com.example.app.provider/table1")

val cursor = contentResolver.query(
  uri,
  projection,
  selection,
  selectionArgs,
  sortOrder
)
```

| query()方法参数 | 对应SQL部分               | 描述                             |
| --------------- | ------------------------- | -------------------------------- |
| uri             | from table_name           | 指定查询某个应用程序下的某一张表 |
| projection      | select column1, column2   | 指定查询的列明                   |
| selection       | where column = value      | 指定where的约束条件              |
| selectionArgs   | -                         | 为where中的占位符提供具体的值    |
| sortOrder       | order by column1, column2 | 指定查询结果的排序方式           |



### 8.5 Kotlin课堂：泛型和委托

#### 8.5.1 泛型的基本用法

泛型主要有两种定义方式：一种是定义泛型类，另一种是定义泛型方法，使用的语法结构都是<T>。`T`并不是固定要求的，事实上使用任何英文字母或单词都可以。

默认情况下，所有的泛型都是可以指定可空类型的，这是因为在不手动指定上界的时候，泛型的上界默认是`Any?`。而如果想让泛型的类型不可为空，只需要将泛型的上界手动指定成`Any`就可以了。

#### 8.5.2 类委托和委托属性

**委托是一种设计模式**，它的基本理念是：操作对象自己不会去处理某段逻辑，而是把工作委托给另外一个辅助对象去处理。

Kotlin中将委托功能分为了两种：类委托和委托属性

##### 类委托

类委托的核心思想在于将一个类的具体实现委托给另一个类去完成。

```kotlin
class MySet<T>(val helperSet: HashSet<T>) : Set<T> {
  override val size: Int = get() = helperSet.size
  
  override fun contains(element: T) = helperSet.contains(element)
  
  override fun containsAll(elements: Collection<T>) = helperSet.containsAll(elements)
  
  override fun isEmpty() = helperSet.isEmpty()
  
  override fun iterator() = helperSet.iterator()
}
```

以上的写法有一定的弊端，如果接口中待实现方法多的话就比较繁琐，我们来看下通过Kotlin的`by`关键字实现委托是怎么样的（Kotlin中委托使用的关键字是`by`，需要在接口声明的后面使用`by`关键字，再接上受委托的辅助对象）：

```kotlin
class MySet<T>(val helperSet: HashSet<T>) : Set<T> by helperSet {
  
}
```

可以看到这样多久免去了之前所写的一大堆模板代码了。

##### 委托属性

委托属性的核心思想是将一个属性（字段）的具体实现委托给另一个类去实现。

```kotlin
class MyClass {
  var param by Delegate()
}
```

代表这将`param`属性的具体实现委托给了Delegate类去完成，当调用`param`属性的时候会自动调用Delegate类的`getValue()`方法，当给`param`属性赋值时会自动调用Delegate类的`setValue()`方法。

下面我们看下Delegate的实现：

```kotlin
class Delegate {
  var propValue: Any? = null
  
  operator fun getValue(myClass: MyClass, prop: KProperty<*>): Any? {
    return propValue
  }
  
  operator fun setValue(myClass: MyClass, prop: KProperty<*>, value: Any?) {
    propValue = value
  }
}
```

这是一种标准的代码实现模板，在Delegate类中必须实现`getValue()`和`setValue()`方法，并且要使用`operator`关键字进行声明。

**`KProperty<*>`是Kotlin中的一个属性操作类，可用于获取各种属性相关的值**，在当前场景下用不着，但必须在方法参数上进行声明。另外，`<*>`这种泛型的写法表示你不知道或者不关心泛型的具体类型，只是为了通过语法编译而已，有点类似于Java中的`<?>`写法，至于返回值可以根据具体的实现逻辑声明成任何类型。



## 第9章

### 9.2 使用通知

#### 9.2.1 创建通知渠道

Android 8.0引入了通知渠道，就是每条通知都有属于一个对应的渠道，每个应用程序都可以自由地创建当前应用拥有哪些通知渠道，但是这些通知渠道的控制权是掌握在用户手上的。用户可以自由地选择这些通知渠道的重要程度，是否响铃、是否振动或者是否要关闭这个渠道的通知。

```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
  val channel = NotificationChannel(channelId, channelName, importance)
  manager.createNotificationChannel(channel)
}
```

##### PendingIntent

要实现通知的点击效果，需要在代码中进行相应的设置，也就是PendingIntent。

相对与Intent，PendingIntent更倾向于在某个合适的实际执行某个动作，也可以理解为延迟执行的Intent。



### 9.5 Kotlin课堂：使用`infix`函数构建更可读的语法

首先在前面的`mapOf()`函数中能使用`A to B`这样的语法结构，并不是Kotlin中的关键字，而是Kotlin提供了一种高技的语法糖特性：`infix`函数。比如`A to B`的写法，实际上等价于A.to(B)，`infix`函数只是把编程语言函数调用的语法规则调整了一下而已。

`infix`函数由于语法糖格式的特殊性，有两个比较严格的限制，只有同时满足这两点，`infix`函数的语法糖才具备使用的条件：

- 不能定义成顶层函数，必须是某个类的成员函数，可以使用扩展函数的方式将它定义到某个类当中。
- 必须接收且只能接收一个参数，至于参数类型是没有限制。



## 第10章

### 10.1 Service

Service依赖于创建Service时所在的应用程序进程，当某个应用程序进程被杀掉时，所有依赖于该进程的Service也会停止运行。

从Android 8.0系统开始，应用的后台功能被大幅削减。现在只有当应用保持在前台可见状态的情况下，Service才能保证稳定运行，一旦应用进入后台之后，Service随时都有可能被系统回收。之所以做这样的改动，是为了防止许多恶意的应用程序长期在后台占用手机资源，从而导致手机变得越来越卡。当然，如果你真的非常需要长期在后台执行一些任务，可以使用前台Service或者WorkManager。

##### 前台Service

前台Service和普通Service最大的区别就在于，它一直会有一个正在运行的图标在系统的状态栏显示，下拉状态栏后可以看到更加详细的信息，非常类似于通知的效果。

##### IntentService

用于操作耗时任务的Service，在`onHandleIntent()`方法中可以处理一些耗时逻辑，而不用担心ANR的问题。

### 10.2 Android多线程编程

#### 10.2.3 解析异步消息处理机制

Android中异步消息处理主要由4个部分组成：

##### Message

##### Handler

##### MessageQueue

##### Looper



### 10.6 Kotlion课堂：泛型的高技特性

#### 10.6.1 对泛型进行实化

了解泛型实化之前我们先了解下Java的泛型擦除机制。

在JDK1.5之前，Java是没有泛型功能的，那个时候诸如List之类的数据结构可以存储任意类型的数据，取出数据的时候也需要手动向下转型才行，这不仅麻烦，而且很危险。比如说我们在同一个List中存储了字符串和整型这两种数据，但是在取出数据的时候却无法区分具体的数据类型，如果手动将它们强制转成同一种类型，那么就会抛出类型转换异常。

于是在JDK 1.5中，Java终于引入了泛型功能。这不仅让诸如List之类的数据结构变得简单好用，也让我们的代码变得更加安全。

但是实际上，**Java的泛型功能是通过类型擦除机制来实现的**。就是说**泛型对于类型的约束只在编译时期存在，运行的时候仍然会按照JDK 1.5之前的机制来运行，JVM是识别不出来我们在代码中指定的泛型类型的**。例如，假设我们创建了一个List<String>集合，虽然在编译时期只能向集合中添加字符串类型的元素，但是在运行时期JVM并不能知道它本来只打算包含哪种类型的元素，只能识别出来它是个List。

所有基于JVM的语言，它们的泛型功能都是通过类型擦除机制来实现的，其中当然也包括了Kotlin。这种机制使得我们不可能使用`a is T/T::class.java`这样的语法，因为T的实际类型在运行的时候已经被擦除了。

然而不同的是，Kotlin提供了一个内联函数的概念，**内联函数中的代码会在编译的时候自动被替换到调用它的地方**，这样的话也就不存在什么泛型擦除的问题了，因为代码在编译之后会直接使用实际的类型来替代内联函数中的泛型声明。

##### reified关键字

那么具体怎么将泛型实化呢？首先函数必须是`inline`内联函数，其次，在声明泛型的地方必须家伙是哪个`reified`关键字来表示该泛型要进行实化。

```kotlin
inline fun <reified T> getGenericType() = T::class.java
```

#### 10.6.2 泛型实化的应用

泛型实化功能允许我们在泛型函数中获得泛型的实际类型。下面我们来举例展示下泛型实化的应用

优化前：

```kotlin
val instant = Intent(context, TestActivity::class.java)
context.startActivity(intent)
```

通过泛型实化优化：

```kotlin
inline fun <reified T> startActivity(context: Context) {
  val instant = Intent(context, T::class.java)
  context.startActivity(intent)
}
```

优化后：

```kotlin
startActivity<TestActivity>(context)
```

#### 10.6.3 泛型的协变

在学习协变和逆变之前，要先了解一个约定：一个泛型类或者泛型接口中的方法，它的参数列表是接收数据的地方，因此可以称它为`in`位置，而它的返回值是输出数据的地方，因此可以称它为`out`位置。

![kotlin-in-out](../../blog/images/kotlin/kotlin-in-out.png)

泛型协变的定义，假设定义了一个MyClass<T>的泛型类，其中A是B的子类型，同时MyClass<A>又是MyClass<B>的子类型，那么我们就可以称MyClass在T这个泛型上是协变的。

##### @UnsafeVariance 注解

我们先来看一个List的源码：

```kotlin
public interface List<out E> : Collection<E> {
  override val size: Int
  override fun isEmpty(): Boolean
  override fun containes(element: @UnsafeVariance E): Boolean
  override fun iterator(): Iterator<E>
  public operator fun get (index: Int): E
}
```

List在泛型`E`前添加了`out`关键字，说明List的泛型是协变的，原则上泛型`E`只能出现在`out`位置上，可在`contains()`方法中，泛型`E`仍出现在了`in`位置上。

这么写本身是不合法的，会出现类型转换的安全隐患，由于`contains()`的有着非常明确的逻辑，只是判断当前集合中是否包含参数中传入的这个元素，并不会修改当前集合的内容，理论上这种操作是安全的，那么如何让编译器理解这种操作是安全的呢？这里就需要在泛型`E`前加上`@UnsafeVariance`注解，这样编译器就允许泛型`E`出现在in的位置上了。

但如果滥用这个功能，导致运行时是有可能出现了类型转换异常的。

#### 10.6.4 泛型的逆变

假如定义了一个MyClass<T>的泛型类，其中A是B的子类型，同时MyClass<B>又是MyClass<A>的子类型，那么我们就可以称MyClass在T这个泛型上是逆变的。

![kotlin-泛型-协变-逆变](../../blog/images/kotlin/kotlin-泛型-协变-逆变.png)

Kotlin在提供协变和逆变功能时，就已经把各种潜在的类型转换安全隐患全部考虑进去了。只要我们严格按照其语法规则，让泛型在协变时只出现在`out`位置上，逆变时只出现在`in`位置上，就不会存在类型转换异常的情况。虽然`@UnsafeVariance`注解可以打破这一语法规则，但同时也会带来额外的风险，所以你在使用`@UnsafeVariance`注解时，必须很清楚自己在干什么才行。



## 第13章

### 13.4 LiveData

LiveData是Jetpack提供的一种响应式编程组件，它可以包含任何类型的书，并在数据发生变化的时候通知观察者。

Android比较推荐的写法是，永远只暴露不可变的LiveData给外部，这样在非ViewModel中只能观察LiveData的数据变化，而不能给LiveData设置数据。

```kotlin
class MainViewModel(countReserved: Int) : ViewModel() {
  val counter: LiveData<Int>
  	get() = _counter
  
  private val _counter = MutableLiveData<Int>()
  
  init {
    _counter.value = countReserved
  }
  
  fun plusOne() {
    val count = _counter.value ?: 0
    _counter.value = count + 1
  }
  
  fun clear() {
    _counter.value = 0
  }
}
```

#### 13.4.2 map和switchMap

##### map()

```kotlin
data class User(var firstName: String, var lastName: String, var age: Int)
```

```kotlin
class MainViewModel(countReserved: Int) : ViewModel() {

  private val userLiveData = MutableLiveData<User>()
  
  val userName: LiveData<String> = Tramsformations.map(userLiveData) { user ->
		"${user.firsetName} ${user.lastName}"
	}
}
```

##### switchMap()

`switchMap()`的工作原理就是将转换函数中返回的LiveData对象转换成另一个可观察的LiveData对象。

`switchMap()`的使用场景：如果ViewModel中的某个LiveData对象是调用另外的方法获取的，那么我们可以借助`switchMap()`方法，将这个LiveData对象转换成另外一个可观察的LiveData对象。

```kotlin
object Repository {
  fun getUser(userId: String): LiveData<User> {
    val livedata = MutableLiveData<User>()
    livedata.value = User(userId, userId, 0)
    return liveData
  }
}
```

```kotlin
class MainViewModel(countReserved: Int) : ViewModel() {
  
  private val useridLiveData = MutableLiveData<String>()
  
  val user: LiveData<User> = Transformations.switchMap(userIdLiveData) { userId ->
    Repository.getUser(userId)
  }
  
  fun getUser(userId: String) {
    userIdLiveData.value = userId
  }
}
```



### 13.5 Room

ORM(Object Relational Mapping) 对象关系映射框架，由于我们使用的编程语言是面向对象的，而使用的数据库则是关系型数据库，**将面向对象的语言和面向关系的数据库之间建立一种映射关系**，就是ORM。

#### 13.5.1 使用Room

首先Room的整体结构由Entity、Dao和Database这三个部分组成。

- Entity：用于定义封装实际数据的实体类，每个实体类都会在数据库中有一张对应的表，并且表中的列是根据实体类中的字段自动生成的。
- Dao：数据访问对象，通常会在这里对数据库的各项操作进行封装，在实际编程的时候，逻辑层只需要与Dao层进行交互，就不需要和底层数据库打交道了。
- Database：用于定义数据库中的关键信息，包括数据库的版本号、包含哪些实体类以及提供Dao层的访问实例。

```kotlin
@Database(version = 1, entities = [User::class])
abstract class AppDatabase : RoomDatabase {
  
  abstract fun userDao() : UserDao
  
  companion object {
    private var instance: AppDatabase? = null
    
    @Synchronized
    fun getDatabase(context: Context): AppDatabase {
      instance?.let {
        return it
      }
      return Room.databaseBuilder(context.applicationContext, AppDatabase::class.java, "app_database")
      .build().apply {
        instance = this
      }
    }
  }
}
```



### 13.6 WorkManager

首先我们来大概看下后台相关的API变更：

1. Android 4.4 开始AlarmManager的触发事件由原来的精准变为不精准
2. Android 5.0 加入了JobScheduler来处理后台任务
3. Android 6.0 引入了Doze和AppStandby模式用于降低手机被后台唤醒的频率
4. Android 8.0 禁止了Service的后台功能，只允许使用前台Service

这么频繁的功能和API变更，让开发者就很难受了，到底该如何编写后台代码才能保证应用程序在不同系统版本上的兼容性呢？为了解决这个问题，Google推出了WorkManager组件。WorkManager很适合用于处理一些要求定时执行的任务，它可以根据操作系统的版本自动选择底层是使用AlarmManager实现还是JobScheduler实现，从而降低了我们的使用成本。另外，它还支持周期性任务、链式任务处理等功能，是一个非常强大的工具。

WorkManager和Service并不相同，也没有直接的联系。Service是Android系统的四大组件之一，它在没有被销毁的情况下是一直保持在后台运行的。而WorkManager只是一个处理定时任务的工具，它可以保证即使在应用退出甚至手机重启的情况下，之前注册的任务仍然将会得到执行，因此WorkManager很适合用于执行一些定期和服务器进行交互的任务，比如周期性地同步数据，等等。

另外，使用WorkManager注册的周期性任务不能保证一定会准时执行，这并不是bug，而是系统为了减少电量消耗，可能会将触发时间临近的几个任务放在一起执行，这样可以大幅度地减少CPU被唤醒的次数，从而有效延长电池的使用时间。



### 13.7 Kotlin课堂：使用DSL构建转悠的语法结构

DSL的全称是领域特定语言（Domain Specific Language），它是编程语言赋予开发者的一种特殊能力，通过它我们可以编写出一些看似脱离其原始语法结构的代码，从而构建出一种专有的语法结构。



## 第14章

### 14.1 全局Context

```kotlin
class MyApplication : Application {
  companion object {
    @SuppressLint("StaticFieldLeak")
    lateinit var context: Context
  }
}
```

### 14.2 Intent传递数据

#### 14.2.2 Parcelable方式

```kotlin
import kotlinx.parcelize.Parcelize

@Parcelize
class Person(var name: String, var age: Int) : Parcelable
```



### 14.5 深色主题

Android 10 引入了深色主题特性，从而让夜间模式成为了官方支持的功能。

Force Dark适配方式，是一种让应用程序快速适配深色主题，并且几乎不用编写额外代码的方式，Force Dark的工作原理是系统会分析浅色主题应用下的每一层View，并且在这些View绘制到屏幕之前，自动将它们的颜色转成更加适合深色主题的颜色。注意，只有原本使用浅色主题的应用才能使用这种方式，如果原本使用的就是深色主题，Force Dark将不会起作用。

由于`android:forceDarkAllowed`属性是从Android 10.0（API 29）才开始有的，所以需要把主题资源声明在`values-v29`目录下：

```xml
<resource>
  <style name="AppTheme" parent="Theme.AppCompt.DayNight.NoActionBar">
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <item name="colorAccent">@color/colorAccent</item>
    <item name="android:forceDarkAllowed">true</item>
  </style>
</resource>
```

判断当前系统是否深色主题：

```kotlin
fun isDarkTheme(context: Context): Boolean {
  val flag = context.resources.configuration.uiMode and Configuration.UI_MODE_NIGHT_MASK
  return flag = Configuration.UI_MODE_NIGHT_YES
}
```

另外，由于Kotlin取消了按位运算符的写法，改成了使用英文关键字，因此上述代码中的`and`关键字其实就对应了Java的`&`运算符，而Kotlin中的`or`关键字对应了Java中的`|`运算符，`xor`关键字对应了Java中的`^`运算符。



## 第16章

##### 类型别名

类型别名为现有类型提供替代名称。 如果类型名称太长，你可以另外引入较短的名称，并使用新的名称替代原类型名。

它有助于缩短较长的泛型类型。 例如，通常缩减集合类型是很有吸引力的：

```
typealias NodeSet = Set<Network.Node>

typealias FileTable<K> = MutableMap<K, MutableList<File>>
```

你可以为函数类型提供另外的别名：

```
typealias MyHandler = (Int, String, Any) -> Unit

typealias Predicate<T> = (T) -> Boolean
```

你可以为内部类和嵌套类创建新名称：

```
class A {
    inner class Inner
}
class B {
    inner class Inner
}

typealias AInner = A.Inner
typealias BInner = B.Inner
```

类型别名不会引入新类型。 它们等效于相应的底层类型。 当你在代码中添加 `typealias Predicate<T>` 并使用 `Predicate<Int>` 时，Kotlin 编译器总是把它扩展为 `(Int) -> Boolean`。 因此，当你需要泛型函数类型时，你可以传递该类型的变量，反之亦然：

```kotlin
typealias Predicate<T> = (T) -> Boolean

fun foo(p: Predicate<Int>) = p(42)

fun main() {
    val f: (Int) -> Boolean = { it > 0 }
    println(foo(f)) // 输出 "true"

    val p: Predicate<Int> = { it > 0 }
    println(listOf(1, -2).filter(p)) // 输出 "[1]"
}
```

