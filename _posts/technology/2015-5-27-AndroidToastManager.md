---
title: Android 实现 Toast Manager  
date: 2015-5-27 19:20
layout: post
category: technology
---  
在项目中我们经常需要使用Toast为用户提供一些提示信息，但如果用户多次触发Toast的话，系统会让每个Toast显示完再显示下一个Toast，这样Toast就会在屏幕上停留相当长一段时间，用户体验不好。  
其实我们可以通过自定义一个ToastManager类，使每次显示Toast时都先取消正在显示中Toast，下面是实现代码，比较简单：  
{% highlight java %}  
	public class ToastManager {
	    private static WeakReference<Toast> toastReference;
	
	    // 不允许实例化
	    private ToastManager() {
	    }
	
	    // 显示Toast，如正在显示Toast则取消之
	    public static void show(Context mCxt, CharSequence content, int duration) {
	        Toast currentToast = getCurrentToast();
	        if (currentToast != null) {
	            currentToast.cancel();
	        }
	        currentToast = Toast.makeText(mCxt, content, duration);
	        currentToast.show();
	        toastReference = new WeakReference<>(currentToast);
	    }
	
	    public static void show(Context mCxt, int contentRes, int duration) {
	        show(mCxt, mCxt.getResources().getString(contentRes), duration);
	    }
	
	    public static Toast getCurrentToast() {
	        if (toastReference != null) {
	            return toastReference.get();
	        }
	        return null;
	    }
	}
{% endhighlight %}  
然后在需要使用Toast的地方我们只要调用ToastManager的show函数就可以了。