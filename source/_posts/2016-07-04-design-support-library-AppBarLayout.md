---
title: CoordinatorLayout, AppBarLayout, CollapsingToolbarLayout使用
date: 2016-07-04 16:41:11
tags: [Android, Design Support Library, AppBarLayout, Toolbar]
categories: [Android, Design Support Library]
---

本文介绍Design Support Library中CoordinatorLayout, AppBarLayout, CollapsingToolbarLayout的使用.
先列出了Design Support Library中的Features, 然后如何set up, 最后附有Demo程序, 介绍CoordinatorLayout, AppBarLayout, CollapsingToolbarLayout的使用.

<!-- more -->

## Design Support Library Features
Design Support Library中有
- [FloatingActionButton](https://guides.codepath.com/android/Floating-Action-Buttons)
- [TabLayout](https://guides.codepath.com/android/Google-Play-Style-Tabs-using-TabLayout)
- [NavigationView](https://guides.codepath.com/android/Fragment-Navigation-Drawer)
- [SnackBar](https://guides.codepath.com/android/Displaying-the-Snackbar)
- [TextInputLayout](https://guides.codepath.com/android/Working-with-the-EditText#displaying-floating-label-feedback)
- [CoordinatorLayout](https://guides.codepath.com/android/Handling-Scrolls-with-CoordinatorLayout)
- [AppBarLayout](https://developer.android.com/reference/android/support/design/widget/AppBarLayout.html)
- [CollapsingToolbarLayout](https://developer.android.com/reference/android/support/design/widget/CollapsingToolbarLayout.html)
- [Bottom Sheets](https://guides.codepath.com/android/Handling-Scrolls-with-CoordinatorLayout#bottom-sheets)
- [PercentRelativeLayout](https://guides.codepath.com/android/Constructing-View-Layouts#percentrelativelayout) and [PercentFrameLayout](https://developer.android.com/reference/android/support/percent/PercentFrameLayout.html)
- [Vector Drawables](https://guides.codepath.com/android/Drawables#vector-drawables)

## Design Support Library Setup

```
android {
   compileSdkVersion 23  // needs to be consistent with major support libs used
}

ext {
  supportLibVersion = '23.4.0'  // variable that can be referenced to keep support libs consistent
}

dependencies {
    compile "com.android.support:appcompat-v7:${supportLibVersion}"
    compile "com.android.support:design:${supportLibVersion}"
}
```

## CoordinatorLayout, AppBarLayout使用

[CoordinatorLayout](https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.html)
实际上是一个更强大的FrameLayout, 可以通过[Behavior](https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.Behavior.html) 来控制其中各个child view的交互行为. 也可以指定anchor来指定floating view相对于其他某个View的位置. 比如Floating Action Button在显示Snackbar的时候自动向上移动.

为了使Toolbar响应滚动事件, 我们需要给它外边包一个[AppBarLayout](https://developer.android.com/reference/android/support/design/widget/AppBarLayout.html).
它是一个纵向的LinearLayout, 必须要作为CoordinateLayout的直接child使用.
然后, 我们需要定义AppBarLayout和我们scroll的内容View的关系.
这里可以是一个RecyclerView, 或者其他支持嵌套scrolling的view, 比如[NestedScrollView](https://developer.android.com/reference/android/support/v4/widget/NestedScrollView.html)

(实际上View类就有方法setNestedScrollingEnabled(), 但是还是需要View自己实现nested scrolling的功能, 否则这个开关也没有效果.)

support library提供了`@string/appbar_scrolling_view_behavior`, 它映射到`AppBarLayout.ScrollingViewBehavior`.
它是用来告诉AppBarLayout下面那个scroll view上的scroll事件什么时候发生.
所以这个属性必须在触发事件的view上指定, 比如:

```xml
<android.support.v7.widget.RecyclerView
        android:id="@+id/rvToDoList"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
```
当CoordinatorLayout看到自己的child(比如RecyclerView)声明了这个属性, 就会在自己的其他child中寻找相关的view(AppBarLayout).
这样, 当RecyclerView发生scroll事件的时候, AppBarLayout和其中的views都会被通知到.

滚动事件怎么通知到AppBarLayout的呢? 还需要一个属性: `app:layout_scrollFlags`.

```xml
<android.support.design.widget.AppBarLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:fitsSystemWindows="true"
    android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:layout_scrollFlags="scroll|enterAlways"/>

</android.support.design.widget.AppBarLayout>
```

`app:layout_scrollFlags`属性中:
- `scroll`必须有, 这样scroll的任何效果才能生效.
- `enterAlways`: 表示只要列表上方内容滚动出现, View就应该出现. 适用的情形: 当把列表滚到底部时, Toolbar被隐藏了, 一旦回滚一点儿, Toolbar就应该立即出现. 如果不设置这个flag, 默认的行为是一直要把列表滚到顶部, Toolbar才会出现.
- `enterAlwaysCollapsed`: 正常情况下, 如果只有`enterAlways`被指定, 在列表向下滚动的过程中Toolbar将会一直展开.
如果同时指定了`enterAlwaysCollapsed`和`minHeight`, 那么开始滚动以后, 只滚动到minHeight为止, 直到滚动到达列表顶部的时候, view才会展开到全部高度.
- `exitUntilCollapsed`: 正常只指定scroll的情况下, scrolling down(即显示列表底部)将会使得整个Toolbar移动到不见.
如果同时指定了`exitUntilCollapsed`和`minHeight`, 那么将会收缩到minHeight为止, Toolbar不会一直滚动和退出屏幕.
- `snap`: 使用了这个属性, 等scroll事件结束的时候, View可见的尺寸小于它的50%, 则它会直接消失, 如果大于50%, 则它会完整地出现.

还可以用`app:layout_scrollInterpolator`属性指定滚动动画效果的插值器.

### 折叠效果
如果想要折叠Toolbar的效果, 可以在Toolbar外面包一层[CollapsingToolbarLayout](https://developer.android.com/reference/android/support/design/widget/CollapsingToolbarLayout.html)
这个类必须作为AppBarLayout的直接child使用.

```xml
<android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:expandedTitleMarginEnd="64dp"
            app:expandedTitleMarginStart="48dp"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_scrollFlags="scroll|enterAlways"></android.support.v7.widget.Toolbar>

</android.support.design.widget.CollapsingToolbarLayout>
```

包了这个类之后, setTitle要调用这个类的方法:
```java
CollapsingToolbarLayout collapsingToolbar = (CollapsingToolbarLayout) findViewById(R.id.collapsing_toolbar);
collapsingToolbar.setTitle("Title");
```

### 背景图平行淡出
这个类使得我们可以做更高级的动画效果, 比如放一个ImageView, 它在折叠的时候淡出.
这时候需要把ImageView的`app:layout_collapseMode`属性置为`parallax`.

```xml
    <android.support.design.widget.AppBarLayout
        android:id="@+id/appbar"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:fitsSystemWindows="true"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:expandedTitleMarginEnd="64dp"
            app:expandedTitleMarginStart="48dp"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            app:layout_scrollInterpolator="@anim/hero_image_interpolator">

            <ImageView
                android:id="@+id/logo"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:contentDescription="@null"
                android:fitsSystemWindows="true"
                android:scaleType="fitCenter"
                android:src="@drawable/android_logo"
                app:layout_collapseMode="parallax"
                app:layout_collapseParallaxMultiplier="0.1" />

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

        </android.support.design.widget.CollapsingToolbarLayout>

    </android.support.design.widget.AppBarLayout>
```

## 项目
这里推荐Demo: [AndroidDesignWidgetsSample](https://github.com/mengdd/AndroidDesignWidgetsSample)
![Android Design Widgets Sample screen video](/images/AndroidDesignWidgetsSample-screen-video.gif)

## Reference
[Handling Scrolls with CoordinatorLayout](https://guides.codepath.com/android/Handling-Scrolls-with-CoordinatorLayout)
[Material Design: scrolling techniques](https://material.google.com/patterns/scrolling-techniques.html#scrolling-techniques-behavior)
[Design Support Library](https://guides.codepath.com/android/Design-Support-Library#official-source-code)
[Mastering the Coordinator Layout](http://saulmm.github.io/mastering-coordinator)