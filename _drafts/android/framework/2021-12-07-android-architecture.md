## 组件化

 组件化就是把App拆成不同功能模块，把模块解耦，行程独立组件，组件间代码不依赖，提供给宿主调用。

### 适用范围

适合于项目大，但功能相对集中，比如金融类App里只包含金融的功能，金融功能有会有借贷、理财、线下交易，可以把这些模块抽成单独的组件。

### 单体项目的缺点

- 无论分包做得再好，随着项目的增大，项目会逐渐失去层次感，别人来接收会很吃力
- 在调试阶段每次修改代码都需要build整个项目，这样显的很不合理
- 多人联合开发在版本管理中很容易出现冲突和代码覆盖问题



## 插件化

App的部分功能模块在打包时并不以传统的方式打包进apk中，而是以另一种形式二次封装进apk内部，或者放在网络上适时下载，在需要的时候动态对这些功能模块进行加载，称之为插件化。

### 插件化的优势

- 动态发布（热更新/热修复）
- 模块间高度解耦，没有交叉依赖

### 插件化的弊端

- 插件多的时候管理起来不方便

### 插件化原理

动态加载，通过自定义ClassLoader来加载新的dex文件，从而让开发原本没有的类可以被使用



### 博客

[Android动态加载dex技术初探](http://blog.csdn.net/u013478336/article/details/50734108)

#### Android动态加载dex技术初探

Android使用Dalvik虚拟机加载可执行程序，所以不能直接加载基于class的jar，而是需要将class转化为dex字节码。

Android支持动态加载的两种方式是：DexClassLoader和PathClassLoader，DexClassLoader可加载jar/apk/dex，且支持从SD卡加载；PathClassLoader据说只能加载已经安装在Android系统内APK文件。

### 

#### Android插件化基础

Android简单来说就是如下操作：

* 开发者将插件代码封装成Jar或者APK
* 宿主下载或者从本地加载Jar或者APK到宿主中
* 将宿主调用插件中的算法或者Android特定的Class（如Activity）

#### 插件化开发—动态加载技术加载已安装和未安装的apk

[http://blog.csdn.net/u010687392/article/details/47121729?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io](http://blog.csdn.net/u010687392/article/details/47121729?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

##### 为什么引入动态加载技术？

* 一个应用程序dex文件的方法数最大不能超过65536个
* 可以让应用程序实现插件化、插拔式结构，对后期维护有益

##### 什么是动态加载技术？

动态加载技术就是使用类加载器加载相应的apk、dex、jar(必须含有dex文件)，再通过反射获得该apk、dex、jar内部的资源（class、图片、color等等）进而供宿主app使用。

##### 关于动态加载使用的类加载器

* PathClassLoader - 只能加载已经安装的apk，即/data/app目录下的apk。
* DexClassLoader  - 能加载手机中未安装的apk、jar、dex，只要能在找到对应的路径。



#### 插件化技术学习

原因：各大厂商都碰到了AndroidNative平台的瓶颈：

1. 从技术上讲，业务逻辑的复杂代码急剧膨胀，各大厂商陆续触到65535方法数的天花板；同时，对模块热更新提出了更高的要求。
2. 在业务层面上，功能模块的解耦以及维护团队的分离也是大势所趋。

插件化技术主要解决两个问题：

1. 代码加载
2. 资源加载

##### 代码加载

类的加载可以使用Java的ClassLoader机制，还需要组件生命周期管理。

##### 资源加载

用AssetManager的隐藏方法addAssetPath。



#### Android插件化原理解析——Hook机制之动态代理

使用代理机制进行API Hook进而达到方法增强。

静态代理

动态代理：可以简单理解为JVM可以在运行时帮我们动态生成一系列的代理类。

##### 代理Hook

如果我们自己创建代理对象，然后把原始对象替换为我们的代理对象，就可以在这个代理对象中为所欲为了；修改参数，替换返回值，称之为Hook。

整个Hook过程简要总结如下：

1. 寻找Hook点，原则是静态变量或者单例对象，尽量Hook public的对象和方法，非public不保证每个版本都一样，需要适配。
2. 选择合适的代理方式，如果是接口可以用动态代理；如果是类可以手动写代理也可以使用cglib。
3. 偷梁换柱－用代理对象替换原始对象

##### Android插件化原理解析——Hook机制之Binder Hook 



