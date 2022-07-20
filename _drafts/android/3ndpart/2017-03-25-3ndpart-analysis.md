# popular frame analysis 

android 流行框架解析

## 第三方解決方案汇总
![img](http://img.my.csdn.net/uploads/201412/13/1418436863_2555.jpg)

## 网络请求库

网络请求框架与网络请求功能的关系
![网络请求框架与网络请求功能的关系](http://upload-images.jianshu.io/upload_images/944365-cd9cf3ca0a179b26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![img](http://upload-images.jianshu.io/upload_images/944365-58819416dfd2767a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### Android-Async-Http
### Volley
### OkHttp
### Retrofit


## 图片相关
### Glide、Picasso、Fresco、ImageLaoder 对比
#### 基础

Glide 加载图片的方式和 Picasso 非常相似；虽然两者看起来一样，但 Glide 更易用，因为 Glide 的 `.with(Context/Fragment/Activity)`方法好处是：图片加载会和 Activity/Fragment 的生命周期保持一致。比如在 Paused 状态时暂停加载， 在Resumed 状态时自动重新加载。

#### 图像和内存

同样将 1920×1080 像素的图片加载到 768×432 的 ImageView 中，Glide 加载的图片质量要差于Picasso，这是因为 **Glide 默认的 Bitmap 格式是 RGB-565 ，比ARGB-8888格式的内存开销要小一半**。
> 想要提高 Glide 的图片效果，可以创建一个新的 GlideModule 将 Bitmap 格式转换到 ARGB-8888。同时在 AndroidManifest.xml 中将 GlideModule 定义为 meta-data。

修改了 Bitmap 格式后，Glide 将花费两倍于上次的内存，但是仍远远小于 Picasso 的内存开销，**原因在于 Picasso 是加载了全尺寸的图片到内存，然后让 GPU 来实时重绘大小**。而 Glide 加载的大小和 ImageView 的大小是一致的，当然，Picasso 也是可以指定加载图片大小的，但是问题在于你需要主动计算 ImageView 的大小，或者说你的 ImageView 大小是具体的值（而不是 wrap_content ）
**在加载图片上 Glide 完胜 Picasso，因为 Glide 可以自动计算出任意情况下的 ImageView 大小**。

#### Image质量的细节

**将 ImageView 还原到真实大小时，Glide 加载的图片没有 Picasso 那么平滑**。



#### 磁盘缓存

Picasso 和 Glide 在磁盘缓存策略上有很大的不同。**Picasso 缓存的是全尺寸的，而 Glide 缓存的是跟 ImageView 尺寸相同的**。我们可以将 ImageView 调整成不同大小，但不管大小如何 Picasso 只缓存一个全尺寸的。Glide 则不同，它会为每种大小的 ImageView 缓存 一次。尽管一张图片已经缓存了一次，但是假如你要在另外一个地方再次以不同尺寸显示，需要重新下载，调整成新尺寸的大小，然后将这个尺寸的也缓存起来。

>假如在第一个页面有一个 200×200 的 ImageView，在第二个页面有一个 100×100 的 ImageView，这两个 ImageView 本来是要显示同一张图片，却需要下载两次。不过，你可以通过代码改变这种行为，让Glide既缓存全尺寸又缓存其他尺寸，这样就使得下次在任何 ImageView 中加载图片的时候，全尺寸的图片将从缓存中取出，重新调整大小，然后缓存。

**Glide 的这种方式优点是加载显示非常快。而 Picasso 的方式则因为需要在显示之前重新调整大小而导致一些延迟。不过 Glide 比 Picasso 需要更大的空间来缓存**。

#### 特性
Glide 可以做到和 Picasso 几乎一样多的事，代码也几乎一样。但 **Glide 可以加载 GIF 动态图，而 Picasso 不能**，但是 Glide 动画会消费太多的内存，因此谨慎使用。除了 gif 动画之外，Glide 还可以将任何的本地视频解码成一张静态图片。还有一个特性是你可以配置图片显示的动画，而 Picasso 只有一种动画：fading in，最后一个是可以使用` thumbnail()`产生一个你所加载图片的 thumbnail。其实还有一些特性，不过不是非常重要，比如将图像转换成字节数组等。


#### Fresco
**Fresco 的最大亮点在于它的内存管理**。解压后的图片，即 Android 中的 Bitmap ，占用大量的内存，在 Android 5.0以下系统中，这会显著地引发界面卡顿。而使用 Fresco 将很好地解决这个问题，Fresco 会将图片放到一个特别的内存区域，当图片不再显示的时候，占用的内存会自动被释放，这会使得 APP 更流畅，减少因图片内存占用而引发的 OOM。当 APP 包含的图片较多时，这个效果尤其明显。

**Fresco 支持图像的渐进式呈现**，渐进式的图片格式先呈现大致的图片轮廓，然后随着图片下载的继续，逐渐呈现清晰的图片，这在低网速情况下浏览图片十分有帮助，可以带来更好地用户体验。另外，Fresco 支持加载 gif 图，支持 WebP 格式。

#### 总结
**ImageLoader 优点：**
1. 支持下载进度监听
   通过 PauseOnScrollListener 接口可以在 View 滚动中暂停图片加载。
2. 可以在 View 滚动中暂停图片加载
   这几个图片缓存都可以配置缓存算法，不过 ImageLoader 默认实现了较多缓存算法，如 Size 最大先删除、使用最少先删除、最近最少使用、先进先删除、时间最长先删除等。
3. 默认实现多种内存缓存算法
4. 支持本地缓存文件名规则定义

**Picasso 优点**
1. 自带统计监控功能
   支持图片缓存使用的监控，包括缓存命中率、已使用内存大小、节省的流量等。
2. 支持优先级处理
   每次任务调度前会选择优先级高的任务，比如 App 页面中 Banner 的优先级高于 Icon 时就很适用。

3. 支持延迟到图片尺寸计算完成加载

4. 支持飞行模式、并发线程数根据网络类型而变
   手机切换到飞行模式或网络类型变换时会自动调整线程池最大并发数，比如 wifi 最大并发为 4， 4g 为 3，3g 为 2。
   这里 Picasso 根据网络类型来决定最大并发数，而不是 CPU 核数。

5. “无”本地缓存
   无”本地缓存，不是说没有本地缓存，而是 Picasso 自己没有实现，交给了 Square 的另外一个网络库 okhttp 去实现，这样的好处是可以通过请求 Response Header 中的 Cache-Control 及 Expired 控制图片的过期时间。


**Glide 优点：**

1. 图片缓存->媒体缓存
   Glide 不仅是一个图片缓存，它支持 Gif、WebP、缩略图。甚至是 Video，所以更该当做一个媒体缓存。

2. 支持优先级处理

3. 与 Activity/Fragment 生命周期一致，支持 trimMemory
   Glide 对每个 context 都保持一个 RequestManager，通过 FragmentTransaction 保持与 Activity/Fragment 生命周期一致，并且有对应的 trimMemory 接口实现可供调用。

4. 支持 okhttp、Volley
   Glide 默认通过 UrlConnection 获取数据，可以配合 okhttp 或是 Volley 使用。实际 ImageLoader、Picasso 也都支持 okhttp、Volley。

5. 内存友好
  - Glide 的内存缓存有个 active 的设计
    从内存缓存中取数据时，不像一般的实现用 get，而是用 remove，再将这个缓存数据放到一个 value 为软引用的 activeResources map 中，并计数引用数，在图片加载完成后进行判断，如果引用计数为空则回收掉。

  - 内存缓存更小图片
    Glide 以 url、view_width、view_height、屏幕的分辨率等做为联合 key，将处理后的图片缓存在内存缓存中，而不是原始图片以节省大小

  - 与 Activity/Fragment 生命周期一致，支持 trimMemory

  - 图片默认使用默认 RGB_565 而不是 ARGB_888
    虽然清晰度差些，但图片更小，也可配置到 ARGB_888。

  - 其他：Glide 可以通过 signature 或不使用本地缓存支持 url 过期


#### 使用场景建议

Picasso 所能实现的功能 Glide 都能做到，只是所需设置不同。两者的区别是 Picasso 比 Glide 体积小很多且图像质量比 Glide 高，但Glide 的速度比 Picasso 更快。

Glide 的长处是处理大型的图片流，如 gif、video，如果要制作视频类应用，Glide 当为首选。

Fresco 可以说是综合了之前图片加载库的优点，其在5.0以下的内存优化非常好，但它的不足是体积太大，按体积进行比较：Fresco>Glide>Picasso，所以 Fresco 在图片较多的应用中更能凸显其价值，如果应用没有太多图片需求，不推荐使用 Fresco。