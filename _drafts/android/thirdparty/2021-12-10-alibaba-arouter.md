---
title: ARouter原理分析
tags: android
aside:
  toc: true
---

# ARouter原理分析




## 框架分析

### module: arouter-annotation

框架中主要支持的注解定义，以及RouteMeta等基础定义。

#### @Route

在编译期间通过注解处理器扫描所有添加`@Route`标注的Activity类，然后将注解中的`Path`和`Activity.class`的映射关系保存到它自己生成的Java文件中。

#### @Autowired

主要完成页面跳转过程中，Intent参数的自动填充，主要的目的在于省去了手动编写解析Intent参数的代码。

#### @Interceptor

用于注册路由拦截器的实现

#### RouteMeta

路由元数据，里面包含了基本的路由信息

```java
public class RouteMeta {
    private RouteType type;         // Type of route
    private Element rawType;        // Raw type of route
    private Class<?> destination;   // Destination
    private String path;            // Path of route
    private String group;           // Group of route
    private int priority = -1;      // The smaller the number, the higher the priority
    private int extra;              // Extra data
    private Map<String, Integer> paramsType;  // Param type
    private String name;

    private Map<String, Autowired> injectConfig;  // Cache inject config.
  	...
}
```



### module: arouter-compiler

用于处理注解，在编译期通过apt生成类文件。

#### RouteProcessor

用来处理`@Route`注解的注解处理器，主要完成以下几部分的代码生成工作：

1. 扫描所有被`@Route`注解的类，生成对应的RouteMeta，分组放到groupMap中，key为groupName，value为支持排序放入的RouteMeta的Set集合。

2. 遍历groupMap中的Set<RouteMeta>中的RouteMeta生成对应代码（这里是个双层的for循环）

   ```java
   // Start generate java source, structure is divided into upper and lower levels, used for demand initialization.
   for (Map.Entry<String, Set<RouteMeta>> entry : groupMap.entrySet()) {
     String groupName = entry.getKey();
     ...
     // Build group method body
     Set<RouteMeta> groupData = entry.getValue();
     for (RouteMeta routeMeta : groupData) {
       ...
     }
     ...
   }
   ```

3. 生成rootMap，key为groupName，value为刚每个group对应生成的Java类的类名，根据rootMap生成对应的Java文件。



#### AutowiredProcessor

注解处理器会在编译阶段扫描有被`@Autowired`标注的变量，根据这个类和变量的情况生成一份Java代码，这份Java代码中主要的方法是`inject()`，它负责对相关变量的解析赋值，然后在`Activity.onCreate()`靠前的位置调用对应`ARouter.getInstance().inject(this)`方法即可。

#### InterceptorProcessor

路由拦截处理器，用于生成标注了`@Interceptor`的路由拦截实现实现类

### module: arrouter-api

路由跳转的实现模块

#### _ARouter

`_ARouter`是个单例类，包含了框架主要的功能调用，但`_ARouter`类并不对外提供使用，而是用外观模式对外提供了ARouter类。

##### afterInit()

在`init()`之后执行，使用框架本身的能力实例化了InterceptorService

##### navigation()

根据Postcard中的信息从路由表中获取对应的Class，然后通过反射，执行跳转或实例化该对象。

```java
protected void navigation(final Context context, final Postcard postcard, final int requestCode, final NavigationCallback callback);
```

##### inject()

通过自身的能力实例化AutowiredService，然后执行`AutowiredService.autowire()`方法。



#### LogisticsCenter

##### loadRouteMap()

由`arouter-gradle-plugin`插件在编译阶段插入注册Router相关代码。

##### init()

主要完成了相关路由信息的填充，如果`loadRouteMap()`中完成了Warehouse中各个路由index的加载则直接使用（`arouter-auto-register`），没有则通过运行时读取所有dex文件遍历每个entry查找指定包内的所有类名加载到Warehouse中。

##### completion()

在路由表中根据Postcard的path找到对应的路由信息RouteMeta，利用RouteMeta中信息为Postcard赋值。

#### Postcard

Postcard继承自RouteMeta，是整个路由过程中的信使，类似于生活中的明信片功能，包含了路由所有需要的信息。

```java
public final class Postcard extends RouteMeta {
    // Base
    private Uri uri;
    private Object tag;             // A tag prepare for some thing wrong.
    private Bundle mBundle;         // Data to transform
    private int flags = -1;         // Flags of route
    private int timeout = 300;      // Navigation timeout, TimeUnit.Second
    private IProvider provider;     // It will be set value, if this postcard was provider.
    private boolean greenChannel;
    private SerializationService serializationService;

    // Animation
    private Bundle optionsCompat;    // The transition animation of activity
    private int enterAnim = -1;
    private int exitAnim = -1;
  	...
}
```

##### greenChannel

`greenChannel()`就是绿色通道，用于忽略Interceptor的拦截。



#### Warehouse

路由映射表，包含了执行路由所需的Map集合。



#### DefaultPollExecutor

一个数组阻塞队列的线程池



#### Interceptor

拦截跳转过程

##### UniqueKeyTreeMap

`UniqueKeyTreeMap<Integer, Class<? extends IInterceptor>>`是具有唯一键值的TreeMap，由于TreeMap是一种有序的集合，所以数值越小，拦截器的优先级就越高。

##### InterceptorService

InterceptorService在`_ARouter.afterInit()`中执行实例化，主要方法有：

- doInterceptions：供外部调用接口方法，会在异步线程中开启拦截器额逐个调用，会去调用_excute方法，完成对拦截器IInterceptor的process方法的调用，通过CountDownLatch来控制是否所有拦截器都调用完成，且在超时时间内。
- _excute：调用IInterceptor.process方法，在callback的onContinue方法中递归调用自己，但调整index值，使用下一个拦截器。



#### PathReplaceService

预处理路径服务，框架提供了预处理的逻辑，需自行实例化该服务来使用。例如可以：重写跳转URL。



#### AutowiredService

`@Autowired`注解的参数注入服务

主要逻辑在`autowire()`方法，`autowire()`会调用`doInject()`方法递归注入注解处理器中生成的对应的Java文件对应的类名，最后通过反射生成对象。



### module: arouter-gradle-plugin

这个库是一个`gradle plugin`，目的是帮助在`com.alibaba.android.arouter.core.LogisticsCenter`类在`loadRouterMap()`方法中生成代码来插入各个module注册的所有的Route、Interceptor和Provider。

其中通过`RegisterTransform`完成扫描以`ROUTER_CLASS_PACKAGE_NAME = 'com/alibaba/android/arouter/routes/'`开头的类，然后调用`RegisterCodeGenerator`类的方法完成向`LogisticsCenter::loadRouterMap`插入代码。作用于class到dex文件的时期，用ASM框架插入字节码。

#### 字节码插桩对`loadRouterMap()`做了什么处理？

```java
//源码代码，插桩前
private static void loadRouterMap() {
	//registerByPlugin一直被置为false
    registerByPlugin = false;
}
//插桩后反编译代码
private static void loadRouterMap() {
    registerByPlugin = false;
    register("com.alibaba.android.arouter.routes.ARouter$$Root$$modulejava");
    register("com.alibaba.android.arouter.routes.ARouter$$Root$$modulekotlin");
    register("com.alibaba.android.arouter.routes.ARouter$$Root$$arouterapi");
    register("com.alibaba.android.arouter.routes.ARouter$$Interceptors$$modulejava");
    register("com.alibaba.android.arouter.routes.ARouter$$Providers$$modulejava");
    register("com.alibaba.android.arouter.routes.ARouter$$Providers$$modulekotlin");
    register("com.alibaba.android.arouter.routes.ARouter$$Providers$$arouterapi");
}
```



##  框架原理





## QA

1. 同一个module下注解相同的Path会怎么样？不用module下又会怎么样？
   答：
   - **在同一个module中定义了相同Path的不同的两个类**
     编译通过，在RouteProcessor中有一个成员变量`private Map<String, Set<RouteMeta>> groupMap = new HashMap<>();`由于有value是Set集合，那么字母表排在后面的元素会因为无法添加到Set中而失效。即只生成了排在前面的相关路由。
   - **不同的module中定义了相同Path的不同的两个类**
     编译报错，由于apt框架是分module编译的，并且每个module都会生成`ARouter$$Root$$module_name` `ARouter$$Group$$group_name`等文件，同时也会生成`ARouter$$Group$${path}`的文件，在合并dex的时候相同的两个文件就会报错。
2. 假设通过接口的方式获取服务的话，如果接口不止一个实现，会怎么样？
   答：字母表排在后面的接口实现会覆盖掉前面的。
3. 为什么不能用抽象类继承IProvider然后实现抽象类，而只能用接口继承IProvider然后实现该接口？
   答：ARouter只处理了接口的情况，没有处理抽象类。
4. 每次通过ARouter获取相同Path的服务，获取的都是同一个对象吗？为什么？
   答：同一个对象
5. `arouter-gradle-plugin`的作用是什么？网上说ARouter加载apk后第一次加载会耗时，又是怎么回事？
   答：
   - 是一个Gradle插件，主要帮助生成代码来插入各个module注册的所有的路由。
   - 由于通过运行时读取所有dex文件遍历每个entry查找指定包内的所有类名需要耗时。在第一次进入apk时，主线程必须等待子线程去扫描指定包名下的所有className，class越多，耗时越长。



## 框架中相关的技术点

### 设计模式

#### 外观模式



### 数据结构

#### TreeMap

TreeMap是一种有序的键值对集合



### 编程思想

#### AOP（切面编程）



### 其它

#### AutoRegister

一种更高效的组件自动注册方案。

**原理**：在编译期（代码混淆之前）扫描所有打到apk包中的类，将**符合条件**的类收集起来，通过字节码操作，并生成注册代码到指定的类的static块中，自动完成注册。尤其使用于命令模式后策略模式下的映射表生成。

在组件化框架中，可有助于分级按需加载的功能：

- 在组件管理类中生成自动注册的代码
- 在组件框架第一次被调用时加载此注册表
- 若组件汇总有很多功能提供给外部调用，可以将这些功能包装成多个Processor，并将它们自动注册到组件中进行管理
- 组件被初次调用时再加载这些Processor



#### Gradle插件开发



#### ASM（字节码修改框架）



#### APT

APT(Annotation Processing Tool)，即注解处理工具，它是在编译期对代码中指定的注解进行解析，然后做一些其它处理（如通过`javapoet`生成新的Java文件）。

##### JavaPoet

`javapoet`是square推出的开源java代码生成框架



### 编译插桩

编译插桩就是在代码编译期间修改已有的代码或者生成新代码。

理解编译插桩之前，先回顾下Android中`.java`文件的编译过程：![img](../../images/android/编译插桩.png)

1. 在`.java`文件编译成`.class`文件时，`APT`、`AndroidAnnotation`等就是在此处触发代码生成的。
2. 在`.class`文件进一步优化称号`.dex`文件时，也就是直接操纵字节码文件，这里就是字节码插桩。

ARouer注解生成用了第一种方法，而启动优化则用了第二种方法

![](../../images/android/alibaba-arouter-annoitaionto-dex.png)



### Java

#### 注解



#### 反射

```java
Object obj = clazz.getConstructor().newInstance();
```

框架中类的生成是采用`getConstructor().newInstance()`反射来进行的。



#### CountDownLatch

