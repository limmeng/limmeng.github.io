---
title: 开源项目学习之实现 Android Snackbar  
date: 2015-5-22 21:45
layout: post
category: technology
---
什么，你还不知道什么是Snackbar！！！  
请狂戳 Google Material Design中 Snackbar 的说明：[Material Snackbar](https://www.google.com/design/spec/components/snackbars-toasts.html "Material Snackbar")  
下面说明开源项目[nispok/snackbar](https://github.com/nispok/snackbar "nispok/snackbar")中是如何实现带有Action按钮的Snackbar：  
首先，想想我们要实现的功能有哪些？  

* 触发Snackbar后Snackbar从页面底部滑进页面；  
* 当以下任意一个条件发生时Snackbar从页面底部滑出页面；
	* 过一段时间（duration）后
	* Snackbar上的Action按钮点击后
	* 新的Snackbar被激活
* 点击Snackbar上Action按钮的响应；
* 在Snackbar从激活到消失不同阶段的事件回调。  

<img src="https://github.com/limmeng/limmeng.github.io/raw/master/images/posts/snackbar.png" height="250"/>  
下面对功能的实现进行分析：  
Snackbar需要继承某个ViewGroup，选择继承LinearLayout，然后重写onMeasure函数，以保证我们的Snackbar能正常设置layout_ width及layout_ height等参数。在实现时，由于Snackbar在手机和平板上显示时长宽的要求不一样，所以使用一个SnackbarLayout继承LinearLayout，Snackbar继承SnackbarLayout，在SnackbarLayout中重写onMeasure，并提供可以改变onMeasure尺寸的接口，在Snackbar中使用该接口控制显示尺寸。以下代码仅实现Snackbar在手机上的显示。  
先给出SnackbarLayout的代码：  
{% highlight java linenos %}  
	// SnackbarLayout.java
	public class SnackbarLayout extends LinearLayout {
	    private int maxWidth = Integer.MAX_VALUE;
	    private int maxHeight = Integer.MAX_VALUE;
	
	    public SnackbarLayout(Context context) {
	        super(context);
	    }
	
	    public SnackbarLayout(Context context, AttributeSet attrs) {
	        super(context, attrs);
	    }
	
	    public SnackbarLayout(Context context, AttributeSet attrs, int defStyleAttr) {
	        super(context, attrs, defStyleAttr);
	    }
	
	    @Override
	    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	        // 调整宽度
	        int width = MeasureSpec.getSize(widthMeasureSpec);
	        if (maxWidth < width) {
	            int mode = MeasureSpec.getMode(widthMeasureSpec);
	            widthMeasureSpec = MeasureSpec.makeMeasureSpec(maxWidth, mode);
	        }
	        // 调整高度
	        int height = MeasureSpec.getSize(heightMeasureSpec);
	        if (maxHeight < height) {
	            int mode = MeasureSpec.getMode(heightMeasureSpec);
	            heightMeasureSpec = MeasureSpec.makeMeasureSpec(maxHeight, mode);
	        }
	
	        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
	    }
	
		// Snackbar中通过调用这两个函数控制显示尺寸
	    public void setMaxWidth(int width) {
	        maxWidth = width;
	        requestLayout();
	    }
	    public void setMaxHeight(int height) {
	        maxHeight = height;
	        requestLayout();
	    }
	}
{% endhighlight %}  
实现Snackbar上的Action点击按钮以及不同阶段的事件回调，需要提供两个接口：    
{% highlight java linenos %}  
	// ActionClickListener.java
	public interface ActionClickListener {
	    void onActionClicked(Snackbar snackbar);
	}
	// EventListener.java
	public interface EventListener {
		// 刚激活Snackbar时
	    void onShow(Snackbar snackbar);
		// 新的Snackbar通过替换旧的Snackbar而出现
	    void onShowByReplace(Snackbar snackbar);
		// Snackbar进入动画结束时
	    void onShown(Snackbar snackbar);
		// Snackbar即将退出时
	    void onDismiss(Snackbar snackbar);
		// Snackbar被新的Snackbar替换而退出时
	    void onDismissByReplace(Snackbar snackbar);
		// Snackbar完全消失时
	    void onDismissed(Snackbar snackbar);
	}
{% endhighlight %}   
之所以需要专门定义Snackbar被Replace掉时的事件是因为Snackbar普通事件与被替换时事件的操作有可能不一样，例如一般情况下Snackbar退出需要把 Floating Button 复位，而当Snackbar被替换而退出时就无需将 Floating Button 复位了。  
接下来给出Snackbar的阶段图：  
<img src="https://github.com/limmeng/limmeng.github.io/raw/master/images/posts/snackbarPeriods.png"/>   
Snackbar显示一段时间后自动消失是通过定义一个Runnable处理退出过程，然后在Snackbar完全显示后调用postDelay设置，一段时间后调用退出操作。  
下面给出Snackbar实现的完整代码：  
{% highlight java linenos %}  
	public class Snackbar extends SnackbarLayout {
	    // Snackbar 高度
	    private static final int lineHeight = 48;
	    // Snackbar 默认停留时间
	    private static final long NORMAL = 3000;
	
	    // 参数
	    private int mUndefineColor = -1000;
	    private long mDuration = NORMAL;
	    private CharSequence mText;
	    private CharSequence mActionLabel;
	    private int mActionColor = mUndefineColor;
	    private ActionClickListener mActionClickListener;
	    private EventListener mEventListener;
	
	    // View参数
	    private TextView snackbarText;
	    private TextView snackbarAction;
	
	    // 状态
	    public boolean actionClicked = false;
	    public boolean mIsShowing = false;
	    public boolean mIsShowByReplace = false;
	    public boolean mIsDismissing = false;
	    public boolean mIsDismissByReplace = false;
	
	    // 回调
	    private Runnable mDismissRunnable = new Runnable() {
	        @Override
	        public void run() {
	            dismiss();
	        }
	    };
	
	    public Snackbar(Context context) {
	        super(context);
	    }
	
	    // 新建一个Snackbar
	    public static Snackbar with(Context context) {
	        return new Snackbar(context);
	    }
	    // 设置Snackbar显示文本
	    public Snackbar text(CharSequence text) {
	        mText = text;
	        if (snackbarText != null) {
	            snackbarText.setText(text);
	        }
	        return this;
	    }
	    // 设置Action按钮显示文本
	    public Snackbar actionLabel(CharSequence actionLabel) {
	        mActionLabel = actionLabel;
	        return this;
	    }
	    // 设置Action按钮文本颜色
	    public Snackbar actionColor(int color) {
	        mActionColor = color;
	        return this;
	    }
	    public Snackbar actionColorResource(int rID) {
	        return actionColor(getResources().getColor(rID));
	    }
	    // 设置Action按钮按下事件监听器
	    public Snackbar actionListener(ActionClickListener listener) {
	        mActionClickListener = listener;
	        return this;
	    }
	    // 设置Snackbar不同阶段事件回调
	    public Snackbar eventListener(EventListener listener) {
	        mEventListener = listener;
	        return this;
	    }
	
	    private void init(Context context) {
	        SnackbarLayout layout = (SnackbarLayout) LayoutInflater.from(context)
	                .inflate(R.layout.sb_template, this);
	
	        Resources res = getResources();
	        // 将高度值转化为dp单位
	        float scale = res.getDisplayMetrics().density;
	        layout.setMaxHeight((int) (lineHeight * scale + 0.5f));
	
	        snackbarText = (TextView) layout.findViewById(R.id.sb_text);
	        if (mText != null) {
	            snackbarText.setText(mText);
	        }
	
	        snackbarAction = (TextView) layout.findViewById(R.id.sb_action);
	        if (mActionLabel != null) {
	            snackbarAction.setText(mActionLabel);
	        }
	        if (mActionColor != mUndefineColor) {
	            snackbarAction.setTextColor(mActionColor);
	        }
	        snackbarAction.setOnClickListener(new OnClickListener() {
	            @Override
	            public void onClick(View v) {
	                if (mActionClickListener != null) {
	                    if (!actionClicked) {
	                        mActionClickListener.onActionClicked(Snackbar.this);
	                        actionClicked = true;
	                        dismiss();
	                    }
	                }
	            }
	        });
	        // TextView默认不可点击，为使states-list有效，设置为Clickable
	        setClickable(true);
	    }
	
	    public void showByReplace(Activity targerActivity) {
	        mIsShowByReplace = true;
	        show(targerActivity);
	    }
	
	    public void show(Activity targerActivity) {
	        // 得到根View
	        ViewGroup root = (ViewGroup) targerActivity.findViewById(android.R.id.content);
	        init(targerActivity);
	        showInternal(targerActivity, root);
	    }
	
	    private void showInternal(Activity targetActivity, ViewGroup parent) {
	        parent.removeView(this);
	
	        // 当安卓版本大于等于LOLLIPOP时，需要设置Snackbar的海拔
	        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
	            for (int i = 0; i < parent.getChildCount(); i++) {
	                View otherChild = parent.getChildAt(i);
	                float elvation = otherChild.getElevation();
	                if (elvation > getElevation()) {
	                    setElevation(elvation);
	                }
	            }
	        }
	        // 设置Snackbar显示位置
	        FrameLayout.LayoutParams params = new FrameLayout.LayoutParams(FrameLayout.LayoutParams.MATCH_PARENT,
	                FrameLayout.LayoutParams.WRAP_CONTENT, Gravity.BOTTOM);
	        parent.addView(this, params);
	
	        // 动画
	        Animation slideIn = AnimationUtils.loadAnimation(getContext(), R.anim.slide_in_bottom);
	        slideIn.setAnimationListener(new Animation.AnimationListener() {
	            @Override
	            public void onAnimationStart(Animation animation) {
	                mIsShowing = true;
	                if (mEventListener != null) {
	                    if (mIsShowByReplace) {
	                        mEventListener.onShowByReplace(Snackbar.this);
	                    } else {
	                        mEventListener.onShow(Snackbar.this);
	                    }
	                }
	            }
	
	            @Override
	            public void onAnimationEnd(Animation animation) {
	                if (mEventListener != null) {
	                    mEventListener.onShown(Snackbar.this);
	                }
	                post(new Runnable() {
	                    @Override
	                    public void run() {
	                        postDelayed(mDismissRunnable, mDuration);
	                    }
	                });
	            }
	
	            @Override
	            public void onAnimationRepeat(Animation animation) {
	
	            }
	        });
	        startAnimation(slideIn);
	    }
	
	    public void dismissByReplace() {
	        mIsDismissByReplace = true;
	        dismiss();
	    }
	
	    public void dismiss() {
	        mIsShowing = false;
	        mIsDismissing = true;
	        if (mEventListener != null) {
	            mEventListener.onDismiss(this);
	        }
	        // 动画
	        Animation slideOut = AnimationUtils.loadAnimation(getContext(), R.anim.slide_out_bottom);
	        slideOut.setAnimationListener(new Animation.AnimationListener() {
	            @Override
	            public void onAnimationStart(Animation animation) {
	            }
	
	            @Override
	            public void onAnimationEnd(Animation animation) {
	                post(new Runnable() {
	                    @Override
	                    public void run() {
	                        finish();
	                    }
	                });
	            }
	
	            @Override
	            public void onAnimationRepeat(Animation animation) {
	            }
	        });
	        startAnimation(slideOut);
	    }
	
	    private void finish() {
	        clearAnimation();
	        ViewGroup parent = (ViewGroup) getParent();
	        if (parent != null) {
	            parent.removeView(this);
	        }
	        if (mEventListener != null) {
	            if (mIsDismissByReplace) {
	                mEventListener.onDismissByReplace(this);
	            }
	            else {
	                mEventListener.onDismissed(this);
	            }
	        }
	        mIsShowing = false;
	    }
	
	    @Override
	    protected void onDetachedFromWindow() {
	        super.onDetachedFromWindow();
	        if (mDismissRunnable != null) {
	            removeCallbacks(mDismissRunnable);
	        }
	    }
	}
{% endhighlight %}   
给出Snackbar的layout，很简单：  
{% highlight xml %}  
	<!-- sb_template.xml -->
	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="wrap_content">
	    <TextView
	        android:id="@+id/sb_text"
	        style="@style/Snackbar.Text"
	        android:layout_width="0dp"
	        android:layout_height="wrap_content"
	        android:layout_weight="1"
	        android:singleLine="true"
	        android:ellipsize="end"/>
	    <TextView
	        android:id="@+id/sb_action"
	        style="@style/Snackbar.Text.Action"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:layout_gravity="end"
	        android:gravity="center_vertical"/>
	</LinearLayout>
{% endhighlight %}   
可以看到，到此能够创建Snackbar了，但新的Snackbar替换旧的还没有实现，想想需求，只能存在一个Snackbar，我们需要写一个SnackbarManager，通过SnackbarManager来创建Snackbar，并控制只存在一个Snackbar。  
{% highlight java linenos %}  
	// SnackbarManager.java
	public class SnackbarManager {
	    private static final Handler MAIN_THREAD = new Handler(Looper.getMainLooper());
	
	    // 只出现一个Snackbar
	    private static WeakReference<Snackbar> snackbarReference;
	
	    private SnackbarManager() {
	    }
	
	    public static void show(final Snackbar snackbar, final Activity activity) {
	        MAIN_THREAD.post(new Runnable() {
	            @Override
	            public void run() {
	                Snackbar currentSnackbar = getCurrentSnackbar();
	                if (currentSnackbar != null) {
	                    if (currentSnackbar.mIsShowing && !currentSnackbar.mIsDismissing) {
	                        currentSnackbar.dismissByReplace();
	                        snackbarReference = new WeakReference<>(snackbar);
	                        snackbar.showByReplace(activity);
	                        return;
	                    }
	                }
	                snackbarReference = new WeakReference<>(snackbar);
	                snackbar.show(activity);
	            }
	        });
	    }
	
	    // 若存在Snackbar，使其消失
	    public static void dismiss() {
	        final Snackbar currentSnackbar = getCurrentSnackbar();
	        if (currentSnackbar != null) {
	            MAIN_THREAD.post(new Runnable() {
	                @Override
	                public void run() {
	                    if (currentSnackbar.mIsShowing && !currentSnackbar.mIsDismissing) {
	                        currentSnackbar.dismiss();
	                    }
	                }
	            });
	        }
	    }
	
	    public static Snackbar getCurrentSnackbar() {
	        if (snackbarReference != null) {
	            return snackbarReference.get();
	        }
	        return null;
	    }
	}
{% endhighlight %}   
到此，Snackbar组件的开发基本上完成了。下面给一个使用的例子： 　
{% highlight java linenos %}  
	SnackbarManager.show(
    	Snackbar.with(mCtx)
                .text("Delete")
                .actionLabel("UNDO")
                .actionColorResource(R.color.primary)
                .actionListener(new ActionClickListener() {
                	@Override
                	public void onActionClicked(Snackbar snackbar) {
                    }
                })
                .eventListener(new EventListener() {
                 	@Override
                	public void onShow(Snackbar snackbar) {
                    }

                    @Override
                    public void onShowByReplace(Snackbar snackbar) {
                    }
					@Override
                    public void onShown(Snackbar snackbar) {
                    }
					@Override
                    public void onDismiss(Snackbar snackbar) {
                    }
					@Override
                    public void onDismissByReplace(Snackbar snackbar) {
                    }
					@Override
                    public void onDismissed(Snackbar snackbar) {
                    }
              	}), (Activity) mCtx);
{% endhighlight %}   