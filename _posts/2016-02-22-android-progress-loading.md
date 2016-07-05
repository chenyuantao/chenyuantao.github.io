---
layout: post
author: chenyuantao
title: android自定义渐变色等待条
category: android
tag: [markdown]
---

效果图如下：
![变色条效果如图](http://img.blog.csdn.net/20160329094829636)
##原理
原理就是自定义view，重写onDraw方法，并开启一个线程来周期调用invalidate()让控件周期地重新绘制，实现渐变色的移动。
##代码
废话不多说，直接上onDraw方法代码。
```java
@Override
	protected void onDraw(Canvas canvas) {
		super.onDraw(canvas);		
		mRect = new RectF(0, 0, mWidth, mHeight);
		mPaint = new Paint();
		mPaint.setAntiAlias(true);
		float[] positions = new float[3];
		positions[0] = 0f;
		//改变第二颜色的位置，类似于photoshop的渐变色位置设置
		positions[1] = pos;
		positions[2] = 1.0f;
		//线性渐变色
		LinearGradient shader = new LinearGradient(0, 0, mWidth, mHeight,
				SECTION_COLORS, positions, Shader.TileMode.MIRROR);
		//设置到画笔
		mPaint.setShader(shader);
		//绘制到画布
		canvas.drawRect(mRect, mPaint);
		if(!isAction){
			//启动周期绘制线程
			action();
		}
	}
```
再来看周期线程，其实就是每10毫秒将第二颜色的位置增加0.01，然后重绘控件
```java
public void action() {
		pos = 0.00f;
		operator = true;
		isAction = true;
		new Thread(new Runnable() {

			@Override
			public void run() {
				try {
					while (isAction) {
						Thread.sleep(10);
						if (operator) { 
							//往右边移动0.01f
							pos += 0.01f; 
						} else { 
							//往左边移动0.01f
							pos -= 0.01f; 
						}
						if (pos >= 1f) {
							//如果超过了1f，就调头
							operator = false;
						} else if (pos <= 0f) {
							//如果低于了0f，就调头
							operator = true;
						}
						//通知handler
						handler.sendEmptyMessage(1);
					}

				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}

		}).start();

	}
```
handler接受到通知后只需要执行invalidate()就可以重绘控件了
```java
private Handler handler = new Handler() {
		@Override
		public void handleMessage(Message msg) {
			if (msg.what == 1) {
				// 通知控件重绘
				invalidate();
			}
		}
	};
```
##完整代码
[下载地址](http://download.csdn.net/detail/cyt528300/9475174)
##使用方法
```xml
<com.cyt.view.RainbowView
        android:layout_width="match_parent"
        android:layout_height="3dp" />
```
