

## 依赖注入（Dependency Injection, DI）

依赖注入是一种软件设计模式，能让一个对象接收它所依赖的其他对象，并且接收方不需要知道这些依赖是如何创建的，就能轻松使用。

依赖注入这种设计模式的目的是分离关注点，分离接收方和依赖，从而提供松耦合以及代码重用性。



## Hilt

### Hilt 组件与Android对应类的活动范围

| Hilt 组件                 | 对应Android类的活动范围                   |
| ------------------------- | ----------------------------------------- |
| ApplicationComponent      | Application                               |
| ActivityRetainedComponent | ViewModel                                 |
| ActivityComponent         | Activity                                  |
| FragmentComponent         | Fragment                                  |
| ViewComponent             | View                                      |
| ViewWithFragmentComponent | View annotated with @WithFragmentBindings |
| ServiceComponent          | Service                                   |

Hilt没有为BoardcastReceiver提供组件，因为Hilt直接从ApplicationComponent中注入了BoardcastReceiver。



## Dagger



## 延伸阅读

### IoC

IoC 是 Inversion of Control 的缩写，是控制反转的意思。IoC的别名：依赖注入(DI)。

IoC中最基本的技术是反射(Reflection)编程。

#### 使用IoC框架应该注意什么？

##### 1. 软件系统中由于引入了第三方IOC容器，生成对象的步骤变得有些复杂

本来是两者之间的事情，又凭空多出一道手续，所以，我们在刚开始使用IOC框架的时候，会感觉系统变得不太直观。所以，引入了一个全新的框架，就会增加团队成员学习和认识的培训成本，并且在以后的运行维护中，还得让新加入者具备同样的知识体系。

##### 2. 由于IOC容器生成对象是通过反射方式，在运行效率上有一定的损耗。

如果你要追求运行效率的话，就必须对此进行权衡。

##### 3. 需要进行大量的配制工作，比较繁琐

对于一些小的项目而言，客观上也可能加大一些工作成本。

##### 4. IOC框架产品本身的成熟度需要进行评估，

如果引入一个不成熟的IOC框架产品，那么会影响到整个项目，所以这也是一个隐性的风险。

我们大体可以得出这样的结论：一些工作量不大的项目或者产品，不太适合使用IOC框架产品。另外，如果团队成员的知识能力欠缺，对于IOC框架产品缺乏深入的理解，也不要贸然引入。最后，特别强调运行效率的项目或者产品，也不太适合引入IOC框架产品，象WEB2.0网站就是这种情况。