## Gson



## Moshi



## kotlinx-serialization



## FastJson



## 对比

- kotlinx-serialization 的优势是支持 Kotlin 的 Multiplatform，对于需要多平台移植的 Kotlin 代码，使用 kotlinx-serialization 显然更合适。
- Moshi 的优势是兼容 Java ，毕竟 Kotlin 的代码 90% 仍然跑在 Jvm 甚至 Android 上，所以如果你的 Kotlin 代码与 Java 代码混合运行在 Jvm 上面，那么考虑使用 Moshi。
- [Benchmarking Kotlin JSON Parsers: Jackson-Kotlin and Kotlinx Serialization](https://www.ericthecoder.com/2020/11/23/benchmarking-kotlin-json-parsers-jackson-kotlin-and-kotlinx-serialization/)
- [java-json-benchmark](https://github.com/fabienrenaud/java-json-benchmark)