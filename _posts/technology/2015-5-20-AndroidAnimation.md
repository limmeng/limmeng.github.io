---
title: Android动画初步
date: 2015-5-20 12:00
layout: post
category: technology
--- 
好的动画效果对用户体验的提升有着很大的作用，比如说某个耗时操作搭配上有趣的载入动画能够避免或者至少减少用户等待的烦躁感。
##逐帧动画
通过xml文件定义，举例：  
{% highlight xml %}  
<?xml version="1.0" encoding="utf-8"?>
<animation-list 
	xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot=true>
    <item android:drawable="xxx"
		android:duration="300" />
</animation-list>  
{% endhighlight %}  
定义animation-list，其中每一个item为一帧，每帧分别定义播放时间duration；  
`oneshot`设置循环，为true不循环播放。
##补间动画
补间动画跟逐帧动画类似，只不过补间动画只定义了关键帧，也就是初始帧与结束帧，在初始与结束之间的过程由系统通过计算提供。
以`<set.../>`元素为根元素，该元素内可以定义：  
alpha(透明度)、scale（缩放）、translate（位移）、rotate（旋转）；在属性中设置from...，to...来定义属性的初始值及结束值，若为scale，rotate还需要定义中心点，通过设置pivotX及pivotY确定；  
举例（从中心消失的动画）：  
{% highlight xml %}  
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:fillAfter="true"
    android:interpolator="@android:anim/linear_interpolator"
    android:shareInterpolator="true">
    <scale
        android:duration="200"
        android:pivotX="50%"
        android:pivotY="50%"
        android:fromXScale="1.0"
        android:toXScale="0.0"
        android:fromYScale="1.0"
        android:toYScale="0.0"/>
</set>  
{% endhighlight %}  
几个属性讲解：  
*  fillAfter：动画结束后是否保持最后的状态  
*  interpolator:动画的变换速度  
##属性动画  
属性动画可以看成是增强版的补间动画：  
1. 补间动画只对那4种属性定义关键帧，而属性动画可以对任意属性定义关键帧；  
2. 补间动画只操作UI元素，而属性动画可以操作任意元素。
##其它
AnimationDrawable的使用：  
载入：使用<code>AnimationUtils</code>载入动画；   
开始动画：<code>startAnimation(Animation)</code>；  
另外，可以通过<code>setAnimationListener</code>控制动画各阶段的操作。
