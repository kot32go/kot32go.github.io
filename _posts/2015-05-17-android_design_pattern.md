---
layout: post
title: Android 开发中常用的设计模式
description: "Android设计模式"
tags: [设计模式]
---

##Bulider 模式
部分内容转载自[Mr.Simple的专栏]("http://blog.csdn.net/bboyfeiyu/article/details/24375481")

在Android 开发中，Builder 模式出现较多的情况就是AlertDialog.Builder了，Buildr模式究竟在什么情况下适用呢？

###使用场景


* 相同的方法，不同的执行顺序，产生不同的事件结果时；
* 多个部件或零件，都可以装配到一个对象中，但是产生的运行结果又不相同时；
* 产品类非常复杂，或者产品类中的调用顺序不同产生了不同的效能，这个时候使用建造者模式非常合适

![UML图]("../images/1.png")




