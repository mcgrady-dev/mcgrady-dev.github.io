## 概述

LiveData 是可感知生命周期的、可观察的，数据持有者。被设计用于更新 UI。



## LiveData 的特性

- 观察者的回调永远发生在主线程
  setValue() 发生在主线程（非主线程调用会抛出异常），postValue() 内部会切到主线程调用 setValue() 。之后遍历所哟普观察者的 onChanged() 方法。

- 仅持有单个且最新的数据
  单个且最新，意味着 LiveData 每次持有一个数据，并且新数据会覆盖上一个。

- 自动取消订阅
  这是 LiveData 可感知生命周期的重要表现，自动取消订阅意味着开发者无需写取消订阅的模板代码，降低了内存泄漏的可能性。
  这背后原理在于生命周期处于 `DESTROYED` 时，移除观察者。

  ```java
  @Override
  public void onStateChanged(@NonNull LifecycleOwner source, @NonNull Lifecycle.Event event) {
    Lifecycle.State currentState = mOwner.getLifecycle().getCurrentState();
    if (currentState == DESTROYED) {
      removeObserver(mObserver);
      return;
    }
    ...
  }
  ```

- 提供「可读可写」和「仅可读」两个版本收缩权限
  通过权限细化，让使用者各取所需，避免由于权限泛滥导致的数据异常。

- 配合 DataBinding 实现「双向绑定」
  可以实现更新数据自动驱动UI变化，还可以实现UI变化影响数据的变化。

- value 是 nullable 的
  LiveData#getValue() 是可空的，使用时应该注意判空。

- Fragment 中订阅 LiveData 时需传入正确的 `lifecycleOwner`
  Fragment 调用 LiveData#observe() 方法时传入 this 和 viewLifecycleOwner 是不一样的。

- 当 LiveData 持有的数据是「事件」时，可能会遇到「粘性事件」
  由于 LiveData 会在观察者活跃时将最新的数据通知给观察者，则会产生「粘性事件」的情况。
  解决方案：**将事件作为状态的一部分，在事件被消费后，不再通知观察者**。

  1. UnPeek-LiveData
  2. 使用 Kotlin 扩展函数和 typealias 解决粘性事件

- LiveData 是不防抖的
  `setValue()` `postValue()` 传入相同的值多次调用，观察者的 `onChanged()` 会被多次调用。

- `transformation` 工作在主线程


