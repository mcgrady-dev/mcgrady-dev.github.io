---
layout: 
title: Activity
date: 2022-03-09 11:41 +0800
tags: android
---

Android的界面是由布局和组件协同完成的，布局好比是建筑里的框架，而组件则相当于建筑里的砖瓦。组件按照布局的要求依次排列，就组成了用户所看见的界面。


<!--more-->

![Alt text](./2012011916121443.jpg)



## LinearLayout (线性布局)

- 以线性方向显示它的子视图(View)元素，即垂直或者水平的顺序依次排列子元素，每一个子元素都位于前一个元素之后（每个元素占一行）。
  ![Alt text](./linearlayout.png)

### android:layout_weight
`android:layout_weight`的含义：**一旦Viwe设置了该属性（假设有效的情况下），那么该View的宽度等于原有宽度(`android:layout_width`)加上剩余空间的占比**。
- `android:layout_weight`只在LinearLayout中生效。
- 当`android:layout_width`设置为`wrap_content`和`match_parent`会造成两种截然相反的效果。

>Google官方推荐，当使用weight属性时，将width设为0dip即可，效果跟设成wrap_content是一样的。这样weight就可以理解为占比了！
>先设置`android:layout_width="0dp"`，然后在再去调配比例。


## RelativeLayout (相对布局)
![Alt text](./relativelayout.png)
- 以相对位置显示它的子视图(view)元素，即按照各子元素之间的位置关系完成布局。
- 在此布局中的子元素里与位置相关的属性将生效。例如：
  - `android:layout_below`
  - `android:layout_above`
  - `android:layout_centerVertical`
>注意在指定位置关系时，引用的ID必须在引用之前先被定义，否则将出现异常。


```
// 相对于给定ID控件
android:layout_above		将该控件的底部置于给定ID的控件之上;
android:layout_below		将该控件的底部置于给定ID的控件之下;
android:layout_toLeftOf		将该控件的右边缘与给定ID的控件左边缘对齐;
android:layout_toRightOf	将该控件的左边缘与给定ID的控件右边缘对齐;

android:layout_alignBaseline	将该控件的baseline与给定ID的baseline对齐;
android:layout_alignTop			将该控件的顶部边缘与给定ID的顶部边缘对齐;
android:layout_alignBottom		将该控件的底部边缘与给定ID的底部边缘对齐;
android:layout_alignLeft        将该控件的左边缘与给定ID的左边缘对齐;
android:layout_alignRight		将该控件的右边缘与给定ID的右边缘对齐;

// 相对于父组件
android:layout_alignParentTop		如果为true,将该控件的顶部与其父控件的顶部对齐;
android:layout_alignParentBottom	如果为true,将该控件的底部与其父控件的底部对齐;
android:layout_alignParentLeft		如果为true,将该控件的左部与其父控件的左部对齐;
android:layout_alignParentRight		如果为true,将该控件的右部与其父控件的右部对齐;

// 居中
android:layout_centerHorizontal		如果为true,将该控件的置于水平居中;
android:layout_centerVertical		如果为true,将该控件的置于垂直居中;
android:layout_centerInParent		如果为true,将该控件的置于父控件的中央;

// 指定移动像素
android:layout_marginTop		上偏移的值;
android:layout_marginBottom		下偏移的值;
android:layout_marginLeft		左偏移的值;
android:layout_marginRight		右偏移的值;
example:
android:layout_below = "@id/***"
android:layout_alignBaseline = "@id/***"
android:layout_alignParentTop = true
android:layout_marginLeft = “10px”

android:layout_contentDescription	为视力有障碍的人增加对控件的解释
```

## TableLayout (表格布局)
- 适用于N行N列的布局格式。
- 一个TableLayout由许多TableRow组成，一个TableRow就代表TableLayout中的一行。
  - TableRow是LinearLayout的子类，ablelLayout并不需要明确地声明包含多少行、多少列，而是通过TableRow，以及其他组件来控制表格的行数和列数， TableRow也是容器，因此可以向TableRow里面添加其他组件，没添加一个组件该表格就增加一列。如果想TableLayout里面添加组件，那么该组件就直接占用一行。在表格布局中，列的宽度由该列中最宽的单元格决定，整个表格布局的宽度取决于父容器的宽度（默认是占满父容器本身）。
- 常用属性
```
android:collapseColumns:		以第0行为序，隐藏指定的列
android:collapseColumns:		该属性为空时
```


　　

- TableLayout继承了LinearLayout，因此他完全可以支持LinearLayout所支持的全部XML属性，除此之外TableLayout还支持以下属性：

| XML属性                   |                              相关用法 |          说明           |
| :---------------------- | --------------------------------: | :-------------------: |
| andriod：collapseColumns | setColumnsCollapsed（int ，boolean） | 设置需要隐藏的列的序列号，多个用逗号隔开  |
| android：shrinkColumns   |      setShrinkAllColumns（boolean） |  设置被收缩的列的序列号，多个用逗号隔开  |
| android：stretchColimns  |     setSretchAllColumnds（boolean） | 设置允许被拉伸的列的序列号，多个用逗号隔开 |

## AbsoluteLayout (绝对布局)
- 在此布局中的子元素的`android:layout_x`和`android:layout_y`属性将生效，用于描述该子元素的坐标位置。
- 屏幕左上角为坐标原点（0,0），第一个0代表横坐标，向右移动此值增大，第二个0代表纵坐标，向下移动，此值增大。
- 在此布局中的子元素可以相互重叠。
>在实际开发中，通常不采用此布局格式，因为它的界面代码过于刚性，以至于有可能不能很好的适配各种终端。
## FrameLayout (帧布局)
- 五大布局中最简单的一个布局，可以说成是层布局方式。
- 在这个布局中，整个界面被当成一块空白备用区域，所有的子元素都不能被指定放置的位置，它们统统放于这块区域的左上角，并且后面的子元素直接覆盖在前面的子元素之上，将前面的子元素部分和全部遮挡。

## 其他布局
### 列表视图（List View）
- List View是可滚动的列表。以列表的形式展示具体内容，并且能够根据数据的长度自适应显示。
  具体应用请看：
  	[用法一](http://www.cnblogs.com/allin/archive/2010/05/11/1732200.html)
  	[用法二](http://blog.csdn.net/koupoo/article/details/7018727)

### 网格视图（Grid View）
- 以网格显示它的子视图（view）元素，即二维的、滚动的网格。

　[具体应用查看](http://www.cnblogs.com/linzheng/archive/2011/01/19/1938760.html)

### 标签布局（Tab Layout）
- 以标签的方式显示它的子视图元素，就像在Firefox中的一个窗口中显示多个网页一样。为了狂创建一个标签UI（tabbed UI），需要使用到TabHost和TabWidget。
- TabHost必须是布局的根节点，它包含为了显示标签的TabWidget和显示标签内容的FrameLayout。

  [具体应用查看](http://www.cnblogs.com/devinzhang/archive/2012/01/18/2325887.html)




## 几个常用的布局属性

- **android:layout_width/height**
  用于设置空间的高度和宽度
  `wrap_content`: 内容包裹，表示这个控件里面的文字大小填充
  `match_parent/fill_parent`: 跟随父类窗口

- **android:layout_margin**
  用于设置控件边缘相对于父控件的边距
  `android:layout_marginLeft`
  `android:layout_marginRight`
  `android:layout_marginTop`
  `android:layout_marginBottom`


- **android:layout_padding**
  用于设置控件内容相对于控件边缘的边距
  `android:layout_paddingLeft`
  `android:layout_paddingRight`
  `android:layout_paddingTop`
  `android:layout_paddingBottom`


- **android:layout_gravity**
  用于本元素和某元素的各方向边缘对齐方式
  `android:layout_alignTop`
  `android:layout_alignLeft`
  `android:layout_alignBottom`
  `android:layout_alignRight`

- **gravity**
  用于设置View组件里面内容的对齐方式
  `top` `bottom` `left` `right` `center`
  
- android:adjustVieBounds
  
  使用 ImageView 时，你可能会用 `android:scaleType` 属性设置图片缩放方式。殊不知，`android:adjustViewBounds` 属性也能起到类似的效果。但要注意的是，后者需要至少指定 ImageView 宽高中的一个属性，或者 maxHeight 之类的，然后另一个属性随之适配。这个属性用在列表中较为合适，比如 App 中的活动列表页面，图片宽度设置为 match_parent，然后高度设为 wrap_content 使其自适应，这样便能保证从服务获取的高分辨率图片在不同的屏幕中不被拉伸变形。（备注：最好在项目资源文件中放置一个与网络图片相同尺寸的默认图，起到 placeholder 作用，避免图片显示前高度为 0 的较差体验。）



