# Development Notes

[TOC]




### Space 使用场景
Space 是一个用于创建视图之间空隙的轻量级View，再onDraw()中不执行任何绘制，所以`android:background` 属性对它不起作用。

通常在创建视图空隙的时候，**在不考虑背景色的情况下，Space效率更高**。

> 注意：由于控件是 API 14 引入，如要向下兼容，需要使用**v4.widget.Space**


### view.performClick()
自动调用View点击事件。用于模拟用户点击行为。
> 类似的还有`performLongClick()`。

### 系统内置方法/工具
#### DateUtils
`DateUtils.formayDateTime()`代替JDK中的`new SimpleDateFormat("yyy-MM-dd HH:mm").format()`。
几个常用的format格式：
- FORMAT_SHOW_DATE：3月3日
- FORMAT_SHOW_TIME：10:37
- FORMAT_SHOW_WEEKDAY：星期五
- FORMAT_SHOW_YEAR：2017年3月3日
- FORMAT_NUMERIC_DATE：3/3
- FORMAT_NO_MONTH_DAY：三月
#### TextUtils
`TextUtils.isEmpty(CharSequence str)`

#### Log
`Log.getStackTraceString(Throwable tr)`可以从Throwable对象中获取错误信息，并以字符串形式返回。当你需要错误信息的数据持久化时，这个方法很方便。
使用场景：保存至本地存储卡或上传服务器。


### 5个常见的内存泄露的问题
内存泄露时造成OOM的主要原因之一。由于Android系统为每个应用程序分配的内存有限，当一个应用中产生内存泄露较多时，就南偏灰导致应用所需要的内存超过分配的限额，这就导致应用Crash。
#### 单例造成的内存泄露
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
#### 非静态内部类创建静态实例造成的内存泄露


#### Handler造成的内存泄露
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
#### 非静态内部类
#### 线程造成的内存泄露
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
#### 资源未关闭造成的内存泄露
对于BroadcastReceiver、File、Stream、 Cursor、Bitmap 等资源的使用，应该在Activity销毁时及时关闭或者注销，否则这些资源将不会被回收，造成内存泄露。

### 资源相关
#### string xml 
<b></b> 加粗字体
<i></i> 斜体字体
<u></u> 给字体加下划线
\n 换行
\u0020 表示空格
\u2026 表示省略号


#### `attr format`属性类型
- reference：参考某一资源ID

```xml
<ImageView
  Android:layout_width = "42dip"
  android:layout_height = "42dip"
  android:background = "@drawable/图片ID"/>
```

- `color`：颜色值

```xml
<TextView
  android:layout_width = "42dip"
  android:layout_height = "42dip"
  android:textColor = "#00FF00"/>
```

- boolean：布尔值
- dimension：尺寸值

```xml
<Button
  android:layout_width = "42dip"
  android:layout_height = "42dip"/>
```

- `float`：浮点值

```xml
<alpha
  android:fromAlpha = "1.0"
  android:toAlpha = "0.7"/>
```
- `integer`：整形值

```xml
<animated-rotate xmlns:android = "http://schemas.android.com/apk/res/android"  
  android:drawable = "@drawable/图片ID"  
  android:pivotX = "50%"  
  android:pivotY = "50%"  
  android:framesCount = "12"  
  android:frameDuration = "100"/>
```
- `string`：字符串
- `fraction`：百分数
- `enum`：枚举值
- `flag`：位或运算
```xml
<activity
  android:name = ".StyleAndThemeActivity"
  android:label = "@string/app_name"
  android:windowSoftInputMode = "stateUnspecified | stateUnchanged　|　stateHidden">
    <intent-filter>
      <action android:name = "android.intent.action.MAIN" />
      <category android:name = "android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

####SpannableString与SpannableStringBuilder
类似于String和StringBuilder
SpannableString 长度不可变
SpannableStringBuilder 长度可变

#### EditText 默认取消焦点

#### EEE
```xml
android:background="@color/colorPrimary"  
android:background="@com.myapp:color/colorPrimary"  
android:background="?colorPrimary"  
android:background="?attr/colorPrimary"  
android:background="?com.myapp:attr/colorPrimary"  
android:background="?com.myapp:colorPrimary"  
android:background="?android:colorPrimary"  
android:background="?android:attr/colorPrimary"
```

#### `@`和`?`之间的区别
@ 标记是引用一个实际的值（color, string, dimension...）。
? 标记是引用一个`style attribute`，其值取决于当前使用的主题。

#### InputFilter
edittext maxLength 在使用InputFilter 之后会失效

#### 什么是线程安全？
线程安全是指多线程访问统一代码，不会产生不确定的结果。

#### Lambda 表达式