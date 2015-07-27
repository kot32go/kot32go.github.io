---
layout: post
title: Android EventBus 原理分析
description: "EventBus 原理分析"
tags: [android]
---


仔细 简单分析 Android 事件通信库 EventBus 的实现原理<!--more-->

首先，EventBus 有四种订阅函数（即事件发生时调用的方法）

 * onEvent

 * onEventMainThread

 * onEventBackground

 * onEventAsync
 
 
使用方法就先略过了，下面看看实现原理。

入口：EventBus.register

	private synchronized void register(Object subscriber, String methodName, boolean sticky, int priority) {
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriber.getClass(),
                methodName);
        for (SubscriberMethod subscriberMethod : subscriberMethods) {
            subscribe(subscriber, subscriberMethod, sticky, priority);
        }
    }
    

一般来说，使用时会传入一个Java类（subscriber）,通过 findSubscriberMethods 找到这个类中所有的订阅方法：

	List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass, String eventMethodName) {
    //通过订阅者类名+"."+"onEvent"创建一个key
    String key = subscriberClass.getName() + '.' + eventMethodName;
    List<SubscriberMethod> subscriberMethods;
    synchronized (methodCache) {
      //判断是否有缓存，有缓存直接返回缓存
      subscriberMethods = methodCache.get(key);
    }
    //第一次进来subscriberMethods肯定是Null
    if (subscriberMethods != null) {
      return subscriberMethods;
    }
    subscriberMethods = new ArrayList<SubscriberMethod>();
    Class<?> clazz = subscriberClass;
    HashSet<String> eventTypesFound = new HashSet<String>();
    StringBuilder methodKeyBuilder = new StringBuilder();
    while (clazz != null) {
      String name = clazz.getName();
      //过滤掉系统类
      if (name.startsWith("java.") || name.startsWith("javax.") || name.startsWith("android.")) {
        // Skip system classes, this just degrades performance
        break;
      }

      // Starting with EventBus 2.2 we enforced methods to be public (might change with annotations again)
      //通过反射，获取到订阅者的所有方法
      Method[] methods = clazz.getMethods();
      for (Method method : methods) {
        String methodName = method.getName();
        //只找以onEvent开头的方法
        if (methodName.startsWith(eventMethodName)) {
          int modifiers = method.getModifiers();
          //判断订阅者是否是public的,并且是否有修饰符，看来订阅者只能是public的，并且不能被final，static等修饰
          if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) {
            //获得订阅函数的参数
            Class<?>[] parameterTypes = method.getParameterTypes();
            //看了参数的个数只能是1个
            if (parameterTypes.length == 1) {
              //获取onEvent后面的部分
              String modifierString = methodName.substring(eventMethodName.length());
              ThreadMode threadMode;
              if (modifierString.length() == 0) {
                //订阅函数为onEvnet
                //记录线程模型为PostThread,意义就是发布事件和接收事件在同一个线程执行，详细可以参考我对于四个订阅函数不同点分析
                threadMode = ThreadMode.PostThread;
              } else if (modifierString.equals("MainThread")) {
                //对应onEventMainThread
                threadMode = ThreadMode.MainThread;
              } else if (modifierString.equals("BackgroundThread")) {
                //对应onEventBackgrondThread
                threadMode = ThreadMode.BackgroundThread;
              } else if (modifierString.equals("Async")) {
                //对应onEventAsync
                threadMode = ThreadMode.Async;
              } else {
                if (skipMethodVerificationForClasses.containsKey(clazz)) {
                  continue;
                } else {
                  throw new EventBusException("Illegal onEvent method, check for typos: " + method);
                }
              }
              //获取参数类型，其实就是接收事件的类型
              Class<?> eventType = parameterTypes[0];
              methodKeyBuilder.setLength(0);
              methodKeyBuilder.append(methodName);
              methodKeyBuilder.append('>').append(eventType.getName());
              String methodKey = methodKeyBuilder.toString();
              if (eventTypesFound.add(methodKey)) {
                // Only add if not already found in a sub class
                //封装一个订阅方法对象，这个对象包含Method对象，threadMode对象，eventType对象
                subscriberMethods.add(new SubscriberMethod(method, threadMode, eventType));
              }
            }
          } else if (!skipMethodVerificationForClasses.containsKey(clazz)) {
            Log.d(EventBus.TAG, "Skipping method (not public, static or abstract): " + clazz + "."
                + methodName);
          }
        }
      }
      //看了还会遍历父类的订阅函数
      clazz = clazz.getSuperclass();
    }
    //最后加入缓存，第二次使用直接从缓存拿
    if (subscriberMethods.isEmpty()) {
      throw new EventBusException("Subscriber " + subscriberClass + " has no public methods called "
          + eventMethodName);
    } else {
      synchronized (methodCache) {
        methodCache.put(key, subscriberMethods);
      }
      return subscriberMethods;
    }
  }
  
是


