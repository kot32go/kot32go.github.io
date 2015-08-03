---
layout: post
title: Android 类加载器 DexClassLoader
description: "Android DexClassLoader"
tags: [android]
---


Android 类加载器 DexClassLoader<!--more-->

类加载器 ClassLoader ，用来将类加载到JVM 中，其中纯Java 有以下几种类加载器

* BootstrapClassLoader
* user-defined ClassLoader 

JVM在运行时会产生三个ClassLoader:Bootstrap ClassLoader、Extension ClassLoader和AppClassLoader.Bootstrap是用C++编写的，我们在Java中看不到它,是null,是JVM自带的类装载器，用来装载核心类库，如java.lang.*等。
AppClassLoader的Parent是ExtClassLoader，而ExtClassLoader的Parent为Bootstrap ClassLoader。

而在Android 中，有所不同的是将编译好的class 文件再次打包成dex 类型的文件，要加载这种特殊的Class 文件就要用到Android 的DexClassLoader。

DexClassLoader 的构造参数意义如下：

* dexPath 即目标类所在的APK 或jar 文件路径，类加载器从该路径中寻找指定的目标类，该路径必须是全路径，如/data/app/com.xxxx.plugin.apk。如果要包含多个路径，需要用System.getProperty("path.separtor") 得到的分隔符分离
* dexOutputDir 指定从jar 或者Dex 文件中解压出的dex 文件的存放路径，可以使用程序的数据路径
* libPath 目标类中使用的C/C++ 库存放的路径
* parent 指该装载器的父装载器，一般为当前执行调用类的装载器


利用DexClassLoader 加载插件 Apk中指定方法：
(这里指将两个APK都安装到手机后调用)
例如宿主程序名为Host，插件程序名为Plugin，Plugin中有一个类名为PluginClass，如下：
	
	public class PluginClass{
		public PluginClass(){
			Log.i("Plugin","initialized");
		}
		public int function1(int a,int b){
			return a+b;
		}
	}
	
宿主Activity中：

    @SuppressLint("NewApi") private void useDexClassLoader(){  
        //创建一个意图，用来找到指定的apk  
        Intent intent = new Intent("com.suchangli.android.plugin", null);  
        //获得包管理器  
        PackageManager pm = getPackageManager();  
        List<ResolveInfo> resolveinfoes =  pm.queryIntentActivities(intent, 0);  
        //获得指定的activity的信息  
        ActivityInfo actInfo = resolveinfoes.get(0).activityInfo;  
          
        //获得包名  
        String pacageName = actInfo.packageName;  
        //获得apk的目录或者jar的目录  
        String apkPath = actInfo.applicationInfo.sourceDir;  
        //dex解压后的目录,注意，这个用宿主程序的目录，android中只允许程序读取写自己  
        //目录下的文件  
        String dexOutputDir = getApplicationInfo().dataDir;  
          
        //native代码的目录  
        String libPath = actInfo.applicationInfo.nativeLibraryDir;  
        //创建类加载器，把dex加载到虚拟机中  
        DexClassLoader calssLoader = new DexClassLoader(apkPath, dexOutputDir, libPath,  
                this.getClass().getClassLoader());  
          
        //利用反射调用插件包内的类的方法  
          
        try {  
            Class<?> clazz = calssLoader.loadClass(pacageName+".Plugin1");  
              
            Object obj = clazz.newInstance();  
            Class[] param = new Class[2];  
            param[0] = Integer.TYPE;  
            param[1] = Integer.TYPE;  
              
            Method method = clazz.getMethod("function1", param);  
              
            Integer ret = (Integer)method.invoke(obj, 1,12);  
              
            Log.i("Host", "return result is " + ret);  
              
        } catch (ClassNotFoundException e) {  
            e.printStackTrace();  
        } catch (InstantiationException e) {  
            e.printStackTrace();  
        } catch (IllegalAccessException e) {  
            e.printStackTrace();  
        } catch (NoSuchMethodException e) {  
            e.printStackTrace();  
        } catch (IllegalArgumentException e) {  
            e.printStackTrace();  
        } catch (InvocationTargetException e) {  
            e.printStackTrace();  
        }  
       }  

