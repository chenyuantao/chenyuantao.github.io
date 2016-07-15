---
layout: post
author: chenyuantao
title: Android自定义仿Siri曲线View
category: android
tag: [android]
---

###效果图
![目标实现](http://img.ui.cn/data/file/2/4/0/236042.gif?imageView2/2/w/900/q/90)
![自定义View](http://img.blog.csdn.net/20160616230719521)
![自定义View](http://img.blog.csdn.net/20160616230739130)
###代码实现
仔细观察效果图可以发现，波浪其实是由4条贝塞尔曲线组成的，可以在自定义View的onDraw函数中，用Path.quadTo函数画出4条曲线。

	Path.quadTo(float x1, float y1, float x2, float y2)

其中，x1，y1为控制点的坐标值，x2，y2为终点的坐标值；当控制点的x1位于起点与终点之间时，将画出正弦曲线，此时y1控制正弦曲线的高度，即效果图中波浪的高度由y1控制。
实现了曲线绘制和高度控制之后，如何让曲线像波浪一样动起来呢？
我的解决方法是在屏幕左边，即x<0的位置，同样绘制4条正弦曲线，并且启动线程让8条曲线都向右移动，当左边4条曲线全部移动到屏幕内后，让这8条曲线复位。如此周期进行。
![解析](http://img.blog.csdn.net/20160616232218323)
###使用方法
在xml中，

	<com.tao.view.SiriView
	        android:id="@+id/siriView"
	        android:layout_width="match_parent"
	        android:layout_height="100dp"
	        android:layout_centerInParent="true"/>

在Activity.java中，

	SiriView siriView = (SiriView) findViewById(R.id.siriView);
	// 停止波浪曲线
	siriView.stop();
	// 设置曲线高度，height的取值是0f~1f
	siriView.setWaveHeight(0.5f);
	// 设置曲线的粗细，width的取值大于0f
	siriView.setWaveWidth(5f);
	// 设置曲线颜色
	siriView.setWaveColor(Color.rgb(39, 188, 136));
	// 设置曲线在X轴上的偏移量，默认值为0f
	siriView.setWaveOffsetX(0f);
	// 设置曲线的数量，默认是4
	siriView.setWaveAmount(4);
	// 设置曲线的速度，默认是0.1f
	siriView.setWaveSpeed(0.1f);

###Github地址
https://github.com/chenyuantao/SiriView


       

