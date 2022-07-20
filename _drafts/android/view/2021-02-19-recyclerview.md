# RecyclerView



## RecyclerView 结构



![RecyclerView 结构](..\..\images\android\recyclerview.png)



### Adapter

处理数据集合并负责绑定视图

### ViewHolder

持有所有的用于绑定数据或者需要操作的View

### LayoutManager

布局管理器，负责 RecyclerView 子View 的布局

#### LinearLayoutManager

#### GridLayoutManager

#### StaggeredGridLayoutManager

#### FlexboxLayoutManager

### Recycler

回收复用机制

### ItemDecoration

Item的装饰器

#### ItemDecoration.onDraw(Canvas c, RecyclerView parent, State state)

可实现类似 padding 的效果

#### ItemDecoration.onDrawOver(Canvas c, RecyclerView parent, State state)

可实现类是绘制背景的效果，内容在上面

#### ItemDecoration.getItemOffsets(Rect outRect, View view, RecyclerView parent, State state)

可绘制在内容上面，覆盖内容

### ItemAnimator

为Item的操作添加动画效果

### DiffUtil

DiffUtil 可以计算两个列表之间的差异并输出一个更新操作列表，将第一个列表转换为第二个列表。



## RecyclerView 源码解析

### 设计思路

RecyclerView 的官方定义：A flexible view for providing a limited window into a large data set.（在有限的视图内展示大量的数据）

![](..\..\images\android\recyclerview-design-model-1.png)

RecyclerView 是一个 ViewGroup ，它只认识 View，不清楚 Data 数据的具体结构，而它最终的职责是将 Datas 的数据以一定的规则展示给用户，因此两个陌生人之间想构建通话，需要通过适配器模式（Adapter）来与 Datas 建立交流。

![](..\..\images\android\recyclerview-design-model-2.png)

如上图所示，Adapter 将 Datas 转化为 RecyclerView 认识的 ViewHolder，因此 RecyclerView 就间接的认识了 Datas。

但是，尽管 Adapter 已经将 Datas 转换为 RecyclerView 认识的 View，RecyclerView 并不包含管理子 View 的能力，所以通过桥接的模式，雇佣了一个 LayoutManager 来完成布局，现在，图示变成了下面的样子：

![](..\..\images\android\recyclerview-design-model-3.png)

如上图所示，LayoutManager 协助 RecyclerView 来完成布局，但 LayoutManager 也存在自身的弱点，它只知道如何将一个个的 View 布局在 RecyclerView 上，并不知道如何管理这些 View，所以要管理好这些 View，需要增加一个 Recycler，LayoutManager 需要 View 的时候向 Recycler 获取，当 View 不存在时Recycler 通过 ViewHolder 获取 ，然后提供给 LayoutManager，当 LayoutManager 不需要 View（视图画出）的时候，将废弃的 View 丢给 Recycler，图示如下：

![](..\..\images\android\recyclerview-design-model-4.png)

到了这里，有了负责翻译 Datas 的 Adapter、负责布局的 LayoutManager、负责管理 View 的 Recycler，看似很完美，当 View 的变动需要优雅的动画时，RecyclerView 并不具备这个能力，所以通过观察者模式，让 ItemAnimator 来辅助 RecyclerView 完成 View 变换动画的能力，如下图所示：

![](..\..\images\android\recyclerview-design-model.png)

### RecyclerView 设计结构

![](..\..\images\android\recyclerview-structure.png)

![](..\..\images\android\recyclerview-structure-2.png)

- RecyclerView 容器
- RecyclerViewDataObserver 数据观察器
- Recycler 循环复用系统，核心部件
- SavedState RecyclerView 状态
- AdapterHelper 适配器更新
- ChildHelper 管理 子View
- ViewInfoStore 存储 子View 的动画信息
- Adapter 数据适配器
- LayoutManager 负责 子View 的布局，核心部件
- ItemAnimator Item动画
- ViewFlinger 快速滑动管理
- NestedScrollingChildHelper 管理 子View 嵌套滑动



### RecyclerView 使用的设计模式

#### 桥接模式

通过桥接模式，使 RecyclerView 将布局方式独立成 LayoutManager，实现对布局的定制化。

#### 组合模式

通过组合模式，使RecyclerView通过 dispatchLayout 对 ItemView 进行布局绘制。

#### 适配器模式

通过适配器模式，ViewHolder 将 RecyclerView 与 ItemView 联系起来， 使得RecyclerView方便操作ItemView。

#### 观察者模式

通过观察者模式，给 ViewHolder 注册观察者，当调用 notifyDataSetChanged 时，执行重新绘制。



### 绘制流程

RecyclerView的measure和layout过程委托给了RecyclerView.LayoutManager来处理。

#### onMeasure

##### dispatchLayoutStep1

Step1的目的是记录VIew的状态，首先遍历当前所有的View依次进行处理，mItemAnimator 会根据每个 View 的信息封装成一个 ItemHolderInfo ，这个 ItemHolderInfo 中主要包含当前 View 的位置状态等。然后 ItemHolder 将存入 mViewInfoStore 中。

##### dispatchLayoutStep2

Step2主要是去布局 View，前面提到 RecyclerView 的布局是由 LayoutManager 负责的，所以Step2的主要工作由 LayoutManager 负责。
这里以常见的 LinearLayoutManager 为例，进行讲解：

**`LinearLayoutManager#onLayoutChildren()`** 

`onLayoutChildren()` 大致过程整理如下：

- 找到 anchor 点
- 根据 anchor 一直向前布局，直至填充满 anchor 点前面的所有区域
- 根据 anchor 一直向后布局，直至填充满 anchor 点后面的所欲区域

`mAnchorInfo` 为布局锚点信息，包含子控件在Y轴上起始绘制偏移量（`coordinate`），ItemView 在 Adapter 中的索引位置（`position`）和布局方向（`mLayoutFromEnd`）。这部分代码的主要功能是：确定布局锚点，以此为起点想开始和结束方向填充 ItemView，如图所示：

![](../../images/android/recyclerview-onLayoutChildren.png)

> anchor 锚点的寻找是由 updateAnchorInfoForLayout 负责

**`updateAnchorInfoForLayout()`**

`updateAnchorInfoForLayout()` 内首先通过 子View 来获取 anchor，如果没有获取到，根据头/尾点来作为 anchor，所以这里主要关注 `updateAnchorFromChildren()`，`updateAnchorFromChildren()`内部主要在寻找 anchor点；获取到 anchor 信息后，接下来就可以根据 anchor 调用 `fill()` 布局了，这里的内容还是相对繁重的。

**`onLayoutChildren()#fill()`**

接下来关注下 `fill()` 中比较主要的函数 `recylceByLayoutState()` ，它会根据当前信息对不需要的View进行回收，`recylceByLayoutState()` 内部的  `recycleViewsFromStart()`，这个函数的作用是遍历所有的子View，找出逃离边界的View进行回收，回收函数在 `recycleChildren()` 里，而这个函数最后又会调用 `removeAndRecycleViewAt()`， `removeAndRecycleViewAt()` 首先调用 `removeViewAt()` ，这个函数的作用是将View从RecyclerView中移除，接着 `recycler` 执行了View的回收逻辑。

以上就是 `fill()` 执行的子View回收的逻辑了，接下来我们继续关注 `fille()` 的 `layoutChunk()` 函数。

**`fill()#layoutChunk()`**

`layoutChunk()` 内部主要是 recycler 获取子View 的逻辑，`recycler` 会根据位置返回一个View，然后调用 `LayoutManager.addView()` 方法，这个方法会多次辗转调用到 `RecyclerView.addView()` 方法，将View添加到RecyclerView中。

综上，此过程就完成了子View的测量和布局。

##### dispatchLayoutStep3

#### 总结

1. RecyclerView 将绘制流程交给 LayoutManager 处理，若没有设置LayoutManager则不会测量子View。
2. 绘制流程区分正向绘制和倒置绘制。
3. 绘制前先确定锚点，然后向上、向下绘制，`fill()`至少会执行两次，如果绘制完还有剩余空间，则会再执行一次`fill()`。
4. LayoutManager获得的View来自RecyclerView中的`Recycler.next()`方法，其中涉及RecyclerView的缓存策略，如果缓存没有拿到，则走一遍我们重写的`Adapter.onCreateViewHolder()`方法。
5. 如果RecyclerView宽高没有写死，`onMeasure()`就会执行完子View的`measure()`和`layout()`方法，`onLayout()`仅仅是重置了一些参数；如果RecyclerView宽高写死，子View的`measure()`和`layout()`会延后到`onLayout()`中执行。



### 缓存机制

#### RecyclerView 回收流程

![img](https://pic3.zhimg.com/80/v2-59434d0fea50fa1ef11477cf3819cec2_720w.jpg)

#### RecyclerView 复用流程

![](../../images/android/recyclerview-recycler-process.jpg)



#### RecyclerView的四级缓存策略

##### 1. mAttachedScrap

##### 2. mCacheViews

##### 3. mViewCacheExtension

##### 4. mRecyclerPool



#### Recycler#tryGetViewHolderForPositionByDeadline() 剖析

##### 第一次尝试获取缓存（mAttachedScrap/mCacheViews）

> 0) If there is a changed scrap, try to find from there
>    如果有更改过的废弃的 ViewHolder，尝试从那里获取

![](../../images/android/recyclerview-cache-1-try.png)

##### 第二次尝试获取缓存（对应hasStableIds情况）

> 1) Find by position from scrap/hidden list/cache
>    从废弃的 ViewHolder 列表缓存中按位置获取

![](../../images/android/recyclerview-cache-2-try.png)

##### 第三次尝试获取缓存（mViewCacheExtension）

> 2) Find from scrap/cache via stable ids, if exists



##### 第四次获取缓存（mRecyclerPool）

> We are NOT sending the offsetPosition because LayoutManager does not know it.
>
> fallback to pool

#### 总结

1. RecyclerView 本身只是一个容器（RecyclerView extends ViewGroup），它的 onLayout 方法重写决定了 itemView 的排布方式，其中由四个分发布局的步骤来处理，将委托给 LayoutManager 来管理，而 LayoutManager 处理布局的关键在于 onLayoutChildren ，重写这个方法决定了 RecyclerView 的 itemView 如何布局
2. Adapter <VH extends ViewHolder> 为什么以这样的方式自定义 Adapter ？因为 Adapter 是 RecyclerView 的数据和 itemView 的连接层，itemView 最终是封装到 ViewHolder 中的，绑定数据就要和 ViewHolder 发生关系
3. RecyclerView内部大体可以分为四级缓存：
   - mAttachedScrap
   - mCacheViews
   - ViewCacheExtension
   - RecyclerPool
4. mAttachedScrap/mCacheViews在第一次尝试时只对View的复用，并不区分type，但在第二次尝试时区分了type，是对于ViewHolder的复用，ViewCacheExtension、RecycledViewPool是对于ViewHolder的复用，且区分了type。
5. 如果缓存ViewHolder时超过了mCachedView的限制，会将最老的ViewHolder（也就是mCachedView缓存队列的第一个ViewHolder）移到RecycledViewPool中。



### RecyclerView与ListView对比浅析 —— 缓存机制

#### 1. 层级不同

RecyclerView比ListView多两级缓存，支持多个离屏ItemView缓存，支持开发者自定义缓存处理逻辑，支持所有RecyclerView共用同一个RecyclerViewPool。

![ListView Cache Level](../../images/android/listview-cache.png)

![RecyclerView Cache Level](../../images/android/recyclerview-cache-level.jpg)

ListView和RecyclerView缓存基本一致：

1. mActiveViews和mAttachedScrap功能相似，意义在于快速重用屏幕上可见的列表项ItemView，而不需要重新createView和bindView。

2. mScrapView和mCacheViews + mRecyclerPool功能相似，意义在于缓存离屏的ItemView，目的是让即将进入屏幕的ItemView重用。

3. RecyclerView的优势在于：

   - mCacheViews的使用，可以做到屏幕外的列表项ItemView进入屏幕内时也无需bindView快速重用。
   - mRecyclerPool可以供多个RecyclerView共同使用，在特定场景下，如ViewPager+多个列表页下有优势。

   客观来说，RecyclerView在特定场景下对ListView的缓存机制做了补强和完善。



#### 2. 缓存不同

1. RecyclerView缓存 **RecyclerView.ViewHolder**，抽象可理解为：View + ViewHolder（避免每次createView时调用findViewById）+ flag（标识状态）。
2. ListView缓存**View**

ListView获取缓存的流程：

![](../../images/android/listview-cache-process.png)

RecyclerView获取缓存的流程：

![](../../images/android/recyclerview-cache-process.png)



1. RecyclerView中mCacheViews 获取缓存时，是通过匹配pos获取目标位置的缓存，这样的好处是，当数据不变的情况下，无需重新bindView；而同样是离屏缓存，ListView从mScrapViews根据pos获取相应的缓存，但并没有直接使用，而是重新getView（即会重新bindView）。
2. ListView中通过pos获取的是View，RecyclerView中通过pos获取的是ViewHolder（View、ViewHolder、flag）；从流程图来看，标志flag的作用是判断View是否需要重新bindView，这也是RecyclerView实现局部刷新的一个核心。

#### 3. 局部刷新

由上文可知， RecyclerView的缓存机制更加完善，但不算质的变化，RecyclerView更大的亮点在于提供了局部刷新的接口，通过局部刷新，就能避免调用许多不用的bindView。

![image](../../images/android/recylcerview-and-listview-part-refresh.gif)

ListView和RecyclerView最大的区别在于数据源改变时的缓存的处理逻辑，ListView是将所有的mActiveViews都移入了二级缓存mScrapViews，而RecyclerView则是更加灵活的对每个VIew修改标志位，区分是否重新bindView。

#### 总结

列表页展示界面，需要支持动画，获取频繁更新，局部刷新，建议用RecyclerView，更加庆大晚上、易扩展。

