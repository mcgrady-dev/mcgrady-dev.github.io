# Data Structure\Design Patterns Interview Questions



1. 简述常见的数据结构？

2. 堆的结构？

3. 树、B+ 树、二叉树、红黑树的了解？

4. 二叉树的深度优先遍历和广度优先遍历？

5. 堆和树的区别？

6. 图的了解？

7. 单例模式

   - 饿汉模式：

     ```java
     //只有内部类可以为static。
     public class SingIn {
         //在自己内部定义自己的一个实例，只供内部调用
         private static final SingIn instance = new SingIn();
         
         private SingIn() {
         }
         
         //这里提供了一个供外部访问本class的静态方法，可以直接访问
         public static SingIn getInstance(){
             return instance;
         }
     }
     ```

   - 懒汉模式：

     ```java
     //一种常用的形式
     private static SingIn instance = null; 
     public static synchronized SingIn getInstance() {         
         // 这个方法比上面有所改进，不用每次都进行生成对象，只是第一次  
         // 使用时生成实例，提高了效率！  
         if (instance == null)  
             instance = new SingIn();  
         return instance;  
     }
     ```

   - 双重锁定：

     ```java
     //将同步内容下方到if内部，提高了执行的效率，不必每次获取对象时都进行同步，只有第一次才同步，创建了以后就没必要了。 
     private static volatile SingIn instance = null;
     private SingIn() {
         
     }
      public static SingIn getInstance() {
         if(instance == null) {
             synchronized(SingIn.class) {
                 if(instance == null) {
                     instance = new SingIn ();
                 }
             }
         }
          
         return instance;
       }
     ```

8. 单例模式有什么缺点？

9. 项目中常用的设计原则有哪些？

   - 单一职责原则
     建议接口一定要做到单一职责，类的设计尽量做到只有一个原因引起的变化
   - 开放封闭原则
     这个原则有两个特性，一个是说“对于扩展是开放的”，另一个是说“对于更改是封闭的”。面对需求，对程序的改动是通过增加新代码进行的，而不是更改现有的代码。这就是“开放-封闭原则”的精神所在。
   - 里氏替换原则
     只要父类能出现的地方子类就可以出现，而且替换为子类也不会产生任何错误或异常，使用者可能根本就不需要知道是父类还是子类。但是，反过来就不行了，有子类出现的地方，父类未必就能适应。
   - 依赖倒置原则
     高层模块不应该依赖底层模块，二者都应该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象；依赖倒置原则就是要我们面向接口编程。
   - 接口分离原则
     建立功能单一细化的接口，接口中的方法尽量少，避免接口庞大臃肿。通过分散定义多个接口，可以预防外来变更的扩散，提高系统的灵活性和可维护性。

10. 谈谈你对 Android 设计模式的理解

    - Builder模式
    - 迭代器模式
    - 模板方法模式
    - 访问者模式
    - 中介者模式
    - 代理模式（委托模式）
    - 组合模式
    - 适配器模式
    - 装饰模式
    - 享元模式
    - 外观模式

11. 项目中常用的设计模式有哪些？

    - 单例模式
    - 观察者模式
    - 原型模式
    - 策略模式

12. 手写生产者-消费者模式？

13. 手写观察者模式？

14. 适配器模式、装饰者模式、外观模式的异同？

15. 代理模式与装饰模式的区别，手写一个静态代理，一个动态代理？

16. 动态代理有什么作用？

17. 讲讲LinkedHashMap的数据结构？

18. android源码中有哪些设计模式？

19. Bundle是什么数据结构?利用什么传递数据？

20. 手写双检查单例模式，各个步骤有什么区别？

21. 动画里面用到了什么设计模式？

22. OkHttp里面用到了什么设计模式？