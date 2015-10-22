---
title: Android事件分发机制总结  
date: 2015-10-22 21:00
layout: post
category: technology
---  
##基础：   
`MotionEvent`:继承自`InputEvent`，有一些静态成员变量来判断触摸的类型，常用的有：`ACTION_DOWN`,`ACTION_UP`,`ACTION_MOVE`,`ACTION_CANCEL`，前几个都很好理解，这里说明一下`ACTION_CANCEL`的触发条件，通常情况下当parent抢占了motion的时候会触发`ACTION_CANCEL`，比如按住ListView中的Button然后手指滑动，当滑动到一定距离后ListView会滚动，而不是触发Button的click事件了。  
当需要判断`MotionEvent`时会调用`getAction()`来获取`MotionEvent`的类型，这里需要说明一下的是还有一个方法叫`getActionMasked()`，这两个方法有什么区别呢？其实就在于`getActionMasked()`去掉了一些额外的信息（触发Touch事件的手指索引），返回简单的信息。实现的原理是将`getAction()`中即将返回的信息与静态常量`ACTION_MASK`做按位与，`ACTION_MASK             = 0xff`，这样就只有原信息的低8位信息保留了下来。  

##几个函数：  

    View.java  
    public boolean dispatchTouchEvent(MotionEvent event);  
    public boolean onTouchEvent(MotionEvent event);  
    
    ViewGroup.java  
    public boolean dispatchTouchEvent(MotionEvent event);  
    public boolean onInterceptTouchEvent(MotionEvent ev);  
    public boolean onTouchEvent(MotionEvent event);  
首先，自定义一个LinearLayout，Button，分别继承相应的类，重写上面5个函数，输出提示信息，代码如下：   
{% highlight java linenos %}  
public class MyButton extends Button {
    public MyButton(Context context) {
        super(context);
    }

    public MyButton(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public MyButton(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                System.out.println("Button---onTouchEvent---DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                System.out.println("Button---onTouchEvent---MOVE");
                break;
            case MotionEvent.ACTION_UP:
                System.out.println("Button---onTouchEvent---UP");
                break;
            default:
                break;
        }
        boolean result = super.onTouchEvent(event);
//        boolean result = false;
        System.out.println("Button---onTouchEvent---" + result);
        return result;
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                System.out.println("Button---dispatchTouchEvent---DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                System.out.println("Button---dispatchTouchEvent---MOVE");
                break;
            case MotionEvent.ACTION_UP:
                System.out.println("Button---dispatchTouchEvent---UP");
                break;
            default:
                break;
        }
        boolean result = super.dispatchTouchEvent(event);
        System.out.println("Button---dispatchTouchEvent---" + result);
        return result;
    }
}
{% endhighlight %}   

{% highlight java linenos %}  
public class MyLinearLayout extends LinearLayout {
    public MyLinearLayout(Context context) {
        super(context);
    }

    public MyLinearLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public MyLinearLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                System.out.println("LinearLayout---dispatchTouchEvent---DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                System.out.println("LinearLayout---dispatchTouchEvent---MOVE");
                break;
            case MotionEvent.ACTION_UP:
                System.out.println("LinearLayout---dispatchTouchEvent---UP");
                break;
            default:
                break;
        }
        boolean result = super.dispatchTouchEvent(ev);
        System.out.println("LinearLayout---dispatchTouchEvent---" + result);
        return result;
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                System.out.println("LinearLayout---onInterceptTouchEvent---DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                System.out.println("LinearLayout---onInterceptTouchEvent---MOVE");
                break;
            case MotionEvent.ACTION_UP:
                System.out.println("LinearLayout---onInterceptTouchEvent---UP");
                break;
            default:
                break;
        }
        boolean result = super.onInterceptTouchEvent(ev);
//        boolean result = true;
        System.out.println("LinearLayout---onInterceptTouchEvent---" + result);
        return result;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                System.out.println("LinearLayout---onTouchEvent---DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                System.out.println("LinearLayout---onTouchEvent---MOVE");
                break;
            case MotionEvent.ACTION_UP:
                System.out.println("LinearLayout---onTouchEvent---UP");
                break;
            default:
                break;
        }
        boolean result = super.onTouchEvent(event);
//        boolean result = true;
        System.out.println("LinearLayout---onTouchEvent---" + result);
        return result;
    }
}
{% endhighlight %}   
定义一个简单布局文件，将自定义的Button放在自定义的LinearLayout中：  
{% highlight xml linenos %}  
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent" android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingBottom="@dimen/activity_vertical_margin" tools:context=".MainActivity">

    <com.example.limeng.mytestapplication.MyLinearLayout
        android:id="@+id/linearLayout"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content">
        <com.example.limeng.mytestapplication.MyButton
            android:id="@+id/button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="click" />
    </com.example.limeng.mytestapplication.MyLinearLayout>

</RelativeLayout>
{% endhighlight %}   
MainActivity:  
{% highlight java linenos %}  
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        MyButton btn = (MyButton) findViewById(R.id.button);
        MyLinearLayout layout = (MyLinearLayout) findViewById(R.id.linearLayout);

        btn.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                System.out.println("Button---onTouch");
                return false;
            }
        });
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                System.out.println("Button---onClick");
            }
        });
    }
{% endhighlight %}   
调试。快速点击自定义的Button，得到输出如下：  
<img src="https://github.com/limmeng/limmeng.github.io/raw/master/images/posts/AndroidDispatch1.jpg" height="250"/>   
然后我们分析这个事件分发流程。  
>首先：  
>对View中的`dispatchTouchEvent`，返回`true`代表事件被该View处理了，否则返回`false`。  
>对ViewGroup中的`dispatchTouchEvent`返回`true`代表事件被该ViewGroup或者该ViewGroup中的某个子View处理了，否则返回`false`。  
>只有ViewGroup中才有`onInterceptTouchEvent`，返回`true`代表拦截当前事件，返回`false`继续往子View分发。  

当我们将LinearLayout中的`onInterceptTouchEvent`返回值设为`true`时，输出如下：  

<img src="https://github.com/limmeng/limmeng.github.io/raw/master/images/posts/AndroidDispatch2.png" height="150"/>   

当我们将Button中`onTouchEvent`返回值设为false时，输出如下：  

<img src="https://github.com/limmeng/limmeng.github.io/raw/master/images/posts/AndroidDispatch3.png" height="250"/>    

当我们将Button中`onTouchEvent`返回值设为false，并将LinearLayout中`onTouchEvent`返回值设为true时，输出如下：  
 
<img src="https://github.com/limmeng/limmeng.github.io/raw/master/images/posts/AndroidDispatch4.png" height="250"/>    

总结一下：touch事件的分发如下图（事件没被消耗的情况就没有标出了，所有onTouchEvent都返回false就未消耗）：  
<img src="https://github.com/limmeng/limmeng.github.io/raw/master/images/posts/AndroidEventDispatch.jpg" height="750"/>   

##分析：
下面从源码的角度分析一下具体的分发过程。  
首先是ViewGroup的dispatchTouchEvent函数：  
{% highlight java linenos %}  
@Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
    			......
    			// Check for interception.
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {
                // There are no touch targets and this action is not an initial down
                // so this view group continues to intercept touches.
                intercepted = true;
            }
				......
				// Dispatch to touch targets.
            if (mFirstTouchTarget == null) {
                // No touch targets so treat this as an ordinary view.
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } else {
                // Dispatch to touch targets, excluding the new touch target if we already
                // dispatched to it.  Cancel touch targets if necessary.
                TouchTarget predecessor = null;
                TouchTarget target = mFirstTouchTarget;
                while (target != null) {
                    final TouchTarget next = target.next;
                    if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                        handled = true;
                    } else {
                        final boolean cancelChild = resetCancelNextUpFlag(target.child)
                                || intercepted;
                        if (dispatchTransformedTouchEvent(ev, cancelChild,
                                target.child, target.pointerIdBits)) {
                            handled = true;
                        }
                        if (cancelChild) {
                            if (predecessor == null) {
                                mFirstTouchTarget = next;
                            } else {
                                predecessor.next = next;
                            }
                            target.recycle();
                            target = next;
                            continue;
                        }
                    }
                    predecessor = target;
                    target = next;
                }
            }
            ......
            if (!handled && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
        }
        return handled;
    }
            
{% endhighlight %}  
其中`intercepted`用来标记是否拦截事件。  
{% highlight java linenos %}  
public boolean onInterceptTouchEvent(MotionEvent ev) {
        return false;
    }
{% endhighlight %}  
如果不重写，`onInterceptTouchEvent `始终return false，保证事件能向下传递。  
其中`handled`用来标记事件是否消耗，代码很多，只需注意`dispatchTransformedTouchEvent`函数，其返回值赋给了handled，也就是这个函数将事件分发给子View看事件是否被消耗。  
{% highlight java linenos %}  
private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
            View child, int desiredPointerIdBits) {
        final boolean handled;

        // Canceling motions is a special case.  We don't need to perform any transformations
        // or filtering.  The important part is the action, not the contents.
        final int oldAction = event.getAction();
        if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
            event.setAction(MotionEvent.ACTION_CANCEL);
            if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                handled = child.dispatchTouchEvent(event);
            }
            event.setAction(oldAction);
            return handled;
        }

        // Calculate the number of pointers to deliver.
        final int oldPointerIdBits = event.getPointerIdBits();
        final int newPointerIdBits = oldPointerIdBits & desiredPointerIdBits;

        // If for some reason we ended up in an inconsistent state where it looks like we
        // might produce a motion event with no pointers in it, then drop the event.
        if (newPointerIdBits == 0) {
            return false;
        }

        // If the number of pointers is the same and we don't need to perform any fancy
        // irreversible transformations, then we can reuse the motion event for this
        // dispatch as long as we are careful to revert any changes we make.
        // Otherwise we need to make a copy.
        final MotionEvent transformedEvent;
        if (newPointerIdBits == oldPointerIdBits) {
            if (child == null || child.hasIdentityMatrix()) {
                if (child == null) {
                    handled = super.dispatchTouchEvent(event);
                } else {
                    final float offsetX = mScrollX - child.mLeft;
                    final float offsetY = mScrollY - child.mTop;
                    event.offsetLocation(offsetX, offsetY);

                    handled = child.dispatchTouchEvent(event);

                    event.offsetLocation(-offsetX, -offsetY);
                }
                return handled;
            }
            transformedEvent = MotionEvent.obtain(event);
        } else {
            transformedEvent = event.split(newPointerIdBits);
        }

        // Perform any necessary transformations and dispatch.
        if (child == null) {
            handled = super.dispatchTouchEvent(transformedEvent);
        } else {
            final float offsetX = mScrollX - child.mLeft;
            final float offsetY = mScrollY - child.mTop;
            transformedEvent.offsetLocation(offsetX, offsetY);
            if (! child.hasIdentityMatrix()) {
                transformedEvent.transform(child.getInverseMatrix());
            }

            handled = child.dispatchTouchEvent(transformedEvent);
        }

        // Done.
        transformedEvent.recycle();
        return handled;
    }
{% endhighlight %}   
接下来看View中的dispatchTouchEvent函数：  
{% highlight java linenos %}  
public boolean dispatchTouchEvent(MotionEvent event) {
        // If the event should be handled by accessibility focus first.
        if (event.isTargetAccessibilityFocus()) {
            // We don't have focus or no virtual descendant has it, do not handle the event.
            if (!isAccessibilityFocusedViewOrHost()) {
                return false;
            }
            // We have focus and got the event, then use normal event dispatch.
            event.setTargetAccessibilityFocus(false);
        }

        boolean result = false;

        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }

        final int actionMasked = event.getActionMasked();
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // Defensive cleanup for new gesture
            stopNestedScroll();
        }

        if (onFilterTouchEventForSecurity(event)) {
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }

            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }

        if (!result && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
        }

        // Clean up after nested scrolls if this is the end of a gesture;
        // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
        // of the gesture.
        if (actionMasked == MotionEvent.ACTION_UP ||
                actionMasked == MotionEvent.ACTION_CANCEL ||
                (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
            stopNestedScroll();
        }

        return result;
    }
{% endhighlight %}  
代码29行`li.mOnTouchListener.onTouch(this, event)`的mOnTouchListener就是我们用setOnTouchListener设置的listener。接下来33行才调用`onTouchEvent(event)`，这也解释了为什么onTouch比onTouchEvent先输出。  

最后一个问题：onClick在哪里调用的？  
先看View的onTouchEvent中一小段：  
{% highlight java linenos %}  
......
// Only perform take click actions if we were in the pressed state
if (!focusTaken) {
    // Use a Runnable and post this rather than calling
    // performClick directly. This lets other visual state
    // of the view update before click actions start.
    if (mPerformClick == null) {
        mPerformClick = new PerformClick();
    }
    if (!post(mPerformClick)) {
        performClick();
    }
}
......
{% endhighlight %}  
再看performClick():  
{% highlight java linenos %}  
 public boolean performClick() {
        final boolean result;
        final ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {
            playSoundEffect(SoundEffectConstants.CLICK);
            li.mOnClickListener.onClick(this);
            result = true;
        } else {
            result = false;
        }

        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
        return result;
    }
{% endhighlight %}  
代码第4行的`li.mOnClickListener`就是我们用setOnClickListener设置的listener。

**总结：一个事件通过dispatchTouchEvent不断向里层分发，然后从里层向外层依次调用onTouchEvent函数，如果返回true，则事件在返回处消耗掉。** 

{% highlight java linenos %}  
{% endhighlight %}  