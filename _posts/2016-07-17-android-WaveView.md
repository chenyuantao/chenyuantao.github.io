---
layout: post
author: chenyuantao
title: Android仿拉勾网app的波浪View
category: android
tag: [android]
---

###前言
最近暑假在找实习单位，各种招聘app都用遍了，感觉拉勾网做的最好看也最实用，于是决定模仿拉勾网app中的波浪视图。
###拉勾网app中的效果
![拉勾网app](http://img.blog.csdn.net/20160717175541692)
###自己实现的效果
![waveview](http://img.blog.csdn.net/20160717175731335)
###原理
惯用套路都是先写一个类继承自View类，然后重写onMeasure和onDraw方法，主要核心在onDraw方法，应该怎么画，重点讲这部分。
仔细观察动画，可以发现该控件其实是由三部分组成，分别是上面的矩形和下面的两条波浪曲线，如图所示。
![原理1](http://img.blog.csdn.net/20160717180911757)
化繁为简，我们目标是先画出这样的图形，然后再增加一条曲线以及令曲线动起来即可。
![原始1](http://img.blog.csdn.net/20160717181403592)
可以看出，该图形其实是一个组合图形，由矩形和一条曲线组合而成，这里需要用到画笔的混合模式Xfermode中的Xor模式，即保留各自不同的部分，删去相同的部分。
![Xfermode](http://img.blog.csdn.net/20160717181729738)
![原理2](http://img.blog.csdn.net/20160717182508296)
绘制如上图形的代码如下：


	@Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        // 新建图层(就像photoshop一样)
        canvas.saveLayer(-0, 0, mViewWidth, mViewHeight, null, Canvas.ALL_SAVE_FLAG);
        mPath.reset();
        //根据之前存好的点绘制四条贝塞尔曲线
        for (int i = 0; i < 4; i++) {
            mPath.moveTo(mStartPoints[i].x , mStartPoints[i].y);
            mPath.quadTo(mControlPoints[i].x , mControlPoints[i].y, mStartPoints[i].x + mViewWidth , mStartPoints[i].y);
        }
        canvas.drawPath(mPath, mPaint);
        //设置混合模式 （交叉部分不绘制）
        mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.XOR));
        //用Xor模式绘制矩形
        canvas.drawRect(0, 0, mViewWidth, mViewHeight - mWaveHeight, mPaint);
        // 还原混合模式
        mPaint.setXfermode(null);
        // 还原图层（与saveLayer方法配对使用）
        canvas.restore();
        }

绘制好了静态的波浪图形，如何让它动起来呢？
![原理3](http://img.blog.csdn.net/20160616232218323)
启动一个线程，周期的让波浪往右移动，形成循环即可。
###源代码
[github](https://github.com/chenyuantao/AndroidWaveView)
###使用方法
在xml中，


	<cn.chenyuantao.view.WaveView
	        android:layout_width="match_parent"
	        android:layout_height="200dp" />


在Activity中，


	//改变颜色
	waveView.setColor(Color.RED);
	//开始
	waveView.start();
	//停止
	waveView.stop();