## Android Profiler

### 运行独立性能分析器

如需运行独立性能分析器，请执行以下操作：

1. 确保性能分析器当前未在 Android Studio 中运行。

2. 转到安装目录，然后转到 `bin` 目录：

   **Windows/Linux**：`<studio-installation-folder>/bin`

   **macOS**：`<studio-installation-folder>/Contents/bin`

3. 根据您的操作系统，运行 `profiler.exe` 或 `profiler.sh`。系统随即会显示 Android Studio 启动画面。





## CPU Profiler

### 1. Call Chart

首先每个横条代表一个函数的运行时间：

- **橙色**：对系统 API 的函数调用
- **绿色**：对应用自有函数的调用
- **蓝色**：对第三方 API（包括 Java 语言 API）的函数调用

![](../../images/android/android_profiler_call_chart.png)

Call Chart 显示了应用内所有函数的调用关系以及运行时间，开发者快速查找某个时间点应用内函数的调用和耗时情况。



### 2. Flame Chart

Flame Chart 看似 Call Chart 的倒置，其实不然，Flame Chart 会将调用顺序完全相同的函数归类成一条横条，如下：

![](../../images/android/android_profiler_call_chart_2.png)

![](../../images/android/android_profiler_flame_chart.png)

此时，X轴不再表示调用时间，而表示每个函数的运行总时间。

Flame Chart 更方便开发者查询哪些函数消耗的时间最多。如果看顶层的哪个函数占据的宽度比较大，就表示该函数可能存在性能问题。

### 3. Top Down

Top Down 显示一个函数调用列表，展开后可以查看函数的被调用方。可以理解，Flame Chart 是 Top Down 的图形化表示形式。

### 4. Bottom Up

Bottom Up 列出了所有执行函数的时间，点开后是它调用方，按运行时间排序。

### 5. Events

Events 表格列出了当前所选线程的所有调用。



### 选择记录配置

- **对 Java 方法采样 (Sample Java Methods)**：在应用的 Java 代码执行期间，频繁捕获应用的调用堆栈。分析器会比较捕获的数据集，以推导与应用的 Java 代码执行相关的时间和资源使用信息。
- **跟踪 Java 方法 (Trace Java Methods)**：在运行时检测应用，从而在每个方法调用开始和结束时记录一个时间戳。系统会收集并比较这些时间戳，以生成方法跟踪数据，包括时间信息和 CPU 使用率。
- **对 C/C++ 函数采样 (Sample C/C++ Functions)**：捕获应用的原生线程的采样跟踪数据。如需使用此配置，您必须将应用部署到搭载 Android 8.0（API 级别 26）或更高版本的设备上。
- **跟踪系统调用 (Trace System Calls)**：捕获非常详细的细节，以便检查应用于系统资源的交互情况。可以检查线程状态的确切时间和持续时间、直观地查看所有内核的 CPU 瓶颈在何处，并添加需分析的自定义跟踪事件。如需使用此配置，您必须将应用部署到搭载 Android 7.0（API 级别 24）或更高版本的设备上。



## Memory Profiler

Memory Profiler显示一个应用内存使用量的**实时图表** ，可以**捕获堆转储** 、以及**跟踪内存分配**。

内存计数类别如下：

- **Java**：从 Java 或 Kotlin 代码分配的对象的内存。

- **Native**：从 C 或 C++ 代码分配的对象的内存。

  即使您的应用中不使用 C++，您也可能会看到此处使用了一些原生内存，因为即使您编写的代码采用 Java 或 Kotlin 语言，Android 框架仍使用原生内存代表您处理各种任务，如处理图像资源和其他图形。

- **Graphics**：图形缓冲区队列为向屏幕显示像素（包括 GL 表面、GL 纹理等等）所使用的内存。（请注意，这是与 CPU 共享的内存，不是 GPU 专用内存。）

- **Stack**：您的应用中的原生堆栈和 Java 堆栈使用的内存。这通常与您的应用运行多少线程有关。

- **Code**：您的应用用于处理代码和资源（如 dex 字节码、经过优化或编译的 dex 代码、.so 库和字体）的内存。

- **Others**：您的应用使用的系统不确定如何分类的内存。

- **Allocated**：您的应用分配的 Java/Kotlin 对象数。此数字没有计入 C 或 C++ 中分配的对象。



### 捕获堆转储（Dump Java heap）

堆转储显示应用中哪些对象正在使用内存。特别是在长时间的用户会话后，堆转储会显示您认为不应再位于内存中却仍在内存中的对象，从而帮组识别内存泄露。

捕获堆转储后，可以查看一下信息：

- 您的应用分配了哪些类型的对象，以及每种对象有多少。
- 每个对象当前使用多少内存。
- 在代码中的什么位置保持着对每个对象的引用。
- 对象所分配到的调用堆栈。（目前，对于 Android 7.1 及更低版本，只有在记录分配期间捕获堆转储时，才会显示调用堆栈的堆转储。）



