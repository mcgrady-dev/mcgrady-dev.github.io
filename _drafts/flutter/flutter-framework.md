## Flutter 中的四棵树

![](https://book.flutterchina.club/assets/img/trees.59d95f72.png)

### Widget Tree

Flutter 中是通过 widget 嵌套 widget 的方式来构建UI的，即构建了Widget Tree。

### Element Tree

根据 Widget 树生成 Element Tree，Element Tree 树种的节点都继承自 Element 类。

### Render Tree

根据 Element Tree 生成 Render Tree（渲染树），Render Tree 中的节点都继承自 RenderObject 类。

真正的渲染逻辑在 Render Tree 中，Element 是 Widget 和 RenderObject 的粘合剂，可以理解为一个中间代理。

### Layer Tree

根据 Render Tree 生成 Layer Tree，然后上屏显示，Layer Tree 中的节点都继承自 Layer 类。



## 状态 State 和 UI 的关系

状态管理方案：package:provider

状态管理要解决的问题

