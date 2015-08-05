---
layout: post
title: Android 滚动类OverSrcoller
description: "Android OverSrcoller"
tags: [android]
---


Android 实现滚动效果的类 OverScroll<!--more-->

Android 中View的滚动，分两种情况，一种是View随着手指的移动而移动，一种是手指离开了，View受“惯性”移动，符合人的认知直觉。

所有View都是“无界”的，也就是滚动相当于把之前看不见的View移动到屏幕当中去，造成滚动的感觉。

所以，滚动的本质就是位移+动画。

构造函数：

	mScroller = new OverScroller(mContext, decelerateInterpolator);
	
传入上下文和插值函数以供计算动画。

OverScroll 提供了以下三个主要的接口:

* public void startScroll(int startX, int startY, int dx, int dy, int duration)

* public void fling(int startX, int startY, int velocityX, int velocityY,
            int minX, int maxX, int minY, int maxY, int overX, int overY)
            
* public boolean computeScrollOffset()

* startScroll 函数接受滚动的起始位置，在x，y 方向的速度，以及滚动动画的时间

* fling 表示手指屏幕离开后的“惯性”滚动动画，比startScroll 多了x，y方向上的最大和最小滚动距离

* computeScrollOffset 能够利用插值函数与当前时间点计算当前时刻的状态

如何使用？（网上的例子）

    public class CustomView extends LinearLayout {  
      
        private static final String TAG = "Scroller";  
      
        private OverScroller mScroller;  
      
        public CustomView(Context context, AttributeSet attrs) {  
            super(context, attrs);  
            mScroller = new OverScroller(context);  
        }  
      
        //调用此方法滚动到目标位置  
        public void smoothScrollTo(int fx, int fy) {  
            int dx = fx - mScroller.getFinalX();  
            int dy = fy - mScroller.getFinalY();  
            smoothScrollBy(dx, dy);  
        }  
      
        //调用此方法设置滚动的相对偏移  
        public void smoothScrollBy(int dx, int dy) {  
      
            //设置mScroller的滚动偏移量  
            mScroller.startScroll(mScroller.getFinalX(), mScroller.getFinalY(), dx, dy);  
            invalidate();//这里必须调用invalidate()才能保证computeScroll()会被调用，否则不一定会刷新界面，看不到滚动效果  
        }  
          
        @Override  
        public void computeScroll() {  
          
            //先判断mScroller滚动是否完成  
            if (mScroller.computeScrollOffset()) {  
              
                //这里调用View的scrollTo()完成实际的滚动  
                scrollTo(mScroller.getCurrX(), mScroller.getCurrY());  
                  
                //必须调用该方法，否则不一定能看到滚动效果  
                postInvalidate();  
            }  
            super.computeScroll();  
        }  
    }  





