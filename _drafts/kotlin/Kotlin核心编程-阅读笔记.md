## 第2章 基础语法

### 2.3 高阶函数和Lambda

#### 2.3.1 抽象和高阶函数

>《计算机程序的构造和解释》这本经典的书籍的开篇，有一段关于抽象这个概念的描述：心智的活动，除了尽力产生各种简单的认识外，主要表现在如下3个方面：
>
>1）将若干简单的认识组合为一个复合认识，由此产生出各种复杂的认识；
>
>2）将两个认识放在一起对照，不管它们如何简单或者复杂，在这样做时并不将它们合二为一；由此得到有关它们的相互关系的认识；
>
>3）将有关认识与那些在实际中和它们同在的所有其他认识隔离开，这就是抽象，所有具有普遍性的认识都是这样得到的。

高阶函数是以其他函数作为参数或返回值的函数。高阶函数是一种更加高级的抽象机制，极大的增强了语言的表达能力。

#### 2.3.3 函数的类型

Kotlin中函数类型声明需遵循以下几个点：

- 通过 `->` 符号来组织参数类型和返回值类型，左边是参数类型，右边是返回值类型
- 必须用一个括号来包括参数类型
- 返回值类型即使是Unit，也必须显式声明

```kotlin
() -> Unit
(Int, String) -> Unit
(errorCode: Int, message: String) -> Unit
```

#### 2.3.4 方法和成员的引用

Kotlin存在一种特殊的语法，通过两个冒号来实现对于某个类的方法进行引用。

```kotlin
contryTest::isBigEuropeanCountry
```

**为什么使用双冒号的语法？**

避免容易让人混淆方法引用表达式和成员表达式的二义性，所以沿用了Java8的使用习惯，让我们更清晰地认识这种语法。

#### 2.3.5 匿名函数

Kotlin支持在缺省函数名的情况下，直接定义一个函数：

```kotlin
fun(country: Country): Boolean {	//没有函数名
		return country.continient == "EU" && country.population > 10000
	}
```

```kotlin
fun filterCountries(countries: List<Country>, test: (Country) -> Boolean): List<Country>

filterCountries(countries, fun(country: Country): Boolean {
  return country.continient == "EU" && country.population > 10000
})
```

#### 2.3.6 Lambda是语法糖

> 在Lisp中，匿名函数是重要组成部分，它也被叫作Lambda表达式，这就是Lambda表达式名字的由来。

Lambda表达式可以把它理解成简化表达后的匿名函数，实质上是一种语法糖。

我们试下用Lambda对 filterCountries 函数进行简化：

```kotlin
filterCountries(countries, { country ->
	country.continient == "EU" && country.population > 10000
})
```

我们再来讲解下Lambda具体的语法：

```kotlin
val sum: (x: Int, y: Int) = { x: Int, y: Int ->
  x + y
}
```

由于支持类型推导，可以采用两种方式进行简化：

```kotlin
val sum = { x: Int, y: Int ->
  x + y
}
```

或者：

```kotlin
val sum: (Int, Int) -> Int = { x, y ->
  x + y
}
```

现在来总结下Lambda的语法：

- 一个Lambda表达式必须通过`{}`来包裹
- 如果Lambda声明了参数部分的类型，且返回值类型支持类型推导，那么Lambda变量就可以省略函数类型声明
- 如果Lambda变量声明了函数类型，那么Lambda的参数部分的类型就可以省略
- 如果Lambda表达式返回的不是Unit，那么默认最后一行表达式的值类型就是返回值类型

**Function类型**

Kotlin在JVM层设计了Function类型（Function0、Function1...Function22）来兼容JavaLambda表达式，其中的后缀数组代表了Lambda参数的数量，如以上的foo函数构建的其实是一个无参Lambda，所以对应的接口是`Function0`，如果有一个参数那么对应的就是`Function1`，以此类推。

```kotlin
package kotlin.jvm.functions

public interface Function1<in P1, out R> : Function<R> {
    /** Invokes the function with the specified argument. */
    public operator fun invoke(p1: P1): R
}
```

> 神奇的数字-22
>
> Kotlin在设计的时候便考虑到了这种情况，除了23个常用的Function类型外，还有一个`FunctionN`。在参数真的超过22个的时候，我们就可以依靠它来解决问题。

#### 2.3.7 函数、Lambada和闭包

在Kotlin中匿名函数体、Lambda在语法上都存在`{}`，由于这对**花括号包裹的代码块如果访问了外部环境变量则被称为一个闭包**。一个闭包可以被当做参数传递或者直接使用，也可以简单地看成”**访问外部环境变量的函数**“。Lambda是Kotlin中最常见的闭包形式。

#### 2.3.8 Currying风格、扩展函数

```kotlin
fun curryingLike(context: String, block: (String) -> Unit) {
  block(content)
}
curryingLike("looks like currying style") { content ->
	println(content)
}
```



### 2.4 面向表达式编程

表达式可以是一个值、常量、变量、操作符、函数，或它们之前的组合，编程语言对齐进行解释和计算，以求产生另一个值。通俗地理解，表达式就是可以返回值的语句。

以下是表达式的例子：

```kotlin
//(Int) -> Int
{x: Int -> x + 1}
// (Int) -> Unit
fun (x: Int) { println(x) }
//if-else
if (x > 1) x else 1
```



#### 2.4.2 Unit类型：让函数调用皆为表达式

在Kotlin中函数再所有的情况下都具有返回类型，所以引入了Unit来代替Java中`void`关键字。

**如何理解Unit？**
其实它和Int一样，都是一种类型，然而它不代表任何信息，用面向对象的术语来描述就是一个单例，它的实例只有一个，可写为`()`。

**Kotlin为什么要引入Unit呢？**

一个很大的原因是函数式编程侧重于组合，尤其是很多高阶函数，在源码实现的时候都是采用泛型来实现的。然而void在涉及泛型的情况下会存在问题。

#### 2.4.3 复合表达式：更好的表达力

**Kotlin中的 `?:`**

虽然Kotlin没有采用三元运算符，然而它存在一个很像的语法 `?:` ，也被叫作**Elvis运算符**，或者**null合并运算符**。表示一种类型的可空性：

```kotlin
val maybeInt: Int? = null
>>> maybeInt ?: 1
```



## 第3章 面向对象

### 3.1 类和构造方法

#### 3.1.1 Kotlin中的类及接口

对象是由状态和行为组成，通过它描述一个事物，称之为对象。

**延迟初始化 by lazy**

by lazy 语法特点：

- 该变量必须是引用不可变的，而不能通过`var`来声明
- 在改首次调用时，才会进行赋值操作。一旦被赋值，后续它将不能被更改。

lazy 的背后接受的是一个Lambda返回一个`Lzy<T>`实例的函数，第一次访问该属性时，会执行 lazy 对应的Lambda表达式并记录结果，后续访问该属性时只是返回记录的结果。

另外系统会给 lazy 属性默认加上同步锁，也就是`LazyThreadSafetyMode.SYNCHRON IZED`，它在同一时刻只允许一个线程对 lazy 属性进行初始化，所以它是线程安全的。



#### 3.2.1 限制的修饰符

**什么是里氏替换原则？**

子类可以扩展父类的功能，但不能改变父类原有的功能。它包含以下4个设计原则：

- 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法
- 子类可以增加自己特有的方法
- 当子类的方法实现父类的方法时，方法的前置条件（即方法的形参）要比父类方法的输入参数更宽松
- 当子类的方法实现父类的抽象方法时，方法的后置条件（即方法的返回值）要比父类更严格。



### 3.4 真正的数据类

#### 3.4.3 copy、componentN与解构

copy 方法的作用是帮我们从已有的数据类型对象中拷贝一个新的数据类对象。在 copy 的执行过程中，若你未指定具体属性的值，那么新生成的对象的属性值将使用被 copy 对象的属性值，便是我们平常所说的**浅拷贝**。实际上，除了基本类型的属性，其他属性还是引用同一个对象，这边是浅拷贝的特点。

componentN 可以理解为类属性的值，其中N代表属性的顺序，比如`component1`代表第1个属性的值。

解构的语法：

```kotlin
val (widght, age, color) = birdInfo.split("")
```

```java
String birdInfo = "20.0,1,blue";
String[] temps = birdInfo.split(",");
double weight = Double.valueOf(temps[0]);
int age = Integer.valueOf(temps[1]);
String color = temps[2];
```

除了数组支持解构外，Kotlin也提供了其他常用的数据类，让使用者不必主动声明这些数据类，它们分别是Pai二元组r和Triple三元组：

```kotlin
//Pair
public data class Pair<out A, out B>(
  public val first: A,
  public val second: B
)
//Triple
public data class Pair<out A, out B, out C>(
  public val first: A,
  public val second: B,
    public val third: C
)
```

所以，数据类型中的结构基于componentN函数，如果主动声明componentN函数，那么就会默认根据**主构造**函数参数来生成具体个数的componentN函数，与**从构造**函数中的参数无关。

#### 3.4.4 数据类型的约定与使用

数据类的另一个典型的应用就是代替我们在Java中的建造者模式。

### 3.5 从static到object

#### 3.5.1 什么是伴生对象

“伴生”是相较于一个类而言的，意为伴随某个类的对象，它属于这个类所有，因此伴生对象跟Java中static修饰效果性质是一样的，全局只有一个单例。它需要声明在类的内部，在类被装载时会被初始化。

伴生对象的另一个作用是可以实现工厂方法模式，可以改进Java延伸过来的工厂方法模式的缺点：

- 利用多个构造方法语意不够明确，只能靠参数区分
- 每次获取对象都需要重新创建对象

```kotlin
class Prize1 private constructor(val name: String, val count: Int, val type: Int) {
    companion object {
        val TYPE_COMMON = 1
        val TYPE_REDPACK = 2
        val TYPE_COUPON = 3

        val defaultCommonPrize = Prize1("普通奖品", 10, TYPE_COMMON)

        fun newRedpackPrize(name: String, count: Int) = Prize1(name, count, TYPE_REDPACK)
        fun newCouponPrize(name: String, count: Int) = Prize1(name, count, TYPE_COUPON)
        fun defaultCommonPrize() = defaultCommonPrize
    }
}
```

#### 3.5.3 object 表达式

```kotlin
val comparator0 = object : Comparator<String> {
    override fun compare(s1: String?, s2: String?): Int {
        if (s1 == null)
            return -1
        else if (s2 == null)
            return 1
        return s1.compareTo(s2)
    }
}
```

用于代替匿名内部类的object表达式在运行中不像我们在单例模式中说的那样，全局只存在一个对象，而是在每次运行时都会生成一个新的对象。

**对象表达式与Lambda表达式哪个更适合代替匿名内部类？**

- 当你的匿名内部类使用的类接口只需要实现一个方法时，使用Lambda表达式更适合
- 当匿名内部类内有多个方法实现的时候，使用object表达式更加合适



## 第4章 代数数据类型和模式匹配



## 第5章 类型系统

### 5.3 比Java更面向对象的设计

在Kotlin的类型系统中，并不区分原始类型（基本数据类型）和包装类型。与Java不同的是，Kotlin不区分“原始类型”（primitive type）和其它的类型，他们都是同一类型层级结构的一部分。

#### 5.3.1 Any：非空类型的根类型

Any? 是 Any的父类型

#### 5.3.3 Nothing与Nothing?

在Kotlin类型层级结构的最底层是Nothing类型。Nothing是没有实例的类型，本质上与`null`没有区别。Nothing类型的表达式不会产生任何值。任何返回值为Nothing的表达式之后的语句都是无法执行的，这有点像`reuturn`或者`break`的作用，在Kotlin中`return`、`throw`等返回值都为Nothing。

#### 5.3.4 自动装箱与拆箱

**装箱与拆箱的含义**

自动装箱基本类型自动转为包装类，自动拆箱指包装类自动转为基本类型。

Kotlin中的`Int`类型等同于`int`，而`Int?`等同于`Integer`。

#### 5.3.5 “新”的数组类型

IntArray等并不是Array的子类，所以两者创建的相同值的对象，并不是相同对象。由于Kotlin对原始数据有特殊的优化（主要体现在避免了自动装箱带来的开销）。所以建议优先使用原始类型数组。



### 5.4 泛型

泛型的优势：

- 类型检查，能在编译时检查出错误
- 更加语义化，比如我们声明一个List<String>，便可以知道里面存储的是String对象，而不是其他对象
- 自动类型转换，获取数据时不需要进行类型强制转换
- 能写出更加通用化的代码

#### 5.4.3 类型约束：设定类型上界

可以通过where关键字给泛型参数添加多个约束条件

```kotlin
interface Ground { }

class Watermelon(weight: Double): Fruit(weight), Ground

fun <T> cut(t: T) where T: Fruit, T: Ground {
  print("You can cut me.")
}
```



### 5.5 泛型擦除

```java
Apple[] appleArray = new Apple[10];
Fruit[] fruitArray = appleArray;	//允许
fruitArray[0] = new Banana();	//编译通过，运行报ArrayStoreException
List<Apple> appleList = new ArrayList<Apple>();
List<fruit> fruitList = appleList;	//不允许
```

这里涉及一个关键点，Java中**数组是协变的，而List是不变的**。简单来说就是`Object[]`是所有对象数组的父类，而`List<Object>`却不是`List<T>`的父类。

Java中的泛型是类型擦除的，可以看做是伪泛型。

Kotlin中的泛型机制与Java中是一样的，但不同的是Kotlin中的数组是支持泛型的，所以不再协变，例如：

```kotlin
val appleArray = arrayOfNulls<Apple>(3)
val anyArray: Array<Any?> = appleArray //不允许
```

#### 5.5.3 类型擦除的矛盾

其实泛型类型擦除并不是真的将全部的类型信息都擦除，还是会将类型信息放在对应class的常量池中的。

```kotlin
open class Plate<T>(val t: T, val calzz: Class<T>) {
  fun getType() {
    println(clazz)
  }
}
val applePlate = Plate(Apple(1.0), Apple::calss.java)
applePlate.getType()

//结果
class Apple
```

匿名内部类之所以能在运行时获取泛型参数的类型是因为，**匿名内部类在初始化的时候会绑定父类或父类接口的相应信息**。这样就能通过父类或父类接口的泛型类型信息来实现。例如Gson的TypeToken也是采用了相同的设计。

#### 5.5.4 使用内联函数获取泛型

**Kotlin中的内联函数在编译时编译器会将相应的函数的字节码插入调用的地方**。

```kotlin
inline fun <reified T> getType() {
  return T::class.java
}
```

`reified`关键字，在编译时会将具体的类型插入相应的字节码中。

Java并不支持主动指定一个函数是否是内联函数，所以在Kotlin中声明的普通内联函数可以在Java中调用，因为它会被当作一个常规函数；而用`reified`来实例化的参数类型的内联函数则不能在Java中调用，因为它永远是需要内联的。



### 5.6 打破泛型不变

#### 5.6.2 一个支持协变的List

`out`关键字，如果定义的泛型类和泛型方法的泛型参数前面加上`out`关键字，说明这个泛型类及泛型方法是协变的。

支持协变的List只可以读取，而不可以添加，加入支持协变的List出入新对象，那么它就不再是类型安全的了，也违背了泛型的初衷。

通常情况下，out T支持协变，那么它的方法的参数类型不能使用T类型（但可以作为方法的返回值），因为那样可能导致错误，可以添加`@UnsafeVariance`注解解除这个限制。比如List中的`indexOf`等方法。

#### 5.6.3 一个支持逆变的Comparator

`in`关键字，不能将泛型参数类型当作方法返回值的类型，但可以作为方法的参数类型。

#### 5.6.4 协变和逆变

`in`和`out`是一个对立面，其中`in`代表**泛型参数类型逆变**，`out`代表**泛型参数类型协变**。从字面意思上也可以理解，`in`代表输入，`out`代表输出，但同时它们由于泛型不变相对立，统称为**型变**。

Kotlin与Java的型变比较

|        | 协变                                             | 逆变                                             | 不变  |
| ------ | ------------------------------------------------ | ------------------------------------------------ | ----- |
| Kotlin | `<out T>` 只能作为消费者，只能读取不能添加       | <in T> 只能作为生产者，只能添加，读取受限        | `<T>` |
| Java   | `<? extends T>` 只能作为消费者，只能读取不能添加 | `<? super T>` 只能作为生产者，只能添加，读取受限 | `<T>` |

如果对泛型参数的类型不感兴趣，可以使用类型通配符 `*` 来代替泛型参数（类似Java中的 `?` ）。



## 第6章 Lambda和集合

### 6.4 惰性集合

惰性求值（Lazy Evaluation）表示一种在需要时才进行求值的计算方式。通过惰性求值可以优化性能，还有就是能够构造出无限的数据类型。

#### 6.4.1 通过序列提高效率

在Kotlin序列中元素的求值是惰性的，这就意味着在利用序列进行链式求值的时候，不需要像操作普通集合那样，每进行一次求值操作就产生一个新的集合保存中间数据。

#### 6.4.2 序列的操作方式

Kotlin中序列的操作分为两类，一类是**中间操作**，另一类则是**末端操作**。



### 6.5 内联函数

内联函数之所以被设计出来，主要是为了优化Kotlin支持Lambda表达式之后所带来的开销。

#### 6.5.1 优化Lambda开销

在Kotlin中每声明一个Lambda表达式，就会在字节码中产生一个匿名类，该匿名类包含了一个invoke方法，作为Lambda的调用方法，每次调用的时候，还会创建一个新的对象，所以使用Lambda很简洁，但额外增加了不小的开销。

**invokedynamic**

在讲述内联函数之前，先来看看Java中是如何解决Lambda带来的开销问题的。在Java SE 7之后，JVM引入了一种叫做`invokedynamic`的技术，实现了在运行期才产生相应的「翻译代码」。在`invokedynamic`被首次调用的时候，就会触发产生一个匿名类来替换中间码`invokedynamic`，后续的调用会直接采用这个匿名类的代码。这种做法的好处体现在：

- 由于具体的转换实现时在运行时产生的，在字节码中能看到的只有一个固定的`invokedynamic`，所以需要静态生成的类的个数及字节码大小都显著减少
- 与编译时写死在字节码中的策略不同，李彤`invokedynamic`可以把实际的翻译策略隐藏在JDK库的实现，这极大提高了灵活性，在确保向后兼容性的同时，后期可以继续对翻译策略不断优化升级
- JVM天然支持了针对该方式的Lambda表达式的翻译和优化，这也意味着开发者在书写Lambda表达式的同时，可以完全不同关心这个问题

**内联函数不是万能的**

一下情况为们应避免使用内联函数：

- 由于JVM对普通函数已经能够根据实际情况智能地判断是偶进行内联优化，所以我们并不需要对其使用inline，那样只会让字节码变得更加复杂
- 尽量避免对具有大量函数体的函数进行内联，这样会导致过多的字节码数量
- 一旦一个函数被定义为内联函数，便不能获取闭包类的私有陈孤雁，除非你把它们声明为`internal`

#### 6.5.3 noinline：避免参数被内联

如果一个声明内联的函数接收了多个参数，但只想对其中部分Lambda参数内联，可以使用`noinline`关键字对参数进行修饰。

#### 6.5.4 非局部返回

Kotlin中，正常情况下Lambda表达式不允许存在`return`关键字。需要`inline`进行修饰，但要注意`return`暴露在调用函数的情况。这种效果也就是所谓的非局部返回。

#### 6.5.6 crossinline

有时候内联函数所接收的Lambda参数常常来自于上下文其他地方。为了避免带有`return`的Lambda参数产生破坏，我们还可以使用`crossinline`关键字来修饰该参数。

#### 6.5.6 具体化参数类型

由于运行时的类型擦除，我们不能直接获取一个参数的类型。然而，由于内联函数会直接在字节码中生成相应的函数体实现，这种情况下我们反而可以获得参数的具体类型，可以用`reified`修饰符来实现这一效果。

```kotlin
inline fun <reified T : Activity> Activity.startActivity() {
  startActivity(Intent(this, T::class.java))
}
startActivity<DetailActivity>()
```



## 第7章 多态和扩展

### 7.1 多态的不同方式

**子类型多态（Subtype polymorphism）**
子类型替换超类型实例的行为

**参数多态（Parametric polymorphism）**

参数多态在程序设计语言与类型论中是指声明与定义函数、复合类型、变量时不指定其具体的类型，而把这部分类型作为参数使用，使得该定义对各种具体类型都适用，所以它建立在运行时的参数基础上，并且所有这些都是在不影响类型安全的前提下进行的。

**特设多态（Ad-hoc polymorphism）**

一个多态函数是有多个不同的实现，依赖于实参而调用相应版本的函数。

#### 7.1.4 特设多态与运算符重载

**operator关键字**
将一个函数标记为重载一个操作符或者实现一个约定

```kotlin
data class Area(val value: Double)
operator fun Area.plus(that: Area): Area {
  return Area(this.value + that.value)
}
fun main() {
  println(Area(1.0) + Area(2.0))
}
```

```
Area(value=3.0)
```



### 7.2 扩展：为别的类添加方法、属性

扩展是Kotlin实现特设多态的一种非常重要的语言特性。

#### 7.2.1 扩展与开闭原则

开闭原则（OCP：Open Closed Principle）：软件实体应该是可扩展，而不可修改的。也就是说，对扩展开放，对修改封闭。

#### 7.2.2 使用扩展函数、属性

**扩展函数的实现机制**

扩展函数可以理解为被`public`修饰静态方法，所以扩展函数不会带来额外的性能消耗。

**扩展属性**

由于扩展没有实际将成员插入类中，因此对扩展属性来说幕后字段是无效的，这也说明为什么扩展属性不能有初始化器的原因，因为它们的行为只能由显式提供的`getters`和`settters`定义。

**幕后字段**

在Kotlin中，如果属性中存在访问器使用默认实现，那么Kotlin会自动提供幕后字段`filed`，其仅可用于自定义`getter`和`setter`中。

#### 7.2.3 扩展的特殊情况

Kotlin中扩展函数与类成员方法同名是，类成员方法优先级总高于扩展函数。



### 7.4 扩展不是万能的

**扩展函数始终静态调度**

成员函数和扩展函数之间最重要的区别：前者是动态调度的，后者总是静态调度的。

**调度接收器和扩展接收器的概念**

扩展接收器（extension receiver）：与Kotlin扩展密切相关的接收器，表示我们为其定义扩展函数的对象。

调度接收器（dispatch receiver）：扩展被声明为成员时存在的一种特殊接收器，它表示声明扩展名的类的实例。

```kotlin
class X {
  fun Y.foo() = "I'm Y.foo"
}
```

- 如果扩展函数是顶级函数或成员函数，则不能被覆盖
- 我们无法访问其接收器的非公共属性
- 扩展接收器总是被静态调度。



## 第8章 元编程

Java反射是元编程的一种方式。

### 8.1 程序和数据

#### 8.1.1 什么是元编程

**描述数据的数据可以称之为元数据，描述程序的数据就是程序的元数据，那么操作元数据的编程可以称之为元编程。**

>程序即是数据，数据即是程序
>
>- 前半句指的是访问描述程序的数据，如我们通过反射获取类型信息；
>- 后半句则是指将这些数据转化成对应的程序，也就是所谓代码生成。

元编程就像高阶函数一样，是一种更高阶的抽象。高阶函数将函数作为输入或输出，而元编程则是将程序本身作为输入或输出。

**同像性**

在计算机编程中，同像性（homoiconicity，来自希腊语单词homo，意为与符号含义表示相同）是某些编程语言的特殊属性，它意味着一个程序的结构与其句法是相似的，因此易于通过阅读程序来推测程序的内在涵义。如果一门编程语言具备了同像性，说明该语言的文本表示（通常指源代码）与其抽象语法树（AST）具有相同的结构（即，AST和语法是同形的）。该特性允许使用相同的表示语法，将语言中的所有代码当成资料来存取以及转换，提供了“代码即数据”的理论前提。

#### 8.1.2 常见的元编程技术

目前主流的元编程的实现方式包括：

- 运行时通过API暴露程序信息，反射就是这种实现思路。
- 动态执行代码，可以动态地将文本作为代码执行。多见于脚本语言，如JavaScript的eval函数。
- 通过外部程序实现目的。如编译器，在将源文件解析为AST之后，可以针对这些AST做各种转化。
  这种实现思路最典型的例子是我们常常谈论的语法糖，编译器会将这部分代码AST转化为相应的等价的AST，这个过程通常被称为desuger（解语法糖）。

**什么是AST**

AST（抽象语法树）是源代码语法结构的一种抽象表示。它以树状的形式表现编程语言的语法结构，树上的每个节点都表示源代码中的一种结构。之所以说语法是“抽象”的，是因为这里的语法并不会表示出真实语法中出现的每个细节。比如，嵌套括号被隐含在树的结构中，并没有以节点的形式呈现；而类似于if-condition-then这样的条件跳转语句，可以使用带有两个分支的节点来表示。

**反射**

反射，也称自反，是指元语言（描述程序的数据结构）和要描述的语言是同一种语言的特性。

Kotlin中的KClass就是如此，它是一个Kotlin的类，同时它的实例又能作为描述其它类的元数据。像这样用Kotlin描述Kotlin自身信息的行为就是反射或自反。

除了运行时反射之外，也有许多语言支持编译器反射，编译器反射通常需要和宏等技术结合使用，编译器将当前程序的信息作为输入传给宏，并将其结果作为程序的一部分。



### 8.2 Kotlin的反射

KParameter 函数的参数

KType 函数的返回值

KTypeParameter 类型参数



### 8.3 Kotlin的注解

```kotlin
annotation class TestAnnotation(val name: String)
```

了解Kotlin注解之前，先认识下Java的几种元注解：

- Documented：文档（通常是API文档）中必须出现该注解。
- Inherited：如果超类标注了该类型，那么其子类型也将自动标注该注解而无须指定。
- Repeatable：这个注解在同一位置可以出现多次。
- Retention：表示注解用途，有3种取值。
- Sourc：仅在源代码中存在，编译后class文件中不包含该注解信息。
- CLASS：class文件中存在该注解，但不能被反射读取。
- RUNTIM：注解信息同样保存在class文件中并且可以在运行时通过反射获取。
- Target：表明注解可应用于何处。

Target的取值

| Kotlin(Annotation Target) | Java(Target)    | 说明（作用于）            |
| ------------------------- | --------------- | ------------------------- |
| CLASS                     | TYPE            | 类                        |
| ANNOTATION_CLASS          | ANNOTATION_TYPE | 注解本身（即元注解）      |
| TYPE_PARAMETER            | TYPE_PARAMETER  | 类型参数                  |
| PROPERTY                  | NA              | 属性                      |
| FIELD                     | FIELD           | 字段（包含Getter/Setter） |
| LOCAL_VARIABLE            | LOCAL_VARIABLE  | 局部变量                  |
| VALUE_PARAMETER           | NA              | val 参数                  |
| CONSTRUCTOR               | CONSTRUCTOR     | 构造函数                  |
| FUNCTION                  | METHOD          | 函数（Java只有Method）    |
| PROPERTY_GETTER           | NA              | Getter                    |
| PROPERTY_SETTER           | NA              | Setter                    |
| TYPE                      | TYPE_USE        | 类型                      |
| EXPRESSION                | NA              | 表达式                    |
| FILE                      | PACKAGE         | 文件开头/包声明           |
| TYPEALIAS                 | NA              | 类型别名                  |

### 8.3.3 获取注解信息

JSR269引入了注解处理器（annotation processors）,允许在编译过程汇总挂钩子实现代码生成。

在了解什么是注解处理器之前，我们先来看看编译器的主要工作。

![](../../blog/images/kotlin/kotlin-driveinto-coroutine-8.3-1.png)

虽然annotation processor允许开发人员访问程序AST，但没有提供行之有效的代码生成方案，目前仅有的代码生成方案也仅仅是将代码以字符串的形式写入新文件，而无法做到直接将生成的AST作为程序。这也说明了Java和Kotlin目前不具备同像性。



## 第9章 设计模式

### 9.1 创建型模式

#### 9.1.1 伴生对象增强工厂模式

简单工厂模式的核心作用是**通过工厂类隐藏对象实例的创建逻辑**，而不需要暴露给客户端。典型的场景就是当父类拥有多个子类的时候可以通过这种模式来创建子类对象。

**用单例代替工厂类**

```kotlin
object ComputerFactory {
  fun produce(type: ComputerType) : Computer {
    return when(type) {
      ComputerType.PC -> PC()
      ComputerType.Server -> Server()
    }
  }
}
```

```kotlin
ComputerFactory.produce(ComputerType.PC)
```

**伴生对象创建静态工厂方法**

```kotlin
interface Computer {
  val cpu: String
  
  companion object Factory {
    operator fun invoke(type: ComputerType) : Computer {
      return when(type) {
        ComputerType.PC -> PC()
        ComputerType.Server -> Server()
      }
    }
  }
}
```

```kotlin
Computer.Factory(ComputerType.PC)
```

**扩展伴生对象方法**

```kotlin
fun Computer.Companion.fromCPU(cpu: String): ComputerType? = when(cpu) {
  "Core" -> ComputerType.PC
  "Xeon" -> ComputerType.Server
  else -> null
}
```

#### 9.1.2 内联函数简化抽象工厂

抽象工厂模式，为创建一组相关或相互依赖的对象提供一个接口，而且无需指定它们的具体类。

```kotlin
abstract class AbstractFactory {
  abstract fun produce() : Computer
  
  companion object {
    inline operator fun <reified T: Computer> invoke() : AbstractFactory = when(T::class) {
      Dell::class -> DellFactory()
      Asus::class -> AsusFactory()
      Acer::class -> AcerFactory()
      else -> throw IllegalArgumentException()
    }
  }
}
```

```kotlin
val dellFactory = AbstractFactory<Dell>()
val dell = dellFactory.produce()
```

#### 9.1.3 用具名可选参数而不是构建者模式

构建者模式，构建者模式与单例模式一样，也是Gof设计模式中的一种。它主要做的事情就是将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

构建者模式的一些不足：

- 如果业务的参数很多，代码依然显得比较冗长
- 可能会在使用Builder的时候忘记在最后调用`build`方法
- 由于传感对象的时候，必须先创建它的构造器，因此额外增加了多余的开销

**具名可选参数**

- 在具体化一个参数的取值时，可以通过带上它的参数名，而不是它所在 所有参数中的位置决定的
- 由于参数可以设置默认值，这允许我们只给出部分参数的取值，而不必所有的参数

**require方法对参数进行约束**

使用`require`关键爱你自进行函数参数限制，本质上是一个内联的方法，有点类似于Java中的`assert`。

```kotlin
class Robot(
  val code: String,
  val battery: String? = null,
  val height: Int? = null,
  val weight: Int? = null
) {
  init {
    require(weight == null || battery != null) {
      "Battery should be determined when setting weight."
    }
  }
}
```

可见，Kotlin的require方法可以让我们的参数约束代码在语义上变得更加友好。

总的来说，在Kotlin中我们应该尽量避免使用构建者模式，因为Kotlin支持具名的可选参数，可以设计出更加简洁并利于维护的代码。



### 9.2 行为型模式

那些用来识别对象之间的常用交流模式就是行为型模式。

#### 9.2.1 Kotlin中的观察者模式

观察者模式定义了一对多的依赖关系，让一个或多个观察者对象监听一个主题对象。当被观察者状态发生改变时，需要通知相应的观察者，是这些观察者对象能够自动更新。

Java自身的标准库提供了Observable和Observer接口，来帮助实现观察者模式。

**Delegates.observable()**

```kotlin
public inline fun <T> observable(initialValue: T, crossinline onChange: (
  property: KProperty<*>, 
  oldValue: T, 
  newValue: T) -> Unit): ReadWriteProperty<Any?, T>
```

**Vetoable**

`vetoable`提供了一种功能，在被赋新值生效之前提前进行截获，然后判断是否接收它。

```kotlin
public inline fun <T> vetoable(initialValue: T, crossinline onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Boolean): ReadWriteProperty<Any?, T>
```

#### 9.2.2 高阶函数简化策略模式、模板方法模式

**遵循开闭原则：策略模式**

策略模式定义了算法族，分别封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。

**高阶函数抽象算法**

利用高阶函数实现策略模式

```kotlin
class Swimmer(val swimming: () -> Unit) {
  fun swim() {
    swimming()
  }
}
fun breaststroke() { println("I am breaststroking...")}
fun backstroke() { println("I am backstroking...")}
fun freestyle() { println("I am freestyleing...")}
```

```kotlin
val weekendShaw Swimmer(::freestyle)
weekendShaw.swim()
val weekdaysShaw Swimmer(::breaststroke)
weekdaysShaw.swim()
```

**模板方法模式：高阶函数代替继承**
模板方法和策略模式要解决的问题是相似的，他们都可以分离通用的算法和具体的上下文。

模板方法模式，定义一个算法中的操作框架，而将一些步骤延迟到子类中，使得子类可以不改变算法结构即可重定义该算法的某些特定步骤。

```kotlin
class CivicCenterTask {
  fun execute(askForHelpe: () -> Unit) {
    this.lineUp()
    askForHelpe()
    this.evaluate()
  }
}
fun pullSocialSecurity() {
  println("ask for pulling the social security")
}
```

```kotlin
val task = CivicCenterTask()
task.execute(::pullSocialSecurity)
```

#### 9.2.3 运算符重载和迭代器模式

迭代器模式，核心作用就是将遍历和实现分离开来，在遍历的同事不需要暴露对象的内部表示。

**通过扩展函数和operator重载iterator方法**

```kotlin
data class Book(val name: String)
class Bookcase(val books: List<Book>) { }

operator fun Bookcase.iterator(): Iterator<Book> = books.iterator()
```

如果想对迭代器的逻辑有更多的控制权，可以通过`object`表达式来实现：

```kotlin
operator fun Bookcase.iterator(): Iterator<Book> = object: Iterator<Book> {
  val iterator = books.iterator()
  override fun hasNext() = iterator.hasNext()
  override fun next() = iterator.next()
}
```

#### 9.2.4 用偏函数实现责任链模式

责任链模式，目的是避免请求的发送者和接收者之间的耦合关系，将这个对象炼成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。

**实现偏函数类型：PartialFunction**

偏函数，是一个数学中的概念，指的是定义域X中可能存在某些值在值域Y中没有对应的值。
通俗来讲，当输入参数处理某个责任链环节的有效接收范围之内，该环节才能对其做出正常的处理操作。

```kotlin
val applyChain = groupLeader orElse president orElse colleage
```

借助PartialFunction类的封装，在构件责任链时，可以用`orElse`获得更好的语法表达。

#### 9.2.5 ADT实现状态模式

状态模式和策略模式存在某些相似性，它们都可以实现某种算法、业务逻辑的切换。

状态模式允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。

状态模式具体表现在：

- 状态决定行为，对象的行为由它内部的状态决定。
- 对象的状态在运行期被改变时，它的行为也会因此而改变。从表面上看，同一个对象，在不同的运行时期，行为是不一样的，像是类被修改了一样。

对比策略模式，策略模式通过在客户端切换不同的策略实现来改变算法，而在状态模式中，**对象通过修改内部的状态来切换不同的行为方法**。



### 9.3 结构型模式

在对象创建之后，对象的组成及对象之间的依赖关系就成了我们关注的焦点。

#### 9.3.1 装饰者模式：用类委托减少样板代码

装饰者模式：在不改变源文件和使用继承的情况下，动态地扩展一个对象的功能。该模式通过创建一个包装对象，来包裹真实的对象。

总的来说，装饰者模式做的以下几件事情：

- 创建一个装饰类，包含一个需要被装饰类的实例
- 装饰类重写所有被装饰类的方法
- 在装饰类中对需要增强的功能进行扩展

```kotlin
//装饰类
class ProcessorUpgradeMackbookPro(val macbook: MacBook) : MacBook by macbook {
  override fun getCost() = macbook.getCost() + 219
  override fun getDesc() = macbook.getDesc() + ", +1G Memory"
}
```

装饰类会把MacBook接口所有的方法都委托给**构造参数对象macbook**，因此只需通过覆写的语法来重写需要变更的cost和desc方法。这样大大减少了装饰者模式中的样板代码。

#### 9.3.2 通过扩展代替装饰者

```kotlin
fun Printer.startDraw(decorated: Printer.() -> Unit) {
  println("++ start drawing ++")
  this.decorated()
  println("++ end drawing ++")
}
```

```kotlin
Printer().run {
  startDraw {
    drawLine()
  }
  startDraw {
    drawDottedLine()
  }
  startDraw {
    drawStarts()
  }
}
```



## 第10章 函数式编程

Kotlin是一门集面向对象与函数式的多范语言。

### 10.1 函数式编程的特征

#### 10.1.1 函数式语言之争

我们从广义和狭义两方面去解读所谓的函数式语言。

- 所谓狭义的函数式语言，有着非常简单且严格的语言标准，即只通过存函数进行编程，不允许有副作用。就像数学中函数一样，给它同样的输入，会有相同的输出，因此程序也非常适合推理。但纯函数的编程语言也有着一些明显的劣势，典型的就是绝对的无副作用，以及所有的数据结构都是不可变的。这使得它在设计一些如今我们认为非常简单的程序时，也变得十分麻烦。
  由于Kotlin继承了Java中的面向对象的特征，在存函数式语言看来，Kotlin'并不能称为真正意义的函数式语言。
- 从广义上看，任何“以函数为中心进行编程”的语言都可称为函数式编程，可以在任何位置定义函数，同时也可以将函数作为值进行传递。因此，广义的函数式编程语言并不需要抢到函数是否都是“存”的。

下面列举一下常见的函数式语言特性：

- 函数是头等公民
- 方便的闭包语法
- 递归式构造列表（list comprehension）
- 柯里化的函数
- 惰性求值
- 模式匹配
- 尾递归优化

如果支持静态类型的函数式语言，那么它们还可能支持：

- 强大的泛型能力，包括高阶类型
- Typeclass
- 类型推导

Kotlin支持以上具有的代表性的函数式语言特性，因此被称为广义上的函数式语言。



### 10.2 实现Typeclass

#### 10.2.1 高阶类型：用类型构造新类型

**值构造器（value constructor）**

很多情况下，值构造器可以是一个函数，我们可以给一个函数传递一个值参数，从而构造出一个新的值。

```kotlin
(x: Int) -> x
```

**类型构造器（type constructor）**

类型构造器，可以传递一个类型变量，然后构造出一个新的类型。

- 一阶构造器：通过传入一个具体的值，然后构造出另一个具体的值。
- 一阶类型构造器：通过传入一个具体的类型变量，然而构造出另一个具体的类型。

高阶函数，可以支持传入一个值构造器，或者返回另一个值构造器。

```kotlin
{ x: (Int) -> Int -> x(1) }
{ x: Int -> { y: Int -> x + y } }
```

#### 10.2.3 用扩展方法实现Typeclass

```kotlin
interface Kind<out F, out A>

interface Functor<F> {
  fun <A,B> Kind<F, A>.map(f: (A) -> B): Kind<F,B>
}
```

```kotlin
sealed class List<out A> : Kind<List.K, A> {
  object K
}
object Nil : List<Nothing>()
data class Cons<A>
```

```kotlin
ListFunctor.run {
  Cons(1, Nil).map( it + 1)
}
```

#### 10.2.4 Typeclass设计常见功能

- 利用类型的扩展语法定义通用的Typeclass接口
- 通过object定义具体类型的Typeclass实例
- 在实例run函数的闭包中，目标类型的对象或值随之支持了响应的Typeclass的功能

**Eq**

```kotlin
interface Eq<F> {
  fun F.eq(that: F): Boolean
}

object IntEq : Eq<Int> {
  override fun Int.eq(that: Int): Boolean {
    reutrn this == that
  }
}
```

```kotlin
IntEq.run {
  val a = 1
  println(a.eq(1))
  println(a.eq(2))
}
```

下面来看如何用**Eq**支持高阶类型，下面来实现一个**ListEq**的Typeclass

```kotlin
interface Kind<out F, out A>

object Nil : List<Nothing>()

data class Cons<A>(val head: A, val tail: List<A>) : List<A>()

abstract class ListEq<A>(val a: Eq<A>) : Eq<Kind<List.K, A>> {
  override fun Kind<List.K, A>.eq(that: Kind<List.K, A>): Boolean {
    val curr = this
		return if (curr is Cons && that is Cons) {
      val headEq = a.run {
        curr.head.eq(that.head)
      }
      if (headEq) curr.tail.eq(that.tail) else false
    } else if (curr is Nil && that is Nil) {
      true
    } else false
  }
}
```



### 10.4 类型代替异常处理错误

函数式数据类型，如Option、OptionT、Either、EitherT，为业务中的错误处理提供了一种新的思路，即抛弃传统的异常处理，基于高阶类型来定义和区分业务中非正常的情况。



## 第11章 异步和并发



## 第12章 基于Kotlin的Android架构

### 12.1 架构方式的演变

#### 12.1.1 MVC

**经典的MVC问题**

- 代码相对冗余
- 灵活性较低
- 可维护性低

#### 12.1.2 MVP

**MV容易产生的问题**

- 接口粒度难以掌控
- Presenter逻辑容易过重
- Presenter和View相互引用

#### 12.1.3 MVVM

**MVVM容易造成的问题**

- 需要更多精力定位Bug
- 通用的View需要更好的设计















