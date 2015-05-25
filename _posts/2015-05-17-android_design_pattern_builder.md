---
layout: post
title: Android 设计模式 Builder
description: "Android设计模式"
tags: [设计模式]
---

## Builder 模式

转载自[Mr.Simple的专栏]("http://blog.csdn.net/bboyfeiyu/article/details/24375481")

在Android 开发中，Builder 模式出现较多的情况就是AlertDialog.Builder了，Buildr模式究竟在什么情况下适用呢？

<!--more-->

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
        
     
        
### 使用场景
       
 
 
     // AlertDialog
    public class AlertDialog extends Dialog implements DialogInterface {
        // Controller, 接受Builder成员变量P中的各个参数
        private AlertController mAlert;

        // 4 : 构造AlertDialog
        AlertDialog(Context context, int theme, boolean createContextWrapper) {
            super(context, resolveDialogTheme(context, theme), createContextWrapper);
            mWindow.alwaysReadCloseOnTouchAttr();
            mAlert = new AlertController(getContext(), this, getWindow());
        }

        // 实际上调用的是mAlert的setTitle方法
        @Override
        public void setTitle(CharSequence title) {
            super.setTitle(title);
            mAlert.setTitle(title);
        }
        // AlertDialog其他的代码省略
        
        // ************  Builder为AlertDialog的内部类   *******************
        public static class Builder {
            // 1 : 存储AlertDialog的各个参数, 例如title, message, icon等.
            private final AlertController.AlertParams P;
            // 属性省略
            // 构造方法
            public Builder(Context context) {
                this(context, resolveDialogTheme(context, 0));
            }
            public Builder(Context context, int theme) {
                P = new AlertController.AlertParams(new ContextThemeWrapper(
                        context, resolveDialogTheme(context, theme)));
                mTheme = theme;
            }     
            // 2 : 设置各种参数..
            public Builder setTitle(CharSequence title) {
                P.mTitle = title;
                return this;
            }
            public Builder setMessage(CharSequence message) {
                P.mMessage = message;
                return this;
            }
            
            // 3 : 构建AlertDialog, 传递参数
            public AlertDialog create() {
                // 调用new AlertDialog构造对象， 并且将参数传递个体AlertDialog 
                final AlertDialog dialog = new AlertDialog(P.mContext, mTheme, false);
                // 5 : 将P中的参数应用的dialog中的mAlert对象中
                P.apply(dialog.mAlert);
                dialog.setCancelable(P.mCancelable);
                if (P.mCancelable) {
                    dialog.setCanceledOnTouchOutside(true);
                }
                dialog.setOnCancelListener(P.mOnCancelListener);
                if (P.mOnKeyListener != null) {
                    dialog.setOnKeyListener(P.mOnKeyListener);
                }
                return dialog;
            }
        }
        
    }


总的来说，Builder 负责将传递来的各种参数进行装配，将构造好的对象进行返回，在创建复杂对象时经常用到。
       
      


### 优点缺点

####优点

* 良好的封装性， 使用建造者模式可以使客户端不必知道产品内部组成的细节；
* 建造者独立，容易扩展；
* 在对象创建过程中会使用到系统中的一些其它对象，这些对象在产品对象的创建过程中不易得到。

####缺点

* 会产生多余的Builder对象以及Director对象，消耗内存；
* 对象的构建过程暴露。






