### 集合

Kotlin 和 Java  一样有三种集合类型：

- `List` 以固定顺序存储一组元素，元素可以重复
  在 Kotlin 中 `List` 支持 `covariant`（协变），可以把子类 `List` 赋值给 父类的 `List`。

- `Set` 存储一组互不相等的元素，通常没有固定顺序

- `Map` 存储 Key-Value 键值对的数据集合，键互不相等，键值可以有相同的值

  ```kotlin
  val map = mapOf("one" to 1, "two" to 2, "thrid" to 3)
  ```

  `to` 表示将 `Key` 和 `Value` 关联，这个叫中辍表达式。 



### List 和 数组的区别

```kotlin
//List
val list: List<String> = listOf("a", "b", "c")

//数组
val strs: Array<String> = arrayOf("a", "b", "c")
```

- 数组不支持协变，因为 Kotlin 的数组编译成字节码时使用的仍是 Java 的数组，在语言层面是泛型的实现，这样会失去协变特性，而 List 是支持协变的。
- Kotlin 中数组 和 MutableList 的API 非常像，主要的区别是数组的元素个数不能变。

### 可变集合

- `MutableList`
- `MutableSet`
- `MutableMap`

在 Kotlin 中集合默认是不可修改的，想要修改集合需要用 `mutableXXXOf()` 创建可变集合。

### 数组和集合的操作符



### Sequence

序列 `Sequence` 又被称为「惰性集合操作」，`Sequence` 和 `Iterable` 一样用来遍历一组数据并可以对每个元素进行特定的处理。

Sequence 有以下优点：

- 一旦满足遍历退出的条件，就可以省略后续不必要的遍历过程。
- 像 `List` 这种实现 `Iterable` 接口的集合类，每调用一次函数就会生成一个新的 `Iterable`，下一个函数再基于新的 `Iterable` 执行，每次函数调用产生的临时 `Iterable` 会导致额外的内存消耗，而 `Sequence` 在整个流程中只有一个。

