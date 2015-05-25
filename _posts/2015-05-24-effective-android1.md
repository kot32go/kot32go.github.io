---
layout: post
title: Effective Android 1
description: "Effective Android"
tags: [effective android]
---


##缘由

最近在读Effective Java ，深感写出的代码尚未达标，故以博文记录自己对其中知识点在Android 开发应用中的领悟。<a id="more"></a>

###1.使用静态工厂方法代替构造器

比如：

	public static Boolean valueOf(boolean b){
		return b?Booean.TRUE:Boolean.FALSE;
	}                                             


####好处：

* 有名称，便于理解（这句话挺废的）
* 不用每次调用时都重新创建一个对象（便于实现缓存复用），这一条对于掌上设备的开发还是很有用的，像什么Fragment 可以考虑缓存一下。
* 能够返回子类型的对象
* 创建参数化了类型的时候使代码更简洁（说白了好看一些）

比如你本来要这么初始化的：

	Map<String,List<String>> m=new HashMap<String,List<String>>();
	
如果在HashMap中有了以下工厂方法：

	public static<K,V> HashMap<K,V> new Instance(){
		return new HashMap<K,V>();
	} 
	
就可以这样：

	Map<String,List<String>> m=HashMap.newInstance();
	
就是简单了一丢丢。

####坏处：

* 类如果没有构造器，就不能子类化
* 与其他静态方法看起来差不多，难以分辨，可以使用以下惯用名称：
* * valueOf，参照String的，其实就是类型转换方法
* * getInstance 返回一个实例，可能是单例的
* * newInstance 返回一个实例，不会是单例


###总结

在一些实体类或Fragment 中，可以考虑使用这个方法

...