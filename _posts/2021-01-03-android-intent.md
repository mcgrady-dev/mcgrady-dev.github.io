---
layout: 
title: Android Intent
date: 2021-01-03 20:53 +0800
tags: android

---

Intent是Activity、Service、BroadcastReceiver 组件间通信的重要媒介。

- Intent为组件的启动提供了一致的编程模型。无论想要启动的组件是Activity,Service，还是BroadcastReceiver，都可以使用Intent封装启动的意图。
- 在某些时候，应用程序只是想启动具有某种特征的组件，并不想和某个特定的组件耦合。使用Intent可以方便的达到这种高层次解耦的目的。

<!--more-->

## Intent的显示和隐式启动

如果**Intent通过组件类名显式指明了唯一的目标组件，那么这个Intent就是显式的**，否则就是隐式，**隐式Intent一般只描述执行动作的类型**，必要时可以携带数据，系统会根据Intent的描述决定激活哪个组件，如果有多个组件符号激活条件，系统一般会弹出选择框让用选择到底激活哪个组件。

### 显示启动

当程序采用这种形式启动组件时，在Intent中明确的指定了待启动的组件类，此时的Intent属于显式intent，显式Intent应用场合比较狭窄，多用于启动本应用中的`component`，因为这种方式需要提前获知目标组件类的全限定名。

### 隐式启动

而隐式Intent则通过Intent中的`action` `category` `data`属性指定目标组件需要满足的若干条件，系统筛选出满足所有条件的`component`，从中选择最合适的`component`或者由用户选择一个`component`作为目标组件启动。
如果Intent中指定了`ComponentName`属性, 则Intent的其他属性将被忽略。



## Intent用来传递对象的两种实现方式

### Serializable

表示将一个对象转换成可存储或可传输的状态

```java
// 实现Serializable接口，将所有的Person对象序列化
public class Person implements Serializable {
    public String name;
    public int age;
    ...
}
// 在FirstActivity使用intent传递对象Person person = new Person();
person.name = "Tom";
person.age = 1;
Intent intent = new Intent(FirstActivity.this, SecondActivity.class);
intent.putExtra("person_data", person);
startActivity(intent);
// 在SecondActivity获取对象
Person person = (Person)getIntent().getSerializableExtra("person_data");
```

### Parcelable

将一个完整的对象进行分解，分解后的每一部分都是Intent所支持的数据类型。

```java
// 实现Parcelable接口，将所有的Person对象序列化
public class Person implements Parcelable {
    public String name;
    public int age;

    @Override
    public int describeContents() {
         return 0;
    }
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(name);    // 写出name
        dest.writeInt(age);    // 写出age
    }

    // 创建Parecelable。Creator接口的实现，并将泛型指定为Person，接着重写createFromParcel()和newArray()方法
    public static final Parcelable.Creator<Person> CREATOR = new Parcelable.Creator<Person>() {
        @Override
        public Person createFromParcel(Parcel source) {
            Person person new Person();

            // 注意：读取顺序需与写出的顺序相同
            person.name = source.readString();    // 读取name
            person.age = source.readInt();        // 读取age
            return person;
        }
        @Override
        public Person[] newArray(int size) {
            return new Person[size];
        }
   };
}
// 在SecondActivity使用Intent传递对象
Person person = (Person)getIntent().getParcelableExtra("person_data");
```



## Component属性

`setComponent(ComponentNamecomp)` 方法用于设置Intent的`Component`属性。

ComponentName包含如下几个构造器：

```java
ComponentName(String pkg, String cls);
ComponentName(Context pkg, String cls);
ComponentName(Context pkg, Class<?> cls);
```

由以上的构造器可知，创建一个ComponentName对象需要指定包名和类名，这就可以唯一确定一个组件类，这样应用程序即可根据给定的组件类去启动特定的组件。

```java
ComponentName comp = new ComponentName(FirstActivity.this,SecondActivity.class);
Intent intent = newIntent();
intent.setComponent(comp);

// 以上三句代码创建了一个intent对象, 并为其指定了Component属性, 完全等价于下面的代码
Intent intent = newIntent(FirstActivity.this,SecondActivity,class);
```

除了使用`setComponent()`之外, 还可以使用`setClass()` `setClassName()`来显式指定目标组件，还可以调用`getComponent()`方法获得Intent中封装的`ComponentName`对象。



## Action属性

action属性是一个字符串, **代表某一种特定的动作**。 
Intent类预定义了一些action常量, 开发者也可以自定义action。
一般来说, 自定义的action应该以application的包名作为前缀， 然后附加特定的大写字符串。

>例如`cn.xing.upload.action.UPLOAD_COMPLETE`就是一个命名良好的action。

`setAction()` 方法用于设定action
`getAction()` 方法可以获取Intent中封装的action

以下是Intent类中预定义的部分action：
`ACTION_VIEW` 是一个Android系统内置的动作。
`ACTION_CALL` 目标组件为activity，代表拨号动作。
`ACTION_EDIT` 目标组件为activity，代表向用户显示数据以供其编辑的动作。
`ACTION_MAIN` 目标组件为activity，表示作为task中的初始activity启动。
`ACTION_BATTERY_LOW` 目标组件为broadcastReceiver，提醒手机电量过低。
`ACTION_SCREEN_ON` 目标组件为broadcast，表示开启屏幕。



## Category属性

**用于指定一些目标组件需要满足的额外条件**。
Intent对象中可以包含任意多个category属性，Intent类预定义了一些category常量，开发者也可以自定义category属性。
`addCategory()` 方法为Intent添加`category`属性。
`getCategories()` 方法用于获取Intent中封装的所有`category`。
以下是Intent类中预定义的部分category：

- `CATEGORY_HOME` 表示目标activity必须是一个显示`homescreen`的activity。
- `CATEGORY_LAUNCHER` 表示目标activity可以作为`task`栈中的初始activity，
  常与`ACTION_MAIN`配合使用。
- `CATEGORY_GADGET` 表示目标activity可以被作为另一个activity的一部分嵌入。



## Data属性

**data属性指定所操作数据的URL**。
`setData()` 方法用于设置`data`属性
`setType()` 方法用于设置`data`的`MIME`类型
`setDataAndType()` 方法可以同时设定两者
`getData()` 方法获取`data`属性的值
`getType()` 方法获取`data`的`MIME`类型



## Extra属性

通过Intent启动一个`component`时，经常需要携带一些额外的数据过去。携带数据需要调用Intent的`putExtra()`方法，该方法存在多个重载方法，可用于携带一下多种等数据：

- 基本数据类型
- 数组
- ``String``
- Serializable
- Parcelable
- Bundle

Serializable和Parcelable类型代表一个可序列化的对象, Bundle与Map类似,可用于存储键值对。



## Flag属性

`flag`属性是一个`int`值，**用于通知android系统如何启动目标activity，或者启动目标activity之后应该采取怎样的后续操作**。所有的`flag`都在intent类中定义，部分常见`flag`如下：
`FLAG_ACTIVITY_NEW_TASK` 通知系统将目标activity作为一个新task的初始activity。
`FLAG_ACTIVITY_NO_HISTORY` 通知系统不要将目标activity放入历史栈中。
`FLAG_FROM_BACKGROUND` 通知系统这个Intent来源于后台操作，而非用户的直接选择。



## IntentFilter

IntentFilter 即 Intent过滤器，大部分情况下，每一个`component`都会定义一个或多个 IntentFilter，用于表明其可处理的Intent。
一般来说，`component`的 IntentFilter 应该在 `AndroidManifest.xml` 文件中定义。

```xml
<activityandroid:name=".FirstActivity">
       <intent-filter>
               <action android:name="android.intent.action.MAIN"/>
               <category android:name="android.intent.category.LAUNCHER"/>
       </intent-filter>
</activity>
```
