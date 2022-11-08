---
title: 日常开发笔记
date: 2018-01-23 21:30:50
updated: 2018-01-23 21:30:50
tags: [android, note]
categories: android
---


# Android Developer Notes

## View 相关
1. RecyclerView 的 LayoutParams 会是 MarginLayoutParams，如果重设一个 ViewGroup.LayoutParams将会报错，在 onBindViewHolder() 的时候，都会有LayoutParams，所以不用手动再去添加。
2. `view.post()`会获取到关联线程（即UI线程，因为只有UI线程才能创建view）的handler，然后调用该handler的post方法，即把这个任务封装成一个消息post到主线程的消息队列中，然后等待执行。


## 资源相关

1. strings.xml 标签使用
	`<b></b>`	加粗字体
	`<i></i>`	斜体字体
	`<u></u> `	给字体加下划线
	`\n`		换行
	`\u0020`	表示空格
	`\u2026`	表示省略号


## 其他
1. 自带浏览器对 `Adobe Flashplayer WebGL` CSS63D 的不友好支持，对WebGL有要求建议使用腾讯x5内核。
2. `gradle.versionName`等参数为空的时候 bugly 会报错，无法统计。
3. `Log.getStackTraceString(Throwable tr)`
    可以从Throwable对象中获取错误信息，并以字符串形式返回。当你需要错误信息的数据持久化时，这个方法很方便。（使用场景：保存至本地存储卡或上传服务器）
4. CountDownTimer 总时间最好加上16ms，不然一开始显示有问题。
5. 用`DateUtils.formayDateTime()`代替JDK中的`new SimpleDateFormat("yyy-MM-dd HH:mm").format()`。

### 双击返回键退出

```java
private long exitTime = 0;

@Override
   public boolean onKeyDown(int keyCode, KeyEvent event) {
       if (keyCode == KeyEvent.KEYCODE_BACK && event.getAction() == KeyEvent.ACTION_DOWN) {
          //两秒之内按返回键就会退出
           if ((System.currentTimeMillis() - exitTime) > 2000) {
               Toast.makeText(getApplicationContext(), "再按一次退出程序", Toast.LENGTH_SHORT).show();
               exitTime = System.currentTimeMillis();
           } else {
               finish();
               System.exit(0);
           }
           return true;
       }
       return super.onKeyDown(keyCode, event);
   }
```

### 代码混淆
1. 开启代码混淆
```
buildTypes {
  release {
    minifyEnabled false
    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-reles.pro'
  }
}
```

2. 添加混淆规则
```
-dontskipnonpubliclibraryclasses # 不忽略非公共的库类
-optimizationpasses 5 # 指定代码的压缩级别
-dontusemixedcaseclassnames # 是否使用大小写混合
-dontpreverify # 混淆时是否做预校验
-verbose # 混淆时是否记录日志
-keepattributes *Annotation* # 保持注解
-ignorewarning # 忽略警告
-dontoptimize # 优化不优化输入的类文件

-optimizations !code/simplification/arithmetic,!field/*,!class/merging/* # 混淆时所采用的算法

#保持哪些类不被混淆
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class com.android.vending.licensing.ILicensingService

#生成日志数据，gradle build时在本项目根目录输出
-dump class_files.txt #apk包内所有class的内部结构
-printseeds seeds.txt #未混淆的类和成员
-printusage unused.txt #打印未被使用的代码
-printmapping mapping.txt #混淆前后的映射

-keep public class * extends android.support.** #如果有引用v4或者v7包，需添加
-libraryjars libs/xxx.jar #混淆第三方jar包，其中xxx为jar包名
-keep class com.xxx.**{*;} #不混淆某个包内的所有文件
-dontwarn com.xxx** #忽略某个包的警告
-keepattributes Signature #不混淆泛型
-keepnames class * implements java.io.Serializable #不混淆Serializable

-keepclassmembers class **.R$* { #不混淆资源类
public static <fields>;
}
-keepclasseswithmembernames class * { # 保持 native 方法不被混淆
native <methods>;
}
-keepclasseswithmembers class * { # 保持自定义控件类不被混淆
public <init>(android.content.Context, android.util.AttributeSet);
}
-keepclasseswithmembers class * { # 保持自定义控件类不被混淆
public <init>(android.content.Context, android.util.AttributeSet, int);
}
-keepclassmembers class * extends android.app.Activity { # 保持自定义控件类不被混淆
public void *(android.view.View);
}
-keepclassmembers enum * { # 保持枚举 enum 类不被混淆
public static **[] values();
public static ** valueOf(java.lang.String);
}
-keep class * implements android.os.Parcelable { # 保持 Parcelable 不被混淆
public static final android.os.Parcelable$Creator *;
}
```


## 内存泄漏

### 常见的5个内存泄露问题

内存泄露时造成OOM的主要原因之一。由于Android系统为每个应用程序分配的内存有限，当一个应用中产生内存泄露较多时，就难免会导致应用所需要的内存超过分配的限额，这就导致应用Crash。



#### 1. 单例造成的内存泄露

由于单例的静态特性使得单例的生命周期和应用的生命周期一样长，也就说明如果一个对象已经不需要使用的时候，而单例对象还持有该对象的引用，那么这个对象将不能被正常回收，这就导致了内存泄露。
举例：

```java
//worng
public class AppManager {
    private static AppManager instance;
    private Context context;
    private AppManager(Context context) {
        this.context = context;
    }
    public static AppManager getInstance(Context context) {
        if (instance != null) {
            instance = new AppManager(context);
        }
        return instance;
    }
}
//right
public class AppManager {
    private static AppManager instance;
    private Context context;
    private AppManager(Context context) {
        this.context = context.getApplicationContext();
    }
    public static AppManager getInstance(Context context) {
        if (instance != null) {
            instance = new AppManager(context);
        }
        return instance;
    }
}
```


#### 2. 非静态内部类创建静态实例造成的内存泄露


#### 2.1 Handler造成的内存泄露
Handler正确的使用方式：
```java
private MyHandler mHandler = new MyHandler(this);
private static class MyHandler extends Handler {
        private WeakReference<Context> reference;
        public MyHandler(Context context) {
            reference = new WeakReference<>(context);
        }
        @Override
        public void handleMessage(Message msg) {
            Activity activity = (Activity) reference.get();
            if(activity != null){
                //do something
                //...
            }
        }
    }
    
    @Override
    protected void onDestroy() {
      super.onDestroy();
      mHandler.removeCallbackAndMessage(null);
    }
```


#### 3. 非静态内部类

#### 3.1 线程造成的内存泄露
```java
//test1
new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                SystemClock.sleep(10000);
                return null;
            }
        }.execute();
        
//test2
new Thread(new Runnable() {
            @Override
            public void run() {
                SystemClock.sleep(10000);
            }
        }).start();
```
上面的一部任务和Runnable都是一个匿名内部类，因此它们对当前Activity都有一个隐士引用。
如果Activity在销毁之前，任务还没完成，那么导致Activity的内存资源无法回收，造成内存泄露。
正确的做法，应该使用静态内部类的方式实现，如下：
```java
static class MyAsyncTask extends AsyncTask<Void, Void, Void> {
        private WeakReference<Context> weakReference;
  
        public MyAsyncTask(Context context) {
            weakReference = new WeakReference<>(context);
        }
  
        @Override
        protected Void doInBackground(Void... params) {
            SystemClock.sleep(10000);
            return null;
        }
  
        @Override
        protected void onPostExecute(Void aVoid) {
            super.onPostExecute(aVoid);
            MainActivity activity = (MainActivity) weakReference.get();
            if (activity != null) {
                //...
            }
        }
    }
    static class MyRunnable implements Runnable{
        @Override
        public void run() {
            SystemClock.sleep(10000);
        }
    }
    
    //-------
    new Thread(new MyRunnable()).start();
    new MyAsyncTask(this).execute();
```
这样就避免了Activity的内存资源泄露，当然在Activity销毁时也应该取消相应的任务`AsyncTask.cancel()`，避免任务在后台执行浪费的资源。

#### 4. 资源未关闭造成的内存泄露
对于BroadcastReceiver、File、Stream、 Cursor、Bitmap 等资源的使用，应该在Activity销毁时及时关闭或者注销，否则这些资源将不会被回收，造成内存泄露。