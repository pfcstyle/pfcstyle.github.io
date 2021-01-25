---
layout:		post
title:		"Android-Animation-Gesture"
description	"important point"
date:		2016-11-17
author:		"PfCStyle"
header-img:	"img/post/2016-11-17/head.jpg"
categories: "Android"
keywords
    - Android
    - Animation
    - Gesture
---

> 代码就像DNA,随机组合过程中总会出现毛病。

# Animation

这里不会介绍动画的详细用法，只是记录一些小问题。

### 动画无效

```java

//这里要时刻注意，你传进去要变化的动画纬度参数是否是正确的
TranslateAnimation animation = new TranslateAnimation(0, 0, mEditView.getHeight(), 0);        
animation.setDuration(500);
animation.setAnimationListener(new Animation.AnimationListener() {            
  @Override  public void onAnimationStart(Animation animation) {                
	mEditView.setVisibility(View.VISIBLE);            
	}   
   @Override   public void onAnimationEnd(Animation animation) {}   
   @Override   public void onAnimationRepeat(Animation animation){}        
});        
mEditView.startAnimation(animation);

```

注意上述代码，在动画开始时，使mEditView显示，在动画前，它是隐藏的。这里需要注意的是，如果你在xml文件中(或者使用代码)设置的是gone，而不是invisible，那么，你在设置动画参数时，传入的mEditView.getHeight()就是0，从而导致没有动画发生。

### 动画之后出现异常情况


```java

TranslateAnimation animation = new TranslateAnimation(0, 0, 0, mEditView.getHeight());
//注意这里，如果设为true，动画结束后，将会强制保持结束时的状态。
//animation.setFillAfter(true);
animation.setDuration(500);
animation.setAnimationListener(new Animation.AnimationListener() {
   @Override	
	public void onAnimationStart(Animation animation) {      }	
	@Override            
	public void onAnimationEnd(Animation animation) {
 		mEditView.setVisibility(View.INVISIBLE);
    }
	@Override
   public void onAnimationRepeat(Animation animation) {}
});
mEditView.startAnimation(animation);
```

这里是隐藏的动画，在上面`//animation.setFillAfter(true);`这句，会导致`onAnimationEnd`中的`mEditView.setVisibility(View.INVISIBLE);`无效，准确来说，mEditView保持了动画结束的状态，在屏幕外面。

但是，不仅如此，setFillAfter(true)这句，还使得事件与view分离了。所以，你往往会点到一些看起来诡异的东西。

# Gesture

### view自身具有的手势监听已经很全面了。


```
mHeaderView.setOnTouchListener(new View.OnTouchListener() {
        @Override
        public boolean onTouch(View view, MotionEvent motionEvent) {
);
            switch (motionEvent.getAction()) {
                case MotionEvent.ACTION_DOWN: {
                    
              	    final float y = motionEvent.getY();
                    Log.d("downY:"+y);
                    // Remember where we started (for dragging)
                    mLastTouchY = y;
                    // Save the ID of this pointer (for dragging)
                    break;
                }
                    
                case MotionEvent.ACTION_MOVE: {
                    // Find the index of the active pointer and fetch its position
                    final float y = motionEvent.getY();
                    Log.d("downY:"+mLastTouchY+"===moveY:"+y);
                    
                    // Only move if the ScaleGestureDetector isn't processing a gesture.
                    //                                        if (!mScaleDetector.isInProgress()) {
                    // Calculate the distance moved
                    //                                        final float dx = x - mLastTouchX;
                    final float dy = y - mLastTouchY;
                    
                    RelativeLayout.LayoutParams params = (RelativeLayout.LayoutParams) mQueryListView.getLayoutParams();
                    params.topMargin = (int) (params.topMargin+dy);
                    mQueryListView.setLayoutParams(params);
                    
                    break;
                }
                    
                case MotionEvent.ACTION_UP: {

                    break;
                }
            }
            return true;
        }
    });
```

这里展示的是使用一个手指上下拖动view的事例。注意，这里监听的是一个控件，而不是整个window。在`ACTION_MOVE`中，拿到的y是你的手指在目标view中的坐标，因此，这里只需要减去`ACTION_DOWN`时的初始坐标，就能得到每次需要移动的距离。

注意，目标view随着你的手指在移动，所以，`ACTION_MOVE`中拿到的y一般波动很小，因为你的手指可能一直在目标view的同一个位置。

你的手指每移动1像素，目标view就会跟随移动1像素。

### GestureDetector

在遇到极其复杂的交互情况，可能会需要使用到这里的手势监听器。


```
mGestureDetector = new GestureDetector(mActivity,UpdateFeaturesProtocol.this);
//这里需要注意，必须设置这一句，否则在GestureDetector的实现中，只能响应down,press,longpress三个事件mHeaderView.setLongClickable(true);
mHeaderView.setOnTouchListener(new View.OnTouchListener() {
        @Override
        public boolean onTouch(View view, MotionEvent motionEvent) {
                return mGestureDetector.onTouchEvent(motionEvent);
        }
    });
```

下面是对`OnGestureListener`的实现


```
private float startPointY;
    
    /**
     * Gesture
     * @param motionEvent
     * @return
     */
    //点下时响应  第一个响应
    @Override
    public boolean onDown(MotionEvent motionEvent) {
        Log.d("down==>e2.x:"+motionEvent.getX()+"====e2.y:"+motionEvent.getY());
        RelativeLayout.LayoutParams params = (RelativeLayout.LayoutParams) mQueryListView.getLayoutParams();
        startPointY = motionEvent.getY();
        return false;
    }
    
    //几乎跟down一样，但是在down后
    //比如down响应需要0.1s,那么press需要0.2s
    //如果你的手指接触屏幕时间不足0.2s，该事件不会响应
    //这里的0.1s和0.2s皆为假设，没有测过
    @Override
    public void onShowPress(MotionEvent motionEvent) {
        Log.d("press==>e2.x:"+motionEvent.getX()+"====e2.y:"+motionEvent.getY());
        
    }
    
    //单击
    @Override
    public boolean onSingleTapUp(MotionEvent motionEvent) {
        Log.d("singletap==>e2.x:"+motionEvent.getX()+"====e2.y:"+motionEvent.getY());
        return false;
    }
    
    //滚动 在手指滑动过程中一直响应
    @Override
    public boolean onScroll(MotionEvent motionEvent, MotionEvent motionEvent1, float v, float v1) {
        Log.d("scroll==>e2.y:"+motionEvent1.getY()+"===e1.y:"+motionEvent.getY()+"distanceY:"+v1);
        RelativeLayout.LayoutParams params = (RelativeLayout.LayoutParams) mQueryListView.getLayoutParams();
        params.topMargin = (int) (params.topMargin + (motionEvent1.getY()-startPointY));
        mQueryListView.setLayoutParams(params);
        return false;
    }
    
    //长按
    @Override
    public void onLongPress(MotionEvent motionEvent) {
        Log.d("longpress==>e2.x:"+motionEvent.getX()+"====e2.y:"+motionEvent.getY());
    }
    
    //扔？
    //此事件是指用手指快速滑动并快速抬起的情况
    //注意，此事件在scoll后是不一定响应的。
    @Override
    public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX,
                           float velocityY) {
        Log.d("fling==>e2.x:"+e2.getX()+"====e2.y:"+e2.getY());
        
        return false;
    }
```

功能和直接使用`mHeaderView.setOnTouchListener`完全一样，思路也是完全一样的。解释都在代码注释里。

需要注意的是里面没有监听手指抬起的地方，所以你还需要回到`setOnTouchListener`中使用`ACTION_UP`













