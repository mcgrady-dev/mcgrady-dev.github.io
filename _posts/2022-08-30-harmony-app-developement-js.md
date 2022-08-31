---
layout: article
title: Harmony应用开发（JS篇）
date: 2022-08-30 10:47 +0800
tags: harmony
---



<!--more-->

## FA模型整体架构

HarmonyOS用户程序的开发本质上就是开发Ability。HarmonyOS系统是通过对Ability调度，结合系统提供的一致性调度契约对Ability进行生命周期管理，从而实现对用户程序的调度。

FA模型中Ability分为PageAbility、ServiceAbility、DataAbility、FormAbility几种类型。

### PageAbility生命周期

![harmonyos-js-pageability-lifecycle](https://s2.loli.net/2022/08/30/jX16z53YsvZTUuN.jpg)

其他类型Ability的生命周期可参考PageAbility生命周期去除前后台切换以及onShow的部分进行理解。

<br>

### 进程线程模型

应用独享独立进程，Ability独享独立线程，应用进程在Ability第一次启动时创建，并为启动的Ability创建线程，应用启动后再启动应用内其他Ability，会为每一个Ability创建相应的线程。每个Ability绑定一个独立的JSRuntime实例，因此Ability之间是隔离的。<br>

![harmonyos-js-app-process](https://s2.loli.net/2022/08/30/IponlC2bTw7v5q4.png)

<br>

## 方舟开发框架

**方舟开发框架（ArkUI）**是一套UI开发框架，提供应用UI开发时所需的能力。<br>

<img src="https://s2.loli.net/2022/08/30/boCixD6FGAPrq7c.jpg" alt="harmony-arkui-framework" style="zoom:80%;" />

JSFramework主要对页面DOM进行管理。

<br>

## 基于Web开发范式的框架

基于JS扩展的类Web开发范式的方舟开发框架，采用经典的HML、CSS、JavaScript三段式开发方式：

- 使用HML标签文件进行布局搭建；
- 使用CSS文件进行样式描述；
- 使用JavaScript文件进行逻辑处理；

UI组件与数据之间通过单向数据绑定的方式建立关联，当数据发生变化时，UI界面自动触发更新。

使用基于JS扩展的类Web开发范式的方舟开发框架：<br>

<img src="https://s2.loli.net/2022/08/30/8Iu7rF4znVA1Jxw.png" alt="harmonyos-js-web-framework" style="zoom:80%;" />

- **应用层（Application）**
  应用层特指基于JS开发的FA应用。
- **前端框架层（Framework）**
  前端框架主要完成前端页面解析，以及提供MVVM开发模式、页面路由机制和自定义组件等能力。
- **引擎层（Engine）**
  引擎层主要提供动画解析、DOM(Document Object Model)树构建、布局计算、渲染命令构建与绘制、事件管理等能力。
- **平台适配层（Porting Layer）**
  适配层主要完成对平台层进行抽象，提供抽象接口，可以对接到系统平台。比如：事件对接、渲染管线对接和系统生命周期对接等。

<br>

## 基于声明式开发范式的框架

方舟开发框架针对不同目的和技术背景的开发者提供了两种开发范式，其中声明式开发范式无需JS Framework进行页面DOM管理，渲染更新链路更为精简，占用内存更少。

基于TS扩展的声明式开发范式的方舟开发框架有以下特征：

- **开箱即用的组件**
- **丰富的动效接口**
  提供svg标准的绘制图形能力，同时开放了丰富的动效接口。
- **状态和数据管理**
  状态数据管理作为基于TS扩展的声明式开发范式的特色，通过功能不同的装饰器给开发者提供了清晰的页面更新渲染流程和管道。状态管理包括UI组件状态和应用程序状态，两者协作可以使开发者完整地构建整个应用的数据更新和UI渲染。
- **系统能力接口**
  开发者可以通过简单的接口调用丰富的系统能力。

基于声明式开发范式的整体框架：<br>

![harmonyos-ts-framework](https://s2.loli.net/2022/08/30/EKkpAjy4cuNG2T5.png)

- **声明式UI前端**

  提供了UI开发范式的基础语言规范，并提供内置的UI组件、布局和动画，提供了多种状态管理机制，为应用开发者提供一系列接口支持。

- **语言运行时（Runtime）**

  选用方舟语言运行时，提供了针对UI范式语法的解析能力、跨语言调用支持的能力和TS语言高性能运行环境。

- **声明式UI后端引擎（Engine）**

  后端引擎提供了兼容不同开发范式的UI渲染管线，提供多种基础组件、布局计算、动效、交互事件，提供了状态管理和绘制能力。

- **渲染引擎（Render Engine）**

  提供了高效的绘制能力，将渲染管线收集的渲染指令，绘制到屏幕能力。

- **平台适配层（Porting Layer）**

  提供了对系统平台的抽象接口，具备接入不同系统的能力，如系统渲染管线、生命周期调度等。

<br>

### UI描述规范

<br>

### 组件化

<br>

### UI状态管理

<br>

<br><br>
