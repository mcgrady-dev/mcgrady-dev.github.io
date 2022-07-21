---
layout: 
title: Android Fragment
date: 2022-03-09 11:49 +0800
tags: android
---

Fragment 表示应用界面中可重复使用的一部分。Fragment 定义和管理自己的布局，具有自己的生命周期，并且可以处理自己的输入事件。Fragment 不能独立存在，而是必须由 Activity 或另一个 Fragment 托管。

<!--more-->



## 生命周期

![android-fragment-view-lifecycle](https://s2.loli.net/2022/07/19/KROYlEzUQxftZ3u.png)

可以看到 Fragment 比 Activity 多了几个额外的回调方法：

- **`onAttach(Activity)`**
  当Fragment与Activity发生关联时调用
- **`onCreateView(LayoutInflater, ViewGroup, Bundle)`**
  创建该Fragment的视图
- **`onActivityCreated(Bundle)`**
  当Activity的`onCreate()`方法返回时调用
- **`onDestoryView()`**
  与`onDestoryView()`相对应，当该Fragment的视图被移除时调用
- **`onDetach()`**
  与`onAttach()`相对应，当Fragment与Activity关联被取消时调用

>注意：除了`onCreateView()`，如果重写了其他的所有方法，必须调用父类对于该方法的实现。

### 举例

下面来举例来看下 Activity 和 Fragment 生命周期的变化。功能如下：共有两个 Fragment：F1 和 F2，F1 在初始化时加入 Activity，点击 F1 中的按钮调用 replace 替换为 F2。

当 F1 在 Activity 的 `onCreate` 中被添加时，日志如下：

```
BasicActivity: [onCreate] BEGIN
BasicActivity: [onCreate] END
BasicActivity: [onStart] BEGIN
Fragment1: [onAttach] BEGIN 
Fragment1: [onAttach] END
BasicActivity: [onAttachFragment] BEGIN
BasicActivity: [onAttachFragment] END
Fragment1: [onCreate] BEGIN
Fragment1: [onCreate] END
Fragment1: [onCreateView]
Fragment1: [onViewCreated] BEGIN
Fragment1: [onViewCreated] END
Fragment1: [onActivityCreated] BEGIN
Fragment1: [onActivityCreated] END
Fragment1: [onStart] BEGIN
Fragment1: [onStart] END
BasicActivity: [onStart] END
BasicActivity: [onPostCreate] BEGIN
BasicActivity: [onPostCreate] END
BasicActivity: [onResume] BEGIN
BasicActivity: [onResume] END
BasicActivity: [onPostResume] BEGIN
Fragment1: [onResume] BEGIN
Fragment1: [onResume] END
BasicActivity: [onPostResume] END
BasicActivity: [onAttachedToWindow] BEGIN
BasicActivity: [onAttachedToWindow] END
```

接下来分两种情况，分别是不加 `addToBackStack` 和加 `addToBackStack`。

1）那个点击 F1 的按钮，调用 replace 替换为 F2，且不加 `addToBackStack` 时，日志如下：

```
Fragment2: [onAttach] BEGIN
Fragment2: [onAttach] END
BasicActivity: [onAttachFragment] BEGIN
BasicActivity: [onAttachFragment] END
Fragment2: [onCreate] BEGIN
Fragment2: [onCreate] END
Fragment1: [onPause] BEGIN
Fragment1: [onPause] END
Fragment1: [onStop] BEGIN
Fragment1: [onStop] END
Fragment1: [onDestroyView] BEGIN
Fragment1: [onDestroyView] END
Fragment1: [onDestroy] BEGIN
Fragment1: [onDestroy] END
Fragment1: [onDetach] BEGIN
Fragment1: [onDetach] END
Fragment2: [onCreateView]
Fragment2: [onViewCreated] BEGIN
Fragment2: [onViewCreated] END
Fragment2: [onActivityCreated] BEGIN
Fragment2: [onActivityCreated] END
Fragment2: [onStart] BEGIN
Fragment2: [onStart] END
Fragment2: [onResume] BEGIN
Fragment2: [onResume] END
```

2）那个点击 F1 的按钮，调用 replace 替换为 F2，且加 `addToBackStack` 时，日志如下：

```
Fragment2: [onAttach] BEGIN
Fragment2: [onAttach] END
BasicActivity: [onAttachFragment] BEGIN
BasicActivity: [onAttachFragment] END
Fragment2: [onCreate] BEGIN
Fragment2: [onCreate] END
Fragment1: [onPause] BEGIN
Fragment1: [onPause] END
Fragment1: [onStop] BEGIN
Fragment1: [onStop] END
Fragment1: [onDestroyView] BEGIN
Fragment1: [onDestroyView] END
Fragment2: [onCreateView]
Fragment2: [onViewCreated] BEGIN
Fragment2: [onViewCreated] END
Fragment2: [onActivityCreated] BEGIN
Fragment2: [onActivityCreated] END
Fragment2: [onStart] BEGIN
Fragment2: [onStart] END
Fragment2: [onResume] BEGIN
Fragment2: [onResume] END
```

可以看到 F1 被替换时，最后只回调了 `onDestroyView`，并没有调用 `onDestory` 和 `onDetach`。当用户点返回按钮回退事务时，F1 会调用 `onCreateView`->`onStart`->`onResume`，因此在 Fragment 事务中加不加 addToBackStack 会影响 Fragment 的生命周期。



## Fragment 的优势

### 模块化（Modularity）

Fragment 允许您将界面划分为离散的区块，从而将模块化和可重用性引入 Activity 的界面。Activity 是围绕应用的界面放置全局元素（如抽屉式导航栏）的理想位置。相反，Fragment 更适合定义和管理单个屏幕或部分屏幕的界面。

### 可重用（Reusability）

多个Activity可以重用一个Fragment。

### 可适配（Adaptability）

根据硬件的屏幕尺寸、屏幕方向，能够方便地实现不同的布局，这样用户体验更好。



## FragmentManger

Fragment管理器负责对应用的Fragment执行一些操作，如添加、移除或替换它们，以及将它们添加到回退栈。

### 跟踪回退栈状态

通过实现 `OnBackStackChangedListener` 接口来实现回退栈状态跟踪：

```pseudocode
/** 设置回退栈监听接口 **／
getSupportFragmentManager().addOnBackStackChangedListener(this);

/** 实现接口所要实现的方法 **/
@Override
public void onBackStackChanged() {
    //do whatevery you want
}
```



## FragmentTransaction

| API                            | Description                                                  |
| ------------------------------ | ------------------------------------------------------------ |
| `commit()`                     | 异步提交事务，不允许状态丢失                                 |
| `commitAllowingStateLoss()`    | 异步提交事务，允许状态丢失                                   |
| `commitNow()`                  | 同步提交事务，不允许状态丢失                                 |
| `commitNowAllowingStateLoss()` | 同步提交事务，允许状态丢失                                   |
| `executePendingTransactions()` | 同步执行事务队列中的全部事务                                 |
| show()                         | 显示Fragment，只是纯粹把`setVisibility`设为`true`，前提是Fragment已经被添加到容器。 |
| hide()                         | 隐藏Fragment，只是纯粹把`setVisibility`设为`false`，前提是Fragment已经被添加到容器。 |
| detach()                       | 从布局中移除Fragment，但仍被FragmentManager管理              |
| attach()                       | 添加到FragmentManager<br />生命周期变化：`onCreateView()`->`onStart()`->`onResume()` |



### setReorderingAllowed(boolean reorderingAllowed)

设置是否允许优化事务内和跨事务的操作。当执行多个Transaction时，可以通过`setReorderingAllowed(true)`优化冗余操作，例如：两个事务一起执行，一个添加FragmentA，下一个替换为FragmentB，则操作将取消，仅添加FragmentB。

> 优化冗余操作的副作用是，Fragment的状态变更，可能会带来超出预期的顺序。

### BackStackRecord

```
final class BackStackRecord extends FragmentTransaction implements
        FragmentManager.BackStackEntry, FragmentManager.OpGenerator
```

BackStackRecord 对象记录了这个事务的全部操作轨迹，随后提交到 FragmentManager 的执行队列中，等待执行。

```java
getSupportFragmentManager().beginTransaction()
  .add(R.id.container, f1, "f1")
  .addToBackStack("")
  .commit();
```

BackStackRecord 的定义有三重含义：

- 继承 FragmentManager，即事务，保存了整个事务的全部操作轨迹
- 实现了BackStackEntry，作为回退栈的元素，正是因为该类拥有事务全部的操作轨迹，因此在popBackStack()时能回退整个事务
- 继承了 OpGenerator，即被放入FragmentManager执行队列，等待被执行。



## Fragment 通信

### Interface 方案

Fragmnet 中定义接口，Activity 实现该接口，在 Fragment onAttach 中，将 Context 强转为 接口对象

### FABridge 方案

[FABridge](https://github.com/hongyangAndroid/FABridge) 通过注解的形式免去了接口形式进行数据传递的麻烦

### ViewModel 方案

Fragment 与 Activity 共享 ViewModel 实现数据共享





## Fragment懒加载

### 1. `add+show+hide` 模式下的方案

<script src="https://gist.github.com/mcgrady-dev/98d97936b047187fdf011390a223d926.js"></script>

通过`onHiddenChanged`监控fragment的状态判断是否执行数据加载

### 2. ViewPager+Fragment 模式下的方案

ViewPager+Fragment模式下需要控制`setUserVisibleHint(boolean isVisibleToUser)`函数，它与`onHiddenChanged()`非常相似，都是通过传入的参数值来判断当前Fragment是否对用户可见。

<script src="https://gist.github.com/mcgrady-dev/beb099861e0f90fe096ff945a93e196c.js"></script>

### 3. 混合模式下方案

我们可能会遇到更为复杂的 Fragment 嵌套组合。比如 `Fragment+Fragment`、`Fragment+ViewPager`、`ViewPager+ViewPager`….等等。

<script src="https://gist.github.com/mcgrady-dev/ed595bbe4ed8a885a42fa8214c7c4f0d.js"></script>

### 4. AndroidX 模式下的方案

<script src="https://gist.github.com/mcgrady-dev/d7f8ed918e18b4c7b5bfb7e9f03a1caa.js"></script>

### 5. ViewPager2下的方案

ViewPager2 本身就支持对实际可见的 Fragment 才调用 onResume 方法。



## Fragment 使用注意事项

1. 使用 newInstance 初始化 Fragment
   在横竖屏情况下 Fragment 发生重建时，是通过反射调用无参构造方法进行的，当 Fragment 含有参构造函数时，以上情况并调用有参构造函数，而是会检查`arguments`中是否有参数存在，有则拿出来重用。
2. 切换 Fragment 时，对用户数据的处理
   情景：在 FragmentA 中的 EditText 填入了一些数据，然后切换到 FragmentB 时
   - 如果希望回到 FragmentA 后还能看到填入的数据，则适合用`hide/show`。
   - 若不希望保留用户操作，可以使用`remove/add`，或者`replace`，两者效果相同。
3. remove 和 detach 的区别
   在不考虑会退栈的情况下，`remove`会销毁整个 Fragment 实例，而`detach`则只是销毁其视图结构（实例不会被销毁）。如果当前Activity一直存在，在不希望保留用户操作的时候，可以优先使用 `detach`。
4. `commit()`
   `coomit()` 操作是异步的，内部有 `checkSateLoss()` 操作；`commit()`操作在`onSaveInstanceState()`之后时，可能会抛出异常，而`commitAllowingStateLoss()`方法则是不会抛出异常版本的`commit()`方法，但是尽量使用`commit()`，而不要使用`commitAllowingStateLoss()`。
5. `commitNow()`
   `commitNow()` 操作是同步的，对应异步的 `commit()`。
6. Fragment 重叠问题
   由于 Fragment 被系统销毁，重新初始化时再次将 Fragment 加入 Activity 导致，通过外围添加 if 语句判断此时是否为系统销毁并重新初始化的情况。
7.  Can not perform this action after onSaveInstanceState 异常
   该异常的出现原因是：`commit()`在`onSaveInstanceState()`后调用。首先，`onSaveInstanceState()`在`onPause()`之后，`onStop()`之前调用。`onRestoreInstanceState()`在`onStart()`之后，`onResume()`之前。
   规避方案：
   - 不要把 Fragment 事务放在异步线程中调用。
   - 逼不得已时使用 `commitAllowingStateLoss()`
8. 







**参考文献：**

[Fragment Lifecycle](https://developer.android.com/guide/fragments/lifecycle)

[《Android基础：Fragment，看这篇就够了》](https://mp.weixin.qq.com/s/dUuGSVhWinAnN9uMiBaXgw?)