---
layout: 
title: Android Jetpack Navigation
date: 2022-02-17 14:23 +0800
tags: [android,jetpack]
---

Navigation是支持用户导航、进入和退出应用中不同内容Fragment的交互。

<!--more-->



### Navigation组件关键组成部分

- Navigation
  在一个集中位置包含所有导航相关信息的XML资源。
- NavHost
  显示Navigation中目标的空白容器。导航组件包含一个默认 `NavHost` 实现 (`NavHostFragment`)，可显示 Fragment 目标。
- NavController
  在 `NavHost` 中管理应用导航的对象。当用户在整个应用中移动时，`NavController` 会安排 `NavHost` 中目标内容的交换。

### Navigation组件的优势

- 处理Fragment事务
- 默认情况下，正确处理往返操作
- 为动画和转换提供标准化资源
- 实现和处理深层连接
- 包括界面模式（例如抽屉式导航栏和底部导航栏），用户只需完成极少的额外工作。
- Safe Args - 可在目标之间导航和传递数据时提供类型安全的Gradle插件。
- ViewModel支持 - 您可以将ViewModel的范围限定为Navigation，以在Navigation的目标之间共享与界面相关的数据。
- Navigation Editor - 可视化的编辑导航图

### Navigation组件的缺点

- Navigation的底层对Fragment的管理直接采取了替换的方式，虽然它可以配合BottomNavigationView使用，但每次都重新加载显然是不合理的



## NavHostFragment

导航宿主容器

## NavController

- 对 navigation.xml 进行解析，获取所有 Destination（目标点）的引用或 Class 的引用
- 记录当前栈中 Fragment 的顺序
- 管理并控制着导航行为



## 导航原则

- 固定的起始目的地
- 使用一个栈来代表导航图的导航状态
- 向上按钮不会退出App
- 回退栈中向上和返回按钮是等价的
- 深度连接到目标或导航到想用的目标应产生相同的堆栈



## SafeArgs

```xml
<fragment
        android:id="@+id/register"
        ...
        >

        <argument
            android:name="EMAIL"
            android:defaultValue="2005@qq.com"
            app:argType="string"/>
    </fragment>
```

**写入argument属性**

```kotlin
btnRegister.setOnClickListener {
            val action = WelcomeFragmentDirections
                .actionWelcomeToRegister()
                .setEMAIL("TeaOf1995@Gamil.com")
            findNavController().navigate(action)
}
```

**使用argument属性**

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // ...
        val safeArgs:RegisterFragmentArgs by navArgs()
        val email = safeArgs.email
        mEmailEt.setText(email)
}
```



## 总结

![android-jetpack-navigation-frame](https://s2.loli.net/2022/07/19/kr73xPjBReE1Kvh.png)