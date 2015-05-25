---
layout: post
title: Effective Android 1
description: "Effective Android"
tags: [effective android]
---


##缘由

最近在读 Effective Java ，深感写出的代码尚未达标，故以博文记录自己对其中知识点在Android 开发应用中的领悟。<!--more-->

### 1.使用静态工厂方法代替构造器

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

### 2.在类的构建比较复杂时使用Builder模式

### 3.可以利用单枚举实现单例模式

优点：自动提供了序列化机制
例如：类A要实现单例模式，就可以这样：

	public enum A{
   	 	INSTANCE; //这句必须写在第一行
   	 	public int number;
    	public void test(){};
   		public void test1(){};
	}
	
生成单例的时候，只需：

	A a=A.INSTANCE;
	
非常方便，特别是在Intent或者PendingIntent 需要传递这个对象时，自动的序列化也是棒棒的。

### 4.私有化构造器来使类不能被实例化

有的时候我们不希望类被实例化，特别是一些工具类，那么就可以这样：

	public class DBUtil{
		private DBUtil(){
			throw new AssertionError();
		}
	}
	
### 5.避免创建不必要的对象

如果在一个需要调用多次的函数中，尽量不要在函数中创建对象，可以将创建过程放在static 块中或是onCreate 中。

### 6.消除过期的对象引用（避免内存溢出，这点在之前的Android 性能优化博文中提到过）

* 在自己管理内存的类中，在元素失效后，应该将其引用至null
* 在使用缓存时，使用弱引用类型（WeakHashMap）避免引用长时间存在于内存
* 在回调注册使用后，应该显示取消注册
* 查询数据库而没有关闭Cursor
* 调用registerReceiver后未调用unregisterReceiver().
* 未关闭InputStream/OutputStream
* Bitmap使用后未调用recycle()
* 如果设置了一个静态的当Drawable 对象，当它被附加到View时，这个View会被设置为这个Drawable的callback (通过调用Drawable.setCallback()实现)。这就意味着，这个Drawable拥有一个TextView的引用，而TextView又拥有一个Activity的引用。这就会导致Activity在销毁后，内存不会被释放。（同样适用于其他static 的情况），所以在使用static 对象时务必小心，可以用全局的getApplicationContext 代替Activity

以上避免操作可以在onDestory中进行。

### 7.避免使用 finalizer 方法