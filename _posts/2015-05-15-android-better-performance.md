---
layout: post
title: Android 性能优化笔记
description: "Android-better-performance"
tags: [Android 优化]
---

Android 的性能优化的点很多很多，但随着Android 系统本身的版本不断的进化，其中有一些问题已经不需要再解决，故这里记录一些目前来说需要注意的地方。

#1.过度绘制
在Android 手机的开发者模式中，可以开启“过度绘制视觉呈现”来让开发者注意到这个问题的出现布局位置。开启后，手机屏幕颜色越深的地方过度绘制越发严重，可以通过 view hierarchy 工具对相应位置的布局进行查看。解决方法：

* 减少不必要的background 设置项
* 使用canvas.clipRect 方法减少不必要的自定义图形绘制。clip之前先调用canvas.save，之后在调用canvas.restore
* 使用Hierarchy Viewer检查布局层级关系
* 布局尽量扁平化，能用一个RelativeLayout 的地方尽量不要用嵌套布局，尤其是在item 布局中
	
#2.Compute
一旦计算超时，会造成卡顿甚至死机等情况，一定要小心避免（不要问我怎么知道的）

* 注意用TraceView 分析工具来分析方法的调用时间情况
* 在计算量大且频繁时，一定要使用缓存来将前一次的计算结果保存
* 不要在主线程进行数据库查询等耗时工作
* 使用Trace.startSection() 和Trace.endSection() 来跟踪方法的使用情况


#3.Memory
注意GC过于频繁和内存泄露，管理不当的话大量的GC期间会造成APP 的卡顿。这里推荐一个小工具：[Leakcanary]("https://github.com/square/leakcanary")（一个内存泄露分析工具），使用方法很简单，参考链接即可
下面是内存泄露的一些常见的情况，转载自：[Linux公社]("http://www.linuxidc.com/Linux/2011-10/44785.htm")

* 查询数据库而没有关闭Cursor
* 调用registerReceiver后未调用unregisterReceiver().
* 未关闭InputStream/OutputStream
* Bitmap使用后未调用recycle()
* 如果设置了一个静态的当Drawable 对象，当它被附加到View时，这个View会被设置为这个Drawable的callback (通过调用Drawable.setCallback()实现)。这就意味着，这个Drawable拥有一个TextView的引用，而TextView又拥有一个Activity的引用。这就会导致Activity在销毁后，内存不会被释放。（同样适用于其他static 的情况），所以在使用static 对象时务必小心，可以用全局的getApplicationContext 代替Activity
	
	
...
