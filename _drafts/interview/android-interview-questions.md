

## 1. Android 系统组件 

### 2.4 RecyclerView

1. RecyclerView 中 回收是什么？复用是什么？回收到哪里？复用从哪里拿？什么时候回收？什么时候复用？

   - **回收是什么**：当一个 `itemView` 从可见到不可见时，RecyclerView 利用回收机制将它放到内存中，以便 `item` 出现时，不用每次都去 `new` 一个新的 `itemView`，而是只去 `onBindViewHolder` 绑定数据就行了。
   - **复用是什么**：滑动过程中出现了新的 itemView，不用每次都去 `new`，而是优先从缓存中拿，缓存不能满足需求再去 `onCreateViewHolder` 创建新的 `itemView` 并封装到 `ViewHolder` 中
   - **回收到哪里**：RecyclerView 中的 `mCachedViews` ``mRecyclerPool``
   - **从哪里复用**：Recycler 中的 `mAttachedScrap` `mChangedScrap` mCachedViews  `ViewCacheExtension`  `mRecyclerPool`
   - **什么时候回收**：以 RecyclerView 的 `onTouchEvent#move` 事件为起点，发生在 itemView 消失的时候
   - **什么时候复用**：以 RecyclerView 的 `onTouchEvent#move` 事件为起点，发生在 itemView  从不可见到可见的时候

2. LayoutManager的作用是什么？LayoutManager样式有哪些？`setLayoutManager()` 源码里做了什么？

   - LayoutManager 是决定 RecyclerView 中 itemView 如何布局的关键
   - LinearLayoutManger、GridLayoutManager
   - 移除 itemView，清理缓存，设置LayoutManager，执行 requestLayout

3. ViewHolder的作用是什么？什么时候停止调用 onCreateViewHolder？

   - ViewHolder 是对 itemView 的封装，方便用于与 Adapter 进行绑定和回收复用机制
   - 当缓存中有可复用的 ViewHolder 时不会调用 onCreateViewHolder

4. 讲一下RecyclerView的缓存机制，滑动10个，再滑回去，会有几个执行onBindView？

   - RecyclerView缓存机制的关键在于内部的 Recycler#tryGetViewHolderFormPositionByDeadline 方法中，分别从 AttachedScrap、CacheViews、ViewCacheExtension、RecyclerPool 中获取可复用的 ViewHolder
   - 20次 onBindView

5. RecyclerView的缓存结构是怎样的？缓存的是什么？cachedView会执行onBindView吗?

   - AttatchedView、CacheViews、ViewCacheExtension、RecyclerPool 四层缓存
   - 缓存 ViewHolder
   - 不会，只有 RecyclerPool 中获取成功才执行 onBindViewHolder

6. RecyclerView的Recycler是如何实现ViewHolder缓存的？如何理解RecyclerView四级缓存是如何实现的？

   - 通过 tryGetViewHolderByPositionFormDeadline 执行 四层缓存机制
   - 四层缓存是如何实现的：
     1. 第一级，屏幕内缓存：AttachedScrap
     2. 第二级，屏幕外缓存：CachedViews
     3. 第三级，自定义缓存：ViewCacheExtension
     4. 第四级，缓存池：RecycleedViewPool

7. ViewHolder的封装如何对findViewById优化？ViewHolder中为何使用SparseArray代替HashMap存储ViweId？

8. RecyclerView绘制原理过程大概是怎样的？

   RecyclerView 不会对子View进行测量，而是交给 LayoutManager 处理；测量的会区分正向绘制和倒置绘制，绘制过程会先去顶一个锚点，然后分别向上或向下绘制。

9. 如何实现RecyclerView的局部更新，用过payload吗？`notifyItemChanged()`方法中的参数？
   payload 为 notifyItemChanged 方法中的一个可选参数，作用是对 itemView 中的指定 子View 做单独的刷新

10. 如何解决RecyclerView嵌套RecyclerView条目自动上滚的BUG，如何解决NestScrollView嵌套ScrollView滑动冲突？

    - 通过 addOnItemTouchListener 监听是否为嵌套的RecyclerView，是则拦截

11. RecyclerView滑动卡顿原因有哪些？如何解决嵌套布局滑动冲突？如何解决RecyclerView时限画廊卡顿？

12. RecyclerView常见的优化有哪些？实际开发中都是怎么做的，优化前后对比性能上有何提升？

13. 如何处理ViewPager嵌套水平RecyclerView横向滑动到底后不滑动ViewPager？如何解决RecyclerView使用Glide加载图片导致图片错乱问题？

14. 讲一下事件分发机制，RecyclerView是怎么处理内部ViewClick冲突的？

15. SnapHelper主要是做什么用的？SnapHelper是怎么实现支持RecyclerView的对齐方式？

16. SpanSizeLookup的作用是什么？SpanSizeLookup如何使用？SpanSizeLookup实现原理如何理解？

17. 如何实现高复杂的Type需求？如果不封装会出现什么问题和弊端？

18. 关于item条目点击事件在onCreateViewHolder中写和在onBindViewHolder中写有何区别？如何优化？

19. RecyclerView 与 ListView 的对比，缓存策略、优缺点？

20. ViewHolder 为什么要被声明成静态内部类？

21. ScrollView 嵌套 RecyclerView 通常会出现什么问题？

22. 如何实现RecyclerView局部刷新？

### 2.5 ViewPager

1. ViewPager中嵌套ViewPager怎么处理滑动冲突？
2. 怎么写一个不能滑动的 ViewPager ？
3. Viewpager切换掉帧有什么处理经验？
4. ViewPager2原理？
5. ViewPager切换Fragment什么最耗时？

### 2.6 CoordinatorLayout

1. CoordinatorLayout自定义behavior,可以拦截什么？

### 2.7 Dialog、PopupWindow

1. PopupWindow 和 Dialog 的区别?
   AlertDialog 是**非阻塞式**对话框，AlertDialog 弹出时，后台还可以做其他事情；而 PopupWindow 是**阻塞式**对话框，PopupWindow 弹出直至 PopupWindow 退出前，程序会一直等待，等待 dismiss 方法执行后，程序才会想下执行。
   这两种区别的表现是：

   - **不同点**：

   1. Popupwindow在显示之前一定要设置宽高，Dialog无此限制。
   2. Popupwindow默认不会响应物理键盘的back，除非显示设置了popup.setFocusable(true)；而在点击back的时候，Dialog会消失。
   3. Popupwindow不会给页面其他的部分添加蒙层，而Dialog会。
   4. Popupwindow没有标题，Dialog默认有标题，可以通过`dialog.requestWindowFeature(Window.FEATURE_NO_TITLE);`取消标题

   - **共同点**：
     1. 二者显示的时候都要设置Gravity，如不设置，Dialog默认是Gravity.CENTER
     2. 二者都有默认的背景，都可以通过`setBackgroundDrawable(new ColorDrawable(android.R.color.transparent));`去掉。

2. 为什么 Dialog 不能用 Application 的 Context ？

## 3. 动画

1. 介绍一下 Android动画？

2. 动画的分类以及区别？

   - 补间动画（Tween animation）
     是指通过指定View的初末状态和变化时间、方式，对View的内容完成一系列的图形变换来实现动画效果，包括移动、渐变、伸缩、旋转，一般是定义在`res-anim`这个资源文件夹下，然后`res-style`中定义这个动画。

     ```xml
     <style name="Anim_Pop_TopOrBotom">
         <item name="android:windowEnterAnimation">@anim/pop_in_bottom_to_top</item>
         <item name="android:windowExitAnimation">@anim/pop_out_top_to_bottom</item>
     </style>
     ```

     - Alpha（渐变透明度动画效果）
     - Scale（渐变尺寸伸缩动画效果）
     - Frame animation（帧动画）
     - 传统的动画方法，通过顺序的播放排列好的图片来实现，类似电影。
     - Translate（画面转换位置移动动画效果）
     - Rotate（画面转移旋转动画效果）
     - Drawable Animation

   - 视图动画（View Animation）

   - 属性动画（Property Animation）
     Android3.0之后增加的特性，用来在特定的时间修改对象的属性，例如背景颜色和alpha等，可定义在`res-animator`资源文件中。

     ```xml
     <set android:ordering=["together" | "sequentially"]]]>  
         <objectAnimator  
             android:propertyName="string"  
             android:duration="int"  
             android:valueFrom="float | int | color"  
             android:valueTo="float | int | color"  
             android:startOffset="int"  
             android:repeatCount="int"  
             android:repeatMode=["repeat" | "reverse"]  
             android:valueType=["intType" | "floatType"]/>  
         <animator  
             android:duration="int"  
             android:valueFrom="float | int | color"  
             android:valueTo="float | int | color"  
             android:startOffset="int"  
             android:repeatCount="int"  
             android:repeatMode=["repeat" | "reverse"]  
             android:valueType=["intType" | "floatType"]/>  
       
         <set]]>  
             ...  
         </set>  
     </set> 
     ```

3. 属性动画和普通动画有什么区别？

4. 属性动画更新时会回调onDraw吗？

5. 补间动画与属性动画的区别，哪个效率更高？

6. 动画连续调用的原理是什么？

7. Android 动画框架及实现原理？

## 4. Android Famework

1. AMS交互调用生命周期是顺序的吗？

2. 广播与RxBus的区别，全局广播与局部广播区别？

3. 说说你对屏幕刷新机制的了解，双重缓冲，三重缓冲，黄油模型？

4. Launcher启动图标，有几个进程？

5. 了解APK打包的过程吗？

7. 后台杀死App怎么恢复数据？

8. IntentFilter 匹配规则？`action` 与 `category` 的区别？

9. WindowManagerService 中 token 到底是什么？

10. android源码 有哪些设计模式

13. Lifecycle原理

14. Android 各个版本 API 的区别？

    - KitKat（Android 4.4/API 19）
      - 沉浸式全屏模式
      - 透明系统状态栏
    - Lollipop（Android 5.0/API 21）

      - Meaterial Design
      - 安装时控制权限
      - RecyclerView/CardView/DrawerLayout/SwipeRefreshLayout/ToolBar
      - 可自定义状态栏、标题栏、导航栏颜色、控件阴影
    - Marshmallow（Android 6.0/API 23）

      - 动态权限
    - Nougat（Android 7.0/API 24）
      - 低耗电模式
      - 后台优化
    - Oreo（Android 8.0/API 26）
      - 自适应图标
      - Notification Channels
    - Pie（Android9.0/API 28）
      - 对全面屏（刘海屏）的支持
      - 神经网络
      - Material Design 2.0

15. 低版本 SDK 如何使用高版本 API？

    - @TargetApi：忽略特定版本的API调用报错。

      ```java
      @TargetApi(Build.VERSION_CODES.GINGERBREAD)
      public void fun() {
          if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.GINGERBREAD) {
              // 此时我们正常使用API 9的方法，如果这里误使用了Api 11中的方法，编译时就会报错
              // 提醒我们只是引入API 9中的方法
          } else {
              // TODO 使用老的方式
          }
      }
      ```

    - @SuppressLint：使用`@SuppressLint("NewApi")`的方式让Lint在编译时忽略所调用API对版本的要求。

      ```java
      @SuppressLint("NewApi")
      public void fun() {
          if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.GINGERBREAD) {
              // 此时我们正常使用API 9的方法，如果这里误使用了Api 11中的方法，编译时就会报错
              // 那么运行在低版本中将会引发Crash的风险
          } else {
              // TODO 使用老的方式
          }
      }
      ```

    - 总结：使用上面两种注释的方式只是让`Lint`在编译时不再报错，在低版本的手机系统中，如果直接使用高版本的API肯定会报：`NoSuchMethod`的 Crash 的。所以正确的做法应该是在注解的方法中，做版本判断，在低版本中依然使用老的方式处理。版本判断时我们需要判断具体的版本号，`@TargetApi`的方式比较安全。

16. 谈谈你对安卓签名的理解?

17. App 是如何沙箱化，为什么要这么做？

18. 说说你看过的Android源码？

## 5. 进程/线程相关

1. 怎么获取当前线程是否为主线程？

2. 多进程怎么实现？一个APP 可以多进程吗？如果启动一个多进程APP，会有几个进程运行？
   
3. 进程优先级？

   前台进程

   可见进程

   服务进

   后台进程

   空进程

4. 共享内存用过吗？

5. Android 线程有没有上限？
   没有，android系统对应用程序资源的限制仅仅是以进程为单位，资源都限制在这个进程里，开多少线程用的都是进程里的资源。

6. Android 为每个应用程序分配的内存大小是多少？

7. 一个应用程序安装到手机上的过程发生了什么？





## 8. Performance

1. 性能优化你做过哪些？有用过什么工具？有没有精确测量的工具？

   - **应用卡顿常见原因**

     1. 人为再UI线程中做轻微耗时操作，导致UI线程卡顿
     2. 布局Layout过于复杂，无法在16ms内完成渲染
     3. 同一时间动画执行的次数过多，导致CPU或GPU负载过重
     4. View过度绘制，导致某些像素在同一帧时间内被绘制多次，从而使CPU或GPU负责过重
     5. View频繁的触发`measure` `layout` 导致整个View频繁的重新渲染
     6. 内存频繁GC过多（同一帧中频繁创建内存），导致暂时阻塞渲染操作
     7. 冗余资源及逻辑等导致加载和执行缓慢
     8. ANR

   - **应用UI卡顿分析解决方法**

     1. 使用GPU过度绘制分析UI性能
        设置->开发者选项->调试GPU过度绘制

        |  颜色  |        含义         |
        | :----: | :-----------------: |
        |  无色  | WebView等的渲染区域 |
        |  蓝色  |     1x过度绘制      |
        |  绿色  |     2x过度绘制      |
        | 淡红色 |     3x过度绘制      |
        |  红色  |    4x(+)过度绘制    |

     2. 使用HierarchyViewer工具分析UI性能

     3. 使用GPU呈现模式图及FPS考核UI性能
        设置->开发者选项->GPU呈现模式

     4. 使用Lint进行资源及冗余UI布局等优化

     5. 使用Memory监测及GC打印与Allocation Tracker进行UI卡顿分析

     6. 使用Traceview和dmtracedump进行分析优化

     7. 使用Systrace进行分析优化

   - **应用UI性能分析解决总结**

     1. 布局优化
     2. 列表及Adapter优化
     3. 背景和图片等内存分配优化
     4. 自定义View等绘图与布局优化
     5. 避免ANR，不要再UI线程中做耗时操作，遵守ANR规避守则，譬如多次数据库操作等

   - **应用内存性能分析优化**

     - 应用级内存管理原理
       App运行在自己的虚拟机中的内存管理基本就是遵循Java的内存管理机制，系统在特定的情况下主动进行垃圾回收。要注意的一点是在Android系统中执行垃圾回收（GC）操作时，所有线程（包含UI线程）都必须暂停，等垃圾回收操作完成之后其他线程才能继续运行。这些GC垃圾回收分别有：

       | 类型              | 含义                                             |
       | ----------------- | ------------------------------------------------ |
       | GC_MALLOC         | 内存分配失败时触发                               |
       | GC_CONCURRENT     | 当分配的对象大小超过一个限定值（不同系统）时触发 |
       | GC_EXPLICIT       | 对垃圾收集的显式调用(System.gc())                |
       | GC_EXTERNAL_ALLOC | 外部内存分配失败时触发                           |

     - 内存泄漏概念
       在Java中有些对象的生命周期是有限的，当它们完成了特定的逻辑后将会被垃圾回收；但是，如果在对象的生命周期本来该被垃圾回收时这个对象还被别的对象所持有引用，那就会导致内存泄漏；这样的后果就是随着我们的应用被长时间使用，他所占用的内存越来越大。

       内存泄露可以引发很多的问题，常见的内存泄露导致问题如下：

       - 应用卡顿，响应速度慢（内存占用高时JVM虚拟机会频繁触发GC）
       - 应用被从后台进程干为空进程（超过了阈值）
       - 应用莫名的崩溃（超过了阈值OOM）

       <u>造成内存泄漏的最核心的原理是一个对象持有了超过自己生命周期意外的对象强引用导致该对象无法被正常垃圾回收</u>。

     - 应用内存泄漏察觉手段

       | 察觉方式                  | 场景                                                         |
       | ------------------------- | ------------------------------------------------------------ |
       | AndroidStudio的Memory窗口 | 平时用来直观了解自己应用的全局内存情况，大的泄露才能有感知。 |
       | DDMS-Heap内存监测工具     | 同上，大的泄露才能有感知。                                   |
       | dumpsys meminfo命令       | 常用方式，可以很直观的察觉一些泄露，但不全面且常规足够用。   |
       | leakcanary神器            | 比较强大，可以感知泄露且定位泄露；实质是MAT原理，只是更加自动化了，当现有代码量已经庞大成型，且无法很快察觉掌控全局代码时极力推荐；或者是偶现泄露的情况下极力推荐。 |

     - 应用开发规避内存泄漏建议

       - 不要对`Activity Context`保持长生命周期的引用，尽量在一切可以使用`Application Context`代替。
       - 非静态内部类的静态实例容易造成内存泄漏；即一个类中如果你不能够控制它其中内部类的生命周期（譬如Activity中的一些特殊`Handler`等），则尽量使用静态类和弱引用来处理（譬如`ViewRoot`的实现）。
       - 警惕新城为终止造成的内存泄漏；譬如在Activity中关联了一个生命周期超过Activity的Thread，在退出Activity时切记结束线程。一个典型的例子就是`HandlerThread`的`run()`方法是一个死循环，它不会自己结束，线程的生命周期超过了Activity生命周期，我们必须手动在Activity的销毁方法中中调运`thread.getLooper().quit()`。
       - 对象的注册与反注册没有成对出现造成的内存泄露；譬如注册广播接收器、注册观察者（典型的譬如数据库的监听）等。
       - 创建与关闭没有成对出现造成的泄露；譬如Cursor资源必须手动关闭，WebView必须手动销毁，流等对象必须手动关闭等。
       - 不要在执行频率很高的方法或者循环中创建对象，可以使用`HashTable`等创建一组对象容器从容器中取那些对象，而不用每次`new`与`release`。
       - 避免代码设计模式的错误造成内存泄露。

2. App 内存优化？

3. App 绘制优化？

4. App 瘦身？

5. 网络优化？

6. App 电量优化？

7. Foce Close 什么时候会出现？如何避免？

   - Error
   - OOM（OutOfMemoryError），内存溢出
   - StackOverFlowError
   - RuntimeException，比如空指针异常

   避免：

   - 注意内存的使用和管理
   - 使用`Thread.UncaughtExceptionHandler`接口

8. 你碰到过什么内存泄漏，怎么处理？

   - 新建线程引起的Activity内存泄漏

     ```java
     /**
      * 错误示范
      * 当Activity执行finish()后，发生了内存泄漏
      * 因为匿名内部类会持有外部类的引用，当外部类销毁，内部类的线程还在执行，
      * 导致外部类所占用的内存不能被GC回收，将匿名内部类改成静态非匿名内部类即可。
      */
     new Thread(new Runnable() {
         @Override
         public void run() {
             try {
                 // 模拟耗时操作
                 Thrad.sleep(15000);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
         }
     }).start();
     
     
     
     // 解决方法
     private static class MyRunnable implements Runnable {
         @Override
         public void run() {
             try {
                 // 模拟耗时操作
                 Thrad.sleep(15000);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
         }
     }
     
     new Thread(new MyRunnable());
     ```

   - Activity添加监听器造成的内存泄漏

     ```java
     /**
      * 错误示范
      * Manager的静态实例持有Activity的实例，当Activity销毁后导致所占用的内存不能被GC回收，从而造成内存泄漏
      */
     Manager.getInstance().addListener(this);
     
     //解决方法
     @Override
     protected void onDestory() {
         super.onDestory();
         //当Activity销毁时，取消Manager的监听绑定
         Manager.getInstance().removeListener(this);
     }
     ```

   - Handler一名内部类造成的内存溢出

     ```java
     //错误示范
     private Handler handler = new Handler() {
         @Override
         public void handleMessage(Message msg) {
             super.handleMessage(msg);
             //TODO
         }
     }
     
     new Thread(new Runnable() {
         @Override
         public void run() {
             handler.sendEmptyMessage(MSG_CODE);
             try {
                 // 模拟耗时操作
                 Thrad.sleep(8000);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             handler.sendEmptyMessage(MSG_CODE);
         }
     }).start();
     
     //解决方法
     private static class MyHandler extends Handler {
         WeakReference<MyActivity> weakReference;
         
         public MyHandler(MyActivity activity) {
             weakReference = new WeakReference<MyActivity>(activity);
         }
         
         @Override
         public void handleMessage(Message msg) {
             super.handleMessage(msg);
             if (weakReference.get() != null) {
                 //TODO
             }
         }
     }
     
     private static class MyRunnable implements Runnable {
         @Override
         public void run() {
             handler.sendEmptyMessage(MSG_CODE);
             try {
                 // 模拟耗时操作
                 Thrad.sleep(15000);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
             handler.sendEmptyMessage(MSG_CODE);
         }
     }
     
     handler = new MyHandler(this);
     new Thread(new MyRunnable());
     
     @Override
     protected void onDestory() {
         super.onDestory();
         //当Activity销毁时，将所有的Callbacks和Messages全部清除
         handler.remoevCallbacksAndMessage(null);
     }
     ```

   - AsyncTask造成的内存泄漏
     通过自定义静态AsyncTask类可解决

   - 单例模式引发的内存泄漏
     尽量使用`MyApplication.getAppInstance().getApplicationContext()`实例初始化，若用Activity的实例，记得在Activity销毁的时候释放引用。

   - 资源对象没有关闭引发的内存泄漏
     比如：Cursor,File,Bitmap,Video等，系统都运用了缓存技术，如不再使用这些资源时，不及时回收可以引发内存泄漏，所以记得调用类似的方法：`close()` 、`recycler()` 、`release()`。

   - 集合对象没有及时清理引发的内存泄漏
     如果集合是`static`的，不断往里添加东西，而又不及时清理时，可能会引起内存泄漏。

9. 你碰到过什么内存溢出，怎么处理？

   - 情景：
   - 避免：
     - 动态回收内存
     - 为应用分配更多的内存
     - 自定义内存大小
     - 内部类使用`static`

10. OOM 是否可以 try catch？
    
    
    


## 项目实现

1. 比如项目中药读取本地一张大图要注意什么？读取IO时长、CUP占用大小，如果经常性的读取会浪费时间，并且多次读取会占用更多的内存，所以要做缓存，缓存解决了多次IO时长问题，并且内存只有一份图片文件，但是一张大图的缓存对于一些低配的机器来说内存就比较吃紧，那么内存不够用的情况应该释放掉，那么这个时候缓存要用强引用、弱引用还是软引用？
2. 做一个日志组件，要用到那些技术？比如设计模式中的生产消费者模式、线程安全、线程池，这个线程池如何设计？如果用缓存线程池会带来什么问题？
   - 线程安全
     线程安全是指多线程访问统一代码，不会产生不确定的结果。
3. 对 volatile 关键字的理解，我们都知道可以保证可见性，可以用来修饰一些状态变量，比如 boolean 之类的，只有两种状态，并且状态切换跟前面状态没有关系，那么 DCL 为什么要加这个关键字？它可不是为了切换状态。
4. kotlin 中的 data class 在定义参数时最好给一个默认值，是因为给了默认值 data class 会默认生成无参构造函数，如果你的项目里用的是 Gson 来反序列化的话，可以减少这些类反序列化时调用反射的次数，这是因为 Gson 在实现反序列化的时候，其实采用了一种尝试机制，通过不断的反射尝试生成一个实例，如果有无参构造函数的话，在第一步就能生成一个实例，减少后续反射的次数。
5. Activity 窗口层级了解吗？PhoneWindow、DecorView、ContentView？组件化中怎么在各个模块的页面设计一个阅读时间奖励，到到时间弹出一个奖励的 View，到时间就消失？如果你看过Activity窗口的层级结构，就不难想到用 ContentView 加载这个布局，对于这个工具类的方法，只需要传一个Activity就可以了。
6. 对于应用更新这块是如何做的？(解答：灰度，强制更新，分区域更新)？
7. 实现一个 Json 解析器（可以通过正则提高速度）？
7. 一张 100x100 的图片在内存中的大小？



### 人事相关

- 请简单做个自我介绍？
- 为什么离开上家公司？您在前一家公司的离职原因是什么？
- 讲一个你认为做的最好的项目/案例
- 你上家公司的薪水/期望的薪金？
- 你对薪资的要求？
- 对待加班看法？
- 自己最擅长的技术点，最感兴趣的技术领域和技术点，做了那些东西？
- 自己的优点和缺点是什么？并举例说明？
- 你朋友对你的评价？
- 说下项目中遇到的棘手问题，包括技术，交际和沟通？
- 项目中遇到最大的困难是什么？如何解决的？
- 在五年的时间内，你的职业规划？
- 给你一个项目，你怎么看待他的市场和技术的关系？
- 你的学习方法是什么样的？实习过程中如何学习？实习项目中遇到的最大困难是什么以及如何解决的？
- 工作上与领导意见不同时，怎么办？
- 若上司在公开会议上误会你了，该如何解决？
- 你和别人发生过争执吗？你是怎样解决的？
- 你为什么愿意到我们公司来工作？
- 你最擅长的技术点，最感兴趣的技术领域和技术点？
- 说说你对行业、技术发展趋势的看法？
- 就你申请的这个职位，你认为你还欠缺什么？
- 如果你在这次面试中没有被录用，你怎么打算？
- 你还要什么了解和要问的吗？
- 评价下自己，评价下自己的技术水平，个人代码量如何？
- 介绍你做过的哪些项目，介绍一个你影响最深的项目？
- 都使用过哪些自定义控件？
- 项目中用了哪些开源库，如何避免因为引入开源库而导致的安全性和稳定性问题？
- 有没有什么开源项目？
- 研究比较深入的领域有哪些？
- 业余都有哪些爱好？
- 为什么考虑换工作？
- 如果让你来开发B站的一个页面，哪一个页面可以很快入手？
- 如果产品要求你开发一个音频播放功能，你会怎么着手？预计会有什么坑？
- 介绍一下你自已和项目？
- 说说为什么考虑离职？
- 说说对你们原来公司的印象？
- 在你们公司这几年感觉怎么样？
- 在这几年里，你有做过什么觉得最有价值的工作？
- 这些年有做一些什么比较难的工作？
- 你还有什么要问我的吗？目前有几个offer，倾向性是怎样的？
- 平常抓包用什么工具？
- 平常是怎么了解一些新知识与业界动态的，最近有什么印象深刻的文章？
- 如果android端和IOS端调一个接口，一个通了一个没通，你会如何解决？
- 如果android端和IOS端调一个接口，一个比较慢，一个比较快，有什么思路？
- 登陆功能，登陆成功然后跳转到一个新Activity，中间涉及什么？从事件传递，网络请求,AMS交互角度分析？
- 伪代码实现一个长按事件?
- 实现一个下载功能的接口？
- 你在团队中是怎样一个角色？
- 你的梦想是什么？



## 关键词

已竣工

已交付

担任

































