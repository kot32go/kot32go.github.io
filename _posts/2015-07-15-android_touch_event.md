---
layout: post
title: Android 触摸事件分析
description: "Android 触摸事件"
tags: [android]
---


仔细 简单分析Android 触摸事件<!--more-->

网上分析Android 触摸事件的文章很多，网上有的地方也有很多错误，我简单总结了一下，加了一些自己的理解。

* public boolean dispatchTouchEvent      这个方法分发 TouchEvent
* public booleanon onInterceptTouchEvent  这个方法拦截 TouchEvent
* public boolean onTouchEvent          这个方法处理 TouchEvent

梳理一下流程：

当Activity 被点击，点击事件传递到了Activity 自身，默认调用自身的dispatchTouchEvent 方法，将事件分发到了子Viewgroup 并返回了true 表示已经将事件事件分发出去，如果存在子Viewgroup，那么自己的onTouchEvent不会被调用，否则会被调用。

这里可以看出，dispatchTouchEvent 起到一个分发事件的作用，返回值的boolean 表示告诉系统时间是否分发成功。

它的核心是调用super.dispatchTouchEvent(ev); 如果没有这句，只写一个返回值,如果返回值是true ，表示告诉系统分发了事件，但由于实际没有调用分发函数，所以什么也不会发生，也没有真正把事件分发出去，因此onTouchEvent 仍然接收不到事件，如果返回值是false ，表示告诉系统没分发事件，仍然没有任何反应。


事件分发到子Viewgroup 后由它的 dispatchTouchEvent 响应，如果调用了分发函数并返回了true ，表示将事件成功分发到了子View，如果返回了 false，则传递到onInterceptTouchEvent 只有Viewgroup 才会有这个方法）拦截函数中决定是否进行拦截。interceptTouchEvent 如果返回true ，表示拦截了事件，调用 Viewgroup 的onTouchEvent。进而，如果ViewGroup 的onTouchEvent 返回了true，表示消费了事件，等待下一次事件，如果返回false ，在这一次触摸中都不会接收到事件。如果onInterceptTouchEvent 返回了false，表示没有拦截事件，因此事件会进行分发到子View。


首先事件传递到子 View 的dispatchTouchEvent 函数中，如果dispatchTouchEvent 返回false，表示不继续分发了，会传给自己的onTouchEvent，如果dispatchTouchEvent返回了true，表示还要分发事件，但是子View此时没有继续分发事件的子控件了，事件会分发传递到父 Viewgroup的dispatchTouchEvent中。

如果子View 的onTouchEvent 返回值是true ，表示消费掉了事件，如果返回了false，自身在这一次触摸中不再接受其他事件，且事件将会向上传递，一层层往上层的onTouchEvent 事件传递。

嗯。




