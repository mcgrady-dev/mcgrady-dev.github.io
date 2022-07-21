---
layout: 
title: Android Application Package (APK/AAB)
date: 2022-03-01 15:52 +0800
tags: android
---

<!--more-->

## 

### APK 文件结构

![apk-structure](https://s2.loli.net/2022/07/19/mfb8e7GJIapRFVL.png)

- **META-INF/**
  包含 `CERT.SF` 和 `CERT.RSA` 签名文件，以及 `MANIFEST.MF` 清单文件。
- **lib/**
  包含特定于处理器软件层的已编译代码。
  - armeabi
  - armeabi -v7a
  - armeabi -v8a
  - x86
  - x86_64
  - mips
- **res/**
  包含未编译到 `resource.arsc` 中的资源
- **assets/**
  包含应用的资源，应用可以使用 AssetManager 对象检索这些资源。
- **AndroidManifest.xml**
  包含核心 Android 清单文件，列举应用的名称、版本、访问权限和引用的库文件。
- **classes.dex**
  包含以 Dalvik/ART 虚拟机可理解的 DEX 文件格式编译的类。
- **resources.arsc**
  包含已编译的资源。此文件包含 `res/values/` 文件夹的所有配置中的 XML 内容。打包工具会提取此 XML 内容，将其编译为二进制文件形式，并压缩内容。
  此内容包括语言字符串的样式，以及未直接包含在 resource.arsc 文件中的内容（例如布局文件和图片）的路径。



## Android App Bundle (AAB)

### 概述

Android App Bundle 是一种发布格式，其中包含应用编译过的代码和资源，它会将 APK 生成及签名交由 Google Play 来完成。

Google Play 会使用开发者的 App Bundle 针对每种设备配置生成并提供经过优化的 APK ，因此只会下载特定设备所需的代码和资源来运行应用。开发者不必再构建、签署和管理多个 APK 来优化对不同设备的支持，而用户也可以获得更小且更优化的安装包。