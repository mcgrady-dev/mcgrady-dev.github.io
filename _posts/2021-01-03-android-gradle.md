---
layout: 
title: Gradle
date: 2021-01-03 20:53 +0800
tags: [android,gradle]
---

**Gradle**是一个基于[Apache Ant](https://zh.wikipedia.org/wiki/Apache_Ant)和[Apache Maven](https://zh.wikipedia.org/wiki/Apache_Maven)概念的项目[自动化建构](https://zh.wikipedia.org/wiki/自動化建構)工具。Gradle 构建脚本使用的是 [Groovy](https://zh.wikipedia.org/wiki/Groovy) 或 [Kotlin](https://zh.wikipedia.org/wiki/Kotlin) 的[特定领域语言](https://zh.wikipedia.org/wiki/特定领域语言)来编写的，而不是传统的 [XML](https://zh.wikipedia.org/wiki/XML)。

<!--more-->

## Gradle

### 什么是构建工具？

最初，Google为了满足 Android 能在 Eclipse 上进行开发的需求，Google 开发了一个叫ADT（Android Developer Tools）的工具，从此可以直接在 Eclipse 上进行编译、运行、签名、打包等一系列流程。所以某种意义上，ADT 就是我们的构建工具。

一般来说，构建工具除了以上提到的编译、运行、签名、打包等，还具备依赖管理的功能。在 Eclipse 上，如果用到第三方库时，一般都是先下载 jar 包，然后把 jar 包添加到 libs 目录，然后项目中引入。假设第三方有更新，需要下载最新的 jar 包然后替换掉原来的，如果手动来操作就很麻烦，所以 Eclipse 在这方面只有依赖，没有管理。

一个完整的构建工具应该包含依赖管理功能，传统的构建工具有 Make、Ant、Maven、lvy等，而 Gradle 是新一代的自动化构建工具。



## Gradle Wrapper

现在 Android Studio 上新建一个项目，Android Studio 默认会安装 Gradle，但这个其实不是真正的Gradle ，而是 Gradle Wrapper，意为 Gradle 的包装，作用是简化 Gradle 本身的安装、部署。

假设我们本地有多个项目，引用的 Gradle 版本都不一致情况下可能需要安装多个 Gradle 版本，为了解决这个问题，就是通过每个项目都配置了个指定版本的 Gradle Wrapper ，每个项目可以用不同的 Gradle 版本来构建项目。

### 命令行区分

- **gradle**：使用系统环境变量定义的 Gradle 进行构建
- **graldew**：使用 Gradle Wrapper 进行构建



## Gradle Deamon

[Gradle Deamon](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.gradle.org%2Fcurrent%2Fuserguide%2Fgradle_daemon.html) 是 Gradle 3.0 引入的构建优化策略，通过壁柜重复创建 JVM 和内存缓存的手段提升构建速度。当构建结束后， Deamon 进程不会立即销毁，而是保存在内存中等待承接下一次构建。

Deamon 的优化效果主要体现在：

- 缩短 JVM 虚拟机启动时间：不重复创建
- JIT 编译：Deamon 进程会执行 JIT 编译，有助于提升后续构建的字节码执行效率
- 构建缓存：构建过程中加载的类、资源或者 Task 的输入和输出会保存在内存中，可以被后续构建复用



## Android Gradle Plugin

Gradle 是一个基于 Apache Ant 和 Apache Maven 概念的项目自动化构建开源工具。而 Android Plugin Gradle 是一堆适合 Android 开发的 Gradle 插件集合（可以理解为一个 java 程序）。

是 Google 在推出 Android Studio 时选中了 Gradle 作为构建工具，为了支持 Gradle 能在 Android Studio 上使用，Google 开发了一个 Android Studio 的插件叫 Android Gradle Plugin，所以我们才得以在 Android Studio 上使用 Gradle 的原因。



## project:build.gradle

### 配置依赖包下载地址

用阿国内阿里云的依赖下载地址替换Google依赖包下载地址

```groovy
repositories {
    maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
    maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter' }
    ...
}

allprojects {
    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter' }
        ...
    }
} 
```



## app:build.gradle

### android { }

```groovy
android {
    // The default value for each feature is shown below. You can change the value to
    // override the default behavior.
    buildFeatures {
        // Determines whether to generate a BuildConfig class.
        buildConfig = true
        // Determines whether to support View Binding.
        // Note that the viewBinding.enabled property is now deprecated.
        viewBinding = false
        // Determines whether to support Data Binding.
        // Note that the dataBinding.enabled property is now deprecated.
        dataBinding = false
        // Determines whether to generate binder classes for your AIDL files.
        aidl = true
        // Determines whether to support RenderScript.
        renderScript = true
        // Determines whether to support injecting custom variables into the module’s R class.
        resValues = true
        // Determines whether to support shader AOT compilation.
        shaders = true
    }
}
```



### buildToolsVersion

用于指定项目构建工具的版本


### compileSdkVersion
指明用编译应用的 Android SDK 版本，强烈推荐总是使用最新的 SDK 进行编译（默认就是最新的）。

修改 compileSdkVersion 不会改变运行时的行为，当你修改了 `compileSdkVersion` 的时候，可能会出现新的编译警告、编译错误，但新的 **`compileSdkVersion` 不会被包含到 APK 中：它纯粹只是在编译的时候使用**。
>注意：如果使用 `Support Library` ，那么使用最新发布的 `Support Library` 就需要使用最新的 SDK 编译。

### minSdkVersion

指明应用程序运行所需的最小API level。如果不指明的话，默认是1。也就是说该应用兼容所有的android版本。

我们应该总是声明这个属性

1. 如果系统的API level低于`android:minSdkVersion`设定的值，那么android系统会阻止用户安装这个应用
2. 如果指明了这个属性，并且在项目中使用了高于这个API level的API， 那么会在编译时报错。(下面会讲解解决方法)


> 注意：你所使用的库，如 `Support Library` 或 `Google Play services`，可能有他们自己的 `minSdkVersion`。你的应用设置的 `minSdkVersion` 必需大于等于这些库的 `minSdkVersion` 。
>

### targetSdkVersion

**targetSdkVersion 是 Android 提供向前兼容的主要依据**
将 target 更新为最新的 SDK 是所有应用都应该优先处理的事情。但这不意味着你一定要使用所有新引入的功能，也不意味着你可以不做任何测试就盲目地更新` targetSdkVersion `，**请一定在更新 `targetSdkVersion` 之前做测试**！

如果平台的API Level高于你的应用程序中的`targetSdkVersion`属性指定的值，系统会开启兼容行为来确保你的应用程序继续以期望的形式来运行。你可以通过指定`targetSdkVersion`来匹配运行程序的平台的 API level来禁用这种兼容性行为。

> 举例：
>
> 设置API level为11或更高，当你的应用运行在Android3.0或更高的系统上时，系统会为你的应用使用新的默认主题（Holo主题），并且当运行在大屏幕的设备上时会禁用屏幕兼容模式（screen compatibility mode），因为支持了 API level 11就暗示了支持大屏幕。



Android 高版本API方法在低版本系统上的兼容性处理

1. 用`@TargeApi($API_LEVEL)`使可以编译通过, 不建议使用`@SuppressLint("NewApi")`
   区别：

   - ` @SuppressLint("NewApi"）`屏蔽一切新api中才能使用的方法报的`android lint`错误

   - `@TargetApi()` 只屏蔽某一新API中才能使用的方法报的`android lint`错误
     举个例子，某个方法中使用了API9新加入的方法，而项目设置的`android:minSdkVersion=8`，此时在方法上加`@SuppressLint("NewApi"）`
     和`@TargetApi(Build.VERSION_CODES.GINGERBREAD)`都可以，以上是通用的情况。
     而当你在此方法中又引用了一个`API11`才加入的方法时，`@TargetApi(Build.VERSION_CODES.GINGERBREAD)`注解的方法又报错了，而
     `@SuppressLint("NewApi"）`不会报错，这就是区别。

2. 判断运行时版本，在低版本系统不调用此方法，同时为了保证功能的完整性，需要提供低版本功能实现。
   例如：

```java
public class MainActivity extends AppCompatActivity {
   private  AlertDialog.Builder builder;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //如果API level大于11 大于11的时候能够指定主体
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.HONEYCOMB) {
             builder = new AlertDialog.Builder(this,
                    AlertDialog.THEME_HOLO_LIGHT);
        }else {
             builder = new AlertDialog.Builder(this);
        }
        
        builder.setItems(new String[] { "拍照","选择" },
                new DialogInterface.OnClickListener() {

                    @Override
                    public void onClick(DialogInterface dialog, int which) {

                    }
                });
        builder.setTitle("选择照片");
        builder.create().show();
    }
}
```



### testInstrumentationRunner

用于当前项目中的启用`JUnit`测试，可以为当前项目编写测试用例，以保证功能的正确性和稳定性。



### buildTypes

#### proguardFiles

- proguard-android-optimize.txt：<Android SDK>/tools/proguard 下的所有项目通用的混淆规则
- proguard-rules.pro：当前项目的特有混淆规则



### applicationIdSuffix

```groovy
android {
  buildTypes {
        release {
            // 这里是在 applicationId 中添加了一个后缀。所以『.』要加上
          applicationIdSuffix ".release"
          minifyEnabled false
          proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
 
        dev{
            applicationIdSuffix ".dev"
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
 
        }
 
    }
}
```



## 依赖项配置 

| 配置                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| implementation      | Gradle 会将依赖项添加到编译类路径，并将依赖项打包到编译输出。不过，当模块配置 implementation 依赖项时，其他模块只有在运行时才能使用该依赖项。 |
| api                 | Gradle 会将依赖项添加到编译类路径和编译输出。当一个模块包含 api 依赖项时，会让 Gradle 了解该模块要以传递方式将该依赖项导出到其他模块，以便这些模块在运行时和编译时都可以使用该依赖项。 |
| compileOnly         | Gradle 只会将依赖项添加到编译类路径（也就是说，不会将其添加到编译输出）。 |
| runtimeOnly         | Gradle 只会将依赖项添加到编译输出，以便在运行时使用。也就是说，不会将其添加到编译类路径。 |
| annotationProcessor | 要添加对作为注解处理器的库的依赖关系，必须使用 annotationProcessor 配置将其添加到注解处理器类路径。 |

## ktlint

ktlint 是一个自带格式化的静态代码分析工具，可用于规范化 kotlin 代码风格，还可以自动格式化代码，节省手动格式化的时间。简单来说，ktlint 是一个包含了 `linter` 和 `formatter` 的静态代码分析工具。

**特性（Features）**

- 无需配置
- 内置格式化工具
- 自定义化输出
- 包含所有依赖的单个 jar 文件

**标准规则（Standard rules）**

- 缩进用4个`space`，除非在`.editorConfig`中设置了不同的`indent_size`值（使用方法参考 `EditorConfig`）
- 无分号，除非用于在同一行上分隔多个语句
- 不使用通配符和无用的`import`
- 无连续空行
- `}`之前没有空行
- 无尾部空白
- 不使用`Unit`返回值，即用`fun method {}`代替
- 没有空的`{}`类主体
- 范围(`..`)运算符周围没有空格
- `+, -，*，/，%，&&，||` 等二进制运算符之前无换行符
- 在链式调用中`， .，?.`和`?:`应该放在下一行
- 在赋值(`=`)操作符处分行时，后续内容紧跟在赋值符号后面
- 当类/函数签名不适合单行时，每个参数必须位于单独一行
- 统一字符串模板`$v`而不是`${v}`，`${p.v}`而不是`${p.v.toString()}`
- 统一修饰剂的顺序
- 统一关键字，逗号，冒号，大括号，中缀运算符，注释等后面的间距
- 在每个文件末尾换行，默认情况下不使用，但推荐使用。使用方法：`.editorConfig`中设置`insert_final_newline=true`

## Gradle Task

### uploadArchives

`uploadArchives`是一个发布类库到中央仓库的task

### dpendencies

```bash
./gradlew app:dependencies
```

### transitive

```groovy
//简单地排除所有 okhttp-integration 库的所有过渡依赖：
implementation (com.github.bumptech.glide:okhttp-integration:4.11.0) {
  transitive = false
}
```

### walle

- 生成单个渠道包: `./gradlew clean assembleReleaseChannels -PchannelList=meituan`

- 生成多个渠道包: `./gradlew clean assembleReleaseChannels -PchannelList=meituan,dianping`

- 生成渠道包&写入额外信息:

  `./gradlew clean assembleReleaseChannels -PchannelList=meituan -PextraInfo=buildtime:20161212,hash:xxxxxxx`

  注意: 这里的extraInfo以`key:value`形式提供，多个以`,`分隔。

- 使用临时channelFile生成渠道包: `./gradlew clean assembleReleaseChannels -PchannelFile=/Users/xx/Documents/channel`

- 使用临时configFile生成渠道包: `./gradlew clean assembleReleaseChannels -PconfigFile=/Users/xx/Documents/config.json`
