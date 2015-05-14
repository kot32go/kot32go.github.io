---
layout: post
title: Android 性能优化笔记
description: "Android-better-performance"
tags: [Android 优化]
---

Android 的性能优化的点很多很多，但随着Android 系统本身的版本不断的进化，其中有一些问题已经不需要再解决，故这里记录一些目前来说需要注意的地方。

#1.过度绘制
在Android 手机的开发者模式中，可以开启“过度绘制视觉呈现”来让开发者注意到这个问题的出现布局位置。开启后，手机屏幕颜色越深的地方过度绘制越发严重，可以通过 view hierarchy 工具对相应位置的布局进行查看。解决方法：

	1.减少不必要的background 设置项
	2.使用canvas.clipRect 方法减少不必要的自定义图形绘制。clip之前先调用save，之后在调用restore
	3.使用Hierarchy Viewer检查布局层级关系
	
...