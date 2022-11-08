引言：分析Android源码6.0的过程，一定离不开Java与C/C++代码直接的来回跳转，那么就很有必要掌握JNI，这是链接Java层和Native层的桥梁。

## JNI概述
JNI（Java Native Interface，Java本地接口），用于打通Java层与Native(C/C++)层。这不是Android系统所独有的，而是Java所有。众所周知，Java语言是跨平台的语言，而这跨平台的背后都是依靠Java虚拟机，虚拟机采用C/C++编写，适配各个系统，通过JNI为上层Java提供各种服务，保证跨平台性。

相信不少经常使用Java的程序员，享受着其跨平台性，可能全然不知JNI的存在。在Android平台，让JNI大放异彩，为更多的程序员所熟知，往往为了提供效率或者其他功能需求，就需要NDK开发。上一篇文章[Linux系统调用(syscall)原理](http://gityuan.com/2016/05/21/syscall/)，介绍了打通android上层与底层kernel的枢纽syscall，那么本文的目的则是介绍打通android上层中Java层与Native的纽带JNI。

## JNI查找方式
Android系统在启动启动过程中，先启动Kernel创建init进程，紧接着由init进程fork第一个横穿Java和C/C++的进程，即Zygote进程。Zygote启动过程中会AndroidRuntime.cpp中的startVm创建虚拟机，VM创建完成后，紧接着调用startReg完成虚拟机中的JNI方法注册。

### startReg
`AndroidRuntime.cpp`[^code]
```cpp
int AndroidRuntime::startReg(JNIEnv* env)
{
    //设置线程创建方法为javaCreateThreadEtc
    androidSetCreateThreadFunc((android_create_thread_fn) javaCreateThreadEtc);

    env->PushLocalFrame(200);
    //进程NI方法的注册
    if (register_jni_procs(gRegJNI, NELEM(gRegJNI), env) < 0) {
        env->PopLocalFrame(NULL);
        return -1;
    }
    env->PopLocalFrame(NULL);
    return 0;
}
```

`register_jni_procs(gRegJNI, NELEM(gRegJNI), env)`这行代码的作用就是就是循环调用`gRegJNI`数组成员所对应的方法。

```cpp
static int register_jni_procs(const RegJNIRec array[], size_t count, JNIEnv* env)
{
    for (size_t i = 0; i < count; i++) {
        if (array[i].mProc(env) < 0) {
            return -1;
        }
    }
    return 0;
}
```

`gRegJNI`数组，有100多个成员变量，定义在`AndroidRuntime.cpp`[^code]：
```cpp
static const RegJNIRec gReggRegJNIJNI[] = {
    REG_JNI(register_android_os_MessageQueue),
    REG_JNI(register_android_os_Binder),
    ...
};
```

该数组的每个成员都代表一个类文件的`jni`映射，其中`REG_JNI`是一个宏定义，在Zygote中介绍过，该宏的作用就是调用相应的方法。

### 如何查找native方法
当大家在看framework层代码时，经常会看到native方法，这是往往需要查看所对应的C++方法在哪个文件，对应哪个方法？下面从一个实例出发带大家如何查看java层方法所对应的native方法位置。

#### 实例(一)
当分析Android消息机制源码，遇到MessageQueue.java中有多个native方法，比如：

```java
private native void nativePollOnce(long ptr, int timeoutMillis);
```

**步骤1：**
`MessageQueue.java`[^code]的全限定名为`android.os.MessageQueue.java`，
方法名：`android.os.MessageQueue.nativePollOnce()`，
而相对应的native层方法名只是将点号替换为下划线，可得`android_os_MessageQueue_nativePollOnce()`。
>Tips： nativePollOnce --> android_os_MessageQueue_nativePollOnce()

**步骤2：**
有了native方法，那么接下来需要知道该native方法所在那个文件。
前面已经介绍过Android系统启动时就已经注册了大量的JNI方法，见附录`gRegJNI`数组。
这些注册方法命令方式：

```cpp
register_[包名]_[类名]
```
那么`MessageQueue.java`[^code]所定义的`JNI`注册方法名应该是`register_android_os_MessageQueue`，的确存在于`gRegJNI`数组，
说明这次`JNI`注册过程是有开机过程完成的。 
该方法在AndroidRuntime.cpp申明为extern方法：
```java
extern int register_android_os_MessageQueue(JNIEnv* env);
```
这些extern方法绝大多数位于`/framework/base/core/jni/`目录，大多数情况下native文件命名方式：
```cpp
[包名]_[类名].cpp
[包名]_[类名].h
```
>Tips： MessageQueue.java --> android_os_MessageQueue.cpp

打开`android_os_MessageQueue.cpp`[^code]文件，搜索`android_os_MessageQueue_nativePollOnce`方法，这便找到了目标方法：
```java
static void android_os_MessageQueue_nativePollOnce(JNIEnv* env, jobject obj,
        jlong ptr, jint timeoutMillis) {
    NativeMessageQueue* nativeMessageQueue = reinterpret_cast<NativeMessageQueue*>(ptr);
    nativeMessageQueue->pollOnce(env, obj, timeoutMillis);
}
```
到这里完成了一次从Java层方法搜索到所对应的C++方法的过程。

#### 实例(二)
对于native文件命名方式，有时并非`[包名]_[类名].cpp`，比如`Binder.java`[^code]

`Binder.java`所对应的native文件：`android_util_Binder.cpp`

```java
public static final native int getCallingPid();
```
根据**实例(一)**方式，找到`getCallingPid --> android_os_Binder_getCallingPid()`，并且在附录`gRegJNI`数组中找到`register_android_os_Binder`。

按**实例(一)**方式则native文名应该为`android_os_Binder.cpp`[^code]，可是在`/framework/base/core/jni/`目录下找不到该文件，这是例外的情况。其实真正的文件名为`android_util_Binder.cpp`，这就是例外，这一点有些费劲，不明白为何google要如此打破规律的命名。
```java
static jint android_os_Binder_getCallingPid(JNIEnv* env, jobject clazz)
{
    return IPCThreadState::self()->getCallingPid();
}
```
有人可能好奇，既然如何遇到打破常规的文件命令，怎么办？这个并不难，首先，可以尝试在`/framework/base/core/jni/`中搜索，对于`binder.java`[^code]，可以直接搜索`binder`关键字，其他也类似。如果这里也找不到，可以通过`grep`全局搜索`android_os_Binder_getCallingPid`这个方法在哪个文件。

#### 实例(三)
前面两种都是在Android系统启动之初，便已经注册过JNI所对应的方法。 那么如果程序自己定义的jni方法，该如何查看jni方法所在位置呢？下面以`MediaPlayer.java`[^code]为例，其包名为`android.media`：
```java
public class MediaPlayer{
    static {
        System.loadLibrary("media_jni");
        native_init();
    }

    private static native final void native_init();
    ...
}
```
通过static静态代码块中`System.loadLibrary`方法来加载动态库，库名为`media_jni`, Android平台则会自动扩展成所对应的`libmedia_jni.so`库。 接着通过关键字native加在native_init方法之前，便可以在java层直接使用native层方法。

接下来便要查看`libmedia_jni.so`库定义所在文件，一般都是通过`Android.mk`文件定义`LOCAL_MODULE:= libmedia_jni`，可以采用`grep`或者`mgrep`来搜索包含`libmedia_jni`字段的`Android.mk`所在路径。

搜索可知，`libmedia_jni.so`位于`/frameworks/base/media/jni/Android.mk`。用前面**实例(一)**中的知识来查看相应的文件和方法名分别为：
```cpp
android_media_MediaPlayer.cpp
android_media_MediaPlayer_native_init()
```
再然后，你会发现果然在该`Android.mk`所在目录`/frameworks/base/media/jni/`中找到`android_media_MediaPlayer.cpp`文件，并在文件中存在相应的方法：
```cpp
static void
android_media_MediaPlayer_native_init(JNIEnv *env)
{
    jclass clazz;
    clazz = env->FindClass("android/media/MediaPlayer");
    fields.context = env->GetFieldID(clazz, "mNativeContext", "J");
    ...
}
```
>Tips：`MediaPlayer.java`[^code]中的`native_init`方法所对应的native方法位于`/frameworks/base/media/jni/`目录下的`android_media_MediaPlayer.cpp`[^code]文件中的`android_media_MediaPlayer_native_init`方法。

### 小结
JNI作为连接Java世界和C/C++世界的桥梁，很有必要掌握。看完本文，至少能掌握在分析Android源码过程中如何查找native方法。首先要明白native方法名和文件名的命名规律，其次要懂得该如何去搜索代码。 JNI方式注册无非是Android系统启动过程中Zygote注册以及通过System.loadLibrary方式注册，对于系统启动过程注册的，可以通过查询下来附录gReg是否存在对应的register方法，如果不存在，则大多数情况下是通过LoadLibrary方式来注册。

## JNI原理分析
再进一步来分析，Java层与native层方法是如何注册并映射的，继续以MediaPlayer为例。

在文件`MediaPlayer.java`[^code]中调用`System.loadLibrary("media_jni")`把`libmedia_jni.so`动态库加载到内存。接下来，以loadLibrary为起点展开JNI注册流程的过程分析。

### loadLibrary
`System.java`[^code]
```java
public static void loadLibrary(String libName) {
    //接下来调用Runtime方法
    Runtime.getRuntime().loadLibrary(libName, VMStack.getCallingClassLoader());
}
```
`Runtime.java`[^code]
```java
void loadLibrary(String libraryName, ClassLoader loader) {
    //loader不会空，则进入该分支
    if (loader != null) {
        //查找库所在路径
        String filename = loader.findLibrary(libraryName);
        if (filename == null) {
            throw new UnsatisfiedLinkError(loader + " couldn't find \"" +
                                           System.mapLibraryName(libraryName) + "\"");
        }
        //加载库
        String error = doLoad(filename, loader);
        if (error != null) {
            throw new UnsatisfiedLinkError(error);
        }
        return;
    }

    //loader为空，则会进入该分支
    String filename = System.mapLibraryName(libraryName);
    List<String> candidates = new ArrayList<String>();
    String lastError = null;
    for (String directory : mLibPaths) {
        String candidate = directory + filename;
        candidates.add(candidate);
        if (IoUtils.canOpenReadOnly(candidate)) {
             //加载库
            String error = doLoad(candidate, loader);
            if (error == null) {
                return;//加载成功
            }
            lastError = error;
        }
    }
    if (lastError != null) {
        throw new UnsatisfiedLinkError(lastError);
    }
    throw new UnsatisfiedLinkError("Library " + libraryName + " not found; tried " + candidates);
}
```
真正加载的工作是由`doLoad()`，该方法内部增加同步锁，保证并发时一致性。
```java
private String doLoad(String name, ClassLoader loader) {
    ...
    synchronized (this) {
        return nativeLoad(name, loader, ldLibraryPath);
    }
}
```
`nativeLoad()`这是一个native方法，再进入ART虚拟机`java_lang_Runtime.cc`，再细讲就要深入剖析虚拟机内部，这里就不再往下深入了，后续博主有空再展开art虚拟机系列的文章，这里直接说结论：
- 调用dlopen函数，打开一个so文件并创建一个handle；
- 调用dlsym()函数，查看相应so文件的JNI_OnLoad()函数指针，并执行相应函数。

总之，**`System.loadLibrary()`的作用就是调用相应库中的`JNI_OnLoad()`方法**。

### JNI_OnLoad
` android_media_MediaPlayer.cpp`[^code]
```cpp
jint JNI_OnLoad(JavaVM* vm, void* reserved)
{
    JNIEnv* env = NULL;
    //【见3.3】 注册JNI方法 
    if (register_android_media_MediaPlayer(env) < 0) {
        goto bail;
    }
    ...
}
```
### register_android_media_MediaPlayer
`android_media_MediaPlayer.cpp`[^code]
```cpp
static int register_android_media_MediaPlayer(JNIEnv *env)
{
    //【见3.4】
    return AndroidRuntime::registerNativeMethods(env,
                "android/media/MediaPlayer", gMethods, NELEM(gMethods));
}
```
其中`gMethods`，记录java层和C/C++层方法的一一映射关系。
```cpp
static JNINativeMethod gMethods[] = {
    {"prepare",      "()V",  (void *)android_media_MediaPlayer_prepare},
    {"_start",       "()V",  (void *)android_media_MediaPlayer_start},
    {"_stop",        "()V",  (void *)android_media_MediaPlayer_stop},
    {"seekTo",       "(I)V", (void *)android_media_MediaPlayer_seekTo},
    {"_release",     "()V",  (void *)android_media_MediaPlayer_release},
    {"native_init",  "()V",  (void *)android_media_MediaPlayer_native_init},
    ...
};
```
这里涉及到结构体`JNINativeMethod`，其定义在`jni.h`[^code]文件：
```h
typedef struct {
    const char* name;  //Java层native函数名
    const char* signature; //Java函数签名，记录参数类型和个数，以及返回值类型
    void*       fnPtr; //Native层对应的函数指针
} JNINativeMethod;
```
关于函数签名`signature`在下一小节展开说明。

### registerNativeMethods
`AndroidRuntime.cpp`[^code]
```cpp
int AndroidRuntime::registerNativeMethods(JNIEnv* env,
    const char* className, const JNINativeMethod* gMethods, int numMethods)
{
    //【见3.5】
    return jniRegisterNativeMethods(env, className, gMethods, numMethods);
}
```
`jniRegisterNativeMethods`该方法是由Android JNI帮助类`JNIHelp.cpp`[^code]来完成。

### jniRegisterNativeMethods
`JNIHelp.cpp`[^code]
```cpp
extern "C" int jniRegisterNativeMethods(C_JNIEnv* env, const char* className,
    const JNINativeMethod* gMethods, int numMethods)
{
    JNIEnv* e = reinterpret_cast<JNIEnv*>(env);
    scoped_local_ref<jclass> c(env, findClass(env, className));
    if (c.get() == NULL) {
        e->FatalError("");//无法查找native注册方法
    }
    //【见3.6】 调用JNIEnv结构体的成员变量
    if ((*env)->RegisterNatives(e, c.get(), gMethods, numMethods) < 0) {
        e->FatalError("");//native方法注册失败
    }
    return 0;
}
```
### RegisterNatives
`jni.h`[*code]
```h
struct _JNIEnv {
    const struct JNINativeInterface* functions;

    jint RegisterNatives(jclass clazz, const JNINativeMethod* methods,
            jint nMethods)
    { return functions->RegisterNatives(this, clazz, methods, nMethods); }
    ...
} functions是指向`JNINativeInterface`结构体指针，也就是将调用下面方法：

struct JNINativeInterface {
    jint (*RegisterNatives)(JNIEnv*, jclass, const JNINativeMethod*,jint);
    ...
}
```

再往下深入就到了虚拟机内部吧，这里就不再往下深入了。 总之，这个过程完成了`gMethods`数组中的方法的映射关系，比如java层的`native_init()`方法，映射到native层的`android_media_MediaPlayer_native_init()`方法。

虚拟机相关的变量中有两个非常重要的量:
- `JavaVM`：是指进程虚拟机环境，每个进程有且只有一个`JavaVM`实例
- `JNIEnv`：是指线程上下文环境，每个线程有且只有一个`JNIEnv`实例，

## JNI资源
`JNINativeMethod`结构体中有一个字段为`signature`

### 数据类型
#### 基本数据类型

| 简称   |    Java |  Native  |
| :--- | ------: | :------: |
| B    |    byte |  jbyte   |
| C    |    char |  jchar   |
| D    |  double | jdouble  |
| F    |   float |  jfloat  |
| I    |     int |   jint   |
| S    |   short |  jshort  |
| J    |    long |  jlong   |
| Z    | boolean | jboolean |
| V    |    void |   void   |


#### 数组数据类型

数组简称则是在前面添加`[`
| 简称   |      Java |    Native     |
| :--- | --------: | :-----------: |
| B    |    byte[] |  jbyteArray   |
| C    |    char[] |  jcharArray   |
| D    |  double[] | jdoubleArray  |
| F    |   float[] |  jfloatArray  |
| I    |     int[] |   jintArray   |
| S    |   short[] |  jshortArray  |
| J    |    long[] |  jlongArray   |
| Z    | boolean[] | jbooleanArray |

#### 复杂数据类型

对象类型简称：`L+classname +`
| 简称                    |      Java |    Native     |
| :-------------------- | --------: | :-----------: |
| Ljava/lang/String;    |    String |    jstring    |
| L+classname +;        |      所有对象 |    jobject    |
| [L+classname +;       |  Object[] | jobjectArray  |
| Ljava.lang.Class;     |     Class |    jclass     |
| Ljava.lang.Throwable; | Throwable |  jthrowable   |
| S                     |   short[] |  jshortArray  |
| J                     |    long[] |  jlongArray   |
| Z                     | boolean[] | jbooleanArray |


#### 方法签名

方法签名都是采用上面3个数据类型表格中的简称： (输入参数...)返回值参数

| Java函数                        |                    函数签名 |
| :---------------------------- | ----------------------: |
| void foo()                    |                     ()V |
| float foo(int i)              |                    (I)F |
| long foo(int[] i)             |                   ([I)J |
| double foo(Class c)           |    (Ljava/lang/Class;)D |
| boolean foo(int[] i,String s) | ([ILjava/lang/String;)Z |
| String foo(int i)             |   (I)Ljava/lang/String; |


#### 其他
##### 垃圾回收
对于Java开发人员来说无需关系垃圾回收，完全由虚拟机GC来负责垃圾回收，而对于JNI开发人员，对于内存释放需要谨慎处理，需要的时候申请，使用完记得释放内容，以免发生内存泄露。在JNI提供了三种Reference类型，Local Reference(本地引用)， Global Reference（全局引用）， Weak Global Reference(全局弱引用)。其中Global Reference如果不主动释放，则一直不会释放；对于其他两个类型的引用都是释放的可能性，那是不是意味着不需要手动释放呢？答案是否定的，不管是这三种类型的那种引用，都尽可能在某个内存不再需要时，立即释放，这对系统更为安全可靠，以减少不可预知的性能与稳定性问题。

另外，ART虚拟机在GC算法有所优化，为了减少内存碎片化问题，在GC之后又坑你会移动对象内存的位置，对于Java层程序并没有影响，但是对于JNI程序可要小心了，对于通过指针来直接访问内存对象是，Dalvik能正确运行的程序，ART下未必能正常运行。

##### 异常处理
Java层出现异常，虚拟机会直接抛出异常，这是需要try..catch或者继续往外throw。但是对于JNI出现异常时，即执行到JNIEnv中某个函数异常时，并不会立即抛出异常来中断程序的执行，还可以继续执行内存之类的清理工作，直到返回到Java层时才会抛出相应的异常。

另外，Dalvik虚拟机有些情况下JNI函数出错可能返回NULL，但ART虚拟机在出错时更多的是抛出异常。这样导致的问题就可能是在Dalvik版本能正常运行的程序，在ART虚拟机上由于没有正确处理异常而崩溃。

###总结
本文主要通过实例，基于Android 6.0源码来分析JNI原理，讲述JNI核心功能：
- 介绍了如何查找JNI方法，让大家明白如何从Java层跳转到Native层；
- 分析了JNI函数注册流程，进一步加深对JNI的理解；
- 列举Java与native以及函数签名方式。

---
```
frameworks/base/core/jni/AndroidRuntime.cpp

libcore/luni/src/main/java/java/lang/System.java
libcore/luni/src/main/java/java/lang/Runtime.java
libnativehelper/JNIHelp.cpp
libnativehelper/include/nativehelper/jni.h

frameworks/base/core/java/android/os/MessageQueue.java
frameworks/base/core/jni/android_os_MessageQueue.cpp

frameworks/base/core/java/android/os/Binder.java
frameworks/base/core/jni/android_util_Binder.cpp

frameworks/base/media/java/android/media/MediaPlayer.java
frameworks/base/media/jni/android_media_MediaPlayer.cpp
```
[^code]: 本文涉及相关源码

