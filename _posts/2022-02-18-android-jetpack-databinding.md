---
layout: 
title: Android DataBinding
date: 2022-02-18 10:14 +0800
tags: [android,jetpack]
---

数据绑定可以使用声明性格式将布局中的界面组件绑定到应用中的数据源。

```xml
<TextView
          android:text="@{viewmodel.userName}" />
```

<!--more-->

### 什么是双向绑定？

**单向绑定**
DataBinding 本身是对 View 层状态的一种观察者模式的实现，通过让 View 与 ViewModel 层可观察的对象（比如 LiveData）进行绑定，当 ViewModel 层数据发生变化，View 层也会自动进行 UI 的更新。

**双向绑定**
除了数据驱动视图之外，视图也在驱动数据，让 View 层和 ViewModel 层保持状态同步。



## 绑定适配器

### BindingMethods

指定自定义方法名称，像一些属性具有名称不符的 setter 方法，可以使用 BindingMethods 注释与 setter 相关联。

```kotlin
@BindingMethods(value = [
    BindingMethod(
        type = 包名.XXXView::class,
        attribute = "app:borderColor",
        method = "setBColor")])
```

### BindingAdapter

一些属性需要自定义绑定逻辑时，使用 BindingAdapter 注释的静态绑定适配器方法支持自定义特性 setter 的调用方式。

```kotlin
@BindingAdapter("addTextChangedListener")
    fun addTextChangedListener(editText: EditText, simpleWatcher: SimpleWatcher) {
        editText.addTextChangedListener(simpleWatcher)
    }
```

第一个参数用于确定与特性关联的视图类型，第二个参数用于确定给特性的绑定表达式中接受的类型。

还可以使用接受多个属性的适配器：

```kotlin
    @BindingAdapter("imageUrl", "error")
    fun loadImage(view: ImageView, url: String, error: Drawable) {
        Picasso.get().load(url).error(error).into(view)
    }
```

```xml
<ImageView 
           app:imageUrl="@{venue.imageUrl}" 
           app:error="@{@drawable/venueError}" />
```

如果 ImageView 对象同时使用了 imageUrl 和 error ，并且 imageUrl 是字符串， error 是 Drawable ，就会调用适配器。

如果希望在设置了任意属性时调用适配器，则可以将适配器的可选 requireAll 标志设置为 false：

```kotlin
    @BindingAdapter(value = ["imageUrl", "placeholder"], requireAll = false)
    fun setImageUrl(imageView: ImageView, url: String?, placeHolder: Drawable?) {
        if (url == null) {
            imageView.setImageDrawable(placeholder);
        } else {
            MyImageLoader.loadInto(imageView, url, placeholder);
        }
    }
```



## 将布局视图绑定到架构组件

在某些情况下，您可能更愿意使用实现 [`Observable`](https://developer.android.com/reference/androidx/databinding/Observable) 接口的 [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel) 组件，而不是使用 `LiveData` 对象，即使这样会失去对 `LiveData` 的生命周期管理功能也不影响。使用实现 `Observable` 的 `ViewModel` 组件可让您更好地控制应用中的绑定适配器。例如，这种模式可让您更好地控制数据更改时发出的通知，您还可以指定自定义方法来设置双向数据绑定中的属性值。

```kotlin
    /**
     * A ViewModel that is also an Observable,
     * to be used with the Data Binding Library.
     */
    open class ObservableViewModel : ViewModel(), Observable {
        private val callbacks: PropertyChangeRegistry = PropertyChangeRegistry()

        override fun addOnPropertyChangedCallback(
                callback: Observable.OnPropertyChangedCallback) {
            callbacks.add(callback)
        }

        override fun removeOnPropertyChangedCallback(
                callback: Observable.OnPropertyChangedCallback) {
            callbacks.remove(callback)
        }

        /**
         * Notifies observers that all properties of this instance have changed.
         */
        fun notifyChange() {
            callbacks.notifyCallbacks(this, 0, null)
        }

        /**
         * Notifies observers that a specific property has changed. The getter for the
         * property that changes should be marked with the @Bindable annotation to
         * generate a field in the BR class to be used as the fieldId parameter.
         *
         * @param fieldId The generated BR id for the Bindable field.
         */
        fun notifyPropertyChanged(fieldId: Int) {
            callbacks.notifyCallbacks(this, fieldId, null)
        }
    }
```



## 双向数据绑定

使用单向数据绑定时，您可以为特性设置值，并设置对该特性的变化作出反应的监听器：

```xml
    <CheckBox
        android:id="@+id/rememberMeCheckBox"
        android:checked="@{viewmodel.rememberMe}"
        android:onCheckedChanged="@{viewmodel.rememberMeChanged}"
    />
```

双向数据绑定为此过程提供了一种快捷方式：

```xml
    <CheckBox
        android:id="@+id/rememberMeCheckBox"
        android:checked="@={viewmodel.rememberMe}"
    />
```

`@={}` 表示法可接收属性的数据更改并同时监听用户更新。

为了对后台数据的变化作出反应，您可以将您的布局变量设置为 `Observable`（通常为 [`BaseObservable`](https://developer.android.com/reference/androidx/databinding/BaseObservable)）的实现，并使用 [`@Bindable`](https://developer.android.com/reference/androidx/databinding/Bindable) 注释，如以下代码段所示：

```kotlin
    class LoginViewModel : BaseObservable {
        // val data = ...

        @Bindable
        fun getRememberMe(): Boolean {
            return data.rememberMe
        }

        fun setRememberMe(value: Boolean) {
            // Avoids infinite loops.
            if (data.rememberMe != value) {
                data.rememberMe = value

                // React to the change.
                saveData()

                // Notify observers of a new value.
                notifyPropertyChanged(BR.remember_me)
            }
        }
    }
```

### 使用自定义特性的双向数据绑定

如果您希望结合使用双向数据绑定和自定义特性，则需要使用 [`@InverseBindingAdapter`](https://developer.android.com/reference/androidx/databinding/InverseBindingAdapter) 和 [`@InverseBindingMethod`](https://developer.android.com/reference/androidx/databinding/InverseBindingMethod) 注释。

1. 使用 `@BindingAdapter` 对用来设置初始值并在值更改时进行更新的方法进行注释：

   ```kotlin
       @BindingAdapter("time")
       @JvmStatic fun setTime(view: MyView, newValue: Time) {
           // Important to break potential infinite loops.
           if (view.time != newValue) {
               view.time = newValue
           }
       }
   ```

2. 使用 `@InverseBindingAdapter` 对从视图中读取值的方法进行注释：

   ```kotlin
       @InverseBindingAdapter("time")
       @JvmStatic fun getTime(view: MyView) : Time {
           return view.getTime()
       }
   ```

3. 在视图上设置监听器，由于它不知道特性何时或如何更改：

   ```kotlin
       @BindingAdapter("app:timeAttrChanged")
       @JvmStatic fun setListeners(
               view: MyView,
               attrChange: InverseBindingListener
       ) {
           // Set a listener for click, focus, touch, etc.
       }
   ```

   该监听器包含一个 InverseBindingListener ，可以告知数据绑定系统，特性已更改。然后系统开始调用 `@InverseBindingAdapter` 注释的方法。

### 转换器

```kotlin
<EditText
        android:id="@+id/birth_date"
        android:text="@={Converter.dateToString(viewmodel.birthDate)}"
    />
```

由于使用了双向表达式，因此还需要使用反向转换器，以告知库如何将用户提供的字符串转换回后备数据类型，通过向转换器添加 `@InverseMethod` 注释并让此注释引用反向转换器来完成。

```kotlin
    object Converter {
        @InverseMethod("stringToDate")
        @JvmStatic fun dateToString(
            view: EditText, oldValue: Long,
            value: Long
        ): String {
            // Converts long to String.
        }

        @JvmStatic fun stringToDate(
            view: EditText, oldValue: String,
            value: String
        ): Long {
            // Converts String to long.
        }
    }
```

### 使用双向数据绑定的无限循环

使用双向数据绑定时，请注意不要引入无限循环。当用户更改特性时，系统会调用使用 `@InverseBindingAdapter` 注释的方法，并且该值将分配给后备属性。继而调用使用 `@BindingAdapter` 注释的方法，从而触发对使用 `@InverseBindingAdapter` 注释的方法的另一个调用，依此类推。

因此，通过比较使用 `@BindingAdapter` 注释的方法中的新值和旧值，可以打破可能出现的无限循环。



## 表达式语言

### 常见功能

```xml
<TextView
          android:text="@{String.valueOf(index + 1)}"
          android:visibility="@{age > 13 ? View.GONE : View.VISIBLE}"
          android:transitionName="@{"image_" + id}" />
```

- 算术运算符 `+ - / * %`
- 字符串连接运算符 `+`
- 逻辑运算符 `&& ||`
- 二元运算符 `& | ^`
- 一元运算符 `+ - ! ~`
- 移位运算符 `>> >>> <<`
- 比较运算符 `== > < >= <=`（请注意，`<` 需要转义为 `<`）
- `instanceof`
- 分组运算符 `()`
- 字面量运算符 - 字符、字符串、数字、`null`
- 类型转换
- 方法调用
- 字段访问
- 数组访问 `[]`
- 三元运算符 `?:`

### Null 合并运算符

```
android:text="@{user.displayName ?? user.lastName}"
```

等价于

```
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```

### 避免出现 Null 指针异常

生成的数据绑定代码会自动检查有没有 `null` 值并避免出现 Null 指针异常。例如，在表达式 `@{user.name}` 中，如果 `user` 为 Null，则为 `user.name` 分配默认值 `null`。如果您引用 `user.age`，其中 age 的类型为 `int`，则数据绑定使用默认值 `0`。



## 事件处理

### onClick 的几种实现方式

1. **`@{click}`**

   ```xml
   android:onClick="@{click}"
   ```

   ```kotlin
   fun void click(view: View) {
   }
   ```

2. **`@{() -> viewModel.click()}`**

   ```xml
   android:onClick="@{() -> viewModel.click()}"
   ```

   ```kotlin
   fun click() {
   }
   ```

3. **`@{viewModel::click}`**

   ```xml
   android:onClick="@{viewModel::click}"
   ```

   ```kotlin
   fun click(view: View) {
   }
   ```

4. 带参数 **`@{() -> viewModel.click(obj.id)}`**

   ```xml
   android:onClick="@{() -> viewModel.click(obj.id)}"
   ```

   ```kotlin
   fun click(id: Long) {
   }
   ```

5. **`ObservableField<OnClickListener>`**

   ```xml
   <variable
          name="iconView"
          type="com.xxxxx.IconView" />
   
   android:onClick="@{iconView.clickListener}"
   ```

   ```kotlin
   val clickListener = ObservableField<OnClickListener>()
   binding.iconView.clickListener.set { view ->
   }
   ```

6. 带View的参数 **`@{(view) -> listener.onCheckBox(obj, view)}`**

   ```xml
   android:onClick="@{(view)->listener.onCheckBoxClick(obj,view)}"
   ```

   ```kotlin
   fun onCheckBoxClick(entity: Object, v: View)
   ```

   



## 使用可观察的数据对象

可观察性是指一个对象将其数据变化告知其他对象的能力。通过数据绑定库，您可以让对象、字段或集合变为可观察。

### 可观察字段

- [`ObservableBoolean`](https://developer.android.com/reference/android/databinding/ObservableBoolean)
- [`ObservableByte`](https://developer.android.com/reference/android/databinding/ObservableByte)
- [`ObservableChar`](https://developer.android.com/reference/android/databinding/ObservableChar)
- [`ObservableShort`](https://developer.android.com/reference/android/databinding/ObservableShort)
- [`ObservableInt`](https://developer.android.com/reference/android/databinding/ObservableInt)
- [`ObservableLong`](https://developer.android.com/reference/android/databinding/ObservableLong)
- [`ObservableFloat`](https://developer.android.com/reference/android/databinding/ObservableFloat)
- [`ObservableDouble`](https://developer.android.com/reference/android/databinding/ObservableDouble)
- [`ObservableParcelable`](https://developer.android.com/reference/android/databinding/ObservableParcelable)

可观察字段是具有单个字段的自包含可观察对象。原语版本避免在访问操作期间封箱和开箱。如需使用此机制，请采用 Java 编程语言创建 `public final` 属性，或在 Kotlin 中创建只读属性，如以下示例所示：

```kotlin
class User {
        val firstName = ObservableField<String>()
        val lastName = ObservableField<String>()
        val age = ObservableInt()
    }
```



### 可观察集合

- ObservableArrayMap
- ObservableList



### 可观察对象

实现 [`Observable`](https://developer.android.com/reference/android/databinding/Observable) 接口的类允许注册监听器，以便它们接收有关可观察对象的属性更改的通知。为便于开发，数据绑定库提供了用于实现监听器注册机制的 [`BaseObservable`](https://developer.android.com/reference/android/databinding/BaseObservable) 类。实现 `BaseObservable` 的数据类负责在属性更改时发出通知。具体操作过程是向 getter 分配 [`Bindable`](https://developer.android.com/reference/android/databinding/Bindable) 注释，然后在 setter 中调用 [`notifyPropertyChanged()`](https://developer.android.com/reference/android/databinding/BaseObservable#notifypropertychanged) 方法，如以下示例所示：

```kotlin
    class User : BaseObservable() {

        @get:Bindable
        var firstName: String = ""
            set(value) {
                field = value
                notifyPropertyChanged(BR.firstName)
            }

        @get:Bindable
        var lastName: String = ""
            set(value) {
                field = value
                notifyPropertyChanged(BR.lastName)
            }
    }
```

数据绑定在模块包中生成一个名为 `BR` 的类，该类包含用于数据绑定的资源的 ID。在编译期间，[`Bindable`](https://developer.android.com/reference/android/databinding/Bindable) 注释会在 `BR` 类文件中生成一个条目。如果数据类的基类无法更改，[`Observable`](https://developer.android.com/reference/android/databinding/Observable) 接口可以使用 [`PropertyChangeRegistry`](https://developer.android.com/reference/android/databinding/PropertyChangeRegistry) 对象实现，以便有效地注册和通知监听器。
