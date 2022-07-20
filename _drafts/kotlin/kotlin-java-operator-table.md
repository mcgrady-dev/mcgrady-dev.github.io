



### 单目前缀运算符

| Java |     Kotlin     | Description |
| :--: | :------------: | :---------: |
| + a  | a.unaryPlus()  |             |
| - a  | a.unaryMinus() |             |
| ! a  |     a.not      |             |

### 自加自减运算符

| Java | Kotlin  | Description |
| :--: | :-----: | :---------: |
| a ++ | a.inc() |             |
| a -- | a.dec() |             |

### 双目算术运算符

|  Java  |    Kotlin    | Description |
| :----: | :----------: | :---------: |
| a + b  |  a.plus(b)   |             |
| a - b  |  a.minus(b)  |             |
| a * b  |  a.times(b)  |             |
| a / b  |   a.div(b)   |             |
| a % b  |   a.rem(b)   |             |
| a .. b | a.rangeTo(b) |             |

### in和!in运算符

|  Java   |     Kotlin     | Description |
| :-----: | :------------: | :---------: |
| a in b  | b.contains(a)  |             |
| a !in b | !b.contains(a) |             |

### 索引访问运算符

|         Java          |        Kotlin        | Description |
| :-------------------: | :------------------: | :---------: |
|         a [i]         |       a.get(i)       |             |
|       a [i, j]        |     a.get(i, j)      |             |
|   a [i_l, ..., i_n]   | a.get(i_l, ..., i_n) |             |
|       a [i] = b       |     a.set(i, b)      |             |
|     a [i, j] = b      |    a.set(i, j, b)    |             |
| a [i_l, ..., i_n] = b | a.set(i_l, ..., i_n) |             |



### 调用运算符

|        Java        |          Kotlin           | Description |
| :----------------: | :-----------------------: | :---------: |
|        a ()        |        a.invoke()         |             |
|       a (b)        |        a.invoke(b)        |             |
|     a (b1, b2)     |     a.invoke(b1, b2)      |             |
| a(b1, b2, b3, ...) | a.invoke(b1, b2, b3, ...) |             |

### 广义赋值运算符

| Java   | Kotlin           | Description |
| ------ | ---------------- | ----------- |
| a += b | a.pluseAssign(b) |             |
| a -= b | a.minusAssign(b) |             |
| a *= b | a.timesAssign(b) |             |
| a /= b | a.divAssign(b)   |             |
| a %= b | a.remAssign(b)   |             |

### 相等于不等运算符

|  Java  |             Kotlin              | Description |
| :----: | :-----------------------------: | :---------: |
| a == b |   a?.equals(b) : (b === null)   |             |
| a != b | !(a?.equals(b)) ?: (b === null) |             |

### 比较运算符

|  Java  |       Kotlin        | Description |
| :----: | :-----------------: | :---------: |
| a > b  | a.compareTo(b) > 0  |             |
| a < b  | a.compareTo(b) < 0  |             |
| a >= b | a.compareTo(b) >= 0 |             |
| a <= b | a.compareTo(b) <= 0 |             |

### 位运算符

|  Java   |  Kotlin   |  Description  |
| :-----: | :-------: | :-----------: |
|   ~ a   |   a.inv   |      非       |
|  a & b  | a.and(b)  |      与       |
| a \| b  |  a.or(b)  |      或       |
|  a ^ b  | a.xor(b)  |     异或      |
| a << b  | a.shl(b)  |    左移b位    |
| a >> b  | a.shr(b)  |    右移b位    |
| a >>> b | a.ushr(b) | 无符号右移b位 |

另外，对于`<<=` `>>=` `>>>=`这三个操作符，Kotlin中没有对应的函数。