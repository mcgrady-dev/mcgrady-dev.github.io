# 单例模式



## 简介

**单列模式**（singleton），是一种常用的软件设计模式，**单列对象的类必须只有一个实例存在，并且自行实例化向整个系统提供**。



## Java

### 构建方式
#### 1. 懒汉模式
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

#### 2. 饿汉模式
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

#### 3. 双重锁模式
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

#### 4. 实现Serializable接口
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

#### 5. 枚举

```java
public enum Singleton {
    INSTANCE;
}
```

`INSTANCE;`是`enum`的一块语法糖，JVM会阻止反射获取枚举类的私有构造方法。使用枚举的方法起到了单例的作用，但也有弊端，就是无法进行懒加载。



## Kotlin

### 使用 Object 实现单例

```kotlin
object Singleton
```



### 使用 by lazy 实现单例

```kotlin
class Singleton private constructor() {

    companion object {
        // 方式二
        @JvmStatic
        val INSTANCE1 by lazy(mode = LazyThreadSafetyMode.SYNCHRONIZED) { WorkSingleton() }

        // 方式三 默认就是 LazyThreadSafetyMode.SYNCHRONIZED，可以省略不写，如下所示
        @JvmStatic
        val INSTANCE2 by lazy { WorkSingleton() }
    }
}
```

lazy 延迟模式的几种使用说明：

#### 1. LazyThreadSafetyMode.SYNCHRONIZED

默认，如果有多个线程访问，只有一条线程可以去初始化 lazy 对象。

#### 2. LazyThreadSafetyMode.PUBLICATION

对于还没有被初始化的 lazy 对象，可以被不同的线程调用，如果 lazy 对象初始化完成，其他的线程使用的是初始化完成的值。

#### 3. LazyThreadSafetyMode.NONE

只能在单线程下使用，不能在多线程下使用，不会有锁的限制，也就是说它不会有任何线程安全的保证以及相关的开销。



### 可接收参数的单例

```kotlin
class Singleton private constructor(context: Context) {
  init {
    //Init using context argument
  }
  
  companion object : SingletionHolder<Singleton, Context>(::Singleton)
}

open class SingletonHolder<out T, in A>(creator: (A) -> T) {

    private var creator: ((A) -> T)? = creator

    @Volatile
    private var instance: T? = null

    fun getInstance(arg: A): T =
        instance ?: synchronized(this) {
            instance ?: creator!!(arg).apply {
                instance = this
            }
        }
}
```

