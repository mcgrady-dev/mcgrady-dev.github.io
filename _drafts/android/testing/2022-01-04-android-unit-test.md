## 1. Junit

JUnit4 是一套**基于注解的Java单元测试框架**。在 AndroidStudio 中，编写在`test`目录下的测试类都是基于该框架实现，该目录下的测试代码运行在本地的JVM上，不需要设备（真机或模拟器）的支持。

- 主要的测试方法——断言



## 2. Mocking framework

### 2.1. Mockito

用于 Java 单元测试的Mocking框架。

**Kotlin + Mockito 的使用问题：**

- Mockito 不支持 mock 匿名类、`Final Class`、`Static Method`、`Private Method`
- 某些函数返回可空类型与Kotlin预设不可空类型冲突
- 与Kotlin保留字`when`冲突



### 2.2. Mockito-kotlin

提供辅助函数以在 Kotlin 中使用 Mockito。

**解决了 Kotlin + Mockito 的使用问题：**

- when 改成 whenever
- 排除某些函数返回可空类型的问题

**缺点：**

- 不支持 mock `Static Method`



### 2.3. MockK





### 2.4. PowerMockito

PowerMockito 是一个扩展了 Mockito 的具有更强大功能的单元测试框架，它支持 mock 匿名类、`Final Class`、`Static Method`、`Private Method`。

**Mockito VS  PowerMock**：PowerMock 支持 Mockito2 的实验性功能，目前还有问题还未解决，且更新比 Mockito 慢。



### 2.5. MockWebServer

- 设置 response 的 header、body、status code
- 记录接收到的 request，获取 request 的 body、header、method、path、HTTP version
- 模拟网速慢的网络环境
- 提供 Dispatcher，让 mockWebServer 可以根据不同的请求进行不同的反馈







### Robolectric

Robolectric 通过**一套能运行在JVM上的Android代码**，解决了在Java单元测试中很难进行Android单元测试的痛点。



### Jacoco

Jacoco 的全称为 Java Code Coverage（Java代码覆盖率），可以**生成java的单元测试代码覆盖率报告**。











Android Studio中的典型项目包含两个放置测试的目录。按如下方式组织测试：

- `androidTest`目录应包含在实际或在虚拟设备上运行的测试。此类测试包括集成测试，端到端测试以及仅JVM无法验证应用程序功能的其他测试。
- `test`目录应包含在可以在本地计算机上运行的测试，例如单元测试。



## 单元测试（小型测试）

单元测试时参与项目开发的工程师在项目代码之外建立的白盒测试工程，用于执行项目中的目标函数并验收其状态或结果，为项目提供可靠的质量保证。其中，单元指的是测试的最小模块，通常指函数。

### Android 单元测试

在 Android 中，单元测试的本质依旧是验证函数的功能，测试框架也是 JUnit。在 Java 中，编写代码面对的只有类、对象、函数，编写单元测试时可以在测试工程汇总创建一个对象出来然后执行其函数进行测试，而在 Android 中，编写代码需要面对的是组件、控件、生命周期、异步任务、消息传递等，虽然本质是 SDK 主动执行了一些实例的函数，但创建一个 Activity 并不能让它执行到 resume 状态，因此需要奥 JUnit 之外的框架支持。





**编写小型测试应该是高度集中的单元测试，能够详尽地验证应用中每个类的功能和约定。**

- **本地单元测试**：当快速运行测试而不需要在真实设备上运行测试相关的保真度和置信度时，可以使用本地单元测试来评估应用的逻辑。通常使用 Robolectric 或 Mockito 来实现依赖项，如何选择工具，通常与测试关联的依赖项决定：
  - 测试对 Android 框架有依赖性（特别是与框架复杂交互的测试），最好用 Robolectric。
  - 测试对 Android 框架的依赖性极小，或测试仅取决于自己的对象，可以使用 Mockito 模拟框架。
- **插桩单元测试**：插桩单元测试是在实体设备和模拟器上运行的测试，此类测试可以利用 Android 框架 API 和辅助性 API。插桩测试提供的保真度比本地测试要高，但运行速度要慢得多。因此，只在必须对真实设备的行为进行测试时才使用插桩单元测试。

#### 单元测试的范围

单元测试的对象是组件状态、控件行为、界面元素和自定义函数。



### 集成测试（中型测试）

集成测试用于验证模块内层级之间的交互，或相关模块之间的交互。建议：20％

**编写中性测试，用于验证一组单元的协作和交互的集成测试。**

可以根据应用的结构和以下中型测试示例来定义表示应用中的单元组的最佳方式：

1. 视图和视图模型之间的互动，如测试 Fragment 对象、验证布局 XML 评估 ViewModel 对象的数据绑定逻辑。
2. 验证不同数据源和数据访问对象（DAO）是否按预期进行互动。
3. 应用的垂直切片，测试特定屏幕上的互动。此类测试目的在于验证应用堆栈的所有各层的互动。（不懂）
4. 多 Fragment 测试，评估应用的特定区域。与本列表中提到的其他类型的中型测试不同，这种类型的测试通常需要真实设备，因为被测互动涉及多个界面元素。



### 端到端测试（大型测试）

是端到端测试，用于验证跨越应用程序的多个模块的用户交互。建议：10％







## Robolectric





## AndroidJUnitRunner



## Espresso

Espresso 可以用来编写简洁、美观且可靠的 Android 界面测试。



## UI Automator

UI Automator 是一个界面测试框架，适用于整个系统上以及多个已安装应用间的跨应用功能界面测试。



## 断言库

### AssertJ

Java 流式断言器

### Truth

[truth-dev](https://truth.dev/)





## UI测试

### Espresso

### UI Automator



## 压力测试

### Monkey
