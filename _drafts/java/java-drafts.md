

# Java 杂记





## Java基础点
1. java中包（package）概念是什么?
   管理Java文件
   解决同名文件冲突
2. java不会给局部变量赋予初始值
3. java中被`static`修饰的成员称为静态成员或类成员。它属于整个类所有，而不是某个对象所有，即被类的所有对象所共享。
4. 内部类的分类：
- 成员内部类
- 静态内部类
- 方法内部类
- 匿名内部类
5. 包装类（Wrapper Class）的作用：
- 实现基本类型之间的转换
- 便于函数传值
- 在某些地方要用到Object的时候方便将基本数据类型装换
6. Java中的访问修饰符
   ![Alt text](./Image.png)
7. 认识Java中的`StringBuilder`类
```java
// 这里在堆内存中创建了一个字符串, 并且创建了一个str字符串对象指向它.
String str = "hello";    
System.out.println(str);  
// 根据String类具有的不可变性, 这里在堆内存又创建了一个字符串 "helloworld"    
System.out.println(str + "world");    
```

从运行结果中可以看到，程序运行时会额外创建一个对象保存“helloworld”。
当频繁操作字符串时，就会额外产生很多临时变量。使用`StringBuilder`或`StringBuffer`就可以避免这个问题。
`StringBuilder`是线程安全的，而`StringBuffer`则没有实现现场安全功能，所以性能略高。
```java
StringBuilder str = new StringBuilder();
StringBuilder str2 = new StringBuilder("helloworld");
System.out.println(str2);
```





## Java编程技巧
1. 把字符串常量放在前面
```java
// Bad
if (variable.equals("literal")) { ... }
// Good
if ("literal".equals(variable)) { ... }
```
通过把字符串常量放在比较函数`equals()`比较项的左侧来防止偶然的`NullPointerException`。
2. 不要相信早期的JDK APIs
   因为Java早期的的API不够成熟
3. 不要相信`-1`
>字符在字符序列中第一次出现的位置将作为结果[被返回]，如果字符不存在则返回-1。
>——Javadoc String.indexOf()

```java
// Bad
if (string.indexOf(character) != -1) { ... }
// Good
if (string.indexOf(character) >= 0) { ... }
```
4. 检查null和长度
   不管什么时候你有一个集合、数组或者其他的，**确保它存在并且不为空**。
```java
// Bad
if (array.length > 0) { ... }
// Good
if (array != null && array.length > 0) { ... }
```
5. 总是在`switch`语句里加上`default`
```java
// Bad
switch (value) {
    case 1: foo(); break;
    case 2: bar(); break;
}
// Good
switch (value) {
    case 1: foo(); break;
    case 2: bar(); break;
    default:
        throw new ThreadDeath("That'll teach them");
}
```

