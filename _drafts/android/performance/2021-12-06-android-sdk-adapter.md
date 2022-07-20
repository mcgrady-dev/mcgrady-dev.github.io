Android 系统版本适配指南，以下列举一些主要的行为变更。



## Android 12.0 (S) 31

### 用户体验

#### OverScroll（拉伸滚动效果）



#### Splash Screen（应用启动画面）

<img src="../framwork/android-splash-screen-example.gif" alt="android-splash-screen-example" style="zoom:67%;" />

![android-splash-screen-composition](../../images/android/android-splash-screen-composition.png)

如果您之前在 Android 11 或更低版本中实现了自定义启动画面，则需要将您的应用迁移到 `SplashScreen` API，以确保它从 Android 12 开始正确显示。如果不迁移您的应用，则可能会导致应用启动体验变差或出乎预期。

如需了解相关说明，请参阅[将现有的启动画面实现迁移到 Android 12](https://developer.android.com/guide/topics/ui/splash-screen/migrate)。

此外，从 Android 12 开始，在所有应用的[冷启动](https://developer.android.com/topic/performance/vitals/launch-time#cold)和[温启动](https://developer.android.com/topic/performance/vitals/launch-time#warm)期间，系统始终会应用新的 [Android 系统默认启动画面](https://developer.android.com/about/versions/12/features/splash-screen)。 默认情况下，此系统默认启动画面由应用的启动器图标元素和主题的 [`windowBackground`](https://developer.android.com/topic/performance/vitals/launch-time#solutions-3)（如果是单色）构成。

如需了解详情，请参阅[启动画面开发者指南](https://developer.android.com/guide/topics/ui/splash-screen)。





#### 自定义通知

标准模板中的自定义通知：

![android-customizable-area](../../images/android/android-customizable-area.png)

以下示例了再收起和展开状态下呈现的自定义通知：

<img src="../../images/android/android-custom-collapsed-view.png" alt="android-custom-collapsed-view" style="zoom:50%;" />

<img src="../../images/android/android-custom-expanded-view.png" alt="android-custom-expanded-view" style="zoom:50%;" />



### 安全和隐私设置

#### 麦克风和摄像头切换开关

在搭载 Android 12 或更高版本的受支持设备上，用户可以通过按一个切换开关选项，为设备上的所有应用启用和停用摄像头和麦克风使用权限。用户可以从[快捷设置](https://support.google.com/android/answer/9083864)访问可切换的选项（如图 1 所示），也可以从系统设置中的“隐私设置”屏幕访问。

详细了解这些[切换开关](https://developer.android.com/training/permissions/explaining-access#toggles)以及如何检查您的应用是否遵循了关于 [`CAMERA`](https://developer.android.com/reference/android/Manifest.permission#CAMERA) 和 [`RECORD_AUDIO`](https://developer.android.com/reference/android/Manifest.permission#RECORD_AUDIO) 权限的最佳实践。

#### 麦克风和摄像头指示标志

在搭载 Android 12 或更高版本的设备上，当应用使用麦克风或相机时，图标会出现在状态栏中。

详细了解这些[指标](https://developer.android.com/training/permissions/explaining-access#indicators)以及如何检查您的应用是否遵循了关于 [`CAMERA`](https://developer.android.com/reference/android/Manifest.permission#CAMERA) 和 [`RECORD_AUDIO`](https://developer.android.com/reference/android/Manifest.permission#RECORD_AUDIO) 权限的最佳实践。

#### 更安全的组件导出 android:exported

一般情况下，使用了 `intent-filter` ，`exported` 默认为 `true` ，不能设置为 `false` ，相反没有 `intent-filter` ，则 exported 默认为 `false` ，不能设置为 `true` 。

#### 大致位置

使用 `targetSDK` 为 31 的 `App` ，用户可以请求应用只能访问大致位置信息。当 `App` 同时请求 `ACCESS_FINE_LOCATION` 和 `ACCES_COARSE_LOCATION` 权限时，系统会为用户提供以下权限选择对话框：

![approximate-location-full-prompt](../../images/android/android-approximate-location-full-prompt.png)



### PendingIntent

在 Android 12 之前，默认创建的 PendingIntent 是可变的，因此其它恶意应用程序可能会拦截，重定向或修改此 Intent 。

为了安全性 Android 12 中需要显示声明是否可变，否则抛出 `IllegalArgumentException` 异常。

- 可变：PendingIntent.FLAG_IMMUTABLE
- 不可变： PendingIntent.FLAG_MUTABLE



### 性能

#### 前台服务限制

以Android 12或更高版本为目标平台的应用无法在后台运行时启动前台服务，少数特殊情况除外。如果应用尝试在后台运行时启动前台服务，则会引发异常。

#### 受限应用待机模式存储分区

Android 11（API 级别 30）引入了[受限存储分区](https://developer.android.com/topic/performance/appstandby#restricted-bucket)作为应用待机模式存储分区。从 Android 12 开始，此存储分区默认处于活跃状态。在所有存储分区中，受限存储分区的优先级最低（限制最高）。存储分区按优先级从高到低的顺序排列如下：

1. 活跃：应用目前正在使用中，或者最近刚刚使用过。
2. 工作集：会定期使用应用。
3. 常用：会经常使用应用，但不是每天都使用。
4. 极少使用：不经常使用应用。
5. 受限：应用会消耗大量的系统资源，或表现出不良行为。

除了使用模式之外，系统还会考虑应用的行为，以决定是否要将您的应用放在受限存储分区中。

如果您的应用更负责地使用系统资源，就不太可能被放在受限存储分区中。此外，如果用户直接与您的应用互动，系统会将其放在一个限制较少的存储分区中。

想获取APP当前的限制域可以使用`getAppStandbyBucket`方法



### Activity 生命周期

按下“返回”按钮不再 finish 根启动器 activity，系统会将 activity 及其任务移到后台，此变更意味着使用“返回”按钮退出应用的用户可以更快地从温状态恢复应用，而不必从冷状态完全重启应用。



## Android 11.0 (R) 30

### 权限变化

#### 强制执行分区存储

Android 11进一步增强了平台功能，为外部存储设备上的应用和用户数据提供了更好的保护。

在 Android 11 上运行但以 Android 10（API 级别 29）为目标平台的应用仍可请求 [`requestLegacyExternalStorage`](https://developer.android.com/reference/android/R.attr#requestLegacyExternalStorage) 属性。应用可以利用此标记[暂时停用与分区存储相关的变更](https://developer.android.com/training/data-storage/use-cases#opt-out-scoped-storage)，例如授予对不同目录和不同类型的媒体文件的访问权限。当您将应用更新为以 Android 11 为目标平台后，系统会忽略 `requestLegacyExternalStorage` 标记。



#### 单次授权

让用户可以选择授予更多对位置信息、麦克风和摄像头的临时访问权限，具体时间取决于应用的行为和用户的操作：

- 当Activity可见时，应用可以访问相关数据。
- 当应用转为后台运行时，应用可以在短时间内继续访问相关数据。
- 当应用的Activity可见时启动了一项前台服务，并且用户随后将应用转到后台，那么应用可以继续访问相关数据，知道该前台服务停止。
- 当用户测西奥单次授权（例如在系统设置中撤销），无论如何，应用都无法访问相关数据。与任何权限一样，如果用户撤销了单次授权，应用进程将会终止。



#### 权限对话框的可见性

一再拒绝某项权限表示用户希望“不再询问”

#### 自动重置权限

如果用户几个月未与应用互动，系统会自动重置应用的敏感权限

#### 在后台访问位置信息的权限

用户必须转到系统设置，才能向应用授予在后台访问位置信息的权限



### 应用打包和安装

#### 现在需要 APK 签名方案 v2

在Android 11上，仅通过v1签名的应用无法在Android 11的设备上安装或更新。必须使用v2或者更高版本进行签名。

同时Android 11添加了对APK v4 签名方案的支持。

#### 安装外部来源应用需要重启APP

Android 11 让应用安装APK的权限变得不再方便，需要重新启动相关应用才能生效。该行为与强制分区存储(Scoped Storage)有关。由于每个应用程序只需要授予权限一次，因此理想情况下，每个应用程序只需要强行停止一次。



### 其他行为变更

#### 自定义View的Toast

targetSdkVersion >= 30 的应用，从后台发送自定义View的Toast消息系统会进行屏蔽（前台使用不受影响）。Toast响应的`setView` `getView`也被废弃不建议使用。



#### 状态栏高度

在Android 11上，targetSdkVersion = 30 时获取状态栏高度为0，低于30获取值正常，因此需要使用`WindowMetrics`适配

```java
public static int getStatusBarHeight(Context context) {

 	if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
        WindowManager wm = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
        WindowMetrics windowMetrics = wm.getCurrentWindowMetrics();
        WindowInsets windowInsets = windowMetrics.getWindowInsets();
        Insets insets = windowInsets.getInsetsIgnoringVisibility(WindowInsets.Type.navigationBars() | WindowInsets.Type.displayCutout());
        return insets.top;
    }
}
```

`WindowMetrics`是Android 11新增的类，用于获取窗口边界，同时可以用来获取导航栏高度。



## Android 10.0 (Q) 29

### Scoped Storage (分区存储)

#### 背景

在Android 10之前的版本，我们在做文件操作时都会申请存储空间的读写权限，但这些权限完全被滥用，造成的问题就是手机的存储空间中充斥着大量不明作用的文件，并且应用卸载后也没有被删除。为了解决这个问题，Android 10引入了分区存储的改变，通过添加外部存储访问限制来实现更好的文件管理。



#### 存储基本知识

![](../../images/android/android-storage.png)

开始之前先确认下目前有已有的存储目录：

##### 内部存储

`/data` 目录，一般我们使用 `getFilesDir()` 或 `getCacheDir()` 方法获取本应用的内部存储路径，读写该路径下的文件不需要申请存储空间读写权限，且卸载应用时自动删除。

##### 外部存储

`/storage`或`/mnt`目录。一般用`getExernalStorageDirectory()`获取路径。

- 私有存储(Private Storage)：每个应用在都拥有自己的私有目录，其它应用看不到，彼此也无法访问到该目录。
- 内部存储私有目录 (/data/data/packageName)
- 外部存储私有目录 (/sdcard/Android/data/packageName)
- 共享存储 (Shared Storage)：存储其它应用可访问文件，包含媒体、文档以及其它文件，对应设备DCIM、Pictures、Alarms、Music、Notifications、Podcasts、Ringtones、Movies、Download等目录。
- 外部存储：`Environment.getExternalStorageDirectory()` 获取sdcard下的任意文件夹，在SDK29以上已经过期、失效。

#### MediaStore

共享存储空间，指所有App都可以访问的的媒体库，其中包含图片、音视频、下载的等文件，即使应用卸载了，这些文件仍会留在用户的设备上。



#### Android版本迭代变化

Android 10 之后，只能操作本身内部存储私有目录、外部存储私有目录、共享存储，但依然可以通过 `android:requestLegacyExternalStorage="true"`来设置不启用分区存储，但Android 11 之后，就强制性的只能操作规定的目录，这个时候依然有个兼容配置设置，`android:preserveLegacyExternalStorage="true"`使得更新至Android 11后，依然不启用分区存储。

#### 多情况下解决方案

| FROM                   | TO                    | OPTION                                                       | RESULT                 |
| ---------------------- | --------------------- | ------------------------------------------------------------ | ---------------------- |
| targetSdkVersion = 28  |                       |                                                              | 正常读写所有文件       |
| targetSdkVersion < 29  | targetSdkVersion = 29 | 覆盖安装                                                     | 正常读写所有文件       |
| targetSdkVersion = 29  |                       | 卸载旧应用                                                   | 读写外部存储，程序崩溃 |
| targetSdkVersion < 30  | targetSdkVersion = 30 | 覆盖安装<br />增加<br />`android:preserveLegacyExternalStorage="true"` | 正常读写所有文件       |
| targetSdkVersion  = 30 |                       | 卸载旧应用                                                   | 读写外部存储，程序崩溃 |



### 权限变化

#### 1. 后台运行时访问设备位置信息需要权限

Android 10 引入了 `ACCESS_BACKGROUND_LOCATION`权限，该权限允许应用程序在后台访问位置，申请此权限同时还必须申请`ACCESS_FINE_LICATION`或`ACCESS_CORASE_LOCATION`权限。

在 Android 10设备上，如果应用`targetSdkVersion < 29`，则在请求`ACCESS_FINE_LOCATION`或`ACCESS_COARSE_LOCATION`权限时，系统会自动同时请求`ACCESS_BACKGORUND_LOCATION`；如果`targetSdkVersion >= 29`，则请求`ACCESS_FINE_LOCATION`或`ACCESS_COARSE_LOCATION`权限达标在前台时拥有访问设备信息的权限。



设备升级至 Android 10 之后位置权限状态变化

| 目标平台     | 是否授予了粗略或精确位置权限 | 是否定义了后台权限         | 更新后的默认权限状态 |
| ------------ | ---------------------------- | -------------------------- | -------------------- |
| Android 10   | 是                           | 是                         | 前台和后台访问权限   |
| Android 10   | 是                           | 否                         | 仅前台访问权限       |
| Android 10   | 否                           | （被系统忽略）             | 无访问权限           |
| <= Android 9 | 是                           | 在设备升级时由系统自动添加 | 前台和后台访问权限   |
| <= Android 9 | 否                           | （被系统忽略）             | 无访问权限           |



**官方建议方案**：官方不推荐使用后台访问位置信息，官方建议使用前台服务来实现

1. 首先在清单中对应的`service`中添加`android:foregroundServiceType="location"`

   ```xml
   <service
   	android:name="GpsService"
   	android:foregroundServiceType="location" ... >
   	...
   </service>
   ```

2. 启动前台服务前检查是否具有前台的访问权限

   ```java
   boolean granted = ActivityCompat.checkSelfPermission(this, Mainifest.permission.ACCESS_COARSE_LOCATION) == PackageManager.PERMISSION_GRANTED;
   if (granted) {
     //启动前台服务
   } else {
     //请求前台访问位置权限
   }
   ```



#### 2. 一些电话、蓝牙、WLAN的API需要精确位置权限

下面列举了Android 10中必须具有 ACCESS_FINE_LOCATION 权限才能使用类和方法:

**电话**

- TelephonyManager
  - `getCellLocation()`
  - `getAllCellInfo()`
  - `requestNetworkScan()`
  - `requestCellInfoUpdate()`
  - `getAvailableNetworks()`
  - `getServiceState()`
- TelephonyScanManager
  - `requestNetworkScan()`
- TelephonyScanManager.NetworkScanCallback
  - `onResults()`
- PhoneStateListener
  - `onCellLocationChanged()`
  - `onCellInfoChanged()`
  - `onServiceStateChanged()`

**WLAN**

- WifiManager
  - `startScan()`
  - `getScanResults()`
  - `getConnectionInfo()`
  - `getConfiguredNetworks()`
- WifiAwareManager
- WifiP2pManager
- WifiRttManager

**蓝牙**

- BluetoothAdapter
  - `startDiscovery()`
  - `startLeScan()`
- BluetoothAdapter.LeScanCallback
- BluetoothLeScanner
  - `startScan()`

### 后台启动Activity的限制

应用处于后台时，无法启动Activity。

### 标识符和数据

#### 对不可重置的设备标识符实施了限制

从Android 10开始，应用必须具有`READ_PRIVILEGED_PHONE_STATE`权限才能正常使用一下方法：

- Build
  - [getSerial()](https://developer.android.google.cn/reference/android/os/Build#getSerial())
- TelephonyManager
  - [getImei()](https://developer.android.google.cn/reference/android/telephony/TelephonyManager#getImei(int))
  - [getDeviceId()](https://developer.android.google.cn/reference/android/telephony/TelephonyManager#getDeviceId(int))
  - [getMeid()](https://developer.android.google.cn/reference/android/telephony/TelephonyManager#getMeid(int))
  - [getSimSerialNumber()](https://developer.android.google.cn/reference/android/telephony/TelephonyManager#getSimSerialNumber())
  - [getSubscriberId()](https://developer.android.google.cn/reference/android/telephony/TelephonyManager#getSubscriberId())

如果没有该权限，却是用了以上方法，则返回的结果会因目标SDK版本而异：

-  targetSdkVersion >= Android 10，则会发生SecurityException
- targetSdkVersion <= Android 9，且具有`READ_PHONE_STATE`权限，则相应的方法会返回null或占位符数据，否则会发生SecurityException。

## Android 9.0 (Pie) 28

### 刘海屏API支持

### 移除对 `Build.serial`的直接访问

现在，需要 `Build.serial` 标识符的应用必须请求 `READ_PHONE_STATE` 权限，然后使用 Android P 中新增的新 `Build.getSerial()` 函数

### ImageDecoder

引入`ImageDecoder`类，可提供现代化的图像解码方法，取代`BitmapFactory`和`BitmapFactory.Options` API。

### Battery Improvements （功耗解决方案）

### Http请求失败

默认情况下启用网络传输层安全协议（TLS），已停用明文支持，要求使用https请求。

``` 
java.net.UnknownServiceException: CLEARTEXT communication to xxxx not permitted by network security policy
```

解决方法：

在res目录下添加 `network_security_config.xml`网络安全配置文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true" />
</network-security-config>
```

AndroidManifest.xml中添加

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest ... >
    <application android:networkSecurityConfig="@xml/network_security_config">
          ...
    </application>
</manifest>
```

以上是一种简单粗暴的配置方法，为了安全灵活，可以这样执行支持的http域名

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
<!-- Android 9.0 上部分域名时使用 http -->
<domain-config cleartextTrafficPermitted="true">
    <domain includeSubdomains="true">secure.example.com</domain>
    <domain includeSubdomains="true">cdn.example1.com</domain>
</domain-config>
</network-security-config>
```



### Apache HTTP 客户端弃用

从 Android 9.0开始，默认情况下该库从bootclasspath中移除。如果想要继续使用Apache HTTP，需要在应用的AndroidManifest。xml中添加

```xml
<uses-library android:name="org.apache.http.legacy" android:required="false"/>
```

### 前台服务

Android 9.0 要求，创建一个前台服务需要请求 `FOREGROUND_SERVICE` 权限，否则系统会引发`SecurityException`。

解决方法：在`AndroidManifest.xml` 中添加 `FOREGROUND_SERVICE`权限

```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```

### 启动Activity

在Android 9.0 中，不能直接非Activity环境(比如：Service、Application)中启动Activity，否则会崩溃

```
java.lang.RuntimeException: Unable to create service com.weilu.test.MyService: android.util.AndroidRuntimeException: Calling startActivity() from outside of an Activity  context requires the FLAG_ACTIVITY_NEW_TASK flag. Is this really what you want?
```

这类问题一般会在点击推送消息跳转页面这类场景，解决方法就是Intent中添加`FLAG_ACTIVITY_NEW_TASK`标志

```java
Intent intent = new Intent(this, TestActivity.class);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);
```



### 自动填充框架

Android 9引入了自动填充服务，可以进一步增强用户填写表单的体验。

