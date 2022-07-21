---
title: Android编码规范
date: 2018-01-23 21:30:50
updated: 2021-12-31 18:30:50
tags: [android, tips]
categories: android
---

## 1. 摘要

- 把密码和敏感数据放在`gradle.properties`中，并在版本控制中忽略掉

## 2. 命名规范

### 2.1 命名规则

#### 2.1.1 保持命名形式一致性

对于代码中同样的事物使用相同的名称，同时应该遵循用户的习惯用语。很多人写变量名称可能会忘记之前的命名方式，从而又来一个新的名称。比如 `Image` 对象，可能在 A 文件称之为 `image`，在 B 文件又称之为 `picture`，这样的话会让人很困惑。 

```dart
// 正确示例
pageCount         // 表示页数的字段
updatePageCount() // 与 pageCount 名称保持一致
toSomething()     // 与 Iterable 的 toList() 的写法一致
asSomething()     // 与 List 的 toMap() 写法一致
Point             // 用户习惯用语
  
// 错误示例
renumberPages()   // 与 pageCount 名称没有一致
convertToSomething()  // 与通常的 toX()不一致
wrappedAsSomething()  // 与通常的 asX() 不一致
Cartesian							// 不是用户的习惯用语
```

#### 2.1.2 避免缩写

除非缩写是众所周知的的词汇，否则不要缩写。即便是使用缩写，也应该遵循习惯的大小写规则。

```dart
// 正确示例
pageCount
buildRectangles
IOStream
HttpRequest
  
// 错误示例
numPages    // num 缩写不合适
buildRects  // Rects 缩写很难和 Rectangles 对应
InputOutputStream  // 没有使用简单易懂的缩写
HypertextTransferProtocolRequest // 没有使用简单易懂的缩写
```

#### 2.1.3 将描述事物最准确的名词放在最后

当有多个名词组合来描述一个对象时，将最贴近事物对应的名词放在最后面，这样在语义上更好理解变量对应的是一个什么样的事物。

```dart
// 正确示例
pageCount             // 页码的数量（count）
ConversionSink        // 用于转换的 sink
ChunkedConversionSink //  一个chunked 的 Conversation Sink
CssFontFaceRule       // CSS 中字体规则（rule）
  
// 错误示例
numPages							   // 并不是 pages 的集合
CanvasRenderingContext2D // 并不是 2D 对象
RuleFontFaceCss					 // 并不是 CSS
```

#### 2.1.4 将代码像语句一样连贯

让代码像语句一样连贯时，读起来的时候就能够知道代码的具体意思了 —— 此时就无需写注释了！，比如下面的例子，一看代码就知道是在做什么事情。

```dart
// 正确示例
if (errors.isEmpty) ...

subscription.cancel();

monsters.where((monster) => monster.hasClaws);

// 错误示例
// 是清空 error 还是盘点 errors 是不是空？
if (errors.empty) ...

// 反转什么？反转后是什么状态？
subscription.toggle();

// 是过滤包含的还是不包含满足条件的？
monsters.filter((monster) => monster.hasClaws);
```

#### 2.1.5 对于非布尔类型的属性或变量使用名词

因为是一个属性，如果不是布尔类型的话，肯定会是某个对象，因此使用名词能够更清晰地表名属性或变量对应的事物。同时，如果涉及操作后的事物，应该用形容词或动词的完成时（表示已完成）来修饰，而不是直接用动词+名词，这样会误以为是操作一个对象的动作。

```dart
// 正确示例
list.length
context.lineWidth
quest.rampagingSwampBeast
sortedList
  
// 错误示例
list.deleteItems
sortList
```

#### 2.1.6 对于代表可执行某类操作布尔属性，优先使用能动词

我们经常会用布尔值标识一个对象能进行什么操作，比如是否能够更新，是否能够关闭。推荐使用能动词，这样的语义更加清晰。而动词转换后的形容词其实语义并不清晰，而且有些词汇并不那么直观。

```dart
// 正确示例
if (widget.canUpdate) ...
if (window.canClose) ...
if (container.hasElements) ...
if (request.shouldResume) ...

// 错误示例
if (widget.updatable) ...
if (window.closeable) ...
if (container.withElements) ...
if (request.resumable) ...
```

#### 2.1.7 对于布尔类型的参数，不要使用动词

对于布尔型函数参数，不使用动词，而是形容词会更易读。

```dart
// 正确示例
Isolate.spawn(entryPoint, message, paused: false);
var copy = List.from(elements, growable: true);
var regExp = RegExp(pattern, caseSensitive: false);
```

#### 2.1.8 对于布尔型属性和变量，应该使用正向名称

这个说起来有点抽象，实际看一个例子就知道了。当你用反向属性名称的时候，赋布尔值的时候理解起来会很r绕。比如 `isNotEmpty = true`，你得思考1-2秒才能够明白什么意思。而实际上，使用反向布尔值的，在写条件表达式的时候，一旦脑子短路，很容易导致 bug 出现。

```dart
// 正确示例
bool isEmpty;

// 错误示例
bool isNotEmpty;
```

#### 2.1.9 使用祈使动词来命名函数或那些产生其他效果的方法

```dart
// 正确示例
var element = list.elementAt(3);
var first = list.firstWhere(test);
var char = string.codeUnitAt(4);
```

#### 2.1.10 不要在方法名称开头加 `get`

大部分情况下，应当移除 `get`。例如，与其使用 `getBreakfastOrder`，还不如定一个一个 `getter` 属性，名称为 `breakfastOrder`。如果确实需要方法获取一个对象，那么也应该避免使用 get，可以参考下面两条建议：

- 如果调用者更关系返回值，那么可以直接去掉 get，使用名词作为方法名词，例如`breakfastOrder()`，或者使用规则11。
- 如果调用这关心这个方法做什么事情，那么可以用更精确的描述动词，例如 `create`、`download`、`fetch`、`calculate`、`request` 等等。

#### 2.1.11 如果方法是将一个对象复制成为另一种表现形式，那么使用 `toX()` 形式

这和下面的规则14可能觉得有点不好区分，其实这个意思就是这样的操作是将对象的转为新的一种形式表述的对象。比如我们常见的 `toJson`，`toString`，`toSet`，`toLocal` 形式。这种方式大部分是不可逆的。

#### 2.1.12 如果返回的是原对象的不同的表现形式，那么使用 `asX()` 形式

```dart
if ((object as String) == 'a') {
  ...
}
```

asX 形式也是类似的，转换后的对象其实还能够反映原对象的特性，也就是可以通过一定的手段再转换回来。例如下面的情况：

```dart
var map = table.asMap();
var list = bytes.asFloat32List();
var future = subscription.asFuture();
```

#### 2.1.13 不要在方法名称上重复参数名

参数名可以和方法的动词连起来读，添加参数名有点多余。

```dart
// 正确示例
list.add(element);
map.remove(key);

// 错误示例
list.addElement(element)
map.removeKey(key)
```

这个不是绝对的，比如如果加上参数能够消除同一个类不同函数的含义的话，那么是可以的，例如下面的例子。

```dart
// 正确示例
map.containsKey(key);
map.containsValue(value);
```

#### 2.1.14 对于泛型，遵循通用的助记符

通常在泛型中，我们会用如下的助记符来表示不同的泛型参数，需要遵循这样的规则来帮助理解泛型。

- E 表示集合类的泛型参数

  ```dart
  class IterableBase<E> {}
  class List<E> {}
  class HashSet<E> {}
  ```

- K 和 V 表示 key 和 value 泛型参数

  ```dart
  class Map<K, V> {}
  class MapEntry<K, V> {}
  ```

- R 表示返回值泛型参数

  ```dart
  abstract class ExpressionVisitor<R> {
    R visitBinary(BinaryExpression node);
    
    R visitLiteral(LiteralExpression node);
    
    R visitUnary(UnaryExpression node);
  }
  ```

- 其它情况，通常使用 `T`、S 和 `U` 来标识泛型，实际中如果能够有助于理解，也可以使用单词（首字母大写）作为泛型参数，下面的3个例子都是没问题的。

  ```dart
  class Future<T> {
    Future<S> then<S>(FutureOr<S> onValue(T value)) => ...
  }
  
  class Graph<N, E> {
    final List<N> nodes = [];
    final List<E> edges = [];
  }
  
  class Graph<Node, Edge> {
    final List<Node> nodes = [];
    final List<Edge> edges = [];
  }
  ```

  



### 方法名

`lowerCamelCase `风格

| 方法                                 | 说明                                                |
| ------------------------------------ | --------------------------------------------------- |
| `initXX()`                           | 初始化相关方法，使用 init 为前缀标识，如初始化布局  |
| `initView()`   `isXX()`, `checkXX()` | 方法返回值为 boolean 型的请使用 is/check 为前缀标识 |
| `getXX()`                            | 返回某个值的方法，使用 get 为前缀标识               |
| `setXX()`                            | 设置某个属性值                                      |
| `handleXX()`, `processXX()`          | 对数据进行处理的方法                                |
| `displayXX()`, `showXX()`            | 弹出提示框和提示信息，使用 display/show 为前缀标识  |
| `updateXX()`                         | 更新数据                                            |
| `saveXX()`, `insertXX()`             | 保存或插入数据                                      |
| `resetXX()`                          | 重置数据                                            |
| `clearXX()`                          | 清除数据                                            |
| `removeXX()`, `deleteXX()`           | 移除数据或者视图等，如 `removeView()`               |
| `drawXX()`                           | 绘制数据或效果相关的，使用 draw 前缀标识            |




## 3. 资源命名
### 3.1 布局文件
| **类型**      | **前缀**             |
| ----------- | ------------------ |
| Activity    | activity_          |
| Fragment    | fragment_          |
| Dialog      | dialog_            |
| PopupWindow | popup_             |
| Menu        | menu_              |
| Adapter     | layout_item_/item_ |

合理布局，注意布局嵌套层次，有效运用`<merge>`、`<ViewStub>`、`<include>`标签



### 3.2 图片资源

| 名称                      | 说明                                        |
| ------------------------- | ------------------------------------------- |
| `btn_main_about.png`      | 主页关于按键 `类型_模块名_逻辑名称`         |
| `btn_back.png`            | 返回按键 `类型_逻辑名称`                    |
| `divider_maket_white.png` | 商城白色分割线 `类型_模块名_颜色`           |
| `ic_edit.png`             | 编辑图标 `类型_逻辑名称`                    |
| `bg_main.png`             | 主页背景 `类型_逻辑名称`                    |
| `btn_red.png`             | 红色按键 `类型_颜色`                        |
| `btn_red_big.png`         | 红色大按键 `类型_颜色`                      |
| `ic_head_small.png`       | 小头像图标 `类型_逻辑名称`                  |
| `bg_input.png`            | 输入框背景 `类型_逻辑名称`                  |
| `divider_white.png`       | 白色分割线 `类型_颜色`                      |
| `bg_main_head.png`        | 主页头部背景 `类型_模块名_逻辑名称`         |
| `def_search_cell.png`     | 搜索页面默认单元图片 `类型_模块名_逻辑名称` |
| `ic_more_help.png`        | 更多帮助图标 `类型_逻辑名称`                |
| `divider_list_line.png`   | 列表分割线 `类型_逻辑名称`                  |
| `sel_search_ok.xml`       | 搜索界面确认选择器 `类型_模块名_逻辑名称`   |
| `shape_music_ring.xml`    | 音乐界面环形形状 `类型_模块名_逻辑名称`     |

多种形态的按钮选择器

| 名称                    | 说明                                |
| ----------------------- | ----------------------------------- |
| `sel_btn_xx`            | 作用在 `btn_xx` 上的 `selector`     |
| `btn_xx_normal`         | 默认状态效果                        |
| `btn_xx_pressed`        | `state_pressed` 点击效果            |
| `btn_xx_focused`        | `state_focused` 聚焦效果            |
| `btn_xx_disabled`       | `state_enabled` 不可用效果          |
| `btn_xx_checked`        | `state_checked` 选中效果            |
| `btn_xx_selected`       | `state_selected` 选中效果           |
| `btn_xx_hovered`        | `state_hovered` 悬停效果            |
| `btn_xx_checkable`      | `state_checkable` 可选效果          |
| `btn_xx_activated`      | `state_activated` 激活效果          |
| `btn_xx_window_focused` | `state_window_focused` 窗口聚焦效果 |



### 3.3 Strings

| Prefix    | Description                          |
| --------- | ------------------------------------ |
| `error_`  | An error message                     |
| `msg_`    | A regular information message        |
| `title_`  | A title, i.e. a dialog title         |
| `action_` | An action such as "Save" or "Create" |



### 3.3 其他资源

| **类型**  | **命名**        | **示例**            |
| ------- | ------------- | ----------------- |
| strings | 模块名_逻辑名称      | main_menu_about   |
| colors  | 模块名_颜色名称      | main_info_bg_gray |
| style   | 模块名_逻辑名称（驼峰法） | main_tabBottom    |
| dimen   | 模块名_逻辑名称      | main_menu_padding |





### 4. 注释规范

#### 4.1 javadoc tag

在给公共类或公共方法添加注释的时候，第一句话应该是一个简短的摘要。注意左侧不要紧挨 * 号，要有一个空格。如果注释有多个段落，使用< p>段落标记来分隔段落。我们还可使用< tt>标签来让特定的内容呈现出等宽的文本效果。见下：

```java
    /**
     * 第一句话是这个方法的<tt>简短</tt>摘要。
     * 如果这个描述太长，记得换行。
     * 
     * <p>如果多个段落可以这样
     * 当回车的时候与标签首部对齐即可
     */
    public void test(){}
复制代码
```

如果注释描述里需要包含一个列表，一组选项等，我们可以使用< li>标签来标识，注意标签后不需要空格，见下：

```java
    /**
     * 第一句话是这个方法的简短摘要。
     * 如果这个描述太长，记得换行。
     * 
     * <p>如果多个段落可以这样
     * 
     * <ul>
     * <li>这是列表1
     * <li>这是列表2...
     * 同样回车后与标签对齐即可
     * </ul>
     */
    public void test(){}
复制代码
```

**@param** 是用来描述方法的输入参数。注意在方法描述和tag 之间需要插入空白注释行。不需要每个参数param的描述都对齐，但要保证同个param的多行描述对齐。param 的描述不需要在句尾加标点。

```java
    /**
     * 第一句话是这个方法的简短摘要。
     * 如果这个描述太长，记得换行。
     *
     * @param builderTest 添加参数的描述，如果描述很长，
     *                    需要回车，这里需要对齐
     * @param isTest 添加参数描述，不需要刻意与其他param
     *               参数对齐
     */
    public void test(String builderTest, boolean isTest){}
复制代码
```

**@return** 是用来描述方法的返回值。要写在@param tag之后，与其他tag 之间不需要换行。**@throws** 是对方法可能会抛出的异常来进行说明的，通常格式为：异常类名+异常在方法中出现的原因。见下：

```java
    /**
     * 第一句话是这个方法的简短摘要。
     *
     * @param capacity 添加参数描述，不需要刻意与其他param
     *                 参数对齐
     * @return 描述返回值的含义，可以多行，不需要句号结尾
     * @throws IllegalArgumentException 如果初始容量为负
     *         <ul>
     *         <li>这是抛出异常的条件1（非必须），注意<li>格式
     *         </ul>
     * @throws 注意如果方法还存在其他异常，可并列多个
     */
    public int test(int capacity){
        if (capacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity");
        return capacity;
    }
复制代码
```

**@deprecated** 用于指出一些旧特性已由改进的新特性所取代，建议用户不要再使用旧特性。常与@**link** 配合，当然@link的使用位置没有任何限制，当我们的描述需要涉及到其他类或方法时，我们就可以使用@link啦，javadoc会帮我们生成超链接：

```java
    /**
     * 第一句话是这个方法的简短摘要。
     * 如果这个描述太长，记得换行。
     * 
     * @deprecated 从2.0版本起不推荐使用，替换为{@link #Test2()}
     * @param isTest 添加参数描述，不需要刻意与其他param
     *               参数对齐
     */
    public void test(boolean isTest){}
复制代码
```

@link 常见形式见下: ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/edd4329ddcec48a1bb83b31f9a31eec9~tplv-k3u1fbpfcp-watermark.awebp)

**@code** 用来标记一小段等宽字体，也可以用来标记某个类或方法，但不会生成超链接。常与@link配合，首次通过@link生成超链接，之后通过@code 呈现等宽字体。

```java
    /**
     * 第一句话是这个方法的简短摘要。
     * 我们可以关联{@link Test}类，随后通过{@code Test}类怎样怎样
     * 也可以标记一个方法{@code request()}
     *
     * @param isTest 添加参数描述，不需要刻意与其他param
     *               参数对齐
     */
    public void test(boolean isTest){}
复制代码
```

**@see** 用来引用其它类的文档，相当于超链接，javadoc会在其生成的HTML文件中，将@see标签链到其他的文档上：

```java
    /**
     * 第一句话是这个方法的简短摘要。
     *
     * @param capacity 添加参数描述，不需要刻意与其他param
     *                 参数对齐
     * @return 描述返回值的含义，可以多行，不需要句号结尾
     * @throws IllegalArgumentException 如果初始容量为负
     * @see com.te.Test2
     * @see #test(int)
     */
    public int test(int capacity){
        if (capacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity");
        return capacity;
    }
复制代码
```

@see形式与@link类似，见下： ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa40c5e741ac4e8f82cbabd388e9a931~tplv-k3u1fbpfcp-watermark.awebp) **@since** 用来指定方法或类最早使用的版本。在标记类时，常与@**version**和@**author**配合，一个用来指定当前版本和版本的说明信息，一个用来指定编写类的作者和联系信息等。我们也可以通过< pre>来添加一段代码示例。见下：

```java
    /**
     * 第一句话是这个类的简短摘要。
     * <pre>
     *     Test<Test2> t = new Test<>();
     * </pre>
     * 
     * <p>同样可以多个段落。
     * 
     * @param <T> 注意当类使用泛型时，我们需要使用params说明。这时格式需要插入空白行
     *
     * @author mjzuo 123@qq.com
     * @see com.te.Test2
     * @version 2.1
     * @since 2.0
     */
    public class Test<T extends Test2> {
        /**
         * 第一句话是这个方法的简短摘要。
         * 
         * @params capacity 参数的描述
         * @return 返回值的描述
         * @since 2.1
         */
        public int test2(int capacity) {
            return capacity;
        }
    }
复制代码
```

**@inheritDoc** 用来从当前这个类的最直接的基类中继承相关文档到当前的文档注释中。如下的test() 方法，会直接继承该类的直接父类的test()方法注释。注意与其他tag 不需要插入空行：

```java
    /**
     * {@inheritDoc}
     * @since 2.0
     */
    public void test(boolean isTest){}
复制代码
```

**@docRoot** 它总是指向文档的根目录，表示从任何生成的页面到生成的文档根目录的相对路径。例如我们可以在每个生成的文档页面都加上版权链接，假设我们的版权页面copyright.html 在根目录下：

```java
    /**
     * <a href="{@docRoot}/copyright.html">Copyright</a>
     */
    public class Test {}
复制代码
```

**@hide** 当我们使用google提供的Doclava时，可以使用 @hide 来屏蔽我们不想暴露在javaDoc文档中的方法。

```java
    /**
     * {@hide}
     */
    public class Test {}
复制代码
```

> 好了，本文到这里，关于常用的javaDoc tag的介绍就结束了。如果本文对你有用，来点个赞吧，大家的肯定也是阿呆i坚持写作的动力。



#### 4.2 类注释

每个类完成后应该有作者姓名和联系方式的注释，对自己的代码负责

> 具体的注释风格可以在AS中自己配制:
>
> Settings → Editor → File and Code Templates → Includes → File Header

```java
/**
 * @author: user_name
 * @date: 2016/8/1
 * @des: xxxx
 */
public class MainActivity {
       ...
}
```



### 5. 代码规范

**使用单例可以减轻加载的负担、缩短加载的时间、提高加载的效率**，但并不是所有地方都适用于单例，简单来说，单例主要适用于一下三个方面：

1. 控制资源的使用，通过线程同步来控制资源的并发访问
2. 控制实例的产生，以达到节约资源的目的
3. 控制数据的共享，在不建立关联的条件下，让多个不相关的进程或线程之间实现通信



把一个基本数据类型转为字符串，基础数据类型`.toString()`最快，·String.valueOf(数据)·其次，`数据 + ""`最慢。



**循环内不要不断创建对象**

例如：

```
for (int i = 0; i <= count; i++) {
  Object obj = new Object();
}
```

如果count的值很大，消耗的内存就会更大，建议改为：

```
Object obj = null;
for (int i = 0; i <= count; i++) {
	obj = new Object();
}
```

这样的话，内存中只有一份Object对象引用，每次new Object()的时候，Object对象引用指向不同的Object罢了，但是内存中只有一份，这样就大大节省了内存空间。



**for 和 foreach的使用场景**

实现RandomAccess接口的类实例比如ArrayList，假如是随机访问的，使用普通`for`循环效率将高于使用`foreach`，反过来如果是顺序访问的，则使用Iterator会效率更高（`foreach`的底层实现就是迭代器）。



**ArrayList 和 LinkedList的使用场景**

顺序插入和随机访问比较多的场景使用ArrayList，元素删除和中间插入比较多的场景使用LinkedList，

详细可查看ArrayList和LinkedList的原理。



**不要让public方法中有太多的形参**

如果给这些方法太多形参的话主要有两点坏处：

1. 违反了面向对象的编程思想 ，Java讲求一切都是对象，太多的形参，和面向对象的变成思想并不契合。
2. 参数太多势必导致方法调用的出错概率增加。



字符串变量和字符串常量equals的时候将字符串常量写在前面

```java
String str = "123";
if ("123".equals(str)) {
  ...
}
```

这么做主要是可以避免空指针异常



**公用集合类不使用的数据一定要及时`remove`掉**

如果一个集合类是公用的（也就是说不是方法里面的属性），那么这个集合里面的元素是不会自动释放的，因为始终有引用指向它们。所以，如果公用集合里面的某些数据不使用而不去remove掉它们，那么将会造成这个公用集合不断增大，使得系统有内存泄露的隐患。



**使用最有效率的方式去遍历Map**

```java
public static void main(String[] args) {
  HashMap<String, String> hm = new HashMap<String, String>();
  hm.put("111", "222");
  
  Set<Map.Entry<String, String>> entrySet = hm.entrySet();
  Iterator<Map.Entry<String, String>> iter = entrySet.iterator(); 
  
  while (iter.hasNext()) {
    Map.Entry<String, String> entry = iter.next();
    System.out.println(entry.getKey() + "\t" + entry.getValue());
  }
}
```

如果只是想遍历下这个Map的key值，那用`Set<String> keySet = hm.keySet()`会比较合适。



**尽量采用懒加载的策略，即在需要的时候才创建的原则**

**尽量避免随意或过度的使用静态变量**