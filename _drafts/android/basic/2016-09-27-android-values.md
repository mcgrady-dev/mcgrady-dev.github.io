## attrs.xml

定义类的属性（声明自定义属性），这些属性会在类的构造函数中用到。
```xml
<declare-styleable name="Gallery">  
     <attr name="android:galleryItemBackground" />  
</declare-styleable>  
```

```java
TypedArray typeArray=obtainStyledAttributes(R.styleable.Gallery);  
int backgroudId=typeArray.getResourceId(R.styleable.Gallery_android_galleryItemBackground, 0);  
```
#### TypeArray
TypeArray：是一组值得容器，用来存放通过`obtainStyledAttributes(AttributeSet, int[], int, int)` or `obtainAttributes(AttributeSet, int[])` 得到的值。
调用结束后务必调用`recycle()`方法，否则这次的设定会对下次的使用造成影响。



## dimens.xml

定义尺寸。
```xml
<dimen name="activity_horizontal_margin">16dp</dimen>  
```
```java
android:paddingRight="@dimen/activity_horizontal_margin"  
```


## strings.xml

定义使用到的字符串变量。



## styles.xml

定义各个控件的样式，样式由一个个属性组成。可在布局文件中引用。



## color.xml

定义各种颜色值。



