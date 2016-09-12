---
layout: post
author: chenyuantao
title: Android自定义Animation实现3D翻转按钮
category: android
tag: [android]
---
##效果图
![效果图](http://img.blog.csdn.net/20160724184828378)
##原理
在代码中继承Animation类，我们只需要重写applyTransformation方法即可完成我们的动画定制，关键代码是使用了Camera类，可以实现视图的平移、远近（大小）和翻转等功能，直接上代码。

	@Override
	protected void applyTransformation(float interpolatedTime, Transformation t) {
		Matrix matrix = t.getMatrix();
		mCamera.save(); //保存当前状态
		if (interpolatedTime > 0.5f) { //当动画进行到一半的时候，替换图片
			mImageView.setImageBitmap(mBitmap);
		} 		
		mCamera.rotateY(180f * interpolatedTime);//旋转180°
		mCamera.getMatrix(matrix);
		matrix.preTranslate(-mCenterX, -mCenterY);
		matrix.postTranslate(mCenterX, mCenterY);
		mCamera.restore(); //载入之前保存的状态
	}

##完整代码
[github](https://github.com/chenyuantao/RotateAnimation)
##使用方法

	// mTargetBitmap is the image which you want to show after click.
	RotateAnimation mRotateAnimation = new RotateAnimation(mImageView, mTargetBitmap);
	mImageView.startAnimation(mRotateAnimation);
