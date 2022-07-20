## WebView Changelog

### Android4.4 (API level 19)

在 Android 4.4 以前，WebView 使用原生自带的 Android WebKit 内核，对 HTML5的支持很不好。

从 Android 4.4 开始，Chromium 内核取代了 Android WebKit 内核。（Android System WebView）

### Android 5.0 (API level 21)

从 Android 5.0 开始，WebView 被移植成了独立的 apk，可以不依赖系统而独立存在和更新。

> 可以通过 系统设置查看 Android System WebView 的当前版本。

### Android 7.0 (API level 24)

从 Android 7.0 开始，如果系统安装了 Chrome(version > 51)，Chrome 将会直接为应用的 WebView 提供渲染，WebView 版本随着 Chrome 而更新，用户也可以选择 WebView 的服务提供方（开发者选项的 WebView Implementation）；同时 WebView 可以脱离应用，在一个独立的沙盒进程中渲染页面（需在开发者选项里启用）。

### Android 8.0 (API level 26)

从 Android 8.0 开始，默认开启 WebView 多进程模式，即 WebView 运行在独立的沙盒进程中。



## WebView 缓存策略

### LOAD_CACHE_ONLY

只使用本地缓存，不进行网络请求

### LOAD_NO_CACHE

不使用本地缓存，只通过网络请求

### LOAD_CACHE_ELSE_NETWORK

只要本地有缓存就进行使用，否则通过网络请求

### LOAD_DEFAULT

根据 Http 协议来决定是否进行网络请求。以请求一个网络上静态文件为例：

![request_favicon](../../images/android/request_favicon.png)

- Cache-Control
  Cache-Control 是 Http 1.1 中新增加的一个用来定义资源缓存策略的报文头，它由一些定义一个响应资源应该何时被缓存、如何被缓存以及缓存多长时间的指令组成，可选值有很多种：

  - no-cache
  - no-store
  - only-if-cached
  - max-age：比如上图所示就使用到了 max-age 来设定资源的最大有效时间，时间单位为秒

  Cache-Control 也是一个通用的 Http 报文头字段，它可以分别在请求头和响应头中使用，具有不同的含义，以 max-age 为例：

  - 请求头：客户端用于告知服务端，希望接收一个有效期不大于 max-age 的资源
  - 响应头：服务端用于告知客户端，该资源在请求发起后的 max-age 时间内均是有效的，上图所示的 2592000 秒也即 30 天，客户端在第一次发起请求后的 30 天内无需再向服务端进行请求，可以直接使用本地缓存

  如果在 WebView 中使用了 LOAD_DEFAULT 的话，就会遵循此 Http 缓存策略，在有效期内 WebView 会直接使用本地缓存。

- Expirse

- ETag
  用于作为资源的唯一标识信息

- Last-Modified
  用于记录资源的最后一次修改时间

但 Http 缓存策略也存在一些问题需要注意，即如何保证**用户在资源更新了时能马上感知到且重新下载最新资源**。



## WebView 问题优化

### 启用JS安全漏洞问题

### 301/302重定向问题

### 白屏处理





## WebView 性能优化

### WebView 预加载

- 触发时机的选择：通过 IdleHandler 提交任务，当 MessageQueue 空闲的时候执行预加载
- 创建 WebView 所需的 Context 选择：预加载 WebView 绑定的 Context 需保证与最终的 Context 之前的一致性。通过 MutableContextWrapper 来解决，MutableContextWrapper 是系统提供的 Context 包装类，起内部包含一个 baseContext，内部的方法都会交由 baseContext 来实现，且MutableContextWrapper 允许外部替换它的 baseContext，因此可以在预创建 WebView 的时候用 Application 作为 baseContext，等 WebView 与 Activity 进行实际关联时再进行替换。

### 预置离线包

精简并抽取公共的 JS 和 CSS 文件作为通用的页面模板，预置到客户端汇总，并为每套末班文件标注也定的版本号，通过 app 后台定时静默更新，通过这种方式来避免每次使用都要去联网请求，从而缩短总耗时

### 并行请求

H5 在加载模板文件的同时，由 Native 端来请求正文数据，Native 端再通过 JS 将正文数据传给 H5，以此来实现并行请求从而缩短总耗时

### 预加载页面模板

当模板和正文数据分离之后，可以直接在预加载 WebView 的同时就让其预热加载模板，这样每次使用时仅需要将正文数据传给 H5，H5 收到数据后直接进行页面渲染即可

### 延迟加载

呈现首屏页面所需要的依赖项越多，就意味着用户需要的等待时间就越长，因此要尽可能地减少在首屏完成前执行的操作，对于一些非首屏必需的网络请求、 JS 调用、埋点上报等，都可以后置到首屏显示后再执行

### 页面静态直出

由后端对正文数据和前端代码进行整合，直出首屏内容，直出后的 HTML 文件已经包含了首屏展现所需的内容和样式，无需进行二次加工，内核可以直接渲染。其它动态内容可以在渲染完首屏后再进行异步加载

### 复用 WebView

当 WebView 使用完毕后可以将其正文数据全部清空并再次存入缓存池中，等下次需要时就可以直接注入新的正文数据进行复用了，从而减少了频繁创建 WebView 和预热模板文件带来的开销，前提是 WebView 使用的都是固定的统一模板。

### 视觉优化

在Android 版本 4.3 到 6.0 之间，Activity 切换过程中无法渲染 H5 页面的问题，产生视觉上的白屏现象，原因是ViewRootImpl 在处理 View 绘制的时候，会通过一个布尔变量 `mDrawDuringWindowsAnimating` 来控制 Window 在执行动画的过程中是否允许进行绘制，该字段默认为 false，可以利用反射的方式去手动修改这个属性，避免这个白屏效果
```java

/**
     * 让 activity transition 动画过程中可以正常渲染页面
     */
    private void setDrawDuringWindowsAnimating(View view) {
        if (Build.VERSION.SDK_INT > Build.VERSION_CODES.M
                || Build.VERSION.SDK_INT < Build.VERSION_CODES.JELLY_BEAN_MR1) {
            // 1 android n以上  & android 4.1以下不存在此问题，无须处理
            return;
        }
        // 4.2不存在setDrawDuringWindowsAnimating，需要特殊处理
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.JELLY_BEAN_MR1) {
            handleDispatchDoneAnimating(view);
            return;
        }
        try {
            // 4.3及以上，反射setDrawDuringWindowsAnimating来实现动画过程中渲染
            ViewParent rootParent = view.getRootView().getParent();
            Method method = rootParent.getClass()
                    .getDeclaredMethod("setDrawDuringWindowsAnimating", boolean.class);
            method.setAccessible(true);
            method.invoke(rootParent, true);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    /**
     * android4.2可以反射handleDispatchDoneAnimating来解决
     */
    private void handleDispatchDoneAnimating(View paramView) {
        try {
            ViewParent localViewParent = paramView.getRootView().getParent();
            Class localClass = localViewParent.getClass();
            Method localMethod = localClass.getDeclaredMethod("handleDispatchDoneAnimating");
            localMethod.setAccessible(true);
            localMethod.invoke(localViewParent);
        } catch (Exception localException) {
            localException.printStackTrace();
        }
    }
```

### Http 缓存策略

见 WebView 缓存策略

### 拦截请求与共享缓存

可以通过 `shouldInterceptRequest` 方法来主动拦截并完成图片的加载操作，这样我们既可以使得两端的资源文件得以共享，也避免了多次 JS 调用带来的效率问题

### DNS优化

能保持客户端整体 API 地址、资源文件地址、WebView 线上地址的主域名都是一致情况下，如果已经解析过某域名，那么在下次使用时就可以直接去访问已知的 IP 地址，而不用先发起 DNS 再访问 IP 地址



### 总结

一个加载网页的过程中，native、网络、后端处理、CPU 都会参与，各自都有必要的工作和依赖关系；让它们相互并行处理而不是相互阻塞才可以让网页加载更快：

- WebView 初始化慢，可以在初始化同时先请求数据，让后端和网络不要闲着
- 后端处理慢，可以让服务器分 trunk 输出，在后端计算的同时前端也加载网络静态资源
- 脚本执行慢，就让脚本在最后运行，不阻塞页面解析
- 合理的预加载、预缓存可以让加载速度的瓶颈更小
- WebView 初始化慢，就随时初始化好一个 WebView 待用
- DNS 和链接慢，可以把框架代码拆分出来，在请求页面之前就执行好







## WebView 监控

### 调试 WebView 加载页面

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    WebView.setWebContentsDebuggingEnabled(true);
}
```

```
chrome://inspect
```



### 通过 Performance API 获取页面耗时



### 通过 UA 获取设备信息


通过 修改 UserAgent 在原有的基础上加上特定后缀，如设备的平台类型、app 版本、系统版本等，用于区分不同平台的特定处理。



## WebView加载网页过程

1. WebView 初始化，内核初始化
2. DNS 解析，建立连接
3. 请求 HTML 页面
4. 解析 HTML
5. 下载静态资源
6. 渲染



## PerformanceTiming API

### Resource Timing

![resource-timing-overview-1](../../web/resource-timing-overview-1.png)

### The PerformanceNavigationTiming attributes

![navigation-timing-attributes](../../web/navigation-timing-attributes.png)

| 属性                       | 含义                                                         |
| -------------------------- | ------------------------------------------------------------ |
| navigationStart            | 准备加载新页面的起始时间                                     |
| redirectStart              | 如果发生了HTTP重定向，并且从导航开始，中间的每次重定向，都和当前文档同域的话，就返回开始重定向的timing.fetchStart的值。其他情况，则返回0 |
| redirectEnd                | 如果发生了HTTP重定向，并且从导航开始，中间的每次重定向，都和当前文档同域的话，就返回最后一次重定向，接收到最后一个字节数据后的那个时间.其他情况则返回0 |
| fetchStart                 | 如果一个新的资源获取被发起，则 fetchStart必须返回用户代理开始检查其相关缓存的那个时间，其他情况则返回开始获取该资源的时间 |
| domainLookupStart          | 返回用户代理对当前文档所属域进行DNS查询开始的时间。如果此请求没有DNS查询过程，如长连接，资源cache,甚至是本地资源等。 那么就返回 fetchStart的值 |
| domainLookupEnd            | 返回用户代理对结束对当前文档所属域进行DNS查询的时间。如果此请求没有DNS查询过程，如长连接，资源cache，甚至是本地资源等。那么就返回 fetchStart的值 |
| connectStart               | 返回用户代理向服务器服务器请求文档，开始建立连接的那个时间，如果此连接是一个长连接，又或者直接从缓存中获取资源（即没有与服务器建立连接）。则返回domainLookupEnd的值 |
| (secureConnectionStart)    | 可选特性。用户代理如果没有对应的东东，就要把这个设置为undefined。如果有这个东东，并且是HTTPS协议，那么就要返回开始SSL握手的那个时间。 如果不是HTTPS， 那么就返回0 |
| connectEnd                 | 返回用户代理向服务器服务器请求文档，建立连接成功后的那个时间，如果此连接是一个长连接，又或者直接从缓存中获取资源（即没有与服务器建立连接）。则返回domainLookupEnd的值 |
| requestStart               | 返回从服务器、缓存、本地资源等，开始请求文档的时间           |
| responseStart              | 返回用户代理从服务器、缓存、本地资源中，接收到第一个字节数据的时间 |
| responseEnd                | 返回用户代理接收到最后一个字符的时间，和当前连接被关闭的时间中，更早的那个。同样，文档可能来自服务器、缓存、或本地资源 |
| domLoading                 | 返回用户代理把其文档的 "current document readiness" 设置为 "loading"的时候 |
| domInteractive             | 返回用户代理把其文档的 "current document readiness" 设置为 "interactive"的时候. |
| domContentLoadedEventStart | 返回文档发生 DOMContentLoaded事件的时间                      |
| domContentLoadedEventEnd   | 文档的DOMContentLoaded 事件的结束时间                        |
| domComplete                | 返回用户代理把其文档的 "current document readiness" 设置为 "complete"的时候 |
| loadEventStart             | 文档触发load事件的时间。如果load事件没有触发，那么该接口就返回0 |
| loadEventEnd               | 文档触发load事件结束后的时间。如果load事件没有触发，那么该接口就返回0 |



### Roadmap of Performance Monitoring

![W3C-WebPerf-deps-graph](../../web/W3C-WebPerf-deps-graph.png)





## Hybrid

Hybrid app 即混合模式移动应用

<img src="../../images/android/hybrid.png" alt="f64ad884041fc3f2226845a078504454_720w" style="zoom: 200%;" />

## JsBridge

### 背景

由于 H5 页面是内嵌到原生应用的 WebView 组件 WebKit 中，而手机浏览器 Javascript 引擎是在一个沙箱环境中运行，因此 JavaScript 的权限受到严格限制，比如没有本地文件读写权限、不能使用GPS、不能修改系统配置等。所以，如果JavaScript 要用到这些受限的能力时，就需要委托原生去实现，原生完成后，再将结果通知 JavaScript，因此，JavaScript 和原生之间就需要一个通信的桥梁，而这个桥梁本质上就是原生的 WebView 组件与Javascript 通信的通道。

### 简介

JsBridge 是 Native 与 JavaScript 之间的桥梁，核心是构建 Native 与 JavaScript 间的消息通信通道，而且是双向的通道。

简单来讲，JsBridge 主要用来给 JavaScript 提供调用 Native 功能的接口，让混合开发中的「前端部分」可以方便的使用地址位置、摄像头甚至支付等 Native 功能。

### 技术形式

移动端混合开发中的 JsBridge，主要被应用在两种形式的技术方案上：

- 基于 Web 的Hybrid 解决方案，例如各公司的 Hybrid 方案、微信浏览器
- 非基于 Web UI 但业务逻辑基于 JavaScript 的解决方案，例如 React-Native

### 通信原理

Hybrid 方案是基于 WebView 的，JavaScript 执行在 WebView 的 Webkit 引擎中。因此，Hybrid 方案中 JsBridge 的通信原理会具有一些 Web 特性。

- **JavaScript 调用 Native**
  - 注入 API
    注入 API 方式的主要原理是，通过 WebView 提供的接口，向 JavaScript 的 Context（window）中注入对象或者方法，让 JavaScript 调用时，直接执行相应的 Native 代码逻辑，达到 JavaScript 调用 Native 的目的。
  - URL SCHEME
    拦截 URL SCHEME 的主要流程是：Web 端通过某种方式（例如 iframe.src）发送 URL Scheme 请求，之后 Native 拦截到请求并根据 URL SCHEME（包括所带的参数）进行相关操作。
- **Native 调用 JavaScript**
  Native 调用 JavaScript，其实就是执行拼接 JavaScript 字符串，从外部调用 JavaScript 中的方法，因此 JavaScript 的方法必须在全局的 window 上。
- **Callback**
  RPC 框架的回调机制



