---
title: 实现Android DrawerLayout侧边栏分类导航
date: 2015-5-22 21:45
layout: post
category: technology
---
先看一下希望实现的效果：
<img src="../drawer_navigation.png" height="400" />  
本来侧边栏的导航是用RecyclerView实现的，现在想分类不同的导航项，难道需要用很多个RecyclerView吗？显然这不是我们所期望的解决方法。  
下面介绍的解决方法是由阅读 [The Google I/O 2014 Android App](https://github.com/limmeng/iosched "The Google I/O 2014 Android App") 的源码得来，阅读优质开源项目的源码是极佳的学习方法。  
首先，构建基础NavigationDrawer的官方教程：[Creating a Navigation Drawer](https://developer.android.com/training/implementing-navigation/nav-drawer.html "Creating a Navigation Drawer")；  
建立基础的NavigationDrawer后，再来构造分类的items，整个构造过程就是把每一项（包括分割线）都当成一个Item，逐个添加到左边栏中去，上代码(基础的NavigationDrawer代码就不给出了，官方教程说的很详细)：  
先是资源的设置：  
{% highlight java %}  
    //所有可能的navDrawer items ID  
    protected static final int NAVDRAWER_ITEM_TODAY = 0;
    protected static final int NAVDRAWER_ITEM_GOAL = 1;
    protected static final int NAVDRAWER_ITEM_TASK = 2;
    protected static final int NAVDRAWER_ITEM_ALARM = 3;
    protected static final int NAVDRAWER_ITEM_STATISTICS = 4;
    protected static final int NAVDRAWER_ITEM_SETTING = 5;
    protected static final int NAVDRAWER_ITEM_SEPARATE = -1; //分割符的ID

    //navDrawer item 的名称  
    private static final int[] NAVDRAWER_TITLE_RES_ID = {
        "今天",
        "目标",
        "任务",
        "提醒",
		"统计",
		"设置"
    };

    // 实际加入navDrawer的item的ID，按序保存在这个list中
    private ArrayList<Integer> mNavDrawerItems = new ArrayList<Integer>();

    // 与navdrawer items相对应的 view，下标与mNavDrawerItems对应
    private View[] mNavDrawerItemViews = null;

    //navDrawer item drawable，icon资源
    private static final int[] NAVDRAWER_ICON_RES_ID = {
        R.drawable.ic_today_grey600_24dp,
        R.drawable.ic_assignment_grey600_24dp,
        R.drawable.ic_format_list_bulleted_grey600_24dp,
        R.drawable.ic_alarm_grey600_24dp,
        R.drawable.ic_trending_up_grey600_24dp,
        R.drawable.ic_settings_grey600_24dp
    };
{% endhighlight %}  
然后在Activity的`onCreate`中调用`populateNavDrawer()`设置NavigationDrawer的items；  
{% highlight java %}    
	private void populateNavDrawer() {
		// 按序设置所有Item的ID
        mNavDrawerItems.clear();
        mNavDrawerItems.add(NAVDRAWER_ITEM_TODAY);
        mNavDrawerItems.add(NAVDRAWER_ITEM_GOAL);
        mNavDrawerItems.add(NAVDRAWER_ITEM_TASK);

        mNavDrawerItems.add(NAVDRAWER_ITEM_SEPARATE);

        mNavDrawerItems.add(NAVDRAWER_ITEM_ALARM);
        mNavDrawerItems.add(NAVDRAWER_ITEM_STATISTICS);

        mNavDrawerItems.add(NAVDRAWER_ITEM_SEPARATE);

        mNavDrawerItems.add(NAVDRAWER_ITEM_SETTING);

		// 根据ID创建对应的View
        createNavDrawerItems();
    }

    private void createNavDrawerItems() {
		// navdrawer_items_list为Activity布局中左边栏部分
        mDrawerItemsListContainer = (ViewGroup) findViewById(R.id.navdrawer_items_list);
        if (mDrawerItemsListContainer == null) {
            return;
        }
		
		// 初始化Views
        mNavDrawerItemViews = new View[mNavDrawerItems.size()];
        mDrawerItemsListContainer.removeAllViews();
        int i = 0;
		// 按照ID创建每一个view然后添加到mDrawerItemsListContainer
        for (int itemId : mNavDrawerItems) {
            mNavDrawerItemViews[i] = makeNavDrawerItem(itemId, mDrawerItemsListContainer);
            mDrawerItemsListContainer.addView(mNavDrawerItemViews[i]);
            ++i;
        }
    }

    private View makeNavDrawerItem(final int itemID, ViewGroup container) {
        boolean isSeperator = false;
        int layoutToInflate = 0;
		// 判断是否为分割符
        if (itemID == NAVDRAWER_ITEM_SEPARATE) {
            layoutToInflate = R.layout.navdrawer_seperator;
            isSeperator = true;
        }
        else {
            layoutToInflate = R.layout.navdrawer_item;
        }
		// 获取对应item的layout
        View view = getLayoutInflater().inflate(layoutToInflate, container, false);
        if (isSeperator) {
            return view;
        }
		
		// 不是分隔符，则设置item中对应views
        ImageView icon = (ImageView) view.findViewById(R.id.item_image);
        TextView title = (TextView) view.findViewById(R.id.item_name);
        icon.setImageResource(NAVDRAWER_ICON_RES_ID[itemID]);
        title.setText(NAVDRAWER_TITLE_RES_ID[itemID]);

        view.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                selectItem(itemID); // 设置点击后的导航
            }
        });

        return view;
    }
{% endhighlight %}  
最后给出布局文件：  
{% highlight xml %}   
<!-- navdrawer_seperator.xml -->  
<?xml version="1.0" encoding="utf-8"?>
<View xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="1dp"
    android:background="@color/divider"
    android:layout_marginTop="7dp"
    android:layout_marginBottom="8dp"/>
{% endhighlight %}  
{% highlight xml %}   
<!-- navdrawer_item.xml -->  
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="?android:selectableItemBackground"
    android:paddingRight="@dimen/single_line_list_padding_left_and_right">
    <ImageView
        android:id="@+id/item_image"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true"
        android:paddingLeft="@dimen/single_line_list_padding_left_and_right"/>
    <TextView
        android:id="@+id/item_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="@dimen/single_line_list_font"
        android:textColor="@color/drawer_selector_color"
        android:paddingBottom="@dimen/single_line_list_text_padding_bottom"
        android:paddingTop="@dimen/single_line_list_text_padding_top"
        android:paddingLeft="@dimen/single_line_list_text_padding_left"/>
</RelativeLayout>
{% endhighlight %}  
