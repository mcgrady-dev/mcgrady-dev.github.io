# Android数据和文件存储

系统提供的几种存储应用数据的选项：

- 应用专属存储空间：指应用专属的存储目录，其它应用不可访问
  - 内部存储空间：适用于存储持久性或缓存数据的存储空间，在API ≥ 29 系统上，会对位置进行加密，使得内部存储更专注于应用本身存储的敏感数据。
  - 外部存储空间：适用于存储持久性或缓存数据的存储空间，其他应用拥有适当的权限情况下可以访问这些目录。
- 共享存储空间：存储可与其他应用共享的的文件。
- Preferences：以键值对形式存储的私有数据。
- Databases：使用Room等持久性结构话数据存储在专用数据库中。



| 内存类型                           | 访问方法                                        | 所需权限                                                     | 外部可访问？ | 卸载是否移除文件？ |
| :--------------------------------- | :---------------------------------------------- | ------------------------------------------------------------ | ------------ | ------------------ |
| 应用专属存储空间<br />内部存储目录 | `getFilesDir()`<br /> `getCacheDir()`           | 不需要任何权限                                               | 否           | 是                 |
| 应用专属存储空间<br />外部存储目录 | `getExternalFilesDir()` `getExternalCacheDir()` | API ≥ 19：不需要任何权限                                     | 可以         | 是                 |
| 共享存储空间                       | MediaStore API                                  | API ≥ 29：访问其他应用的文件需要读写权限<br />API ≤ 28：访问所有文件均需要相关权限 | 可以         | 否                 |
| SharedPreferences                  | SharedPreferences、Jetpack-Preferences          | 无                                                           | 否           | 是                 |
| Databases                          | Room持久性库等                                  | 无                                                           | 否           | 是                 |



## 存储位置的类别

Android 提供两类物理存储位置：内部存储空间和外部存储空间。在大多数设备上，内部存储空间小于外部存储空间。不过，所有设备上的内部存储空间都是始终可用的，因此在存储应用所依赖的数据时更为可靠。
可移除卷（例如 SD 卡）在文件系统中属于外部存储空间。Android 使用路径（例如 `/sdcard`）表示这些存储设备。



## 对外部存储空间的访问和所需权限

Android 为对外部存储空间的读写访问定义了以下权限：[`READ_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission#READ_EXTERNAL_STORAGE) 和 [`WRITE_EXTERNAL_STORAGE`](https://developer.android.com/reference/android/Manifest.permission#WRITE_EXTERNAL_STORAGE)。

在较低版本的 Android 系统中，应用需要声明这些权限才能访问位于外部存储空间中[应用专属目录](https://developer.android.com/training/data-storage/app-specific#external)之外的任何文件。Android 系统的版本越新，就越依赖于文件的用途而不是位置来确定应用对文件的访问能力。这种基于用途的存储模型可增强用户隐私保护，因为应用只能访问其在设备文件系统中实际使用的区域。

### 分区存储

