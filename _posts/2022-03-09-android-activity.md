---
layout: 
title: Android Activity
date: 2022-03-09 11:41 +0800
tags: android
---

**Activity 是 Android 的应用程序组件，用于处理用户操作**。
Activity 提供用户一个交互的接口，Activity 本身没有界面，所以 Activity 创建了一个窗口，开发人员可以通过 `setContentView(view)` 接口把 UI 放到 Activity 创建的窗口上。

<!--more-->



## Activity生命周期

![activity-lifecycle](https://s2.loli.net/2022/07/19/VrPd2OFWQCfgAqR.png)

| Activity生命周期回调方法 | 方法描述                                                     |
| :----------------------: | ------------------------------------------------------------ |
|        onStart()         | **用户可见不可交互** 的状态，表示活动将被展现给用户。        |
|       onRestart()        | 处于停止状态的活动需要再次展现给用户的时候，触发该方法。     |
|        onResume()        | **用户可交互**状态，当活动和用户发送交互的时候，触发该方法。 |
|        onPause()         | 当一个正在前台运行的活动因其他活动需要前台运行而转入后台运行时，触发该方法。(这时需要将活动的状态持久化，比如正在编辑的数据库记录等) |
|         onStop()         | 当活动不需要展示给用户的时候，触发该方法。（如果内存吃紧，系统会直接结束这个活动，而不会触发 `onStop()`方法） |
|       onDestroy()        | 当活动销毁时，触发该方法。如果内存吃紧，系统会直接结束这个活动，而不会触发`onDestroy()`方法。 |



## Activity的4种启动模式

### standard（默认模式）

系统在启动该 Activity 的任务中创建 Activity 的新实例，Activity 可以多次实例化，每个实例可以属于不同的任务，一个任务可以拥有多个实例。

应用场景：Mainfest 中没有配置就默认标准模式。

### singleTop

如果当前任务的顶部已存在 Activity 的实例，则系统会通过调用其 `onNewIntent()` 方法来将 intent 转送给该实例，而不是创建 Activity 的新实例。Activity 可以多次实例化，每个实例可以属于不同的任务，一个任务可以拥有多个实例（但前提是返回堆栈顶部的 Activity 不是该 Activity 的现有实例）。

### singleTask

系统会创建新 Task，并实例化新 Task 的根 Activity。但是，如果另外的 Task 中已存在该 Activity 的实例，则系统会通过调用其 `onNewIntent()` 方法将 intent 转送到该现有实例，而不是创建新实例。Activity 一次只能有一个实例存在。

![android_diagram_backstack_singletask_multiactivity](../../images/android/android_diagram_backstack_singletask_multiactivity.png)

应用场景：程序模块逻辑入口:主页面（Fragment的containerActivity）、WebView页面、扫一扫页面、电商中：购物界面，确认订单界面，付款界面。

### singleInstance

与 `"singleTask"` 相似，唯一不同的是系统不会将任何其他 Activity 启动到包含该实例的 Task 中。该 Activity 始终是其 Task 唯一的成员；由该 Activity 启动的任何 Activity 都会在其他的 Task 中打开。



## Intent 标记

| Intent 标记                        | 说明                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| FLAG_ACTIVITY_SINGLE_TOP           | 等同于 singleTop                                             |
| FLAG_ACTIVITY_NEW_TASK             | 等同于 singleTask                                            |
| FLAG_ACTIVITY_CLEAR_TOP            | 在 launchMode 中没有对应的属性值。<br />如果要启动的 activity 已经在当前 task 中运行，<br />则不再启动一个新的实例，且所有在之上的 activity 将被销毁。 |
| FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS | 其对应在AndroidManifest中的属性为android:excludeFromRecents=“true”<br />当用户按了“最近任务列表”时候，<br />该Task不会出现在最近任务列表中，可达到隐藏应用的目的。 |



## Activity 被回收的状态和信息保存/恢复过程

```java
public class MainActivity extends Activity {
  
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    //判断是否有之前保存的状态信息
    if (savedInstanceState != null) {
      savedInstanceState.get("key");
    }
    super.onCreate(savedInstanceState);
  }
  
  @Override
  protected void onSaveInstanceState(Bundle outState) {
    //可能被回收内存前保存状态和信息
    Bundle data = new Bundle();
    data.putString("key", "last words before be kill");
    outState.putAll(data);
    super.onSaveInstanceState(outState);
  }
  
  @Override
  protected void onRestoreInstanceState(Bundle savedInstanceState) {
    //判断是否有以前的保存状态信息
    if (savedInstanceState != null) {
      savedInstanceState.get("key");
    }
    super.onRestoreInstanceState(savedInstanceState);
  }
}
```

### onSaveInstanceState()

即当系统“未经你许可”时销毁了你的activity，则`onSaveInstanceState`会被系统调用

当系统“未经你许可”时销毁了 Activity，在 Activity 可能被回收之前，调用 `onSaveInstanceState` 保存状态和信息，以便回收后重建时恢复数据（在 `onCreate()` 或 `onRestoreInstanceState()` 中恢复）。如果要调用就一定发生在 `onStop`方法之前，但并不保证发生在 `onPause` 的前面还是后面。

### onRestoreInstanceState()

这个方法在 `onStart()` 和 `onPostCreate()` 之间调用，在 `onCreate()` 中也可以状态恢复，但有时候需要所有布局初始化后再恢复状态。

需要注意的是，`onSaveInstanceState`方法和`onRestoreInstanceState方法`“不一定”是成对的被调用的。



## Activity的任务和返回堆栈

Task 是用户在执行某项工作时与之互动的一系列 Activity 的集合。这些 Activity 按照每个 Activity 打开的顺序排列在一个返回堆栈中。

- 每个 Activity 的状态是由它在 Activity 栈中的位置决定的。
- 一个应用程序的优先级是受最高优先级的 Activity 影响的。
- 当决定某个应用程序是否要终结且释放资源时，Android 内存管理将使用栈来决定基于Activity的应用程序的优先级。
