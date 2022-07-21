

#### 类的加载过程



- 加载 loading   加载阶段主要完成[三件](http://www.chinabyte.com/keyword/三件/)事，即通过一个类的全限定名来获取定义此类的二进制字节流，将这个字节流所代表的静态[存储](http://storage.chinabyte.com/)结构转化为方法区的运行时数据结构，在Java堆中生成一个代表此类的Class对象，作为访问方法区这些数据的入口。这个加载过程主要就是靠类加载器实现的，这个过程可以由用户自定义类的加载过程。

- 验证 verification               这个阶段目的在于确保Class文件的字节流中包含信息符合当前虚拟机要求，不会危害虚拟机自身安全。主要包括四种验证：

- 文件格式验证：基于字节流验证，验证字节流是否符合Class文件格式的规范，并且能被当前虚拟机

  - 元数据验证：基于方法区的存储结构验证，对字节码描述信息进行语义
  - 字节码验证：基于方法区的存储结构验证，进行数据流和控制流的
  - 符号引用验证：基于方法区的存储结构验证，发生在解析中，是否可以将符号引用成功解析为直接引用。

- 准备 preparation               仅仅为类变量(即static修饰的字段变量)分配内存并且设置该类变量的初始值即零值，这里不包含用final修饰的static，因为final在编译的时候就会分配了，同时这里也不会为实例变量分配初始化。类变量会分配在方法区中，而实例变量是会随着对象一起分配到Java堆中。

- 解析 resolution               解析主要就是将常量池中的符号引用替换为直接引用的过程。符号引用就是一组符号来描述目标，可以是任何字面量，而直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。有类或接口的解析，字段解析，类方法解析，[接口方法](http://www.chinabyte.com/keyword/接口方法/)解析。

  ```
             　　这里要注意如果有一个同名字段同时出现在一个类的接口和父类中，那么编译器一般都会拒绝编译。
  ```

- 初始化 initialization               初始化阶段依旧是初始化类变量和其他资源，这里将执行用户的static字段和静态语句块的赋值操作。这个过程就是执行类构造器方法的过程。

  ```
             　　方法是由编译器收集类中所有类变量的赋值动作和静态语句块的语句生成的，类构造器方法与实例构造器方法不同，这里面不用显示的调用父类的方法，父类的方法会自动先执行于子类的方法。即父类定义的静态语句块和静态字段都要优先子类的变量赋值操作。
  ```

### android事件分发机制



| Touch 事件相关方法                                   | 方法功能 | ViewGroup | Activity |
| :--------------------------------------------------- | -------- | --------- | -------- |
| public boolean dispatchTouchEvent(MotionEvent ev)    | 事件分发 | Yes       | Yes      |
| public boolean onInterceptTouchEvent(MotionEvent ev) | 事件拦截 | Yes       | No       |
| public boolean onTouchEvent(MotionEvent ev)          | 事件响应 | Yes       | Yes      |

#### onTouch和onTouchEvent有什么区别，又该如何使用？

这两个方法都是在View的dispatchTouchEvent中调用的，onTouch优先于onTouchEvent执行。如果在onTouch方法中通过返回ture将事件消费掉，onTouchEvent将不再执行。

- android事件分发是先传递到ViewGroup，再由ViewGroup传递到View。
- ViewGroup中可以通过onInterceptTouchEvent方法对事件传递进行拦截，onInterceptTouch方法返回true代表不允许事件继续向子View传递，返回false代表不对事件进行拦截，默认返回false。
- 子View中如果将传递的事件消费掉，ViewGroup中将无法接收到任何事件。

#### Touch事件的传递机制

- 所有Touch事件都被封装成MotionEvent对象，包括Touch的位置、时间、历史记录以及第几个手指（多点触控）等。
- 事件类型分为：
  - ACTION_DOWN
  - ACTION_UP
  - ACTION_MOVE
  - ACTION_POINTER_DOWN
  - ACTION_POINTER_UP
  - ACTION_POINTER_CANCEL
- 每个事件都是以ACTION_DOWN开始，以ACTION_UP结束。

1. 事件从Activity的dispatchTouchEvent开始传递，只要onInterceptTouchEvent不进行拦截，从最上层的View(ViewGroup)开始一直往下（子View）传递。子View可以通过onTouchEvent对事件进行处理。
2. 如果事件从上往下传递过程中一直没有被停止，且最底层子View没有消费事件，事件会反向上传递，这时父View(ViewGroup)可以进行消费，如果还是没有被消费的话，最后会到Activity的onTouchEvent。
3. 如果View没有对ACTION_DOWN进行消费，之后的其他事件不会传递过来。
4. OnTouchListener优先于onTouchEvent执行。

#### Activity Window View 三者的差别

Activity 控制单元，Window 承载模型，View 显示视图

Activity在onCreate中调用attach方法，在attach方法中创建一个window对象。window对象创建时并没有创建DocerView对象，而是当用户在Activity中调用setContentView方法之后，接着转到window的setContentView，这时会检查DecorView是否存在，如果不存在则创建DecorView对象，然后把用户自己的View添加到DecorView中。

#### LayoutInflater

LayoutInflater只做一件事，就是把xml表述的layout转化为View。

#### LaunchMode 应用场景

- standard：创建一个新的Activity
- singleTop：栈顶不是该类型的Activity，创建一个行的Activity。否则，onNewIntent。
- singleTask：回退栈中没有该类型的Activity，创建Activity，否则，onNewIntent + ClearTop。
- singleInstance：回退栈中只有这个Activity，没有其他Activity。

#### ListView优化

- 复用convertView，使用ViewHolder
- item中有图片时，异步加载，适当对图片进行压缩
- 监听滚动事件，滑动时不加载图片
- 实现上拉加载更多，进行分页加载

#### 自定义View的基本流程

1. 覆写onDraw、onMeasure、onLayout
2. 覆写dispatchTouchEvent、onTouchEvent
3. 覆写自定义属性，编写attr.xml，然后在代码中通过TypedArray等类获取自定义属性值
4. 处理滑动冲突、像素转换等问题 Other
5. 编写attr.xml文件
6. 在layout布局文件中引用，同时引用命名空间
7. 在自定义控件中进行读取（构造方法拿到attr.xml文件值）
8. 覆写onMeasure
9. 覆写onLayout



混淆的作用

1. 将代码进行优化和缩短方法名、字段名
2. 删除没有被调用的变量和方法



1. 什么是类？什么是对象？类和对象之间的关系？

   类是对象的集合，是抽象的；对象是类的实例化，是具体的。
   
2. 简述C/C++的核心思想

   http://www.cnblogs.com/lanxuezaipiao/p/4135105.html
   
3. 泛型的本质

   泛型的本质是参数化类型，也就是说所操作的数据类型被指定为某个参数类型。

   这种参数类型可以在类、接口和方法中创建，分别称为泛型类、泛型接口、泛型方法。

   优点：在编译的时候检查类型安全，并且素有的强制转换都是自动和隐式的，提高代码的重用率。

   缺点：
   
4. 静态变量的使用场景

   - 变量所包含的对象提及较大，占用内存较多
   - 变量所包含的对象生名周期较长。
   - 变量所包含的对象数据稳定。
   - 该类的对象实例有对该变量所包含的对象共享需求。
     

5. 类加载原理及类加载器

6. 对Clone的理解

7. HashMap的实现

8. Collection和Collections的区别

9. 数组浅析

10. 代码优化编程

11. 事件处理机制与"恋爱关系"

12. JNDI(Java命名与目录接口)

13. Comparable和Comparator实现对象比较

14. String、StringBuffer与StringBuilder之间的区别

    String：字符串常量

    StringBuffer：字符串变量，线程安全

    StringBuilder：字符串变量，线程非安全

    三者在执行速度方面的比较：StringBuilder > StringBuffer > String

    **总结**：

    - 操作少量的数据用 String
    - 单线程操作字符串缓冲区下操作大量数据 StringBuilder
    - 多线程操作字符串缓冲区下操作大量数据 StringBuffer

15. Heap和Stack的区别

16. 反射机制

17. Synchronized的使用

    synchronized代码块：

    synchronized方法：

    synchronized静态方法：

    synchronized类：

    synchronized()：
    
1.  android:background="?/attr/..."
2.  DrawerLayout
3.  @SuppressLint("...")
4.  @SuppressWarnings("deprecation")
5.  @TargetApi(Build.VERSION_CODES.HONEYCOMB)
6.  CharSequence
7.  attr
8.  dimen
9.  @Nullable
10.  <merge>
11.  android:shadowDx
12.  android:shadowDy
13.  android:shadowRadius
14.  layout_alignWithParentIfMissing



遵循 app arch layer design 提高应用的质量和稳健性，并使应用更易于测试。



命名惯例：一般现在时动词 + 名词/内容 （可选）+ 用例
