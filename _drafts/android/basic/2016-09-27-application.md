## 自定义 Application
创建一个类继承Application并在`AndroidManifest.xml`文件中的`application`标签中进行注册（只需要给application标签增加`name`属性，并添加自己的 `Application名字`即可）。

### 作用：
启动Application时，系统会创建一个`PID（进程ID）`，所有的Activity都会在此进程上运行。那么我们在Application创建的时候初始化全局变量，同一个应用的所有Activity都可以取到这些全局变量的值，换句话说，我们在某一个Activity中改变了这些全局变量的值，那么在同一个应用的其他Activity中值就会改变。

**Application对象的生命周期等于这个程序的生命周期。因为它是全局的单例的，所以在不同的Activity、Service中获得的对象都是同一个对象。所以可以通过Application来进行一些，如：数据传递、数据共享和数据缓存等操作**。

### 应用场景：
在Android中，**可以通过继承Application类来实现应用程序级的全局变量**，这种全局变量方法相对静态类更有保障，直到应用的所有Activity全部被destory掉之后才会被释放掉。
#### 实现步骤：
1. 继承Application
```java
public class CustomApplication extends Application
{
    private static final String VALUE = "Harvey";
   
    private String value;
   
    @Override
    public void onCreate()
    {
        super.onCreate();
        setValue(VALUE); // 初始化全局变量    }
   
    public void setValue(String value)
    {
        this.value = value;
    }
   
    public String getValue()
    {
        return value;
    }
}
```
>注：继承Application类，主要重写里面的onCreate（）方法（android.app.Application包的onCreate（）才是真正的Android程序的入口点），就是创建的时候，初始化变量的值。然后在整个应用中的各个文件中就可以对该变量进行操作了。

2. 在`ApplicationManifest.xml`文件中配置自定义的Application
```xml
<application
        android:name="CustomApplication">
</application>
```
实例代码：
```java
FirstActivity.java
public class FirstActivity extends Activity
{
    private CustomApplication app;
   
    @Override
    public void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
       
        app = (CustomApplication) getApplication(); // 获得CustomApplication对象       
        Log.i("FirstActivity", "初始值=====" + app.getValue()); // 获取进程中的全局变量值，看是否是初始化值       
        app.setValue("Harvey Ren"); // 重新设置值       
        Log.i("FirstActivity", "修改后=====" + app.getValue()); // 再次获取进程中的全局变量值，看是否被修改       
        Intent intent = new Intent();
        intent.setClass(this, SecondActivity.class);
        startActivity(intent);
    }
}
```
>注：只需要调用Context的 getApplicationContext或者Activity的getApplication方法来获得一个Application对象，然后再得到相应的成员变量即可。它是代表我们的应用程序的类，使用它可以获得当前应用的主题和资源文件中的内容等，这个类更灵活的一个特性就是可以被我们继承，来添加我们自己的全局属性。



