## 项目实现

1. 比如项目中药读取本地一张大图要注意什么？读取IO时长、CUP占用大小，如果经常性的读取会浪费时间，并且多次读取会占用更多的内存，所以要做缓存，缓存解决了多次IO时长问题，并且内存只有一份图片文件，但是一张大图的缓存对于一些低配的机器来说内存就比较吃紧，那么内存不够用的情况应该释放掉，那么这个时候缓存要用强引用、弱引用还是软引用？
2. 做一个日志组件，要用到那些技术？比如设计模式中的生产消费者模式、线程安全、线程池，这个线程池如何设计？如果用缓存线程池会带来什么问题？
3. 对 volatile 关键字的理解，我们都知道可以保证可见性，可以用来修饰一些状态变量，比如 boolean 之类的，只有两种状态，并且状态切换跟前面状态没有关系，那么 DCL 为什么要加这个关键字？它可不是为了切换状态。
4. kotlin 中的 data class 在定义参数时最好给一个默认值，是因为给了默认值 data class 会默认生成无参构造函数，如果你的项目里用的是 Gson 来反序列化的话，可以减少这些类反序列化时调用反射的次数，这是因为 Gson 在实现反序列化的时候，其实采用了一种尝试机制，通过不断的反射尝试生成一个实例，如果有无参构造函数的话，在第一步就能生成一个实例，减少后续反射的次数。
5. Activity 窗口层级了解吗？PhoneWindow、DecorView、ContentView？组件化中怎么在各个模块的页面设计一个阅读时间奖励，到到时间弹出一个奖励的 View，到时间就消失？如果你看过Activity窗口的层级结构，就不难想到用 ContentView 加载这个布局，对于这个工具类的方法，只需要传一个Activity就可以了。
6. 对于应用更新这块是如何做的？(解答：灰度，强制更新，分区域更新)？
7. 实现一个 Json 解析器（可以通过正则提高速度）？
8. 一张 100x100 的图片在内存中的大小？



## Window

1. Window是什么？在Android中哪些地方用到？

   - Window是Android中唯一展示视图的中介，所有的视图都是通过Window来呈现的，无论Activity、Dialog或Toast，它们的视图都是附加在Window上的，所以Window是View的直接管理者。
   - Window的具体实现在WindowManagerService中，但创建Window或者访问Window的操作都需要WindowManager。所以这就需要WindowManager和WindowMangerService进行交互，交互的方式就是通过IPC，具体涉及的参数就是token。
   - 每个Window都对应着一个View和ViewRootImpl，Window和View通过ViewRootImpl建立联系，所以Window并不是实际存在的，而是以View的形式存在。

   涉及View的地方：

   - 事件分发机制：
     界面上事件分发机制的开始都有这么一个过程：DecorView –> Activity –> PhoneWindow –> DecorView –> ViewGroup。
   - 各种视图的显示：
     比如Activity的setContentView，Dialog，Toast显示视图等等都是通过Window完成的。

2. Window的分层和类别？
   由于界面上不止一个WIndow，所有就有了分层的概念。每个Window都有自己对应的Window层级––z-ordered，层级打的会覆盖到层级小的上面，类似HTML中的z-index。

   Window主要分为三个类别：

   - 应用Window：对应着一个Activity，Window层级为1-99，在视图在下层。
   - 子Window：不能单独存在，需要附属在特定的父Window之中（如Dialog就是子Window），Window层级为1000-1999。
   - 系统Window：需要声明权限才能创建的Window，比如Toast和系统状态栏，Window层级为2000-2999，处于视图最上层。

3. Window的内部机制––添加、删除、更新

   Window的操作是通过WindowManager来完成的，而WindowManager是一个接口，它的实现类是WindowManagerImpl，并且全部交给WindowManagerGlobal来处理。

   1. `addView()`
      通过add方法修改了WindowManagerGlobal中的一些参数，比如：
      mViews––存储了所有Window对应的View
      mRoots––存储了所有Window对应的ViewRootImpl
      mParams––村粗了所有Window对应的布局参数
      其次，`ViewRootImpl.setView()`主要完成了两件事，一是通过`requestLayout()`完成一部刷新界面请求，进行View绘制流程；二是通过WindowSession进行一次IPC调用，交给WindowManagerService来实现Window的添加。

   2. `updateViewLayout()`

      这里更新了`WindowManager.LayoutParams`和`ViewRootImpl.LayoutParams`，然后ViewRootImpl内部同样会对View进行刷新，最后通过IPC通讯，调用到WindowManagerService完成更新。

   3. `removeView()`
      通过View找到mRoots中对应的索引，然后同样走到ViweRootImpl中进行View删除工作，通过`die()`方法，最终走到`dispatchDetachedFromWindow()`中，执行了一下几件事：

      - 回调`onDetachedFromWindow()`
      - 垃圾回收相关操作
      - 通过Session的`remove()`在WindowManagerService
      - 通过Choreographer移除监听器

   4. Window中的Token是什么？

      - 是Window类中的一个变量，`是一个Binder对象`。在Window中主要是实现WindowManagerService和应用所在的进程通信，也就是上文说到的WindowManager和WindowManagerService进行交互。
      - 同时也是添加View的权限标识，拥有token的context可以创建界面、进行UI操作，而没有token的Context，如Service、Application，是不允许添加View到屏幕上的。所以它存在的意义就是为了保护Window的创建，也是为了防止Application或Service来做进行View或者UI相关的一些操作。

   5. Activity、Dialog、Toast的Window创建过程？

   - Dialog
     创建一个Dialog会经历一下几个步骤：
     1. 创建一个新的Window，类型为PhoneWindow，与Activity创建Window过程类似，并设置setCallback回调。





1. 深度理解Android平台架构、主要组成和工作模式；

2. 有linux内核、framework和jni、虚拟机、安全逆向等底层技术经验；

3. 深入理解设计模式，能够设计出高內聚，低耦合的系统框架，提升开发效率；

4. 具有比较强的客户端系统架构设计能力，在通用性、跨平台和复用性方面有深刻的理解；

5. 有大型软件架构设计和重构经验。

   

1. **学习基础知识**

**Activity相关实体知识体系**

• Task启动原理与调用栈内核

• 生命周期与内核管理原理

• ViewGroup源码解析

• View源码分析与高级自定义View项目实战

• 事件分发的核心机制

• Handler通信原理与框架手写

• Intent数据传递原理和内核

• Hook Resource源码实现

**Fragment 内核**

• Fragment事务管理机制与控件混合应用原理

• Fragment事务管理的原理

**Service 内核原理**

• 生命周期及AMS关系

• 两种启动方式启动原理

• 基于内核的应用实战

• Service进程优先级调优与实战

• Service职责原理

**实体间的通信方案**

**实体中数据存储专题





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

