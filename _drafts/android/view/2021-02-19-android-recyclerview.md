---
layout: article
title: Android RecyclerView
date: 2021-02-19 15:22 +0800
tags: android

---

A flexible view for providing a limited window into a large data set.（在有限的视图内展示大量的数据）

<!--more-->

![4e9c18b463f00bf7](https://developer.android.com/static/codelabs/basic-android-kotlin-training-recyclerview-scrollable-list/img/4e9c18b463f00bf7.png)

## RecyclerView概览

![recyclerview-structure-2](/Users/mcgrady/Pictures/gallery/android/recyclerview/recyclerview-structure-2.png)

### Adapter

获取数据并准备数据供RecyclerView显示

### ViewHolder

持有所有的用于绑定数据或者需要操作的View

### LayoutManager

布局管理器，负责RecyclerView子View的布局

- LinearLayoutManager
- GridLayoutManager
- StaggeredGridLayoutManager
- FlexboxLayoutManager

### Recycler

回收复用机制

### ChildHelper

管理子View

### ItemDecoration

Item的装饰器

- **onDraw()**：可实现类似 padding 的效果
- **onDrawOver()**：可实现类是绘制背景的效果，内容在上面
- **getItemOffsets()**：可绘制在内容上面，覆盖内容

### ItemAnimator

为Item的操作添加动画效果

### DiffUtil

DiffUtil 可以计算两个列表之间的差异并输出一个更新操作列表，将第一个列表转换为第二个列表。

### RecyclerViewDataObserver

数据观察器

### ViewInfoStore

存储子View的动画信息

### ViewFlinger

快速滑动管理

### NestedScrollingChildHelper

管理子View嵌套滑动



## RecyclerView的结构设计

![android-recyclerview](/Users/mcgrady/Pictures/gallery/android/recyclerview/android-recyclerview.png)

- RecyclerView是一个ViewGroup ，它只认识View，不清楚Data的具体结构，而它最终的职责是将Datas以一定的规则展示给用户，因此两个陌生人之间想构建通话，需要通过适配器模式（Adapter）来与建立交流；
- 通过Adapter将Datas转化为RecyclerView认识的ViewHolder，而ViewHolder包含了ItemView及在RecyclerView中的位置信息，因此RecyclerView间接的认识了Datas；
- 尽管Adapter已经将Datas转换为RecyclerView认识的View，但RecyclerView并不包含管理子View的能力，所以通过桥接的模式，雇佣了一个LayoutManager来完成子View的布局；
- LayoutManager协助RecyclerView来完成布局，并具备管理这些View的能力，由于RecyclerView需要在有限的空间展现更多的内容，所以需要对View进行缓存以加快显示的速度，而缓存有涉及回收和恢复，所以还需要增加一个Recycler来管理这些View，在LayoutManager需要View的时候向Recycler获取，当LayoutManager不需要View（视图出画）的时候，将废弃的View丢给Recycler；
- 还有当View的变动需要优雅的动画时，RecyclerView并不具备这个能力，所以通过观察者模式，让ItemAnimator来辅助RecyclerView完成View变换动画的能力；

通过上面RecyclerView的结构设计，我们了解到其中运用了很多的设计模式：

### RecyclerView中的设计模式

- **桥接模式**：RecyclerView将布局方式独立成LayoutManager，实现对布局的定制化；
- **组合模式**：RecyclerView通过`dispatchLayout()`对ItemView进行布局绘制；
- **适配器模式**：ViewHolder将RecyclerView与ItemView联系起来， 使得RecyclerView方便操作ItemView；
- **观察者模式**：给ViewHolder注册观察者，当调用`notifyDataSetChanged()`时，执行重新绘制；



## RecyclerView的绘制流程

RecyclerView的measure和layout过程委托给了LayoutManager来处理。

### dispatchLayout()

- **dispatchLayoutStep1()**：
  Step1的目的是记录VIew的状态，首先遍历当前所有的View依次进行处理，mItemAnimator会根据每个View的信息封装成一个ItemHolderInfo ，这个ItemHolderInfo中主要包含当前View的位置状态等。然后ItemHolder将存入mViewInfoStore中；
- **dispatchLayoutStep2()**：
  Step2主要是去布局View，主要工作由LayoutManager负责；
- **dispatchLayoutStep3()**：

### onLayoutChildren()

确定锚点，以此为起点向START或END方向填充ItemView，直至填满所有的区域。

### 锚点

**mAnchorInfo**是布局锚点信息，包含子View在Y轴上起始绘制的偏移量(coordinate)，ItemView在Adapter中的索引位置(position)和布局方向(mLayoutFromEnd)。锚点的寻找是由`updateAnchorInfoForLayout()`负责

**`updateAnchorInfoForLayout()`**

`updateAnchorInfoForLayout()`内首先通过子View来获取锚点，如果没有获取到，根据头尾点来作为锚点，所以这里主要关注 `updateAnchorFromChildren()`。

`updateAnchorFromChildren()`内部主要在寻找点；获取到 anchor 信息后，接下来就可以根据锚点调用`fill()填充剩余的空间。

### fill()

`fill()`中比较主要的函数`recylceByLayoutState()`它会根据当前信息对不需要的View进行回收，`recylceByLayoutState()` 内部的  `recycleViewsFromStart()`，这个函数的作用是遍历所有的子View，找出逃离边界的View进行回收，回收函数在 `recycleChildren()` 里，而这个函数最后又会调用 `removeAndRecycleViewAt()`， `removeAndRecycleViewAt()` 首先调用 `removeViewAt()` ，这个函数的作用是将View从RecyclerView中移除，接着 `recycler` 执行了View的回收逻辑。

以上就是 `fill()` 执行的子View回收的逻辑了，接下来我们继续关注 `fille()` 的 `layoutChunk()` 函数。

**`fill()#layoutChunk()`**

`layoutChunk()` 内部主要是 recycler 获取子View 的逻辑，`recycler` 会根据位置返回一个View，然后调用 `LayoutManager.addView()` 方法，这个方法会多次辗转调用到 `RecyclerView.addView()` 方法，将View添加到RecyclerView中。

综上，此过程就完成了子View的测量和布局。



如果RecyclerView宽高没有写死，`onMeasure()`就会执行完子View的`measure()`和`layout()`方法，`onLayout()`仅仅是重置了一些参数；如果RecyclerView宽高写死，子View的`measure()`和`layout()`会延后到`onLayout()`中执行。



## RecyclerView的缓存机制

RecyclerView缓存ViewHolder对象有4个级别，从优先级从高到地依次为：

- **ArrayList<ViewHolder> mAttachedScrap**：缓存屏幕可见范围的ViewHolder，当`LayoutManager.onLayoutChildren()`布局视图时，RecyclerView上所有的子View都会暂存到mAttachedScrap集合中，如果匹配到RecyclerView上的`position`或`itemId`，可以直接使用，而不需要调用`onBinderViewHolder()`重新绑定数据；
- **ArrayList<ViewHolder> mChangedScrap**：mChangedScrap和mAttachScrap属于同一级别的缓存，不过mChangedScrap的调用场景是`notfityItemChanged()`和`notifyItemRangeChanged()`，只有发生变化的ViewHolder才回放入mChangedScrap中。mChangedScrap缓存中的ViewHolder是需要调用`onBindViewHolder()`重新绑定数据的；
- **ArrayList<ViewHolder> mCachedViews**：缓存滑动时不可见的ViewHolder，默认缓存2个，与mAttachedScrap一样，当`position`与`itemId`匹配上，将不需要重新绑定数据；
- **ViewCacheExtension mViewCacheExtension**：可自定义扩展的缓存策略；
- **RecyclerViewPool mRecyclerPool**：ViewHolder缓冲池，本质上是SparseArray，其中key是`ViewType`，value是`ArrayList<ViewHolder>`，默认情况下，每个ArrayList最多存储5个ViewHolder；



### RecyclerView的四级缓存对比

| 缓存级别 | 成员                            | 操作说明                                        | 是否重新<br>创建视图 | 是否重新<br>绑定数据 |
| -------- | ------------------------------- | ----------------------------------------------- | -------------------- | -------------------- |
| 一级缓存 | mAttachedScrap<br>mChangedScrap | 缓存屏幕可见范围的ViewHolder                    | 否                   | 否                   |
| 二级缓存 | mCachedViews                    | 缓存滑动时屏幕不可见的ViewHolder<br>默认上限为2 | 否                   | 否                   |
| 三级缓存 | mViewCacheExtension             | 可自定义扩展的缓存策略                          | ？                   | ？                   |
| 四级缓存 | mRecyclerPool                   | ViewHolder缓冲池<br>默认上限为5                 | 否                   | 是                   |



### RecyclerView的复用过程

tryGetViewHolderForPositionByDeadline

### RecyclerView的回收过程

detachAndScrapAttachedViews





#### Recycler#tryGetViewHolderForPositionByDeadline() 剖析

##### 第一次尝试获取缓存（mAttachedScrap/mCacheViews）

> 0) If there is a changed scrap, try to find from there
>    如果有更改过的废弃的 ViewHolder，尝试从那里获取

##### 第二次尝试获取缓存（对应hasStableIds情况）

> 1) Find by position from scrap/hidden list/cache
>    从废弃的 ViewHolder 列表缓存中按位置获取

##### 第三次尝试获取缓存（mViewCacheExtension）

> 2) Find from scrap/cache via stable ids, if exists

##### 第四次获取缓存（mRecyclerPool）

> We are NOT sending the offsetPosition because LayoutManager does not know it.
>
> fallback to pool

#### 总结

1. RecyclerView 本身只是一个容器（RecyclerView extends ViewGroup），它的 onLayout 方法重写决定了 itemView 的排布方式，其中由四个分派布局的步骤来处理，将委托给LayoutManager来管理，而LayoutManager处理布局的关键在于onLayoutChildren ，重写这个方法决定了 RecyclerView 的 itemView 如何布局
4. mAttachedScrap/mCacheViews在第一次尝试时只对View的复用，并不区分type，但在第二次尝试时区分了type，是对于ViewHolder的复用，ViewCacheExtension、RecycledViewPool是对于ViewHolder的复用，且区分了type。
5. 如果缓存ViewHolder时超过了mCachedView的限制，会将最老的ViewHolder（也就是mCachedView缓存队列的第一个ViewHolder）移到RecycledViewPool中。



参考文章：

[RecyclerView 必知必会](https://juejin.cn/post/6844903459699982343#heading-19)
[Android ListView 与 RecyclerView 对比浅析 -- 缓存机制](https://juejin.cn/post/6844903448974983181)

