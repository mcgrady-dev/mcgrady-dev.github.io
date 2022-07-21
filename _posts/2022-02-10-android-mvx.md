---
layout: 
title: Android MVX 开发模式
date: 2022-02-09 15:06 +0800
tags: [android,architecture]
---

Android MV**X**系列开发模式

<!--more-->



## MVC(Model-View-Controller)

![mvc](https://s2.loli.net/2022/07/19/9vWRi3DwhFlAe6I.png)

**MVC模式分为三部分：**

* 视图(View)：用户界面，它会接收用于的交互请求并展示数据信息给用户。
* 控制器(Controller)：主要担任Model与View之间的桥梁，用于控制程序的流程。Contrller负责确保View可以访问到需要显示的Model对象数据，并充当View了解Model更改的渠道。View接收到用户的交互请求之后，会将请求转发给Contrller，Contrller解析用户的请求之后，会交给对应的Model去处理。因此，理论上，Contrller应该是「很轻的」。
* 模型(Model)：主要管理业务模型的数据和行为，它即保存程序的数据，也定义了处理数据的逻辑。

**各部分之间的通信方式如下：**

1. 所有的通信都是单向的
2. View传送指令到Controller
3. Controller完成业务逻辑后，要求Model改变状态
4. Model将新的数据发送到View，用户得到反馈

**互动模式**
接受用户指令时，MVC可以分为两种方式。一种是通过View接受指令，传递给Controller；另一种是直接通过Controller接受指令。



## MVP (Model-View-Presenter)

![mvp](https://s2.loli.net/2022/07/19/SoJ9iOUjm1Dp58R.png)

MVP 的本质：是广义上的架构模式，适用于面向实体或虚拟用户接口的开发。它主要是在 MVC 的背景下，通过依赖倒置，来解决「逻辑复用难」 以及「实现替换难」的问题。

**MVP里他通常包含的4个要素：**

1. View：负责绘制UI元素、与用户进行交互(在Android中体现为Activity)
2. View Interface：需要View实现的接口，View通过View interface与Presenter进行交互，降低耦合，方便进行单元测试
3. Model：负责存储、检索、操纵数据(有时也实现一个Model interface用来降低耦合)
4. Presenter：作为View与Model交互的中间纽带，处理与用户交互的负责逻辑

MVP模式将Controller改名为Presenter，同时改变了通信方向。

1. 各部分之间的通信，都是双向的
2. View和Model不发生联系，都通过Presenter传递
3. View非常薄，不部署任何业务逻辑，称为"被动视图"(Passive View)，即没有任何主动性，而Presenter非常厚，所有逻辑都部署在那里。

**MVP 的优点**

- 降低耦合度，实现了Model和View真正的完全分离，可以修改View而不影响Modle
- 模块职责划分明显，层次清晰
- 隐藏数据
- Presenter可以复用，一个Presenter可以用于多个View，而不需要更改Presenter的逻辑（当然是在View的改动不影响业务逻辑的前提下）
- 利于测试驱动开发。以前的Android开发是难以进行单元测试的（虽然很多Android开发者都没有写过测试用例，但是随着项目变得越来越复杂，没有测试是很难保证软件质量的；而且近几年来Android上的测试框架已经有了长足的发展——开始写测试用例吧），在使用MVP的项目中Presenter对View是通过接口进行，在对Presenter进行不依赖UI环境的单元测试的时候。可以通过Mock一个View对象，这个对象只需要实现了View的接口即可。然后依赖注入到Presenter中，单元测试的时候就可以完整的测试Presenter应用逻辑的正确性。
- View可以进行组件化。在MVP当中，View不依赖Model。这样就可以让View从特定的业务场景中脱离出来，可以说View可以做到对业务完全无知。它只需要提供一系列接口提供给上层操作。这样就可以做到高度可复用的View组件。
- 代码灵活性

**MVP 的缺点**

- 由于V/P之间的互相耦合，从代码分层角度来说，层之间未做到单向引用；无法做到P层的业务复用。
- 不符合单一职责原则，由于V/P是一对一的，如果业务很复杂的话，P会承担大量的责任。
- 声明周期不易于管理，实际上大部分App并不想需要处理转屏等复杂应用场景，但即使这样，我们经常要关注页面关闭后，运行中的网络请求是否需要停止，是否会造成空指针，甚至内存泄露。
- 不利于单元测试，一般情况下，QA写的单元测试case是针对于业务逻辑的，但是如果没有独立的业务逻辑层，是非常不利于实施的。
- 编码风格无法统一，如果编码风格得不到统一，每个人在做业务需求，或帮助其他人调试代码，亦或进行 code review 的时候，会非常困难，这时候一套能够让每个人都写成风格相似代码的框架显得尤为重要。

**MVP 内存泄露**

- P层的耗时任务再页面销毁时是否执行很关键，假设当页面销毁时，P层内的任务执行完，由于P层没有再被内部类等持有引用，所以P层是会被回收的，那V层也不被P层持有引用，所以即使没在V层销毁时清空软引用和置空（V层），V层同样会被销毁，不存在内存泄露问题。
- V层是否被P层弱引用持有决定V层是否会内存泄露，假设当页面销毁时，P层内的任务再执行，由于V层是被P层弱引用持有，所以V层是会被GC回收的，而P层由于任务还在执行，所以回收不了。



## MVVM（Model-View-ViewModel）

![mvvm](https://s2.loli.net/2022/07/19/lPC2rz1KiDG5Wud.jpg)

MVVM 的本质：是狭义上的架构模式，专用于页面开发。

它主要是在多人协作的软件工程的背景下，通过只操作 ViewModel 中映射的视图数据来刷新视图状态，以此来解决视图调用的一致性问题从而规避不可预期的错误。

MVVM模式将Presenter改名为ViewModel，基本上与MVP模式完全一致，唯一的区别是，它采用`Data-Binding`双向绑定：View的变动，自动反映在ViewModel，反之亦然。



**数据绑定**

MVVM 中最重要的一个特性就是数据绑定，通过将View的属性绑定到ViewModel，可以使两者之间松耦合，也完全不需要在ViewModel里写代码去直接更新一个View。数据绑定系统还支持输入验证，这将提供了将验证错误传输到View的标准化方法。

通过数据绑定，当 ViewModel 的数据发生改变之后，与之绑定的 View 也会随之自动更新。反过来，如果 View 发生了变化，那 ViewModel 是否也同样会随之变化呢？这就涉及到数据绑定的两种类型：

- **单向绑定**：ViewModel 与 View 绑定之后，ViewModel 变化后，View 会自动更新，但反之不然，即数据传递的方向是单向的。**（ViewModel —> View）**
- **双向绑定**：ViewModel 与 View 绑定之后，如果 View 和 ViewModel 中的任何一方变化后，另一方都会自动更新，这就是双向绑定。（**Model <—> View**）

一般情况下，在视图中只显示而无需编辑的数据用单向绑定，需要编辑的数据才用双向绑定。

前面我们已经了解到，ViewModel 封装的数据包含 View 的属性和命令两种，因此，数据绑定其实也可分为**属性绑定**和**命令绑定**。比如，TextView 的内容绑定的就是属性，Button 的点击事件绑定的就是命令。



## MVI

![mvi](https://s2.loli.net/2022/07/19/EfyQ4McjsDgawbt.png)



## 表现层逻辑（Presenter/ViewModel）

为什么要让Presenter/ViewModel处理几乎所有的表现层逻辑？主要是为了提高可测试性，将尽可能多的表现层逻辑纳入到单元测试的范围中。因为对视图控件的显示等等进行单元测试太难了，所以View是基本上没发进行单元测试的。但是Presenter/ViewModel完全可以进行单元测试：

```kotlin
class ProfilePresenterTest {
  private val presenter: ProfilePresenter
  private val view: ProfileView
  
  @Test
  fun testShowEditStateOnBtnClick() {
    // 浏览状态下点击编辑按钮，验证View是否显示了编辑状态视图
		// 也就是验证view.showEditState()方法是否被调用了
		presenter.setState(State.NORMAL)
		presenter.onEditStateButtonClicked()
		Mockito.verify(view).showEditState()
  }
  
  @Test
  fun testShowNormalStateOnBtnClick() {
    // 编辑状态下点击完成按钮，验证View是否显示了浏览状态视图
		// 也就是验证view.showNormalState()方法是否被调用了
		presenter.setState(State.EDIT)
		presenter.onEditStateButtonClicked()
		Mockito.verify(view).showNormalState()
  }
}
```



## 业务层逻辑（Model）

静态的业务数据不能代表Model层，业务数据以及针对业务数据的操作功能构成了Model层，这也就是业务逻辑。Model层如何处理业务逻辑，来自于Presenter/ViewModel层的业务指令。

```kotlin
class RecommendBlogFeedPresenter {
  private val view: RecommendBlogFeedView
  private val model: BlogModel
  
  fun onStart() {
    view.showLoadWait()
    model.loadRecommendBlogs { blogs: List<Blog> ->
				view.showBlogs(blogs)
    }
  }
}
```

```kotlin
interface BlogModel {
  void loadRecommendBlogs(callback: LoadCallback<List<Blog>>)
}
```

```kotlin
class BlogModelImpl : BlogModel {
  private val repo: BlogFeedRepository
  
  override fun loadRecommendBlogs { callback: LoadCallback<List<Blog>> ->
    callback.onLoaded(repo.fetch("recommend"))
  }
```

```kotlin
interface BlogFeedRepository {
  fun fetch(tag: String): List<Blog>
}
```







