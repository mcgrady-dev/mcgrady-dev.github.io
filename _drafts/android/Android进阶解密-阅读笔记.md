



## 第17章 内存优化

## 17.2 Memory Monitor

### 17.3 Allocation Tracker

Allocation Tracker 用来跟踪内存分配，它允许你在执行某些操作的同时监视在何处分配对象，了解这些分配使你能够调整与这些操作相关的方法调用，以优化应用程序性能和内存使用。Allocation Tracker 能够做到如下事情：

- 显示代码分配对象类型、大小、分配线程、堆栈跟踪的时间和位置。
- 通过重复的分配/释放模式帮助识别内存变化。
- 当与 HPROF Viewer 结合使用时，可以帮助你跟踪内存泄露。例如：如果你在堆上看到一个 Bitmap 对象，你可以使用 Allocation Tracker 来找到其分配的位置。
