---
layout: post
title: Android 动态加载
description: "Android Dynamic Loader"
tags: [android]
---


Android 动态加载<!--more-->

大致思路如下：

1.通过apk获取ClassLoader

	final File optimizedDexOutputPath = context.getDir("outdex", Context.MODE_PRIVATE);//先找到默认的输出路径
	pluginClassLoader = new DexClassLoader(apkFile.getAbsolutePath(), optimizedDexOutputPath.getAbsolutePath(), null, baseClassLoader);

baseClassLoader 通过 Setting.class.getClassLoader() 得到,是主app 的classLoader

2.通过类名让ClassLoader 去加载类

	Class<?> componentClass;
	StringBuilder builder = classNameStrBuilder.insert(0, "com.alibaba.android.wing.hybrid.product.plugin.");
	String className = builder.toString();
	if (classLoader != null) {
			//通过类名将类加载到内存中
	       componentClass = classLoader.loadClass(className);
	} else {
			//如果插件中没找到，那么去主客中查找
	       componentClass = Setting.class.getClassLoader().loadClass(className);
	}