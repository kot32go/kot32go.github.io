---
layout: post
title: Android 开发中常用的设计模式
description: "Android设计模式"
tags: [设计模式]
---

## Bulider 模式

转载自[Mr.Simple的专栏]("http://blog.csdn.net/bboyfeiyu/article/details/24375481")

在Android 开发中，Builder 模式出现较多的情况就是AlertDialog.Builder了，Buildr模式究竟在什么情况下适用呢？

### 使用场景



- 相同的方法，不同的执行顺序，产生不同的事件结果时；
- 多个部件或零件，都可以装配到一个对象中，但是产生的运行结果又不相同时；
- 产品类非常复杂，或者产品类中的调用顺序不同产生了不同的效能，这个时候使用建造者模式非常合适


![UML图](http://img2.imgtn.bdimg.com/it/u=3669888755,2800577221&fm=15&gp=0.jpg)

- Product 产品类 :  产品的抽象类。
- Builder : 抽象类， 规范产品的组建，一般是由子类实现具体的组件过程。
- ConcreteBuilder : 具体的构建器.
- Director : 统一组装过程(可省略)。


在Android源码中，我们最常用到的Builder模式就是AlertDialog.Builder， 使用该Builder来构建复杂的AlertDialog对象。简单示例如下 : 


    private void showDialog(Context context) {  
            AlertDialog.Builder builder = new AlertDialog.Builder(context);  
            builder.setIcon(R.drawable.icon);  
            builder.setTitle("Title");  
            builder.setMessage("Message");  
            builder.setPositiveButton("Button1",  
                    new DialogInterface.OnClickListener() {  
                        public void onClick(DialogInterface dialog, int whichButton) {  
                            setTitle("点击了对话框上的Button1");  
                        }  
                    });  
            builder.setNeutralButton("Button2",  
                    new DialogInterface.OnClickListener() {  
                        public void onClick(DialogInterface dialog, int whichButton) {  
                            setTitle("点击了对话框上的Button2");  
                        }  
                    });  
            builder.setNegativeButton("Button3",  
                    new DialogInterface.OnClickListener() {  
                        public void onClick(DialogInterface dialog, int whichButton) {  
                            setTitle("点击了对话框上的Button3");  
                        }  
                    });  
            builder.create().show();  // 构建AlertDialog， 并且显示
        } 








