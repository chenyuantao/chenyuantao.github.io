---
layout: post
author: chenyuantao
title: Android使用ValueAnimator开发仿余额宝数值增加控件
category: android
tag: [android]
---

###效果图
![效果图](http://img.blog.csdn.net/20160715225912628)
###原理
首先我们需要写一个NumberView继承自TextView。

	public class NumberView extends TextView{
		public NumberView(Context context) {
			super(context);
		}

		public NumberView(Context context, AttributeSet attr) {
			super(context, attr);
		}

		public NumberView(Context context, AttributeSet attr, int defStyle) {
			super(context, attr, defStyle);
		}
	}

增加一个私有方法实现数值增加动画，用到了ValueAnimator，其中mStartNumber为起始数值，mEndNumber为结束数值，mDuration为动画持续的时间。为该valueAnimator添加一个AnimatorUpdateListener，监听每一次的数值更新，实现onAnimationUpdate方法，将更新后的数值格式化后设置到TextView中，就是这么简单。

	private void runAnimator() {
		ValueAnimator valueAnimator = ValueAnimator.ofFloat(mStartNumber,
				mEndNumber);
		valueAnimator.setDuration(mDuration);
		valueAnimator.addUpdateListener(new AnimatorUpdateListener() {

			@Override
			public void onAnimationUpdate(ValueAnimator va) {
				Float f = Float.parseFloat(va.getAnimatedValue().toString());
				if(f.floatValue()==mEndNumber){
					isRunning=false;
				}
				setText(mDecimalFormat.format(f.floatValue()));
			}
		});
		valueAnimator.start();
	}

之后再写一个start()方法，包装runAnimator方法。

	public void start(float startNumber, float endNumber, long duration) {
		if(!isRunning){
			isRunning=true;
			mStartNumber = startNumber;
			mEndNumber = endNumber;
			mDuration = duration;
			mDecimalFormat = new DecimalFormat(".00");
			runAnimator();
		}		
	}

###完整代码
[github](https://github.com/chenyuantao/AndroidNumberView)

