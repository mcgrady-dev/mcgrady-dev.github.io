## 第1章 Android特性

### 1.1 Android 5.0 新特性

#### 1.1.1 Android 5.0 主要新特性概述

##### 支持64位ART虚拟机

弃用Dalvik虚拟机，改用了ART（Android Running Time）虚拟机，实现真正的跨平台编译，在arm、x86、mips等无处不在。

#### 1.1.2 替换ListView和GridView的RecyclerView

RecyclerView架构提供了一种插拔式的体验，具有高度解耦、灵活性和更高的效率。

### 1.2 Android 6.0 新特性

#### 1.2.1 Android 6.0 主要新特性概述

##### 应用权限管理

##### Doze电量管理

手机静止一段时间后，会进入Doze电量管理模式



## 第3章 View体系与自定义View

### 3.1 View和ViewGroup

### 3.2 坐标系

Android系统中有两种坐标系，分别为**Android坐标系**和**View坐标系**。

#### 3.2.1 Android 坐标系

![64e8d195690394e88f6cd067d2b20987](../../../blog/images/android/android-coordinate-system.png)

#### 3.2.2 View坐标系

它与Android坐标系并不冲突，两者是共同存在的。

<img src="../../../blog/images/android/android-view-coordinate-system.png" alt="android-view-coordinate-system" style="zoom:50%;" />

##### View自身的坐标 

通过如下方法可以获得View到其父控件（ViewGroup）的距离:

- getTop()：获取View自身顶边到其父布局顶边的距离。 
- getLeft()：获取View自身左边到其父布局左边的距离。 
- getRight()：获取View自身右边到其父布局左边的距离。 
- getBottom()：获取View自身底边到其父布局顶边的距离。 

##### MotionEvent提供的方法 

上图中间的那个圆点，假设就是我们触摸的点。我们知道无论是View还是ViewGroup，最终的点击事件都会由`onTouchEvent(MotionEvent event)`方法来处理。MotionEvent在用户交互中作用重大，其内部提供了很多事件常量，比如我们常用的`ACTION_DOWN`、`ACTION_UP`和`ACTION_MOVE`。此外，MotionEvent也提供了获取焦点坐标的各种方法：

- getX()：获取点击事件距离控件左边的距离，即视图坐标。 
- getY()：获取点击事件距离控件顶边的距离，即视图坐标。 
- getRawX()：获取点击事件距离整个屏幕左边的距离，即绝对坐标。 
- getRawY()：获取点击事件距离整个屏幕顶边的距离，即绝对坐标。

### 3.3 View的滑动

View的滑动是Android实现自定义控件的基础，同时在开发中我们也难免会遇到View的滑动处理。其实 

不管是哪种滑动方式，其基本思想都是类似的：当点击事件传到View时，系统记下触摸点的坐标，手指移 

动时系统记下移动后触摸的坐标并算出偏移量，并通过偏移量来修改View的坐标。实现View滑动大概有6种滑动方法，分别是：

#### 3.3.1 layout()

```java
public boolean onTouchEvent(MotionEvent event) {
  //获取手指触摸点的横坐标和纵坐标
  int x = (int) event.getX();
  int y = (int) event.getY();
  
  switch (event.getAction()) {
    case MotionEvent.ACTION_DOWN:
      lastX = x;
      lastY = y;
      break;
    case MotionEvent.ACTION_MOVE:
      //计算移动距离
      int offsetX = x - lastX;
      int offsetY = y - lastY;
      //调用layout方法重新防止View的位置
      layout(getLeft()+offsetX, getTop()+offsetY, getRight()+offsetX, getBottom+offsetY);
      break;
  }
}
```

#### 3.3.2 offsetLeftAndRight() 与 offsetTopAndBottom()

```kotlin
override fun onTouchEvent(event: MotionEvent?): Boolean {
        val x = event?.x ?: 0F
        val y = event?.y ?: 0F

        when (event?.action) {
            MotionEvent.ACTION_DOWN -> {
                lastX = x
                lastY = y
            }
            MotionEvent.ACTION_MOVE -> {
                val offsetX = x.minus(lastX ?: 0F).roundToInt()
                val offsetY = y.minus(lastY ?: 0F).roundToInt()

                offsetLeftAndRight(offsetX)
                offsetTopAndBottom(offsetY)
            }
            else -> {}
        }
        return true
    }
```

#### 3.3.3 LayoutParams

LayoutParams主要保存了一个View的布局参数，因此我们可以通过LayoutParams来改变View的布局参数从而达到改变View位置的效果。

```kotlin
override fun onTouchEvent(event: MotionEvent?): Boolean {
        val x = event?.x ?: 0F
        val y = event?.y ?: 0F

        when (event?.action) {
            MotionEvent.ACTION_DOWN -> {
                lastX = x
                lastY = y
            }
            MotionEvent.ACTION_MOVE -> {
                val offsetX = x.minus(lastX ?: 0F).roundToInt()
                val offsetY = y.minus(lastY ?: 0F).roundToInt()

                var lp = (layoutParams as ConstraintLayout.LayoutParams).apply {
                    leftMargin = left + offsetX
                    topMargin = top + offsetY
                }
                layoutParams = lp

            }
            else -> {}
        }
        return true
    }
```

#### 3.3.4 动画



scollTo 与 scollBy

Scroller

