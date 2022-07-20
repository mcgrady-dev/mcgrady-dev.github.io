##  Android Studio 项目结构

### .gradle 和 .idea

### gradle

### gradlew 和 gradlew.bat

这两个文件时用来在命令行中执行gradle命令的：

- `gradlew`是在Linux或Mac系统中使用
- `gradlew.bat`是在Window系统中使用

### *.iml

iml文件是自动生成的用于标识这是一个IntelliJ IDEA项目的文件





## AndroidStudio 3.0 依赖变化

### compile（api）
这种是我们最常用的方式，使用该方式依赖的库将会参与编译和打包。

### provided（compileOnly）
只在编译时有效，不会参与打包，可以在自己的`moudle`中使用该方式依赖。比如`com.android.support`，`gson`这些使用者常用的库，避免冲突。

### apk（runtimeOnly）
只在生成apk的时候参与打包，编译时不会参与，很少用。

### testCompile（testImplementation）

### testCompile 
只在单元测试代码的编译以及最终打包测试apk时有效。

### debugCompile（debugImplementation）

### debugCompile 
只在debug模式的编译和最终的debug apk打包时有效。

### releaseCompile（releaseImplementation）



## Android设备的CPU类型（通常称为“APIs”）

- **armeabiv-v7a**：第7代及以上的 ARM 处理器。2011年15月以后的生产的大部分Android设备都使用它.
- **arm64-v8a**：第8代、64位ARM处理器，很少设备，三星 Galaxy S6是其中之一。
- **armeabi**：第5代、第6代的ARM处理器，早期的手机用的比较多。
- **x86**：平板、模拟器用得比较多。
- **x86_64**：64位的平板。



## Emulator

模拟器`INSTALL_FAILED_NO_MATCHING_ABIS` 解决方案

```groovy
splits {
        abi {
            enable true
            reset()
            include 'x86', 'armeabi-v7a','x86_64'
            universalApk true
        }
    }
```

