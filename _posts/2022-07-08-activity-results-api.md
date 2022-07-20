---
layout:
title: Android Activity Results API
date: 2022-07-08 10:20 +0800
tags: [android]
---

通常启动一个 Activity 多数时候不是单向操作，而是启动一个 Activity 并接收返回的结果，例如，启动相机应用并接收拍摄的照片作为结果。或者，启动“通讯录”应用接收联系人详细信息作为结果等。

传统的方式是通过 Intent 携带数据，使用 `startActivityForResult` 启动一个 Activity ，然后通过 `onActivityResult` 来接收返回的结果。但随着应用的扩展，`onActivityReuslt` 回调方法各种嵌套、耦合严重、难以维护，Google 可能也意识到了这些问题，推出了 Activity Result API 来处理用于注册结果、启动结果以及在系统分派结果对其进行处理的组件。

<!--more-->

## ActivityResultContract

ActivityResultContract 定义生成结果所需的输入类型以及结果的输出类型。这些 API 可为拍照和请求权限等基本 Intent 操作提供默认协定，同时还可以创建自定义协定。

### 预定义的 Contract

- `StartActivityForResult()`: 不进行任何类型转换，将原始 Intent 作为输入，将 ActivityResult 作为输出。
- `RequestMultiplePermissions()`: 请求多个权限
- `RequestPermission()`: 请求单个权限
- `TakePicturePreview()`: 用于获取小图片预览，并返回 Bitmap。
- `TakePicture()`: 用于拍摄照片并将其保存到提供的 content-Uri 中。如果图像保存到给定的 Uri 中，则返回 true。
- `TakeVideo()`: 将视频保存到提供的 content-Uri 中。返回缩略图 Bitmap。
- `PickContact()`: 从通讯录中获取联系人，作为 Uri 返回。
- `CreateDocument()`: 提示用户选择新建文档路径，并创建新文档，返回已创建项目的 content-Uri。
- `OpenDocumentTree()`: 提示用户选择文档目录，作为 content-Uri 返回用户选择的路径。
- `OpenMultipleDocuments()`: 打开多个文档
- `OpenDocument()`: 打开单个文档
- `GetMultipleContents()`: 选择多条内容
- `GetContent()`: 选择一条内容

## ActivityResultCallback

ActivityResultCallback 是单一方法（`onActivityResult()`）接口，可接受 ActivityResultContract 中定义的输出类型的对象：

```kotlin
val getContent = registerForActivityResult(StartActivityForResult()) { result ->
	// Handle the ActivityResult
}
```

## ActivityResultLauncher

调用 ActivityResultLauncher 的 `launch()` 方法来启动页面跳转，相当于 `startActivity()`













**参考文献**

[Getting a result from an activity](https://developer.android.com/training/basics/intents/result)

[再见！onActivityResult！你好，Activity Results API！](https://juejin.cn/post/6887743061309587463)

