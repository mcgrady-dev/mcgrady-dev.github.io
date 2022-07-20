---
layout: 
title: Android ImageLoad Lib Analysis
date: 2021-03-09 21:53 +0800
tags: [android,3ndpart]
---

图片加载库：Glide、Picasso、Fresco 分析

<!--more-->

## Glide

### Glide 总体设计及流程

![overall-design-glide](https://s2.loli.net/2022/07/19/YwCurIf4LlHM6t9.jpg)

- RequestManager（请求管理器）
- Engine（数据获取引擎）
- Fetcher（数据获取器）
- MemoryCache（内存缓存）
- DiskLRUCache（磁盘缓存）
- Transformation（图片处理）
- Encoder（本地缓存存储）
- Registry（图片类型及解析 器配置）
- Target（目标）

Glide 收到加载及显示资源任务（Job），创建 Request 并将它交给 RequestManager，Request 启动 Engine 去数据源获取资源（通过 Fetcher），获取到后 Transformation 处理后交给 Target。

> Glide 依赖于 DiskLRUCache GifDecoder 等开源库去完成本地缓存和Gif图片解码工作。



### Glide 缓存机制

Glide 将缓存分成了两个模块，一个是内存缓存，一个是磁盘缓存。

这两个缓存模块的作用各不相同，**内存缓存的主要作用是防止应用重复将图片数据读取到内存当中，而硬盘缓存的主要作用是防止应用重复从网络或其它地方重复下载和读取数据**。

#### 内存缓存

Glide 内存缓存实现使用了LruCache算法，除了LruCache之外，Glide还结合了一种弱引用的机制，共同完成了内存缓存功能。

**实现原理：** 正在使用中的图片使用弱引用来进行缓存，不再使用中的图片使用LruCache进行缓存的功能。

##### LRUCache

LRUCache（Least Recently Used 近期最少使用算法），它的主要算法原理就是**把最近使用的对象强引用存储在LinkedHashMap中，并且把最近最少使用的对象在缓存值达到预设定值之前从内存中移除。**

##### LinkedHashMap

LinkedHashMap 使用了一个双向的链表来存储 Map 中的 Entry 顺序关系，这种顺序有两种，一种是 LRU 顺序，一种是插入顺序。

#### 磁盘缓存

当Glide去加载一张图片的时候，Glide默认不会将原始图片展示出来，而是对图片进行压缩和转换之后的图片进行展示，而Glide默认情况下再磁盘缓存的就是转换过后的图片，可以通过调用 `diskCacheStrategy()` 改变这一策略。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       



## Picasso

### Picasso 总体设计及流程

![Picasso](https://s2.loli.net/2022/07/19/ITKlCs1eFrYLN7P.jpg)



## Picasso vs Glide vs Fresco

|                    | Picasso | Glide   | Fresco    |
| ------------------ | ------- | ------- | --------- |
| 发布时间           | 2013/05 | 2014/09 | 2015/05   |
| 是否支持GIF        | false   | true    | true      |
| 是否支持webP       | true    | true    | true      |
| 视频缩略图         | false   | true    | true      |
| 大小               | 100 KB  | 500 KB  | 2~3M      |
| 加速度             | 中      | 高      | 高        |
| Disk+Menmory Cache | true    | true    | true      |
| Easy to use        | low     | mediun  | difficult |
| 开发者             | Square  | Google  | Facebook  |



### Glide 优缺点

**优点：**

- 支持 Memory / Disk 缓存
- Picaso 只会缓存原始尺寸图片，而 Glide 缓存多种规格
- 内存开销小，默认 RGB_565，Picaso 默认是 ARGB_8888

**缺点：**

- 使用方法复杂，实现方法较多
- 使用比 Fresco 简单，但性能（加载/缓存）比不上 Fresco

### Fresco 优缺点

**优点：**

- 大大减少 OOM，底层使用 C++ 解决图片缓存问题
- 使用简单，几乎全部功能都能在 xml 上定制

**缺点：**

- 用法更加复杂
- 依赖包更大（2~3M）
- 底层 C++，阅读源码困难

### Picasso 优缺点

**优点：**

- 使用简单，代码简洁
- 与 Square 出品的 Retrofit / OkHttp 等搭配兼容性好
- 自带统计监测功能
  支持图片缓存使用的监控，包括缓存命中率、已使用内存大小、节省的流量等。
- 支持优先级处理
  每次任务调度钱会选择优先级高的任务，比如App页面中Banner的优先级高于Icon时，适用。

**缺点：**

- 功能简单
- 性能（加载速度等）较差
- 自身没有实现 “本地缓存”
  Picaso 没有实现本地缓存功能，而是交给了 OkHttp 去实现，这样的好处可以通过请求 Response Header 中的 Cache-Control / Expired 来控制图片的过期时间
