---
layout: 
title: AndroidManifest
date: 2016-09-27 11:46 +0800
tags: android
---

`AndroidManifest.xml`配置文件采用XML作为描述语言，每个XML标签都不同的含义，大部分的配置参数都放在标签的属性中。

<!--more-->



## `<manifest>`

AndroidManifest.xml 配置文件的根元素，必须包含一个`<application>`元素并且指定`xlmns:android`和`package`属性。

`xlmns:android`指定了Android的命名空间，默认情况下是`http://schemas.android.com/apk/res/android`

`package`是标准的应用包名，也是一个应用进程的默认名称，以本书微博应用实例中的包名为例，即`com.app.demos`就是一个标准的Java应用包名，我们为了避免命名空间的冲突，一般会以应用的域名来作为包名。

`android:versionCode`是给设备程序识别版本用的，必须是一个整数值代表app更新过多少次

`android:versionName`是给用户查看版本用的，需要具备一定的可读性，比如`1.0.0`这样的。`<manifest>`标签语法范例如下：
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="string"
    android:sharedUserId="string"
    android:sharedUserLabel="string resource" 
    android:versionCode="integer"
    android:versionName="string"
    android:installLocation=["auto" | "internalOnly" | "preferExternal"] >
... ...
</manifest>
```

## `<uses-permission>`

为了保证Android应用的安全性，应用框架制定了比较严格的权限系统，一个应用必须声明了正确的权限才可以使用相应的功能，例如我们需要让应用能够访问网络就需要配置`android.permission.INTERNET`

如果要使用设备的相机功能，则需要设置`android.permission.CAMERA`等。

`<uses-permission>`是最常使用的**权限设定标签**，通过设定`android:name`属性来声明相应的权限名。

根据应用的所需功能声明了对应的权限，相关代码如下：
```xml
<manifest ...>
... ...
    <!-- 网络相关功能 -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <!-- 读取电话状态 -->
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <!-- 通知相关功能 -->
    <uses-permission android:name="android.permission.VIBRATE" />
... ...
</manifest>
```

## `<permission>`
权限声明标签，定义了供给`<uses-permission>`使用的具体权限，通常情况下我们不需要为自己的应用程序声明某个权限，除非需要给其他应用程序提供可调用的代码或者数据，这个时候你才需要使用`<permission>`标签。该标签中提供了`android:name`权限名标签，权限图标`android:icon`以及权限描述`android:description`等属性，另外还可以和`<permission-group>`以及`<permission-tree>`配合使用来构造更有层次的、更有针对性权限系统。

`<permission>`标签语法范例如下：
```xml
<permission android:description="string resource"
    android:icon="drawable resource"
    android:label="string resource"
    android:name="string"
    android:permissionGroup="string"
    android:protectionLevel=["normal" | "dangerous" | "signature" | "signatureOrSystem"] />

```

## `<instrumentation>`
用于声明`Instrumentation`测试类来监控Android应用的行为并应用到相关的功能测试中，其中比较重要的属性有：
- 测试功能开关`android:functionalTest`
- profiling调试功能开关`android:handleProfiling`
- `测试用例目标对象android:targetPackage`

另外，我们需要注意的是`Instrumentation`对象是在应用程序的组件之前被实例化的，这点在组织测试逻辑的时候需要被考虑到。

`<instrumentation>`标签语法范例如下：
```xml
<instrumentation android:functionalTest=["true" | "false"]
    android:handleProfiling=["true" | "false"]
    android:icon="drawable resource"
    android:label="string resource"
    android:name="string"
    android:targetPackage="string" />

```

## `<uses-sdk>`
`<uses-sdk>`标签还可以指定最高版本和目标版本，语法范例如下：
```xml
<uses-sdk android:minSdkVersion="integer" 
    android:targetSdkVersion="integer"
    android:maxSdkVersion="integer" />
```

## `<uses-configuration>` `<uses-feature>`
这两个标签都是用于描述应用所需要的硬件和软件特性，以便防止应用在没有这些特性的设备上安装。
- `<uses-configuration>`标签中，比如有些设备带有`D-pad`或者`Trackball`这些特殊硬件，那么`android:reqFiveWayNav`属性就需要设置为`true`
- 如果有一些设备带有硬件键盘，`android:reqHardKeyboard`也需要被设置为`true`。
- 如果设备需要支持蓝牙，我们可以使用`<uses-feature android:name="android.hardware.bluetooth" />`来支持这个功能。

这两个标签主要用于支持一些特殊的设备中的应用，两个标签的语法范例分别如下：
```xml
<uses-configuration android:reqFiveWayNav=["true" | "false"] 
    android:reqHardKeyboard=["true" | "false"]
    android:reqKeyboardType=["undefined" | "nokeys" | "qwerty" | "twelvekey"]
    android:reqNavigation=["undefined" | "nonav" | "dpad" | "trackball" | "wheel"]
    android:reqTouchScreen=["undefined" | "notouch" | "stylus" | "finger"] />

<uses-feature android:name="string"
    android:required=["true" | "false"]
    android:glEsVersion="integer" />
```

## `<uses-library>`
用于指定Android应用可使用的用户库，除了系统自带的android.app、android.content、android.view和android.widget这些默认类库之外，有些应用可能还需要一些其他的Java类库作为支持，这种情况下我们就可以使用`<uses-library>`标签让ClassLoader加载其类库供Android应用运行时用。`<uses-library>`标签的用法很简单，以下是语法范例。

```xml
<uses-library android:name="string"
    android:required=["true" | "false"] />
```

>小贴士：
>当运行Java程序时，首先运行JVM（Java虚拟机），然后再把Java类加载到JVM里头运行，负责加载Java类的这部分就叫做ClassLoader。当然，ClassLoader是由多个部分构成的，每个部分都负责相应的加载工作。当运行一个程序的时候，JVM启动，运行BootstrapClassLoader，该ClassLoader加载java核心API（ExtClassLoader和AppClassLoader也在此时被加载），然后调用ExtClassLoader加载扩展API，最后AppClassLoader加载CLASSPATH目录下定义的Class，这就是一个Java程序最基本的加载流程。

## `<supports-screens>`
对于一些应用或者游戏来说，只能支持某些屏幕大小的设备或者在某些设备中的效果比较好，我们就会使用<supports-screens>标签来指定支持的屏幕特征。其中比较重要的属性包括：屏幕自适应属性android:resizeable，小屏（android:smallScreens）、中屏（android:normalScreens）、大屏（android:largeScreens）和特大屏（android:xlargeScreens）支持属性，按屏幕渲染图像属性android:anyDensity以及最小屏幕宽度属性android:requiresSmallestWidthDp等。

`<supports-screens>`标签的语法范例如下。
```xml
<supports-screens android:resizeable=["true"| "false"]
    android:smallScreens=["true" | "false"]
    android:normalScreens=["true" | "false"]
    android:largeScreens=["true" | "false"]
    android:xlargeScreens=["true" | "false"]
    android:anyDensity=["true" | "false"]
    android:requiresSmallestWidthDp="integer"
    android:compatibleWidthLimitDp="integer"
    android:largestWidthLimitDp="integer"/>

<application>
```

应用配置的根元素，位于`<manifest>`下层，包含所有与应用有关配置的元素，其属性可以作为子元素的默认属性，常用的属性包括：应用名android:label，应用图标android:icon，应用主题android:theme等。当然，`<application>`标签还提供了其他丰富的配置属性，由于篇幅原因就不列举了，大家可以打开Android SDK文档来进一步学习，

以下是语法范例：
```xml
<application android:allowTaskReparenting=["true" | "false"]
    android:backupAgent="string"
    android:debuggable=["true" | "false"]
    android:description="string resource"
    android:enabled=["true" | "false"]
    android:hasCode=["true" | "false"]
    android:hardwareAccelerated=["true" | "false"]
    android:icon="drawable resource"
    android:killAfterRestore=["true" | "false"]
    android:label="string resource"
    android:logo="drawable resource"
    android:manageSpaceActivity="string"
    android:name="string"
    android:permission="string"
    android:persistent=["true" | "false"]
    android:process="string"
    android:restoreAnyVersion=["true" | "false"]
    android:taskAffinity="string"
    android:theme="resource or theme" >
... ...
</application>
```


## `<activity>`
Activity活动组件（即界面控制器组件）的声明标签，Android应用中的每一个Activity都必须在`AndroidManifest.xml`配置文件中声明，否则系统将不识别也不执行该Activity。

`<activity>`标签中常用的属性有：
- Activity对应类名`android:name`
- 对应主题`android:theme`
- 加载模式`android:launchMode`
- 键盘交互模式`android:windowSoftInputMode`

另外，`<activity>`标签还可以包含用于消息过滤的`<intent-filter>`元素，当然还有可用于存储预定义数据的`<meta-data>`元素。

以下是`<activity>`标签的语法范例：

```xml
<activity android:allowEmbedded=["true" | "false"]
          android:allowTaskReparenting=["true" | "false"]
          android:alwaysRetainTaskState=["true" | "false"]
          android:autoRemoveFromRecents=["true" | "false"]
          android:banner="drawable resource"
          android:clearTaskOnLaunch=["true" | "false"]
          android:colorMode=[ "hdr" | "wideColorGamut"]
          android:configChanges=["mcc", "mnc", "locale",
                                 "touchscreen", "keyboard", "keyboardHidden",
                                 "navigation", "screenLayout", "fontScale",
                                 "uiMode", "orientation", "density",
                                 "screenSize", "smallestScreenSize"]
          android:directBootAware=["true" | "false"]
          android:documentLaunchMode=["intoExisting" | "always" |
                                  "none" | "never"]
          android:enabled=["true" | "false"]
          android:excludeFromRecents=["true" | "false"]
          android:exported=["true" | "false"]
          android:finishOnTaskLaunch=["true" | "false"]
          android:hardwareAccelerated=["true" | "false"]
          android:icon="drawable resource"
          android:immersive=["true" | "false"]
          android:label="string resource"
          android:launchMode=["standard" | "singleTop" |
                              "singleTask" | "singleInstance"]
          android:lockTaskMode=["normal" | "never" |
                              "if_whitelisted" | "always"]
          android:maxRecents="integer"
          android:maxAspectRatio="float"
          android:multiprocess=["true" | "false"]
          android:name="string"
          android:noHistory=["true" | "false"]  
          android:parentActivityName="string" 
          android:persistableMode=["persistRootOnly" | 
                                   "persistAcrossReboots" | "persistNever"]
          android:permission="string"
          android:process="string"
          android:relinquishTaskIdentity=["true" | "false"]
          android:resizeableActivity=["true" | "false"]
          android:screenOrientation=["unspecified" | "behind" |
                                     "landscape" | "portrait" |
                                     "reverseLandscape" | "reversePortrait" |
                                     "sensorLandscape" | "sensorPortrait" |
                                     "userLandscape" | "userPortrait" |
                                     "sensor" | "fullSensor" | "nosensor" |
                                     "user" | "fullUser" | "locked"]
          android:showForAllUsers=["true" | "false"]
          android:stateNotNeeded=["true" | "false"]
          android:supportsPictureInPicture=["true" | "false"]
          android:taskAffinity="string"
          android:theme="resource or theme"
          android:uiOptions=["none" | "splitActionBarWhenNarrow"]
          android:windowSoftInputMode=["stateUnspecified",
                                       "stateUnchanged", "stateHidden",
                                       "stateAlwaysHidden", "stateVisible",
                                       "stateAlwaysVisible", "adjustUnspecified",
                                       "adjustResize", "adjustPan"] >   
    . . .
</activity>
```

Activity组件别名的声明标签，简单来说就是Activity的快捷方式，属性`android:targetActivity`表示的就是其相关的Activity名，当然必须是前面已经声明过的Activity。

除此之外，其他比较常见的属性有：
- Activity别名名称`android:name`
- 别名开关`android:enabled`
- 权限控制`android:permission`

另外，我们还需要注意的是，Activity别名也是一个独立的Activity，可以拥有自己的`<intent-filter>`和`<meta-data>`元素。

其语法范例如下：
```xml
<activity-alias android:enabled=["true" | "false"]
    android:exported=["true" | "false"]
    android:icon="drawable resource"
    android:label="string resource"
    android:name="string"
    android:permission="string"
    android:targetActivity="string" >
</activity-alias>
```



## `<intent-filter>`

用于 Intent 消息过滤器的声明，`<intent-filter>`元素可以放在`<activity>`、`<activity-alias>`、`<service>`和`<receiver>`元素标签中。

下面是标准`<intent-filter>`元素标签的语法范例：

```xml
<intent-filter android:icon="drawable resource"
    android:label="string resource"
    android:priority="integer" >
    <action android:name="string" />
    <category android:name="string" />
    <data android:host="string"
        android:mimeType="string"
        android:path="string"
        android:pathPattern="string"
        android:pathPrefix="string"
        android:port="string"
        android:scheme="string" />
</intent-filter>
```

另外，我们还知道Intent消息还包含有名称、动作、数据、类别等几个重要属性。

### `<action>`

`<intent-filter>`中必须包含有`<action>`元素，即用于描述具体消息的名称。

### `<category>`

`<category>`标签则用于表示能处理消息组件的类别，即该 Action 所符合的类别

### `<data>`

`<data>`元素则用于描述消息需要处理的数据格式，还可以使用正则表达式来限定数据来源。



## `<meta-data>`

**用于存储预定义数据**，和`<intent-filter>`类似，`<meta-data>`也可以放在`<activity>`、`<activity-alias>`、`<service>`和`<receiver>`这四个元素标签中。**Meta数据一般会以键值对的形式出现**，个数没有限制，而这些数据都将被放到一个Bundle对象中，程序中我们则可以使用`ActivityInfo`、`ServiceInfo`甚至`ApplicationInfo`对象的`metaData`属性中读取。假设我们在一个Activity中定义了一个`<meta-data>`元素。

相关示例用法如下：
```xml
<activity>
    <meta-data android:name="testData" android:value="Test Meta Data"></meta-data>
</activity>

ActivityInfo info = this.getPackageManager()
    .getActivityInfo(getComponentName(), PackageManager.GET_META_DATA);
String testData = info.metaData.getString("testData");
System.out.println("testData:" + testData);
```

## `<service>`

Service服务组件的声明标签，**用于定义与描述一个具体的Android服务**。主要属性有：
- 服务类名`android:name`
- 服务图标`android:icon`
- 服务描述`android:label`
- 服务开关`android:enabled`

以下是`<service>`标签的语法范例：
```xml
<service android:enabled=["true" | "false"]
    android:exported=["true" | "false"]
    android:icon="drawable resource"
    android:label="string resource"
    android:name="string"
    android:permission="string"
    android:process="string" >
</service>
```

## `<receiver>`

用于定义与描述 BroadcastReceiver 广播接收器。其主要属性和`<service>`标签有些类似：

- 接收器类名`android:name`
- 接收器图标`android:icon`
- 接收器描述`android:label`
- 接收器开关`android:enabled`

```xml
<receiver android:enabled=["true" | "false"]
    android:exported=["true" | "false"]
    android:icon="drawable resource"
    android:label="string resource"
    android:name="string"
    android:permission="string"
    android:process="string" >
```



## `<provider>` `<grant-uri-permission>`

`<provider>`标签除了和其他组件相同的`android:name`、`android:icon`和`android:label`等基础属性之外，还提供了用于支持其功能的特殊属性，如：

- 内容提供者标识名称`android:authorities`
- 对指定URI授予权限标识`android:grantUriPermission`
- 具体的读、写权限，即`android:readPermission`和`android:writePermission`等。

以下是`<provider>`标签的语法范例：

```xml
<provider android:authorities="list"
    android:enabled=["true" | "false"]
    android:exported=["true" | "false"]
    android:grantUriPermissions=["true" | "false"]
    android:icon="drawable resource"
    android:initOrder="integer"
    android:label="string resource"
    android:multiprocess=["true" | "false"]
    android:name="string"
    android:permission="string"
    android:process="string"
    android:readPermission="string"
    android:syncable=["true" | "false"]
    android:writePermission="string" >
</provider>
```

