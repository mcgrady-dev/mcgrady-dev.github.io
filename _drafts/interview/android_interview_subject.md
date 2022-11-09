## App Components

### Activity

1. **Activity启动流程？**

   1. Activity启动请求都会通过Binder调用AMS来处理；
   2. AMS首先判断目标App进程是否存在，如果目标App进程不存在，将通过Socket向Zygote进程发起创建App进程请求；
   3. Zygote进程接收到来自AMS的请求后，将fork新的App进程，并进入App进程执行`ActivityThread.main()`完成初始化工作；
   4. ActivityThread通过ActivityManagerProxy代理类（Binder）向AMS发起attachApplication请求；
   5. 这时目标App进程已就绪，AMS通过Binder将Activity启动请求发给ApplicationThreadProxy代理类，再次进入目标App进程处理；
   6. ApplicationThreadProxy代理类将Activity启动请求交给ActivityThread的Handler处理；
   7. 最后Handler接收到启动Activity消息后，通过反射机制创建目标Activity，并回调Activity的生命周期方法；

   <img src="https://s2.loli.net/2022/11/01/ZlAGYonuNRaOQIg.jpg" alt="start_activity_process" style="zoom:75%;" />

2. **Activity启动模式？**

   - standard：默认模式，总是在任务栈中启动新的Activity实例；
   - singleTop：如果当前任务栈的顶部已存在Activity的实例，则调用`onNewInstance()`方法，否则启动新的Activity实例。该模式允许一个任务栈中有多个Activity实例，但前提是栈顶的Activity不是该Activity现有的实例；
   - singleTask：首先会创建新的任务栈，并实例化根Activity的实例，但是如果另外的任务栈中已存在该Activity的实例，则会调用`onNewInstance()`方法。该模式只允许一个Activity实例存在；
   - singleInstance：与singleTask类似，唯一不同的是Activity始终是其任务栈的唯一成员，由该Activity启动的任何Activity都会在其他的任务栈中打开。该模式启动的Activity不允许任何其他Activity成为其任务栈中的一部分；
   - singleInstancePerTask：该模式在Adnroid 12中加入，与singleTask类似，只不过singleInstancePerTask不需要为启动的Activity设置一个特殊的taskAffinty才能创建一个新的Task，默认情况下会为启动的Activity新建任务栈，同时配合`Intent.FLAG_ACTIVITY_MULTIPLE_TASK`或`Intent.FLAG_ACTIVITY_NEW_DOCUMENT`，可以同时在多个任务栈中存在单个Activity实例；

3. **如何保存Activity的状态？**

   - `onSaveInstanceState()`中保存，`onRestoreInstanceState()`中恢复；
   - ViewModel；

4. **请描述横竖屏切换时，Activity的生命周期变化？**

   - 默认情况，切屏会重新调用各个生命周期方法：
     ```mermaid
     flowchart 
         subgraph 切到横屏的状态
             direction LR
             Create("onCreate()") --> Start("onStart()") --> Restore("onRetoreInstanceState()") --> Resume("onResume()")
         end
         subgraph 竖屏的状态
             direction LR
             Pause("onPause()") --> Save("onSaveInstanceState()") --> Stop("onStop()") --> Destory("onDestory()")
         end
     ```

   - 设置`android:screenOrientation`指定方向，切屏不会重新创建Activity；

   - 设置`android:configChange="orientation"`，切屏不会重新创建Activity，会调用`Activity.onConfigChange()`方法；

5. **弹出Dialog的时候按Home键时Activity的生命周期？**
   与正常按下Home键Activity退到后台生命周期一样，Dialog的弹出Activity并没进入后台，所以不要被Dialog所迷惑。

6. **说说App的启动过程，在`ActivityThread.main()`方法里面做了什么事，什么时候启动第一个Activity？**

   - `ActivityThread.main()`中主要做了几件事：
     1. 创建主线程Looper；
     2. 初始化ActivityThread；
     3. 初始化Handler；
     4. 开启消息循环；
   - 在SystemServer进程启动动系统服务方法中的`AMS.systemReady()`的方法启动Launcher应用；

7. **生命周期都是通过什么调用的？**
   在ActivityThread中通过反射实例化Activity，通过Handler接收Activity生命周期事件来执行方法回调。

8. **Activity之间的通信方式？如果需要在Activity间传递大量的数据怎么办？**

   Intent，大数据通过SQLite或File来实现。



### Fragment

1. **什么是Fragment？**
   Fragment是Activity中可重用的一部分，Fragment可以定义和管理自己的布局，具有自己的生命周期。

2. **Fragment的生命周期？**
   
   ![fragment-add-lifecycle](https://s2.loli.net/2022/11/02/68aLnMPHWxDQtNw.png)
   
3. **Fragment调用`hide/show/replace`方法时生命周期的变化？**

   - `show/hide`仅回调`onHiddenChanged()`方法；

   - `replace`替换其他Fragment时生命周期如下：
     
     ![fragment-replace-lifecycle](https://s2.loli.net/2022/11/02/xLmgItuVs8qeNWh.png)

4. **Fragment的`add`与`replace`的区别？分别对Fragment生命周期的影响？**
   add添加，replace替换，add不会销毁当前Fragment实例，replace会销毁当前Fragment实例。

5. **Activity和Fragment之间怎么通信， Fragment和Fragment又怎么通信？**
   Interface、ViewModel、Fragment暴露公开方法、setArguments、BroadcastReceiver、EventBus。

6. **Fragment回退栈？**
   Fragment回退栈是类似于Activity任务站的一种栈式的事务管理机制，由FragmentManager管理，默认情况下，Fragment事务不会加入回退栈，可以通过`addToBackStack()`方法加入。如果没加入回退栈，则返回会将Activity出栈，如果加入了，则回滚Fragment事务。



### Context

1. **什么是Context？一个应用程序中有多少Context？**
   - Context是应用环境当前状态的上下文，用于访问特定的资源和类，以及一些应用程序级别操作的调用；
   - 一个ApplicationContext + Service个数 + Activity个数；
2. **Application 、 Activity 、 Service的Context之间的区别？**
   - 它们都是ContextWrapper的子类，而ContextWrapper的成员变量`mBase`可以用来存放系统实现的ContextImpl，所以继承ContextWrapper的子类在调用用Context方法时，都是通过静态代理的方式最终调用到ContextImpl的方法；
   - 在功能上，只有Activity显示显示界面，因为Activity继承与ContextThemeWrapper，提供了一些主题和界面显示相关的能力，同时简介的继承了ContextWrapper；而Application、Service直接继承ContextWrapper，则不具备UI相关的处理能力；
3. **ContextImpl实例是什么时候生成的？在Activity的`onCreate()`里能拿到这个实例吗？**
   可以，`getApplicationContext()`最终调用的就是ContextImpl，ContextImpl在`ActivityThread.perfromLauncherActivity()`中创建，创建完成后将Activity和Application以及ContextImpl进行关联，最后才调用Activity的`onCreate()`生命周期回调。



### Window

1. **Activity、Window、View 三者的关系？**
   
   Activity是控制单元，专注于与用户交互，并且为用户创建一个Window，Window用于承载View的显示和用户输入事件的传递，View用于在Window上的一块区域绘制显示的内容，并响应用户输入事件；
   
2. **Window是什么？在Android中哪些地方用到？**

   - 每个Activity启动时都会创建一个Window，用于承载View的显示和用户输入事件的传递；
   - 需要在屏幕上显示内容的地方就要用到Window，例如：Activity、PopupWindow、Toast、Dialog等；

3. **Window的分层和类别？**
   Window定义了不同的窗口类型，用于确定Window的显示次序。

4. **Window的内部机制：添加、删除、更新？**

   - addView()：由WindowManager转交到WindowManagerService处理，WMS对窗口申请进行权限和令牌校验，然后为Window创建对应的WindowState和DisplayContent以完成渲染前的准备工作，最终WMS为添加的Window分配Surface，Surface将渲染的可见的图像数据交由SurfaceFlinger合并到屏幕；
   - updateViewLayout()：由ViewRootImpl通过Choreographer接收VSYNC信号回调下对View进行测量、布局、绘制，除了View本身的绘制外，还通过WindowSession调用`WMS.relayoutWindow()`更新Window；
   - removeView()：由ViewRootImpl将View添加到待删除列表中，并分别从ViewRootImpl列表、布局参数列表和View列表中移除View，然后判断是否可以直接执行删除操作，通过同步或异步调用执行View中的`dispatchDetachDetachedFromWindow()`方法，将View从Window中移除，同时处理一些清理工作，最后通过WindowManagerGlobal刷新数据；

5. **Window中的Token是什么？**
   作为向WMS申请Window时需出示的令牌，同时将同一类组件的WindowState集合进行管理。

6. **Activity、Dialog、Toast的Window创建过程？**

   - Activity、Dialog的Window创建过程类似，都是由WindowManager向WMS申请一个窗口，WMS为添加的Window分配Surface，Surface将渲染可见的图像数据交由SurfaceFlinger合并到屏幕；
   - Toast的与区别在于它具有定时取消的功能，Window创建过程是通过ITransientNotification调用NotificationManagerService来执行，与Activity、Dialog类似；



### Intent

1. **显式Intent和隐式Intent的区别？**
   - 显式Intent明确指出了目标组件名称，多用于在应用内部传递消息；
   - 隐式Intent没有明确指出目标组件名称，可用于不同应用间传递消息；
2. **Intent传递数据时，哪些数据类型可以被传递？**
   Serializable、Parcelable、Charsequence、基本数据类型。
3. IntentFilter匹配规则？`action` 与 `category` 的区别？



### Service

1. Service有几种启动方式？分别有什么区别？
2. 描述Service的生命周期？
3. Service是否在主线程中执行？Service里可以执行耗时操作吗？
4. Service里可以弹toast吗？
   可以通过`getApplicationContext()`获取上下文显示toast。

5. Activity怎么启动Service，Activity与Service交互？

6. 用什么方式使Service和Activity之间传递长久保存的对象？
7. IntentService与Service有什么不同，区别在哪？
8. IntentService的使用场景与特点？



### Broadcast

1. **注册广播有几种方式，这些方式有何区别？**
   - 静态注册：常驻型广播，在App启动是初始化，App关闭后，如果有广播消息，也可以被系统调用自动运行；
   - 动态注册：在代码中动态注册，跟随Activity生命周期，优先级比静态广播高；
2. **广播的分类？**
   - 系统广播（SystemBroadcast）：Android内置的系统广播，如：开机、网络状态变化等；
   - 普通广播（NormalBroadcast）：自定义intent的广播，支持跨进程通信；
   - 有序广播（OrderedBroadcast）：普通广播是并行无序执行的，而有序广播按照优先级串行执行；
   - 本地广播（LocalBroadcast）：限制在发送者与接受者位于同一应用中广播，无需跨应用时效率更高；
   - 粘性广播（StickyBroadcast）：粘性广播在发送后存储在消息容器里，如果暂时还没处理这个消息，则一直处于等待状态，一旦有接收器注册就可以自动接收到消息；
3. **广播的工作原理？**
   `registerReceiver()`方法通过AMS注册广播，将广播添加到队列中，`sendBroadcast()`通过AMS发送广播，创建接受者对象，并添加到相应的队列中，然后调用BroadcastQueue的`scheduleBroadcastLocked()`方法来完成不同广播的处理；接着广播的几首主要是对并行和串行广播队列的处理，最后都是调用`performReceiveLocked()`进行处理，如果是跨应用的广播，通过Binder讲消息传递给接收广播的进程进行处理，如果该进程还没启动则先启动进程，最后回调BroadcastReceiver的`onReceive()`方法。
4. **可以在 `onReceive()`中开启线程吗？会有什么问题？**
   可以，首先广播是运行在UI线程上的，阻塞15s会报ANR，`onReceive()`方法返回后，进程和线程处于等待状态，系统任意时刻可以终止和回收该线程和进程占有资源，面对耗时操作，如果在`onReceive()`中开启线程处理耗时任务，有可能任务没处理完系统就回收进程了，所以面对长时间的耗时任务建议在`onReceive()`中开启Service来处理。



### ContentProvider

1. ContentProvider 是如何实现数据共享的？
2. ContentProvider 的权限管理？
3. ContentProvider 如何自定义？使用场景是什么？
4. ContentProvider、ContentResolver、ConentObserver 之间的关系



## UI Components

### Graphics

1. **布局层级/主线程耗时较多是如何造成丢帧（Jank）的？**
   延迟显示，因为当前帧的时机无法处理，只能等下一个VSYNC信号的时机。
2. **16.6ms是什么意思？是每16.6ms走一次`measure/layout/draw`吗?**
   - 假如屏幕固定的刷新率是每秒60Hz，即一秒内刷新60帧，相当于每隔16.6刷新一帧；
   - 在View的绘制中最后会经历`scheduleTraversals()`，这里通过Choreographer控制VSYNC信号以触发缓存交换和刷新，所以并非没16.6ms就执行一次`measure/layout/draw`；
3. **屏幕刷新机制中的双缓存、三缓存是什么意思？**
   - 双缓存：是让Display和GPU拥有各自的BufferQueue（Back Buffer和Frame Buffer），避免共用一个BufferQueue下，屏幕刷新速率和帧速率不同步时出现的上一帧未刷新完，BufferQueue中的数据就被下一帧重写覆盖的画面撕裂情况；
   - 三缓存：在双缓存的基础之上增加一个Graphic Buffer缓冲区，有效利用等待VSYNC的时间，CPU/GPU不必再等待下一个VSYNC才处理，而是当双缓存不够用时，通过新增的Graphic Buffer来处理下一帧的绘制工作，减少Jank出现的频率；
4. **VSYNC信号是如何产生的？**
   对于支持HardwareComposer的设备而言，VSYNC信号由HWC设备产生并回调到`HWCompoer.hook_vsync()`方法中。
5. **可能你知道VSYNC，这个具体指啥？在屏幕刷新中如何工作的？**
   当扫描完一个屏幕后，需要重新回到第一行以进行下一次循环，此时会出现VBI，VSYNC就是在这个期间由Display发出，用于保证双缓冲在最佳时间点进行交换。
6. **VSYNC在哪个进程中捕获的？**
   HWC设备是在SurfaceFlinger进程中打开并维护的，在`HWCompoer.hook_vsync()`中接收，然后传递到`SurfaceFlinger.onVSyncReceived()`函数中，并封装成DispSync信号源。
7. **如果界面没动静止了，还会刷新吗？**
   屏幕会固定每16.6ms刷新一次，即从Frame Buffer中获取上一帧的内容进行显示，但CPU/GPU不执行渲染工作。
8. **Choreographer是什么？**
   Choreographer是用于协调动画、输入和绘制的时机，也就是CPU/GPU的绘制是在VSYNC信号到来时开始。
9. **什么是Surface？**
   Android中一切渲染的内容都有Surface来提供，Surface为生产方（WindowManager）和消耗放（SurfaceFlinger）提供交换缓冲区（BufferQueue），SurfaceFlinger将多块Surface按照Z-Order次序进行混合并输出到FrameBuffer。
10. **对SurfaceView的了解？**
    当使用外部缓冲区来源（例如，Camera API或OpenGL ES）进行渲染时，需要从缓冲区来源复制缓冲区以便在屏幕上显示这些缓冲区，为此使用SurfaceView进行渲染时，SurfaceFlinger会直接将缓冲区合成到屏幕上，从而省去了一些额外的工作。



### View

1. 自定义 View 与 ViewGroup 的区别？
2. 自定义实现一个九宫格如何实现？
3. `margin` `padding` 的区别？
4. `gone` `invisible` 的区别？
5. `requestLayout` `invalidate` `postInvalidate` 的区别？
6. DecorView、ViewRootImpl、View之间的关系？`ViewGroup.add()`会多添加一个 `ViewRootImpl` 吗？
7. 自定义 LinearLayout，怎么测量子 View 宽高？
8. `requestLayout()`、`onLayout()`、`onDraw()`、`drawChild()`区别于联系？
9. 一个 wrap_content 的 ImageView，加载远程图片，传什么参数裁剪比较好?
10. 如何求当前 Activity View 的深度？
11. Android 支持的单位有？
13. View的绘制过程？
    measure、layout、draw
14. MeasureSpec 的三种模式？
    - MeasureSpec.EXACTLY：确定模式；
    - MeasureSpec.AT_MOST：最多模式；
    - MeasureSpec.UNSPECIFIED；未指定模式；
14. `onResume`中可以测量宽高么？
15. View的绘制流程是从Activity哪个生命周期方法开始执行的？
16. `onCreate`、`onResume`、`onStart`里面，什么地方可以获得宽高？
17. 为什么 view.post 可以获得宽高，有看过 view.post 的源码吗？
19. `getWidth()`和`getMeasureWidth()`的区别？



### Drawable

1. Drawable 与 View 有什么区别，Drawable 有哪些子类？
2. 两个 `getDrawable` 取得的对象，有什么区别？



### 事件分发机制

1. 说说事件分发机制？责任链模式的优缺点？

2. 事件冲突怎么解决？
3. `onTouch`和`onTouchEvent`有什么区别，又该如何使用？
4. `onTouchListener`、`onTouchEvent`、`onClick` 的执行顺序？
5. ViewGroup在Action_Move时onIntercept返回true，事件怎么传递？
6. dispatchTouchEvent,onInterceptEvent,onTouchEvent顺序，关系？
7. setOnTouchListener,onClickeListener和onTouchEvent的关系？
8. 怎么拦截 `onTouchEvent` ？如果返回 false `onClick` 还会执行吗？



### RecyclerView

1. RecyclerView 中 回收是什么？复用是什么？回收到哪里？复用从哪里拿？什么时候回收？什么时候复用？

2. LayoutManager的作用是什么？LayoutManager样式有哪些？`setLayoutManager()` 源码里做了什么？

3. ViewHolder的作用是什么？什么时候停止调用 onCreateViewHolder？

4. 讲一下RecyclerView的缓存机制，滑动10个，再滑回去，会有几个执行onBindView？

5. RecyclerView的缓存结构是怎样的？缓存的是什么？cachedView会执行onBindView吗?

6. RecyclerView的Recycler是如何实现ViewHolder缓存的？如何理解RecyclerView四级缓存是如何实现的？

7. ViewHolder的封装如何对findViewById优化？ViewHolder中为何使用SparseArray代替HashMap存储ViweId？
8. RecyclerView绘制原理过程大概是怎样的？

9. 如何实现RecyclerView的局部更新，用过payload吗？`notifyItemChanged()`方法中的参数？
10. 如何解决RecyclerView嵌套RecyclerView条目自动上滚的BUG，如何解决NestScrollView嵌套ScrollView滑动冲突？

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



### ViewPager

1. ViewPager中嵌套ViewPager怎么处理滑动冲突？
2. 怎么写一个不能滑动的 ViewPager ？
3. Viewpager切换掉帧有什么处理经验？
4. ViewPager2原理？
5. ViewPager切换Fragment什么最耗时？



### Dialog

1. 为什么 Dialog 不能用 Application 的 Context ？



## UI Layouts

### Layout

1. 布局文件中`@`和`?`的区别？
   - `@` 标记是引用一个实际的值（`color`, `string`, `dimension`...）
   - `?` 标记是引用一个`style attribute`，其值取决于当前使用的主题
2. LayoutInflater 的作用？

### CoordinatorLayout

1. CoordinatorLayout自定义behavior，可以拦截什么？



## Animation

1. 介绍一下 Android动画？
2. 动画的分类以及区别？

3. 属性动画和普通动画有什么区别？
4. 属性动画更新时会回调onDraw吗？
5. 补间动画与属性动画的区别，哪个效率更高？
6. 动画连续调用的原理是什么？
7. Android 动画框架及实现原理？



## Thread Handing

### Thread

1. 怎么获取当前线程是否为主线程？
2. 多进程怎么实现？一个APP 可以多进程吗？如果启动一个多进程APP，会有几个进程运行？
3. 进程优先级？

4. 共享内存用过吗？
5. Android 线程有没有上限？
6. Android 为每个应用程序分配的内存大小是多少？



### Handler

1. **Handler的机制原理？**
   Handler是一个消息分发对象，而消息分发依赖于Looper消息循环，Looper将MessageQueue中的消息取出分配到Handler进一步分发处理。

2. **Handler是怎么切换线程的？**
   消息入队后，`loop()`方法从MessageQueue取出待处理的Message，根据Message中的`target`分配到指定线程的Handler分发处理。

3. **Handler休眠是怎样的？epoll的原理是什么？如何实现延时消息，如果移除一个延时消息会解除休眠吗？**

   - 没有消息时会调用`epoll_wait()`方法使得线程被挂起休眠，且不耗费CPU；有消息时，通过pipe管道写端写入数据来唤醒主线程工作；
   - epoll机制是一种I/O多路复用机制，可以同时监听多个描述符，当某个描述符就绪（读或写就绪），则立即通知相应程序进行读或写操作；
   - 消息入队后，通过循环判断延迟时间是否抵达，抵达则唤醒线程执行；
   - 在延迟消息抵达触发时间前不会唤醒线程，故移除消息不回接触休眠；

4. **Handler如何保证MessageQueue并发访问安全？**
   对`enqueueMessage()`和`next()`方法加上同步锁。

5. **主线程为什么不用初始化Looper？**
   ActivityThread中会提前初始化主线程的Lopper。

6. **Looper死循环为什么不会导致应用卡死，会耗费大量资源吗？**
   Looper是维持应用运行的基本，在没有任务需要执行时Looper会进入休眠阻塞状态，核心是通过epoll机制的`epoll_wait()`方法阻塞在pipe管道的读端，使得县城被挂起休眠，且不消耗CPU，当有消息时，通知相应的程序通过写端写入数据来唤醒线程。

7. **Looper退出后是否可以重新运行？**
   可以，但通常主线程的Looper在退出应用时执行`quit()`方法，工作线程执行完毕或销毁时执行`quit()`方法，quit()方法会调用MessageQueue的`quit()`方法来清理队列中的消息。当Looper中没有消息时则就进入了阻塞状态。

8. 能不能让一个Message加急被处理？

9. **什么是同步屏障？**
   同步屏障可以理解为拦截同步消息的执行，来确保某些优先级更高的异步消息能尽快被执行。

10. **IdleHandler用过吗？IdleHandler应用场景？**

    IdleHandler提供一种在Looper线程处于空闲状态时执行一些优先级不高的操作，例如ActivityThread就向主线程MessageQueue中添加了一个GcIdler，用于在主线程空闲时尝试去执行GC操作。

11. **`View.post`和`Handler.post`的区别？**
    `View.post()`的调用时机跟整个View的绘制和渲染有关系，`View.post()`的执行是在View创建之后。

12. **`sendMessage`与`postDelay`的区别？**
    `sendMessage()`发送普通消息，通过`Handler.handleMessage()`接收事件，`postDelay()`发送延迟消息，通过Runnable参数接收事件，但两者都是调用`sendMessageAtTime()`来完成。

13. **为什么不建议在子线程中访问UI？**
    因为Android中设计的UI刷新操作是非线程安全的，若在子线程中更新UI，有可能主线程已经将UI销毁，将会带来不安全的异常。

14. **为什么系统不对UI控件的访问加上锁机制？**
    由于UI的更新操作对刷新速度有一定的要求，加上锁机制会降低UI的刷新操作的速度。

15. **一个线程有几个Looper？为什么？**
    每个线程有且只有一个Looper，Looper是Handler中消息接收和转发的核心。

16. **多个Handler可以往同一个MessageQueue里添加消息吗？如果可以，多个Handler都会处理对应的消息么？为什么？**

    - 多个Handler绑定到同一个Looper时，都是往同一个MessageQueue中发送消息；
    - Handler只处理与自己相关的消息，因为Looper循环中时通过Mesage的`target`回调到Handler来处理消息的；

17. **如果主线程处于休眠状态，点击屏幕时是怎么被唤醒的？点击事件是通过什么机制传递过来的？**
    首先输入事件的传递是依赖socket通信的，在native层的Looper中的`addFd()`方法通过epoll_ctl注册socket监听，当有输入事件需要传递时，只需要往socket里写入数据，就可以唤醒主线程。



### IPC

1. **什么是Binder机制？**

   Binder是Android特有的通过轻量级远程过程调用（RPC）机制实现的进程间（IPC）通信技术。通过Binder可以实现不同地址空间的进程进行传输数据或远程方法调用。

2. **Binder的通信过程？**

   1. Server进程向ServiceManager中注册服务；
   2. Client进程向ServiceManager查询Server进程的服务地址；
   3. Client进程得到ServiceManager返回的Server进程服务地址，并且使用该服务；
   4. Server进程向Client进程提供服务返回数据；

3. **Binder线程池的工作过程是怎么样的？**
   每当Zygote进程`fork`新进程的同时，都会为该进程分配一个Binder线程池，并向其中注册一个Binder线程（主线程），之后Server进程也可以向Binder线程池注册新的线程，或者Binder驱动在探测没有空闲Binder线程时会主动向Server进程注册新的Binder线程，对于Server进程有一个最大的Binder线程数（默认16）的限制，对于Client进程的Binder事务请求都是交由Server进程的Binder线程处理。

4. **什么是`ioctl`？**
   ioctl是一个独立的系统调用，对设备的I/O通道进行管理，通过它用户空间可以跟内核设备驱动沟通。

5. Binder进程间通信可以调用原进程方法吗？

6. **Binder怎么验证`PID`？**
   对比Binder事务的`binder_transaction_data`中的`sender_pid`和接收端`binder_proc`中的`pid`是否相同。

7. **Binder驱动了解吗？**
   Binder驱动是一种通过动态扩展的设备驱动模块，定义了一套Binder的通信协议，负责建立进程间的Binder通信，提供数据包在数据之间传递的一些底层支持。

8. **为什么在Android中使用Binder进行跨进程通信？**

   - **从稳定性的角度**：基于C/S架构，Server端与Client端相对独立，结构清晰，Binder由于共享内存；
   - **从安全的角度**：传统IPC无法获取对方进程可靠的UID/PID来辨别身份，而Android为每个进程分配了UID，这是鉴别进程身份的重要标志，通过判断UID/PID是否满足访问权限，以增加安全性；
   - 基于上述原因，Android 需要建立一套新的 IPC 机制来满足系统对稳定性、传输性能和安全性方面的要求，这就是 Binder；

9. **跨进程通信（IPC）了解多少？管道了解吗？**

   - IPC是进程间通信的一些技术或方法；
   - 管道是一种使用消息传递的进程间通信机制，实际上管道是一块进程共享的区域，发送方进程以字符流形式将大量数据送入管道，接收方进程可从管道接收数据；

10. **AIDL的全称是什么？如何工作？能处理哪些类型的数据？**

   - 定义：Android接口定义语言（Android Interface Definition Language）是Android提供的进程间通信IPC工具Binder的具体使用方法。通过AIDL可以定义客户端和服务端之间均认可的编程接口，以便二者使用进程间通信（IPC）进行互相通信。
   - 如何工作：AIDL编译器将AIDL文件生成相关的Binder类，当Client端与Sever端处于同一进程时，将该Binder类对象直接返回，否则将返回封装后的Binder代理对象，Client端通过该Binder对象发起远程调用请求，最终由Server端的线程池中的Binder线程执行的`onTransact()`方法接收并处理，成功完成方法调用后，将返回参数通过replay封装进行返回给Client端；
   - 参数类型：基本类型、String、List、Map、Parcelable以及AIDL本身也可在AIDL文件中使用；

11. **AIDL的`oneway`/`in`/`out`/`inout`代表什么意思？**

    - **oneway关键字**：修饰方法为异步调用方法；
    - **in关键字**：表示输入型参数，由Client端输入，Server端执行，执行完不会影响原参数对象；
    - **out关键字**：表示输出型参数，由Server端输入，Client端执行，执行完不会影响原参数对象；
    - **inout关键字**：表示输入输出型参数，可由Client端或Server端输入;

12. **AIDL有什么缺陷？**

    - 服务端公开方法变更，AIDL需要修改，同时Client端都要变更AIDL文件；
    - 如果AIDL中用到了自定义的Parcelable对象，必须新建一个和它同名的AIDL文件，并且在其中声明为parcelable类型；
    - 不支持声明静态常量；
    - AIDL对应的接口名称必须与AIDL文件名相同不然无法自动编译；
    - AIDL对应的接口方法不能加访问权限修复；

13. **什么情况下需要使用AIDL？**

    - AIDL是Android提供的轻量级进程间通信的具体实现方法，由于内存之间内存等资源时隔离的，所以要实现跨进程的内存访问，比如数据传输、函数跨进程同步调用等，就需要用到AIDL。
    - 准确来讲：当你允许来自不同的客户端访问你的服务并且需要处理多线程问题时必须是使用AIDL，否则其他情况可以选择Messenger或自己实现Binder来完成。

14. **AIDL与Messenger的区别？**

    - Messenger同样是一种轻量级的IPC机制，底层实现自AIDL，进行了封装，开发时不用写AIDL文件，比直接使用AIDL更简单；
    - Messenger一次只能处理一个请求（串行），适用于访问不同进程的服务，但不需要处理多线程的情况；AIDL一次可以处理多个请求（并行），适用于访问不同进程的服务，同时要处理多线程的情况；



### NDK

1. 请介绍一下 NDK？
   Native Developer Kit
2. 请简述JNI在Android中的调用过程？
   - 安装和下载 `Cygwin`，下载 `NDK`
   - 在 `NDK` 项目中 `JNI` 接口的设计
   - 使用`C/C++`实现本地方法
   - `JNI`生成动态链接库`.so`文件
   - 将动态链接库复制到`java`工程，在`java`工程中调用，运行`java`工程即可



## Network Handing

### OkHttp

1. 为什么选择 OkHttp？
2. OkHttp连接池是怎么实现的？里面怎么处理SSL？
3. OkHttp网络拦截器，应用拦截器?OKHttp有哪些拦截器，分别起什么作用？

### Retrofit

1. Retrofit中的泛型是怎么解析的？
2. 编译时注解与运行时注解，为什么retrofit要使用运行时注解？什么时候用运行时注解？

### Json Converters

1. 用过哪些Json解析框架？为什么会选择它？



### Native\H5

1. H5与Native通信你做过什么工作？
2. H5与Native交互，`webView.loadUrl`与`webView.evaluateUrl`区别？
3. Native如何对H5进行鉴权，让某些页面可以调，某些页面不能调？
4. 项目中的Webview与Native通信？
5. 项目中对WebView的功能进行了怎样的增强？
6. RN与Flutter的相同点和区别？
7. webview 如何做资源缓存？
8. `@JavaScriptInterface` 为什么不通过多个方法来实现？
9. webview 中与 js 通信的手段有哪些？
10. 系统上安装了多种浏览器，能否指定某浏览器访问指定页面？请说明原由。
    通过发送URI把参数带过去，或者通过`manifest`里的`intentfilter`里的`data`属性。



## Storage

### Data Storage

1. Android 数据持久化？

   - SharedPreferences

   - File（手机内存、硬盘、SDCard磁盘）：通过Context提供的两个方法来打开数据文件里的文件IO流：

     ```java
     FileInputStream openFileInput(String name)
     FileOutputStream(String name, int mode)
     ```

   - SQLite：SQLite是轻量级嵌入式数据库引擎，它支持 SQL 语言，并且只利用很少的内存就有很好的性能；

   - ContentProvider

     - 数据在Android当中是私有的，当然这些数据包括文件数据和数据库数据以及一些其他类型的数据；
     - Content Provider提供了一种多应用间数据共享的方式，比如：联系人信息可以被多个应用程序访问；
     - 标准的 Content Provider：Android提供了一些已经在系统中实现的标准 ContentProvider，比如联系人信息，图片库等等，你可以用这些 ContentProvider 来访问设备上存储的联系人信息，图片等等；

   - Network

     - 可以调用WebService返回的数据或是解析HTTP协议实现网络数据交互。
     - 可以通过地区名称查询该地区的天气预报，以POST发送的方式发送请求到webservicex.net站点，访问WebService.webservicex.net站点上提供查询天气预报的服务；



### SharedPreferences

1. `commit`和`apply`的区别？

2. SharedPreferences的原理？

   保存给予XML文件存储的`key-value`键值对数据。
   SharedPreferences本身是一个接口，程序无法直接创建实例，只能通过`Context`提供的`getSharedPreferences(String name, int mode)`方法获取实例；且只能获取数据而不支持存储和修改，存储修改时通过`SharedPreferences.edit()`获取内部接口`Editor`对象实现。

3. SharedPreferences读取XML是在哪个线程?

4. SharedPreferences可以跨进程通信吗？如何改造成可以跨进程通信？

### 序列化与反序列化

1. Seriazable与Parceable的区别？

   - Parceable

     Android的序列化API，它的出现是为了解决 Serializable 在序列化的过程中消耗资源严重的问题；Parcelable是直接在内存中读写，性能上要优于 Serializable。但是代码写起来相比 Serializable 方式麻烦一些。，一般只获取内存数据的时候使用。
     Parcelable 对象用来在进程间、Activity 间传递数据，保存实例状态也是用它，Bundle 是它的一个实现，最好只用它存储和传递少量数据，别超过 50k，否则既可能影响性能又可能导致崩溃。

   - Seriazable
     是Java的序列化API，Serializable使用IO读写存储在硬盘上。序列化过程使用了反射技术，并且期间产生临时对象。优点代码少，实现容易。

### SQLite

1. 什么时候用到数据库？
   - 本地数据库SQLite主要作用是存储和保留网络数据，保证无网络数据或网络异常下能够使用，同时减少网络流量的损耗。
   - 如果数据是已知的、静态的，可以在本地SQLite中存储、读取。这样不会因网络问题而降低效率和成功率。
   - 如果数据未知、有实时的变化或者有与其他用户交互、共享的数据必然需要后台服务器数据。
2. SQLite数据库升级，数据迁移问题？
   可以将`dictionary.db`文件复制到Android工程中的`resaw`目录中。所有在`resaw`目录中的文件不会被压缩，这样可以直接提取该目录中的文件。
3. 如何将SQLite数据库（`dictionary.db`文件）与apk文件一起发布？
4. 如何将打开`resaw`目录中的数据库文件？
   - 在Android中不能直接打开`resaw`目录中的数据库文件，而需要在程序第一次启动时将该文件复制到手机内存或SD卡的某个目录中，然后再打开该数据库文件。
   - 复制的基本方法是使用`getResources().openRawResource`方法获得`resaw`目录中资源的`InputStream`对象，然后将该`InputStream`对象中的数据写入其他的目录中相应文件中。
   - 在AndroidSDK中可以使用`SQLiteDatabase.openOrCreateDatabase`方法来打开任意目录中的SQLite数据库文件。

### Bitmap

1. 对Bitmap对象的了解？
2. Bitmap内存复用限制条件？
3. 一张`100x100`的图片在内存中的大小？
4. 谈谈图片压缩的算法？

### Resource

1. `raw` 与 `assets` 文件夹的区别？



## Performance

1. 性能优化你做过哪些？有用过什么工具？有没有精确测量的工具？

2. 内存优化？

3. 绘制优化？

4. App瘦身？

5. 网络优化？

6. 电量优化？

7. FoceClose什么时候会出现？如何避免？

8. 你碰到过什么内存泄漏，怎么处理？

9. 你碰到过什么内存溢出，怎么处理？

10. OOM是否可以try-catch吗？
    为了避免应用在分配Bitmap内存的时候出现`OutOfMemory`异常以后Crash掉，通常在实例化Bitmap的代码中，要对`OutOfMemory`异常进行捕获。但是对于`OutOfMemoryError`来说，这样做是捕获不到的。因为`OutOfMemoryError`是一种`Error`，而不是`Exception`。

11. MVP怎么处理内存泄漏？

12. 什么是ANR？有没有实际的ANR定位问题的经历？
    应用无响应（Application Not Response）。

13. 为什么会发生ANR ？如何定位？如何避免？

    - 原因：高耗时操作（如图像变换）、磁盘和数据库读写操作、大量的创建新对象；

    - 定位：

      - 分析log；
      - 检查`/data/anr/traces.txt`文件；
      - LeakCanary；

    - 避免：

      - UI线程尽量只做跟UI相关的工作；

      - 耗时操作（比如数据库操作，I/O，链接网络或者别的有可能阻塞UI线程的操作）把它们放在单独的线程处理；

      - 尽量用Handler来处理UIThread和别的Thread之前的交互；

14. 有什么实际解决UI卡顿优化的经历？

15. 有做过什么Bitmap优化的实际经验？

16. 如何在网络框架里直接避免内存泄漏，不需要在Presenter中释放订阅？

17. 屏幕适配做过什么工作？

18. 有没有做过什么WebView秒开的一些优化？

19. 图片加载优化有什么经验吗？

20. Bitmap如何处理大图？如何预防OOM？

21. Handler内存泄漏的GCRoot是什么？

22. 怎么优化XML的inflate的时间，涉及IO与反射。了解compose吗？

23. Android本身的API并未声明会抛出异常，则其在运行时有无可能抛出Runtime异常，你遇到过吗？若有的话会导致什么问题？如何解决？

    - 情景：比如`textview.setText()`时，`textview`没有初始化。会导致程序无法正常运行出现 ForceClose；
    - 问题：比如 NullpointerException；
    - 解决：查看 logcat 信息找出异常信息并修改程序；

24. Android程序运行时权限与文件系统权限的区别？

    - 运行时权限：Dalvik（Android授权）
    - 文件系统权限：Linux内核授权

25. DDMS和TraceView的区别？

    - DDMS是一个程序执行查看器，在里面可以看见线程和堆栈等信息
    - TraceView是程序性能分析器

26. 性能优化如何分析systrace？

27. 用IDE如何分析内存泄漏？

28. 启动页白屏、黑屏、太慢怎么解决？

29. App启动崩溃异常怎么捕捉？

30. 如何保持应用的稳定性？



## ThreeParty Library

### RxJava

1. 对 RxJava 的理解，功能与原理，优缺点？
2. RxJava怎么切换线程？
3. Rxjava自定义操作符？
4. RxJava1 与 RxJava2 的区别有哪些？

### Glide

1. Glide的缓存，有用过Glide的什么深入的API，自定义Model是在Glide的什么阶段？
1. LRUCache 原理？

### ARouter

1. ARouter详细原理？
2. ARouter怎么实现接口调用？
3. ARouter怎么实现页面拦截？
4. 如果不用ARouter，你会怎么去解藕接口？设计接口有什么需要注意的？

### Hotfix

1. Tinker的原理是什么，还用过什么热修复框架？Robust的原理是什么？
2. 热修复的原理，资源的热修复的原理，会不会有资源冲突的问题？
3. bugly日志收集的原理是什么？
4. 热修复原理，资源的热修复原理，会不会有资源冲突问题

### EventBus

1. 讲讲 EventBus 的原理？



## Android Platform Architecture

### Android Runtime

1. **Dalvik VM与JVM的差异？**

   - JVM基于栈式，Dalvik基于寄存器；
   - Dalvik并不是按照JVM的标准来实现，Dalvik使用`.dex`字节码格式，Java使用的是`.class`字节码；

2. **`Dalvik`与`ART`虚拟机的区别？**

   - Dalvik使用JIT编译器，每次运行都要编译再运行，拖慢运行效率，ART在第一次安装时使用AOT编译，提高了后续每次运行的效率，但拖慢了首次安装的速度；
   - 由于ART使用了AOT编译，字节码变为机器码之后，占用空间可能会增加10%~20%，理论上ART占用空间比Dalvik大；
   - 由于ART使用了AOT编译，不需要应用程序每次运行时重复的编译，理论上可以减少CPU的使用率，降低功耗；
   - 每次打开应用时，Dalvik会读取classes.dex并解释执行，而ART则在安装应用时将classes.dex转换成oat本地机器码，打开应用时读取oat执行即可；

3. **PathClassLoader与DexClassLoader的区别？**

   PathClassLoader和DexClassLoader均可加载外部的`dex`、`apk`，但PathClassLoader无法指定`.odex`生成目录。





## Architecture

### Presentation

1. MVC、MVP、MVVM原理和区别？
2. MVP怎么处理内存泄露？
3. 如何在网络框架里直接避免内存泄露，不需要在Presenter中释放订阅？
4. MVVM怎么更新UI，DataBinding的原理？
5. 介绍一下你们项目的架构？
6. 组件化有详细了解过吗？module与app之间的区别？moduler通信是如何实现的？
7. 插件化的原理是怎样的？
8. 插件化的主要优点和缺点是什么？
9. 插件化的原理，startActivity hook了哪个方法？
10. 说说插件化中资源的插件化id重复如何解决？



### Design Pattern

1. 谈谈你对 Android 设计模式的理解？
2. 手写生产者-消费者模式？
3. 手写观察者模式？
4. 适配器模式、装饰者模式、外观模式的异同？
5. 单例模式
6. 单例模式有什么缺点？
7. 手写双检查单例模式，各个步骤有什么区别？
8. 项目中常用的设计原则有哪些？
9. 代理模式与装饰模式的区别，手写一个静态代理，一个动态代理？
10. 动态代理有什么作用？
11. OkHttp里面用到了什么设计模式？
12. 动画里面用到了什么设计模式？



## Jetpack

### Lifecycle

1. Lifecycle原理？



### LiveData

1. LiveData的生命周期如何监听？



### ViewModel

1. ViewModel为什么在旋转屏幕后不会丢失状态？
2. ViewModel为什么可以在Activity销毁后保存数据?
3. ViewModel是怎么感知声明周期的？
4. ViewModel怎么实现自动处理生命周期？
5. Google为什么要设计成屏幕旋转后继续存留ViewModel？
6. ViewModel在Activity初始化与在Fragment中初始化，有什么区别？
7. ViewModel是怎么实现双向数据绑定的？
8. ViewModel的使用中有什么坑？



### ViewBinding/DataBinding

1. DataBinding的原理了解吗？



## Build Configuration

### Gradle

1. Gradle源码解析？
2. 对热修复和插件化的理解？
3. 插件化原理分析？
4. 模块化实现（好处，原因）？
5. 项目组件化的理解？
6. 描述清点击AndroidStudio的build按钮后发生了什么？
7. 项目搭建过程中有什么经验，有用到什么gradle脚本，分包有做什么操作？



## App Publishing

1. 了解APK打包的过程吗？
2. 后台杀死App怎么恢复数据？
3. 谈谈你对安卓签名的理解？
4. 一个应用程序安装到手机上的过程发生了什么？



## Other

1. mainfest中配置LargeHeap，真的能分配到大内存吗？
2. 断点续传？
3. SIM卡的EF文件有何作用？
   - SIM卡的文件系统有自己规范，主要是为了和手机通讯，SIM本身可以有自己的操作系统
   - EF就是作存储并和手机通讯用的
4. SpannableString与SpannableStringBuilder的区别？
   - SpannableString长度不可变
   - SpannableStringBuilder长度可变
5. 说说你对屏幕刷新机制的了解，双重缓冲，三重缓冲，黄油模型？
6. Android各个版本API的区别？
7. 低版本SDK如何使用高版本API？
8. App是如何沙箱化，为什么要这么做？
9. JAR和AAR的区别？