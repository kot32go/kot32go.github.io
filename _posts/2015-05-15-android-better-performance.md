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

这里插一句，TextView 是为显示富文本乃至图片所设计的，因此渲染速度并不理想。在一般的显示静态文本或者大型文本时最好进行优化，以下是Instagram 的优化方式，值得借鉴：

>上周，成千上万来自全世界的IG用户齐聚在社区组织的先下聚会 Worldwide InstaMeet11上。WWIM11 是历史上最大，最具地域多样性的Instagram聚会，从Muscat到Bushwick，成千上万用户分享了大约10万张照片。我们最近的一项改进是关于渲染庞大复杂的文本以及如何通过改进它优化IG的feed滚动。我们希望你可以从我们的经验中找到提升自己app速度的方法。

##使用text.Layout，缓存text.Layout

Android有很多用于文字展示的控件，但实际上，他们都用text.Layout进行渲染。例如，TextView会将String转化为一个text.Layout对象，并通过canvas API将它绘制到屏幕上。

由于text.Layout需要在构造函数中测量文本的高度，因此它的创建效率不高。缓存text.Layout和复用text.Layout实例 可以节省这部分时间。Android的TextView控件并没有提供设置TextLayout的方法，但是添加一个这样的方法并不困难：

!["缓存text.Layout"](http://static.open-open.com/lib/uploadImg/20150424/20150424111625_487.png)

通过使用TextLayoutView，我们可以缓存和复用text.Layout，从而避免了每次调用TextView的setText(CharSequence c)方法时都要花费20ms来创建它。

##下载feed后准备好Layout缓存

由于我们确定会在下载评论后展示他们，一个简单的改进是在下载它们后就准备好text.Layout的缓存。
![s](http://static.open-open.com/lib/uploadImg/20150424/20150424111632_140.png)

##停止滚动后准备好TextLayoutCache

在可以设置text.Layout缓存后，我们的到来常数级的测量（measure）和绑定（binding）时间。但是初次绘制的时间仍然很长。50ms的绘制时间可能会导致明显的卡顿。

这50ms中的大部分被用于测量文本高度以及产生文字符号。这些都是CPU操作。为了提升文本渲染速度，Android在ICS中引入了 TextLayoutCache用于缓存这些中间结果。TextLayoutCache是一个LRU缓存，缓存的key是文本。如果查询缓存时命中，文本 的绘制速度会有很大提升。

在我们的测试中，这种缓存可以将绘制时间从30ms-50ms减少到2ms-6ms。
![](http://static.open-open.com/lib/uploadImg/20150424/20150424111633_237.png)

为了更好的提升绘制性能，我们可以在绘制文本到屏幕前准备好这个缓存。我们的思路是在一块屏幕外的canvas上虚拟的绘制这些文本。这样在我们绘制文本到屏幕前，TextLayoutCache就已经在一个背景线程中被准备好了。

![](http://static.open-open.com/lib/uploadImg/20150424/20150424111634_594.png)
默认情况下，TextLayoutCache的大小为0.5M，这足以缓存十几张图片的评论。我们决定在用户停止滑动时准备缓存，我们向用户滑动的方向提前缓存5个图片的评论。在任何时候，我们都至少在任何一个方向上缓存了5个图片的评论.

![](http://static.open-open.com/lib/uploadImg/20150424/20150424111635_50.png)
在应用了所有的这些优化后，掉帧情况减少了60%，而卡顿的情况减少了50%。我们希望这些能帮助你提升你app的速度和性能。告诉我们你的想法吧，我们期待听到你的经验。
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
