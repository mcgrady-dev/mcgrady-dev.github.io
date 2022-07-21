---
layout: 
title: MAD：Jetpack Overview
date: 2022-03-09 11:41 +0800
tags: [android,jetpack,MAD]
---

Jetpack 组件概览

<!--more-->



## Lifecycle

**主要为了解决 生命周期管理的一致性问题。**

Lifecycle 通过**模板方法模式**和**观察者模式**，将生命周期管理的复杂操作，全部在作为 `LifecycleOwner` 的基类中（例如视图控制器的基类）封装好，默默地在背后为开发者运筹帷幄。

开发者因而得以在视图控制器（子类）中调用 `getLifecycle().addObserver(GpsMananger.getInstance())`，优雅地完成第三方组件在自己内部对 `LifecycleOwner` 生命周期的感知。



## LiveData

**通过唯一可信源分发状态的标准化开发理念，来完成生命周期安全的事件统一分发，以避免收到不可预期的推送或脏数据。**

### LiveData 存在前的混沌世界

在 LiveData 面世前，分发状态 多数是通过 `EventBus` 或 `Java Interface` 来完成，不管是用于网络请求回调，还是跨页面通信的情况。

首先 `EventBus/Java Interface` 缺乏了上述提到的标准化开发理念的约束，容易因去中心化地滥用，而造成诸如毫无防备的收到预期外不明来源的推送、拿到过时的数据 及 事件追溯复杂度为 n² 的局面；并且 EventBus 本身缺乏 `Lifecycle` 的加持，存在生命周期管理一致性的问题，这也是 EventBus 的硬伤。

### LiveData 为什么能解决上述这些问题？
LiveData 是 Google 希望确立标准化、规范化的开发模式，在这样一种背景下诞生的，LiveData 被十分克制的设计为，**仅支持状态的输入和监听，并且可基于“访问权限控制”来实现“读写分离”**。

这使得任何一次数据推送，都可被限制为“只能单方面地从唯一可信源推送而来”，从而避免了消息同步不一致、不可靠、或在事件追溯复杂度为 n² 的迷宫中白费时间。（即，无论从哪个视图控制器发起的 对某个共享状态改变的请求，状态最终的改变都由作为唯一可信源的单例或 ViewModel 在其内部统一决策，并一对多地通知改变）。

并且，这种承上启下的方式，使得单向依赖成为可能：单例无需通过 `Java Interface` 回调通知视图控制器，从而规避了视图控制器被生命周期更长的单例依赖所埋下的内存泄露隐患。

### LiveData 有个坑需要注意

为了在视图控制器发生重建后，能够 <u>自动倒灌</u> 所观察的 LiveData 的最后一次数据，LiveData 被设计为粘性的事件。

> 自动倒灌：即，页面通信（事件回调）的场景下，通过 ViewModel 的 LiveData 给当前页通知过一次，并返回上一页，下次再进入当前页时重复收到旧数据推送的情况。

因为 `ViewModel` 支持共享作用域，并且官方文档都推荐了通过共享 `ViewModel` 来实现跨页面通信的需求，那么基于“开闭原则”，LiveData 理应提供一个与 `MutableLiveData` 平级的底层支持，专门用非粘性的事件通信的情况，否则直接在跨页面通信中使用 `MutableLiveData` 必造成回调一致性问题及难以预期的错误。

关于非粘性 LiveData 的实现，网上存在通过 “事件包装类”（只适合 kotlin 的情况） 和 “反射干预 LastVersion” （适用于 Java 的情况）两种方式来解决：

- [《在 SnackBar 和其他事件中使用 LiveData》（SingleLiveEvent 案例）](https://juejin.im/post/6844903623252508685)
- [《LiveDataBus实现原理#用法详解#LiveData扩展》（反射案例）](https://blog.csdn.net/geyuecang/article/details/89028283)



## ViewModel

**ViewModel的存在，是为了建立起作用域可控、可共享的状态管理，主要用于状态管理和托管页面状态的分治。**

- 对于轻量状态：当视图控制器重建（saveInstanceState 机制）时以序列化的方式完成状态存储和恢复。
- 对于重量级状态：当网络请求获取的 Bean，可以通过生命周期长于试图控制器的 ViewModel 持有，从而得以直接从 ViewModel 恢复，而不是以效率较低的序列化方式。

### ViewModel 存在前的混沌世界

在 ViewModel 面世前，MVP 的Presenter 和 MVVM-Clean 的 ViewModel 都不具备状态管理分治的能力。

Presenter 和 Clean ViewModel 的生命周期都与视图控制器同生死，因而它们顶多是为 DataBinding 提供状态的托管，而无法实现状态分治。

### ViewModel 为什么能做到状态管理的分治？

ViewModel 基于工厂模式实现，使得 ViewModel 被 LifecycleOwner 所持有，通过 ViewModelProvider 来引用。

所以 ViewModel **类似于单例**，当作为 LifecycleOwner 的 Activity 所持有时，能够脱离 Activity 旗下 Fragment 的生命周期，从而实现作用域共享。

但 ViewModel **实际上又不是单例**，生命周期跟随作为 LifecycleOwner 的视图控制器，当 Owner （Activity/Fragment）被销毁时，它也被 `clear`。

此外，处于视图控制器重建的考虑，Google 在视图控制器基类中通过 retain 机制对 ViewModel 进行了保留，因此对于 作用域共享和视图重建的情况，状态因完好地被保留，而得以在视图控制器恢复时被直接使用。

再者，由于存在共享作用域的考虑，所以 ViewModel 本身也承担了跨页面通信（如事件回调）的职责。

### ViewModelProviders

#### ViewModelProviders.of() 的作用

1. 初始化 ViewModelProviders 内部维护的用于创建 VM 的 Factory（默认DefaultFactory），和用户存放 VM 的ViewModelStore
2. 通过 ViewModelStres 静态方法实例化 HolderFragment，并实例化 ViewModelStore

#### ViewModelProviders.get() 的作用

检查 ViewModelStore 是否已存在，否则通过 Factroy 实例化，并存到 ViewModelStore 中



## DataBinding

**主要用于在软件工程的背景下，解决视图调用的一致性问题。**

在使用 DataBinding 后，唯一改变的是，无需手工调用视图来 set 新状态，只需要 set 数据本身；因而，DataBinding 并非许多人不假思索人为的，将 UI 逻辑搬到 XML 中写从而难以调试。

**DataBinding 只负责绑定数据，负责作为 UI 逻辑末端的改变**（即，它是一个不可再分的原子操作，本来就不需要调试），原本在视图控制器中 UI 逻辑怎么写，现在还怎么写，只不过不再需要 `textView.setText(xxx)` ，而是直接 `xxx.set()` 。

**所以在 DataBinding 的帮助下，解决了以下的问题：**

1. 规避了视图调用的一致性问题，无需手工判空，乃至无需手动调用视图，从而完全不用编写 `findViewById`。

### DataBinding 为什么能解决这些问题？

首先，数据驱动意味着，控件的状态被分离到 ViewModel 中管理，并且 ViewModel 这一层只需负责状态变量本身的变化，至于该变量在布局中究竟哪些视图绑定、当前没有视图来绑定，ViewModel 不用管。

**那么控件是如何做到被通知且更新状态的呢？**

DataBinding 是通过适配器模式和观察者模式来管理控件刷新状态的。当状态变量发生变化时，需要开发者手动完成状态的更新。这将通知 DataBinding 中绑定该变量的控件刷新数据。

- 包含了 ViewBinding 所有功能
- 需要在模块级 `build.gradle` 添加 `dataBinding = true` 
- 需要在布局文件中添加 layout 标签才可以使用
- 支持 data 和 view 的双向绑定
- 效率低于 ViewBinding ，因为注释处理器会影响数据绑定的构建时间
- 会对每个 XML 文件生成绑定类

### BindingAdapter

```kotlin
class ImageHelper {
  
  @BindingAdapter({"imageUrl", "placeHolder", "error"})
  fun loadImage(view: ImageView, url: String? = null, holderDrawable: Drawable? = null, errorDrawable: Drawable? = null) {
    Glide.with(imageView.getContext())
    	.load(url)
    	.placeholder(holderDrawable)
    	.error(errorDrawable)
    	.into(view)
  }
}
```

xml中使用自定义属性

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <import type="com.databindingdemo.bean.UserBean" />
        <variable
            name="user"
            type="UserBean" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="15dp"
        android:orientation="vertical">

        <!-- 当imageUrl属性存在时，会自动调用ImageHelper的loadImage方法 -->
        <ImageView
            android:layout_width="120dp"
            android:layout_height="120dp"
            android:scaleType="centerCrop"
            app:error="@{user.errorUrl}"
            app:placeHolder="@{user.placeHolder}"
            app:imageUrl="@{user.picUrl}" />
    </LinearLayout>
</layout> 
```



DataBinding动态更新数据的两种方式：

### 单向绑定

#### 1. BaseObservable

BaseObservable实现了字段变动的通知，在变量的`getter`上使用`Bindable`注解，并通过`notifyPropertyChanged`通知更新即可。

```kotlin
class DoubleBindBean : BaseObservable(var content: String?) {
  @Bindable
  fun getContent() = content
  
  fun setContent(content: String) {
    this.content = content
    notifyPropertyChanged(BR.content)	//通知系统数据源发生变化，刷新UI界面
  }
}
```

#### 2. ObservableField

如果想省时或者数据类的字段很少时，可以使用ObservableField以及它的派生类：

- ObservableBoolean
- ObservableInt
- ObservableParcelable
- ...

```kotlin
class DoubleBindBean {
  username: ObservableField<String> = ObservableField()
}
```

ObservableField除了支持以上基础类型以外，还支持集合框架**Observable Collections**：

- ObservableArrayMap
- ObservableArrayList
- ...

### 双向绑定

![android-databinding-double](https://s2.loli.net/2022/07/19/dGAbjoyTPDZt6rv.png)

```xml
<EditText
    android:id="@+id/etPassword"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="@={ viewModel.password }" />
```

#### 1. @InverseBindingAdapter

```java
@InverseBindingAdapter(attribute = "android:currentTab")
public static int getCurrentTab(TabHost view) {
  return view.getCurrentTab();

```

#### 2. @InverseBindingMethod/@InverseBindingMethods

```java
@InverseBindingMethods({
        @InverseBindingMethod(type = SeekBar.class, attribute = "android:progress"),
})
public class SeekBarBindingAdapter {}
```

使用说明：

- @InverseBindingMethods注解用于标记类
- @InverseBindingMethod注解需要与@InverseBindingMethods注解结合使用才能发挥其功效
- @InverseBindingMethods需要与@BindingAdapter配合使用才能发挥功效

#### 3. @InverseMethod

为某个方法指定一个相反的方法。正方法与反方法的要求：

- 正方法与反方法的参数数量必须相同
- 正方法的最终参数的类型与反方法的返回值必须相同
- 正方法的返回值类型必须与反方法的最终参数类型相同

```java
@InverseMethod("convertIntToString")
public static int convertStringToInt(String value) {
    try {
        return Integer.parseInt(value);
    } catch (NumberFormatException e) {
        return -1;
    }
}
public static String convertIntToString(int value) {
    return String.valueOf(value);
}
```

#### 4. @Bindable

使用`@Bindable`注解标记的`get`方法，在编译时会在`BR`类中生成对应的字段，然后与`notifyPropertyChanged()`方法配合使用，当该字段中的数据被修改时，DataBinding会自动刷新对应view的数据，而不用拿到数据后重新`setText()`一遍。

```kotlin
class User : BaseObservable {
  private val name: String
  
  @Bindable
  fun getName(): String {
    return name
  }
}
```



## ViewBinding

允许您更容易地编写与视图交互的代码。

- 仅支持绑定View
- 不需要在布局文件中添加 layout 标签
- 需要在模块级 `build.gradle` 添加 `viewBindding = true` 
- 效率高于 DataBinding ，因为避免了与数据绑定相关的开销和性能问题
- 相比于 `kotlin-android-extensions` 插件避免了空异常
- 会对每个 XML 文件生成绑定类
- 绑定类的实例包含对相应布局中具有 ID 的所有 view 的直接引用



## kotlin-android-extensions

- 它公开了以 view#id 为名的全局变量，但该名称与实际的布局无关，没有针对无效查找进行检查
- 只适用于 kotlin
- 当 view 只存在于某些配置中时，它们没有空安全提示



## Navigation

通过声明式编程来解决应用内路由导航的一致性问题。

















