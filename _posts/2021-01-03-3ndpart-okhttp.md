---
layout: 
title: OkHttp Source Code Analysis
date: 2021-01-03 20:53 +0800
tags: [android,3ndpart]
---

OkHttp 源码详解

- 支持 HTTP2，允许对同一主机的所有请求共享一个套接字
- 使用连接池减少请求延迟（HTTP2不可用情况下）
- 使用 GZIP 压缩，减少下载大小
- 使用缓存避免网络重复请求

<!--more-->

## OkHttp 整体架构

![OkHttp整体架构](https://s2.loli.net/2022/07/19/okATFVOJwsWtX9f.jpg)

## OkHttp 请求流程

![](https://s2.loli.net/2022/07/19/lYTNo9k6V8azyiF.png)



## OkHttpClient

OkHttpClient 是整个库的核心管理类，所有的内部逻辑和对象归 OkHttpClient 来统一管理，它通过 Builder 构造器生成。

OkHttpClient 采用**Builder模式**包装了很多功能模块，采用**外观模式** 简洁的提供对外的API。

### Builder 内部参数

| 参数                                            | 作用             | 说明                                                         |
| ----------------------------------------------- | ---------------- | ------------------------------------------------------------ |
| Dispatcher dispatcher                           | 调度器           | 用于调度后台发起的网络请求，含后台请求数和单主机总请求数的控制。 |
| List<Protocol> protocols                        | 协议             | 支持的应用层协议，即HTTP1.1、HTTP/2等。                      |
| List<ConnectionSpec> connectionSpecs            | Socket设置       | 应用层支持的Socket设置，即使用明文传输（用于HTTP）还是某个版本的TLS（用于HTTPS）。 |
| List<Interceptor> interceptors                  | 自定义调度器     | 开发者自定义调度器                                           |
| List<Interceptor> networkInterceptors           | 自定义调度器     | 开发者自定义调度器，区别于`interceptors`，添加的位置不一样   |
| CookieJar cookieJar                             | cookie           | 管理Cookie的控制器，OkHttp提供了Cookie存取的判断支持，但没给出具体的实现，需要自行处理。 |
| Cache cache                                     | 缓存             | Cache存储的配置，默认没有，需要自行配置Cache存储位置及存储空间上限。 |
| HostnameVerifier hostnameVerifier               | 主机名校验       | 用于验证HTTPS握手过程中针对某个Host的Certificate Public Key Pinner（即把网站证书链中的每一个证书公钥直接拿过来提前配置进OkHttpClient中，以跳过本地根证书，直接从代码里进行认证。一般用于防止网络证书被人仿制。） |
| Authenticator authenticator                     | 本地身份认证     | 用于自动重新认证，在请求收到401状态码的响应是，直接调用authenticator，手动加入 Authenticator header 后自动重新发起请求。 |
| boolean followRedirects                         | 本地重定向       | 遇到重定向的要求时，是否自动follow。                         |
| boolean followSslRedirects                      | 安全套接层重定向 | 在重定向时，如果原先请求的是HTTP而重定向的目标是HTTPS（或者相反），是否依然自动follow（follow在HTTP和HTTPS之间切换的重定向）。 |
| boolean retryOnConnectionFailure                | 重试失败连接     | 在请求失败的时候是否自动重试。这种重试只适用于同一个域名的多个IP切换重试、Socket失效重试等情况。 |
| int connectTimeout                              | 连接超时         | 建立连接（TCP或TLS）的超时时间。                             |
| int readTimeout                                 | read超时         | 发起请求到读取响应数据的超时时间。                           |
| int writeTimeout                                | write超时        | 发起请求并被目标服务器接受的超时时间。                       |
| Proxy proxy                                     | 代理             |                                                              |
| ProxySelector proxySelector                     | 代理选择         |                                                              |
| InternalCache internalCache                     | 内部缓存         |                                                              |
| SocketFactory socketFactory                     | Socket工厂       |                                                              |
| SSLSocketFactory sslSocketFactory               | 安全套接层Socket | 用于HTTPS                                                    |
| CertificateChainCleaner certificateChainCleaner | 证书链           | 验证确认响应证书 适用 HTTPS 请求连接的主机名。               |
| CertificatePinner certificatePinner             | 证书链           |                                                              |
| Authenticator proxyAuthenticator                | 代理身份验证     |                                                              |
| ConnectionPool connectionPool                   | 连接池           | 复用连接                                                     |
| Dns dns                                         | 域名             |                                                              |

### Request

Request 是对网络请求参数的统一封装，例如：URL、请求方式、请求头等。

#### Headers

#### RequestBody

##### FormBody

`application/x-www-urlencoded`

##### MultipartBody

`multipart/form-data`



### Response 

即网络响应，包括响应行，响应头和响应体（允许为空），响应体只能被消费一次，且除响应体外的属性不可变。

#### Headers

#### ResponseBody

##### RealResponseBody

##### CacheResponseBody



### Request & Response

1. `FormBody` 和 `MultipartBody` 分别对应了两种不同的 [MIME](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types) 类型 （Multipurpose Internet Mail Extensions –– 多用途互联网邮件扩展类型）。
2. Request 和 Response 的 Header 标准不同，OkHttp 的 Request 和 Response 会把一些常用的 Header 信息提取出来，作为局部变量。比如：`contentType` `contentLength ` `code` `message` `cacheControl` `tag` ... 它们都是以 `name-value` 的形式储存在网络请求头部信息中。



### Call

Call 是一个接口，实现了对Request的封装，可以看做是一次请求任务，`Call`只能被执行一次，不论`execute`还是`enqueue`。

### RealCall

`RealCall`为具体的`Call`实现，每个 Request 最终将会被封装成一个 RealCall 对象，所以 RealCall 与 Request是一一对应的关系，最终会调用`RealCall.getResponseWithInterceptorChain()`发起请求，该方法将返回一个相应结果 Response；所以 RealCall 同时肩负了请求调度和责任链组织两大重任。


#### execute
同步请求

#### enqueue
异步请求



## Dispatcher

Dispatcher用于管理其对应OkHttpClient的所有请求，使用`enqueue()`异步请求时会将请求委托给Dispatcher处理。

## Interceptor

Interceptor 接口作为一个拦截器的抽象概念，被设计为责任链上的单位节点，用于观察、拦截、处理请求等，代表着网络请求的各个阶段，例如：添加Header、重定向、数据处理等。Interceptor之间互相独立，每个Interceptor只负责自己关注的任务，不与其他Interceptor接触，从而实现了各层的解耦。

![okhttp-interceptors](https://s2.loli.net/2022/07/19/iVSUeKEOqH742CZ.png)

### 1. client.interceptors

开发者自定义的拦截器，会按照开发者的要求，在所有其他Interceptor处理之前，进行最早的预处理工作，以及在收到Response之后，做最后的善后工作。如果你有统一的Header需要添加，可以在这里设置。

### 2. RetryAndFollowUpInterceptor

失败和重定向拦截器，负责在请求失败时的重试，以及重定向的自动后续请求。它的存在，可以让重试和重定向对开发者是无感知的。

### 3. BridgeInterceptor

负责把用户构造的请求转换为发送到服务器的请求、把服务器返回的响应转换为用户友好的响应，是从应用层到网络层的桥梁。

- Content-Length 的计算和添加
- Accept-Encoding: gzip
- gize 压缩数据的解包
- 添加 cookie
- 设置其它报头，如 User-Agent、Host、Keep-alive 等

### 4. CacheInterceptor

缓存相关的拦截器，负责 Cache 的处理，把它放在网络交互相关 `Interceptor`的前面的好处是，如果本地有了可用的Cache，一个请求可以在没有发生实质网络交互的情况下就返回缓存结果，而完全不需要开发者做出任何额外工作，让Cache 更加无感知。

![okhttp-cache-interceptor](https://s2.loli.net/2022/07/19/Sd23TN5Gy6ngEsK.webp)

### 5. ConnectInterceptor

负责和服务器建立连接，OkHttp 会创建出网络请求所需要的TCP连接（如果是HTTP），或者建立在TCP连接之上的TLS连接（如果是HTTPS），并且会创建出对应的`HttpCodec`对象（用于编码解码HTTP请求）。

CacheInterceptor 的缓存流程如下：

1. 通过 Request 尝试到 Cache 中获取缓存（前提是 OkHttpClient 配置了缓存，默认是不支持的）
2. 根据 `response` `time` `request` 创建一个缓存策略，用于判断怎么样那使用缓存
3. Response 的缓存步骤是先缓存 header，再缓存 body

### 6. client.networkInteceptors

开发者使用自定义的拦截器，它的行为逻辑和使用 `addInterceptor`创建的一样，但由于位置不同，所以这里创建的Interceptor 会看到每个请求和响应的数据（包括重定向以及重试的一些中间请求和响应），并且看到的是完整的原始数据，而不是没有加`Content-Length`的请求数据，或者 Body 还没被 gzip 解压的响应数据，多数情况，这个方法不需要被使用。

#### Interceptors 与 NetworkInterceptors 的区别

- 顺序不同，定位和职责也不同



### 7. CallServiceInterceptor

负责实质的请求和响应的I/O操作，即向服务器发送请求数据、从服务器读取响应数据，进行 HTTP 请求报文的封装与请求报文的解析。

### RealInterceptorChain





## OkHttp中运用到的设计模式

### 单例模式

OkHttpClient对象最好是共享的，建议用单例模式创建，因为每个OkHttpClient对象都管理自己独有的线程池和链接池，如果每个请求都创建一个OkHttpClient对象可能会导致内存溢出。

### 外观模式

OkHttpClient里组合了很多类的对象，其实OkHttp讲很多功能模块，都包装进了OkHttpClient中，让这个类单独提供对外的API，隐藏系统的复杂性，并提供一个客户端可简单访问系统的接口，这就是外观模式的设计。

### Builder模式

OkHttpClient很多属性比较复杂，而且客户的组合需求多样化，所以OkHttp使用建造者模式对多个简单的对象一步步构建成一个最终复杂的对象。

### 工厂方法模式

Call接口提供了内部接口Factory，用于将对象的创建延迟到该工厂类的子类中运行，从而实现动态的配置。

### 享元模式

在Dispatcher的线程池中，一个不限容量的线程池， 线程空闲时存活时间为60秒。线程池实现了对象复用，降低线程创建开销，从设计模式上来讲，使用了享元模式。

### 责任链模式

在OkHttp的拦截器模块的执行过程中用到，OkHttp的拦截器链中， 内置了5个默认的拦截器，分别用于重试、请求对象转换、缓存、链接、网络读写（责任链模式：为请求创建了一个接收者对象的链。这种模式给予请求的类型，对请求的发送者和接收者进行解耦；通常每个接收者都包含对另一个接收者的引用。如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推。）

### 策略模式

CacheInterceptor实现了数据的选择策略，来自网络还是本地，这个场景也是比较契合策略模式场景，CacheInterceptor需要一个策略提供者提供它一个策略（锦囊），CacheInterceptor根据这个策略去选择走网络数据还是本地缓存。



## HTTP 缓存

### 服务器支持缓存

如果服务器支持缓存，请求返回的 Response 会带有这样的 Header：Cache-Control、max-age=xxx，这种情况下我们只需要手动给 okhttp 设置缓存就可以让 okhttp 自定缓存 response 了。

### 服务器不支持缓存

如果服务器不支持缓存就可能没有指定的头部，这种情况下我们就需要使用 Interceptor 来重写 Response 的头部信息，从而让 okhttp 支持缓存。



## QA

1. OkHttp 如何处理网络缓存的？
   通过 OkHttpClient 配置缓存策略，根据 `request` `time` `response`





**参考文献：**

[OkHttp Source Code Analysis](https://betterprogramming.pub/okhttp-source-code-analysis-b9fe7d5b7b8a)

[使用 HTTP 缓存避免不必要的网络请求](https://web.dev/http-cache/)
