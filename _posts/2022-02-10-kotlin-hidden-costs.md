---
layout: 
title: Kotlin hidden costs
date: 2022-02-10 10:33 +0800
tags: kotlin
---
Kotlin 的隐藏开销

<!--more-->



## Kotlin字节码检查器

Kotlin语法特性转化字节码的情况：

- 原始类型的装箱，如何分配短期对象
- 实例化代码不直接可见的额外对象
- 生成额外的方法，如果dex文件超出限制，需配置multidex，但该方式存在限制和性能的损失

##  

## 高阶函数和Lambda表达式

### Function对象

Kotlin中Lambda表达式和匿名函数在编译后，会被编译为Function对象。

```kotlin
class MyClass$myMethod$1 implements Function1 {
   // $FF: synthetic method
   // $FF: bridge method
   public Object invoke(Object var1) {
      return Integer.valueOf(this.invoke((Database)var1));//被装箱成Integer对象，这个下一节会具体讲到
   }

   public final int invoke(@NotNull Database it) {
      Intrinsics.checkParameterIsNotNull(it, "it");
      return it.delete("Customers", null, null);
   }
}
```

这些Function对象的新实例仅在必要的时候创建，所以在使用中需要知道什么情况下会创建Function对象的新实例，以便给程序带来更好的性能：

- 对于捕获表达式情况，每次将Lambda作为参数传递，然后执行后进行垃圾回收，就会每次创建一个新的Function实例
- 对于非捕获表达式（纯函数）情况，将在下次调用期间创建并复用单例函数实例

> 如果要调用**捕获Lambda**来减少垃圾收集器的压力，请避免重复调用标准（非内联）高阶函数。

**捕获代码**

```kotlin
//捕获局部函数
fun someMath(a: Int): Int {
    fun sumSquare(b: Int) = (a + b) * (a + b)//注意:局部函数这里的a是直接引用外部函数的参数a, 
    //因为局部函数特性可以访问外部函数的作用域，这里实际上就存在了变量的捕获，所以这里sumSquare称为捕获局部函数

    return sumSquare(1) + sumSquare(2)
}
```

**非捕获代码**

```kotlin
//非捕获局部函数
fun someMath(a: Int): Int {
    //注意: 可以明显发现改写后a参数，直接由函数参数传入，而不是在局部函数直接引用外部函数的参数变量，这就是非捕获局部函数
    fun sumSquare(a: Int, b: Int) = (a + b) * (a + b)
    return sumSquare(a,1) + sumSquare(a,2)
}
```



### 装箱带来的性能开销

```kotlin
/** A function that takes 1 argument. */
public interface Function1<in P1, out R> : Function<R> {
    /** Invokes the function with the specified argument. */
    public operator fun invoke(p1: P1): R
}
```

当函数涉及输入值或返回值是基本类型时，调用在高阶函数中作为参数传递的函数实际上将涉及系统的装箱和拆箱，可能对性能上产生不可忽视的影响。

> *在编写涉及使用基本类型作为输入或输出值的参数函数的标准（**非内联**）高阶函数时要小心。反复调用此参数函数将通过装箱和拆箱操作对垃圾收集器施加更大的压力*。



### 内联函数

内联函数中作为参数传递的Lambda表达式的主体被内联的实际效果：

- 声明Lambda表达式时，不会实例化Function对象
- 没有装箱或拆箱操作将应用于基于原始类型的Lambda输入和输出值
- 没有方法将添加到总方法计数中
- 不会执行实际的函数调用，这可以提高对此使用该函数带来的CPU占用性能

使用内联函数时需要注意的一些地方：

- 内联函数不能直接调用自身或通过其它内联函数调用自身
- 在类中声明公有的内联函数只能访问该类的公有函数或成员变量
- 代码的大小会增加。内联多次引用代码较长的函数会使生成的代码更大。

> 如果可能，将高阶函数声明为内联函数，保存简短，将大段代码移动到非内联函数。



## 伴生对象

在编译时，伴生对象会被实现为单例类。意味着像任何需要从其它类访问其私有字段的Java类一样，**从伴随对象访问外部类的私有字段（或构造函数）将生成其他合成getter和setter方法**。对类字段的每次读取或写入访问都将导致伴生对象中的静态方法调用。

> 如果需要从伴生对象重复读取或写入类字段，则可以将其值缓存在局部变量中，以避免重复的隐藏方法调用。



### 访问伴生对象中声明的常量

为了存储常量值，Kotlin编译器是在主类级别中生成实际的`private static final`字段，而不是伴生对象内，**所以需要另一种合成方法来从伴生对象汇总访问它**。

当从Kotlin类访问半生对象中的私有常量字段时，而不是像Java那样直接读取`private static final`静态字段，实际上是：

- 在伴生对象中调用静态方法
- 它将依次调用伴随对象中的实例方法
- 然后反过来调用类中的静态方法
- 读取静态字段并返回其值

```kotlin
class MyClass {
    companion object {
        private val TAG = "TAG"
    }

    fun helloWorld() {
        println(TAG)
    }
}
```

```java
public final class MyClass {
    private static final String TAG = "TAG";
    public static final Companion companion = new Companion();

    // synthetic 
    public static final String access$getTAG$cp() {
        return TAG;
    }

    public static final class Companion {
        private final String getTAG() {
            return MyClass.access$getTAG$cp();
        }

        // synthetic
        public static final String access$getTAG$p(Companion c) {
            return c.getTAG();
        }
    }

    public final void helloWorld() {
        System.out.println(Companion.access$getTAG$p(companion));
    }
}
```

当常量声明为`public`时，此`getter`方法是公共的且可以直接调用，因此不需要上一步的合成方法。但Kotlin仍然需要调用`getter`方法来读取常量。

通过`const`关键字可以获得更轻量的常量的字节码，可以完全避免任何方法调用。将直接在调用代码汇总有效的内联值，但是**只能将它用于原始类型和字符串**。

其次，可以在伴生对象的公共字段上使用`@JvmField`注解，以指示编译器不生成任何`getter`或`setter`，将其作为类中的静态字段公开，就像纯Java常量一样。此外，它只能用于公有字段。

> 从伴生对象中读取“静态”常量，与Java相比，在Kotlin中增加了两到三个额外的间接级别，并且将为这些常量中的每一个生成两到三个额外的方法。
>
> 1. 始终使用`const`关键字声明基本类型和字符串常量以避免这种情况
> 2. 对于其它类型的常量，不能使用`const`，因此如果需要重复访问常量，可能需要将值缓存在局部变量中
> 3. 此外，更推荐将公有的全局常量存储在它们自己的对象中而不是伴生对象。



## 局部函数

在其它函数体内部声明的函数，被称为局部函数，能访问到外部函数的作用域。

**局部函数的局限性**

局部函数不能被声明`inline`内联，并且函数体内含有局部函数的函数也不能被声明成`inline`内联。

从外部函数调用局部函数时，不会进行基本类型的转换或装箱操作。

> 局部函数是私有函数的替代品，其附加好处是能够访问外部函数的局部变量。然而这种好处会伴随着为外部函数每次调用创建Function对象的隐形成本，因此首选使用非捕获的局部函数。



## 空安全

实际上，每个公有的函数都有一个对`Intrinsics.checkParameterIsNotNull()`的静态调用，该调用为每个`非null`引用参数添加。这些检查不会被添加到私有函数中，因为编译器保证了Kotlin类中的代码为`null`安全的。

### 可空的原生类型

可空类型始终是引用类型。将原生类型的变量声明成**可空类型**可以防止Kotlin使用Java基本数据类型(例如`int`或`float`)，而是使用**装箱的引用类型**(例如`Integer`或`Float`)，这会避免装箱和拆想操作带来的额外开销。

> 尽可能使用非null的原生类型，以此来提高代码可读性和性能。



## 数组

在Kotlin中存在3中类型的数组：

- `IntArray` `FloatArray` 以及其它原生类型的数组
  最终会编译成`int[]` `float[]` 以及其它对应基本数据类型的数组
- `Array<T>`
  非空对象引用类型数组，涉及到原生类型的装箱过程
- `Array<T?>`
  可空类型引用类型数组，涉及原生类型的装箱过程

> 如果需要一个`非null`原生类型的数组，最好使用`IntArray`而不是`Array<Int>`，以避免装箱过程带来的性能开销



## varargs

类似Java，Kotlin允许使用可变数量的参数声明函数。

```kotlin
fun printDouble(vararg values: Int) {
    values.forEach { println(it * 2) }
}
```

```kotlin
printDouble(1, 2, 3)
```

编译器会将可变数量的参数转换为数组

```java
printDouble(new int[]{1, 2, 3});
```

**传递单个数组**

与Java不同，在Kotlin中则需要使用`spread`伸展操作符：

```kotlin
val vlaues = intArrayOf(1, 2, 3)
printDouble(*values)
```

编译后

```java
int[] values = new int[]{1, 2, 3};
printDouble(Arrays.copyOf(values, values.length));
```

在Java中，数组引用按原样传递给函数，而无需分类额外的数组空间，而Kotlin在调用函数时，始终会复制现有数组。好处是代码更安全，它允许函数修改数组而不影响调用者代码，但是**会分配额外的内存**。

Kotlin伸展符`spread`运算符的主要好处是它还**允许在同一调用中将数组与其它参数混合使用**。

**传递数组和参数的混合**

```kotlin
val values = intArrayOf(1, 2, 3)
printDouble(0, *values, 42)
```

编译后

```java
int[] values = new int[]{1, 2, 3};
IntSpreadBuilder var10000 = new IntSpreadBuilder(3);
var10000.add(0);
var10000.addSpread(values);
var10000.add(42);
printDouble(var10000.toArray());
```

除了创建新数组之外，还是用了一个临时生成器对象来计算最终数组大小并填充它。给方法调用又增加了一笔开销。

>即使在使用现有数组中的值时，在Kotlin中调用具有可变数量参数的函数也会增加创建新临时数组的成本。对于重复调用该函数的性能至关重要的代码，请考虑添加具有实际数组参数而不是**vararg**的方法



## 代理属性和Range

### 代理属性

代理属性是一种集getter和可选setter的内部实现可由dialing的外部对象提供的属性。允许复用自定义属性的内部实现。

```kotlin
class Example {
  var p: String by Delegate()
}
```

代理对象必须实现一个operator getValue() 函数，以及一个setValue() 函数来用于属性的读写。这些函数将接收**包含对象实例** 以及**属性的metadata元数据** 作为额外参数(比如它的属性名)。

编译后的Java代码

```java
public final class Example {
   @NotNull
   private final Delegate p$delegate = new Delegate();
   // $FF: synthetic field
   static final KProperty[] ?delegatedProperties = new KProperty[]{(KProperty)Reflection.mutableProperty1(new MutablePropertyReference1Impl(Reflection.getOrCreateKotlinClass(Example.class), "p", "getP()Ljava/lang/String;"))};

   @NotNull
   public final String getP() {
      return this.p$delegate.getValue(this, ?delegatedProperties[0]);
   }

   public final void setP(@NotNull String var1) {
      Intrinsics.checkParameterIsNotNull(var1, "<set-?>");
      this.p$delegate.setValue(this, ?delegatedProperties[0], var1);
   }
}
```

### 代理实例

当代理对象需要通过其构造函数传递额外参数时，可以通过将dialing实例声明成object对象表达式，而不需要创建一个新的代理实例：

```kotlin
object FragmentDelegate {
  operator fun getValue(thisRef: Activity, property: KProperty<*>): Fragment? {
    return thisRef.fragmentManager.findFragmentByTag(property.name)
  }
}
```

> 在类中声明的每个代理属性都涉及到**其关联的代理对象创建的性能开销**，并向该类中添加一些metadata元数据。必要的时候，可以尝试为不同属性**复用**同一个代理实例。在你声明大量代理属性的时候，还需要考虑代理属性是否你的最佳选择。

### 泛型代理

还可以以泛型的方式声明代理函数，因此同一个代理类可以用任意的属性类型。

```kotlin
private var maxDelay: Long by SharedPreferencesDelegate<Long>()
```

但是，如果像上面例子那样使用具有原生类型属性的泛型代理的话，即便声明的原生类型为非null，每次读取或写入该属性时都避免不了**装箱和拆箱的发生**。

> 对于`非null`原生类型的代理属性，最好使用为该特定值类型创建特定的代理类，而不是泛型代理，以避免在每次访问该属性时产生的装箱开销

### 标准库代理 lazy()

```kotlin
private val dateFormat: DateFormat by lazy {
    SimpleDateFormat("dd-MM-yyyy", Locale.getDefault())
}
```

这是一种将昂贵的初始化操作延迟到实际需要使用之前的巧妙方法，可以在保持代码可读性的同时又提高了性能。

lazy()另一重载函数实际上还隐藏一个可选的模式参数来确定应该返回不同类型的代理：

```kotlin
public fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer)
public fun <T> lazy(mode: LazyThreadSafetyMode, initializer: () -> T): Lazy<T> =
        when (mode) {
            LazyThreadSafetyMode.SYNCHRONIZED -> SynchronizedLazyImpl(initializer)
            LazyThreadSafetyMode.PUBLICATION -> SafePublicationLazyImpl(initializer)
            LazyThreadSafetyMode.NONE -> UnsafeLazyImpl(initializer)
        }

```

默认模式是**`LazyThreadSafetyMode.SYNCHRONIZED`** 将执行相对开销昂贵的**双重锁的检查**，这是为了保证在**多线程**环境下读取属性时，初始化块可以安全运行。

如果你明确知道当前环境是单线程(例如主线程)访问属性，那么可以通过显式使用 **`LazyThreadSafetyMode.NONE`** 来完全避免双重锁的检查所带来昂贵的开销。

```kotlin
val dateFormat: DateFormat by lazy(LazyThreadSafetyMode.NONE) {
    SimpleDateFormat("dd-MM-yyyy", Locale.getDefault())
}
```

> 使用`lazy()`代理可以按需延迟昂贵的初始化，此外可以指定线程安全的模式以避免不必要的双重锁检查。

### Range区间

区间是一种用于表示Kotlin中的一组有限值的特殊表达式。这些值可以是任意`Comparable`类型。这些表达式由创建用于实现ClosedRange对象的函数形成。用于创建区间的主要函数是 `..`操作符。

区间表达式主要目的是使用**in** 和 **!in** 运算符来判断是否包含某个值

```kotlin
if (i in 1..10) {
    println(i)
}
```

编译后

```java
if(1 <= i && i <= 10) {
   System.out.println(i);
}
```

性能开销几乎为0，没有额外的对象分配。区间也可以和任意其他非原生`Comparable`类型一起使用。

```kotlin
if (name in "Alfred".."Alicia") {
    println(name)
}
```

编译后

```java
if(name.compareTo("Alfred") >= 0) {
   if(name.compareTo("Alicia") <= 0) {
      System.out.println(name);
   }
}
```



## 小结

- 局部函数是私有函数的替代品，其附加好处是能够访问外部函数的局部变量。然而这种好处会伴随着为外部函数每次调用创建Function对象的隐性成本，因此首选使用非捕获的局部函数。

- 对于release版本应用来说，特别是Android应用，可以使用`-Xno-param-assertions`编译器选项或添加以下Proguard规则来禁止运行时的空检查。

  ```groovy
  -assumenosideeffects class kotlin.jvm.internal.Intrinsics {
      static void checkParameterIsNotNull(java.lang.Object, java.lang.String);
  }
  ```



