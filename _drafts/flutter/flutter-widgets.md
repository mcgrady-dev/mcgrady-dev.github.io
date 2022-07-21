# Flutter Widgets



## StatelessWidget

无状态的组件，StatelessWidget 的属性不可变，所有的值都是最终的。

### 使用场景

StatelessWidget 用于不需要维护状态的场景。

## StatefulWidget

有状态的组件，StatefulWidget 可以拥有状态(State)，这些状态在 widget 生命周期中是可变的。
实现一个 StatefulWidget 至少需要两个类：

1. 一个 StatefulWidget 类
2. 一个 State 类，StatefulWidget 类本身是不可变的，但是 State 类在 widget 生命周期中始终存在。



### createState()

createState() 方法用于创建和 StatefulWidget 相关的状态，它在 StatefulWidget 的生命周期中可能被多次调用。



### State

一个 StatefulWidget 类会对应一个 State 类，State 表示与其对应的 StatefulWidget 要维护的状态，State 中的保存的状态信息可以：

1. 在 widget 构建时可以被同步读取。
2. 在 widget 生命周期中可以被改变，当 State 被改变时，可以手动调用其 setState() 方法通知 Flutter 框架状态发生改变，Flutter 框架在收到消息后，会重新调用其 build 方法重新构建 widget 树，从而达到更新UI的目的。



State 中两个常用属性：

1. widget，表示与该 State 实例关联的 widget 实例，由 Flutter 框架动态设置。
2. context，StatefulWidget 对应的 BuildContext，作用同 StatelessWidget 的 BuildContext。



#### setState()

`setState()` 的作用是通知 Flutter 框架，有状态发生了改变，Flutter 对此方法做了优化，使重新执行变的很快，所以你可以重新构建任何需要更新的东西，而无需分别去修改各个 widget。



## Context

`build`方法有一个`context`参数，它是`BuildContext`类的一个实例，表示当前 widget 在 widget 树中的上下文，每一个 widget 都会对应一个 context 对象（因为每一个 widget 都是 widget 树上的一个节点）。实际上，`context`是当前 widget 在 widget 树中位置中执行”相关操作“的一个句柄(handle)，比如它提供了从当前 widget 开始向上遍历 widget 树以及按照 widget 类型查找父级 widget 的方法。

### 使用场景

```dart
class ContextRoute extends StatelessWidget  {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Context测试"),
      ),
      body: Container(
        child: Builder(builder: (context) {
          // 在 widget 树中向上查找最近的父级`Scaffold`  widget 
          Scaffold scaffold = context.findAncestorWidgetOfExactType<Scaffold>();
          // 直接返回 AppBar的title， 此处实际上是Text("Context测试")
          return (scaffold.appBar as AppBar).title;
        }),
      ),
    );
  }
}
```





## Scaffold

Scaffold 叫做页面脚手架，是 Material Library 中提供的一个 widget，它提供了默认的导航栏、标题和包含主屏幕的 widget 树的 body 属性。widget 树可以很复杂。

## Widget

Flutter 中是通过 widget 嵌套 widget 的方式来构建UI和进行实践处理的，几乎所有的对象都是一个 widget，所以 Flutter 中万物皆 widget。

widget 的主要工作是提供一个 build() 方法来描述如何根据其他较低级别的 widget 来显示自己。

### DiagnosticableTree

widget 继承自 DiagnosticableTree，DiagnosticableTree 即 `诊断树`，主要作用是提供调试信息。

## 基础组件

- `Text`：创建一个带格式的文本
- `Row`、`Column`：具有弹性空间的水平、垂直方向的布局
- `Stack`：取代线性布局（LinearLayout相似）
- `Container`：可以创建矩形视觉元素，如：background、边框、阴影

## Material Components

## Cupertino(iOS风格的widget)

## Layout

## Text

## Assets、图片、Icons

## Input

## Animator、Motion

## 交互模型

## 样式

## 绘制效果

## Async

## 滚动

## 辅助功能

