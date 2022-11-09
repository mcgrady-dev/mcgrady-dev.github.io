---
layout: article
title: Android Animation
date: 2022-11-09 14:59 +0800
tags: android

---

Android提供了平移（Translate）、缩放（Scale）、旋转（Rotate）以及透明度（Alpha）4中类型的动画。

<!--more-->





### 使用动画的注意事项

- OOM问题：这个问题主要出现在帧动画中，当图片数量较多且图片较大时极易出现OOM，应尽量避免使用帧动画。
- 内存泄露：在属性动画中有一类无线循环的动画，这类动画需要在Activity退出时及时停止，否则将导致Activity无法释放从而造成内存泄露，通过验证后发现View动画并不存在此问题。
- View动画的问题：View动画是针对View的影响做动画，并不会真正改变View的状态，因此有时候会出现动画完成后View无法隐藏的现象，即 `setVisible(View.GONE)`失效了，这个时候调用 `view.clearAnimation()`清除View动画即可解决问题。
- 不要使用`px`：在进行动画过程中，尽量使用`dp`，使用`px`会导致在不同的设备上有不同的效果。
- 硬件加速：使用动画的过程中，建议开启硬件加速，这样会提高动画的流畅性。





























