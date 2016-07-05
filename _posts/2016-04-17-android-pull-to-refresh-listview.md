---
layout: post
author: chenyuantao
title: android自定义下拉更新上拉刷新的ListView
category: android
tag: [android]
---
###**引言**
之前做安卓项目，都没有绕过下拉刷新上拉更新的ListView这个需求，每次遇到这样的问题，都是从网上找开源的ListView或者用回以前用过的开源ListView来实现，现在大三出来实习了，公司项目又遇到这样的问题，因此决定自己动手自定义一个来用。
###**效果图**
####下拉刷新
![下拉刷新](http://img.blog.csdn.net/20160417112513479)
####上拉更新
![上拉更新](http://img.blog.csdn.net/20160417112538558)
###**实现原理**
简单的实现方法是，为ListView添加HeaderView和FooterView，然后将HeaderView和FooterView隐藏起来，等检测到下拉动作或上拉动作的时候，将相应的View显示出来。
本例中，HeaderView和FooterView采用不同的隐藏和显示方式，HeaderView通过setPadding方法隐藏在第一项中，而FooterView则是直接setVisibility(View.GONE);来隐藏的，初始化的代码如下：

    {% highlight java linenos %}
    public void initView(Context context) {
        //关闭View的OverScroll
        setOverScrollMode(OVER_SCROLL_NEVER);
        //设置滑动监听
        setOnScrollListener(this);
        //加载头尾布局
        headerView = (RelativeLayout) LayoutInflater.from(context).inflate(R.layout.layout_header,this,false);
        footerView = (RelativeLayout) LayoutInflater.from(context).inflate(R.layout.layout_footer, this,false);
        progressWheel = (ProgressWheel) headerView.findViewById(R.id.progressWheel);
        progressBar = (ProgressBar) footerView.findViewById(R.id.progressBar);
        //测量头布局的高度
        measureView(headerView);
        headerViewHeight = headerView.getMeasuredHeight();
        //添加到ListView
        addHeaderView(headerView);
        addFooterView(footerView);
        //隐藏headerView到第一项里面
        headerView.setPadding(0, -headerViewHeight, 0, 0);
        //直接隐藏footerView
        footerView.setVisibility(View.GONE);
        //初始化默认状态量
        state = DONE;
        isRefreable = false;
        isRecord = false;
        isUpdatable = false;
    }
    {% endhighlight %}



初始化界面之后，我们就可以开始检测上拉和下拉动作了，不过在检测之前，我们还要确定当前状态是否可以刷新或更新，令我们的ListView继承OnScrollListener，然后重写onScroll方法：

    @Override
    public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
        View lastVisibleItemView = this.getChildAt(this.getChildCount() - 1);
        View firstVisibleItemView = this.getChildAt(0);
        if (firstVisibleItem == 0 && firstVisibleItemView 
                && firstVisibleItemView.getTop() == 0) {
            isRefreable = true;
        } else if ((firstVisibleItem + visibleItemCount) == totalItemCount
                && lastVisibleItemView != null) {
            isUpdatable = true;
        } else {
            isRefreable = false;
            isUpdatable = false;
        }
    }

做好准备之后，关键部分就是要重写onTouchEvent方法，来实现对用户的滑动监听。首先我们要把ListView的状态细分为7个，分别是：    

        private static final int DONE = 0; // 已完成状态
        private static final int PULL_TO_REFRESH = 1; // 下拉刷新状态
        private static final int OK_TO_REFRESH = 2; // 可以刷新的状态
        private static final int REFRESHING = 3; // 正在刷新状态
        private static final int PULL_TO_UPDATE = 4; // 上拉更新状态
        private static final int OK_TO_UPDATE = 5; // 可以更新的状态
        private static final int UPDATING = 6; // 正在更新状态

DONE（已完成状态）：就是正常状态下的ListView。
PULL_TO_REFRESH（下拉刷新状态）：下拉ListView时，HeaderView被拖出来但是又没有拖到可以刷新的状态。
OK_TO_REFRESH（可以刷新状态）：下拉ListView时，手指移动距离大于HeaderView的高度，HeaderView被完整的拖出来的状态。
REFRESHING（正在刷新状态）：当ListView处于可以刷新状态的时候手指释放了，就进入正在刷新状态，HeaderView被固定的显示出来。
上拉更新的状态同下拉刷新，就不一一阐述了。
重写OnTouchEvent的代码如下：

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        //如果当前状态是正在刷新或正在更新，则返回
        if (state == REFRESHING || state == UPDATING) {
            return super.onTouchEvent(ev);
        }
        switch (ev.getAction()) {
            //当用户按下手指
            case MotionEvent.ACTION_DOWN:
                //如果没有记录过初始Y坐标，则记录
                if (!isRecord) {
                    isRecord = true;
                    startY = ev.getY();
                }
                break;
            //当用户拖动手指
            case MotionEvent.ACTION_MOVE:
                //再次判断是否记录了初始Y坐标，若没记录则记录
                if (!isRecord) {
                    isRecord = true;
                    startY = ev.getY();
                }
                //计算出y的偏移量
                offsetY = ev.getY() - startY;
                //如果处于已完成状态
                if (state == DONE && isRecord) {
                    if (offsetY < 0) {
                        if (isUpdatable) {
                            // 将状态改为上拉更新状态
                            state = PULL_TO_UPDATE;
                            //显示尾部
                            footerView.setVisibility(View.VISIBLE);
                        }
                    } else {
                        if (isRefreable) {
                            //将状态改为下拉更新状态
                            state = PULL_TO_REFRESH;
                        }
                    }
                }
                //如果可以更新，并且处在上拉状态或可以更新状态中
                if (state == PULL_TO_UPDATE || state == OK_TO_UPDATE && isUpdatable && isRecord) {
                    //如果偏移量＜0 （即手指往上拖动）
                    if (offsetY < 0) {
                        //根据偏移量计算出进度条的进度
                        int progress = Math.abs((int) (offsetY / RATIO));
                        //如果进度条的值超过了120（100为刚好合并的状态）,就将状态量设置为
                        if (progress > 120) {
                            progress = 120;
                            state = OK_TO_UPDATE;
                        } else {
                            state = PULL_TO_UPDATE;
                        }
                        //根据进度值绘制进度条（让进度条跟随手指的拖动变化）
                        progressBar.setProgress(progress);
                    }
                }
                //如果可以刷新，并且处于下拉状态或者可以刷新状态中
                if (state == PULL_TO_REFRESH || state == OK_TO_REFRESH && isRefreable && isRecord) {
                    setSelection(0);
                    //如果当前滑动的距离小于headerView的高度
                    if (offsetY / RATIO < headerViewHeight) {
                        state = PULL_TO_REFRESH;
                    } else {
                        state = OK_TO_REFRESH;
                    }
                    headerView.setPadding(0, (int) (-headerViewHeight + offsetY
                            / RATIO), 0, 0);
                }
                break;
            // 当用户手指抬起时
            case MotionEvent.ACTION_UP:
                // 如果当前状态为下拉未到刷新位置状态，则不刷新
                if (state == PULL_TO_REFRESH) {
                    //将状态设置为已完成
                    state = DONE;
                    // 隐藏headerView
                    headerView.setPadding(0, -headerViewHeight, 0, 0);
                }
                // 如果当前状态为可以刷新状态
                if (state == OK_TO_REFRESH) {
                    //将状态设置为正在刷新
                    state = REFRESHING;
                    //将headerView放到刚好刷新的位置
                    headerView.setPadding(0,0,0,0);
                }
                // 如果当前状态为上拉未到刷新位置状态，则不更新
                if (state == PULL_TO_UPDATE) {
                    //将状态设置为已完成
                    state = DONE;
                    // 隐藏footerView
                    footerView.setVisibility(View.GONE);
                    //重置progressBar
                    progressBar.setProgress(0);
                    progressBar.stopAnimation();
                }
                //如果当前状态为可以更新状态
                if (state == OK_TO_UPDATE) {
                    //将状态设置为正在刷新
                    state = UPDATING;
                    //开启进度条动画
                    progressBar.startAnimation();
                }
                //将记录y坐标的isRecord改为false，以便于下一次手势的执行
                isRecord = false;
                break;
        }
        return super.onTouchEvent(ev);
    }

另外不要忘了做回调接口

    private OnRefreshUpdateListener listener; //回调接口

    public interface OnRefreshUpdateListener {
        void onRefresh();

        void onUpdate();
    }

    public void setOnRefreshUpdateListener(OnRefreshUpdateListener onRefreshUpdateListener) {
        listener = onRefreshUpdateListener;
    }

    /**
     * 刷新完毕之后，主线程调用该方法隐藏头部布局
     */
    public void setRefreshComplete() {
        //将状态量设置为已完成
        state = DONE;
        //隐藏头部布局
        headerView.setPadding(0, -headerViewHeight, 0, 0);
    }

    /**
     * 更新完毕之后，主线程调用该方法隐藏尾部布局
     */
    public void setUpdateComplete() {
        //将状态量设置为已完成
        state = DONE;
        //隐藏尾部布局
        footerView.setVisibility(GONE);
        //停止进度条动画
        progressBar.stopAnimation();
    }


###**完整代码**
完整代码我上传到了Github上，注释都写好了，研究起来很方便
[Github地址](https://github.com/chenyuantao/RefreshUpdateListView)

关于HeaderView中的ProgressWheel，我是用的一个开源的Material Design风格的LoadingView，Github地址是：
[ProgressWheel](https://github.com/pnikosis/materialish-progress)

关于FooterView中的ProgressBar，是我自定义控件实现的，有兴趣的可以看看，原理不难。

###**备注**
由于添加了HeaderView，在计算position的时候，第0项是HeaderView，而第1项才是用户传进去的数组的第0项，在ListView.OnItemClickListener事件中，需对position进行-1操作。
    