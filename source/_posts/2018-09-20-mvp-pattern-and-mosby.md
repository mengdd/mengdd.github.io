---
title: MVP模式, 开源库mosby的使用及代码分析
date: 2018-09-20 16:33:08
tags: [Android, Architecture, MVP, mosby]
categories: [Android, Architecture]
---

Android中的构架模式一直是一个很hot的topic, 近年来Architecture components推出之后, MVVM异军突起, 风头正在逐渐盖过之前的MVP.
其实我觉得MVP还是有好处的, 比如灵活多变(其实只是我用起来更熟悉顺手一些吧).
个人是没有什么偏见的, 关于项目的构架, 只要找到适合的就行.
最近打算实际用一下mosby这个开源库, 帮助构建一下mvp模式, 本文是我的心路历程和代码心得记录.

<!-- more -->

## 关于MVP模式
前几年MVP模式的风很大, 之前工作的项目也用的MVP模式, 所以对这个模式在team有很多讨论. 

可以说一千个人眼中有一千种MVP吧, 比如Presenter之后的数据逻辑, 是用Interactor呢, 还是用Repository呢, 如果用了CursorLoader, 那么数据和View层直接耦合怎么办. 要不要给Presenter也定义接口呢, Presenter是注入呢还是在哪里(比如基类Fragment里)初始化呢, P和V的attach到底是在P里做呢还是在V里做呢.


### MVP的原则
尽管结合项目实际, 可能有很多变种, 但是不管怎么变, MVP有几个原则是要遵守的:
* Activity/Fragment实现View接口, View中的方法都只是和UI显示相关的. View要尽可能的dummy, 不涉及业务逻辑, presenter告诉它干什么它干什么就行了.
* Presenter中没有Android相关的类, 是一个纯Java的程序. 这样有利于解耦和测试. (所以一个检查方法是看你的presenter的import中有没有android的包名.)
* 注意生命周期的处理, 因为异步任务callback返回之后View的状态不一定还是活跃的, 所以要有一定的措施检查View是否还在以及处理注销等, 避免crash或内存泄露.

### MVP的官方例子
MVP模式Google有个官方例子: [android-architecture](https://github.com/googlesamples/android-architecture), 我之前写了一篇解读在这里[Google官方MVP Sample代码解读](https://www.cnblogs.com/mengdd/p/5988104.html). (我刚看了一下官方sample代码又更新了, 还得再看一下.)

官方的例子属于比较正统的, 比如每个界面会定义一个Contract, 里面分别定义View和Presenter的接口. 用Repository包装local和remote的数据, local和remote的数据源会和repository实现相同的data source接口, 我非常喜欢RxJava版本的三级缓存处理.

### 我的一些小Demo
之前自己写的一些比较完整的使用MVP的Demo:
- [TodoRealm](https://github.com/mengdd/TodoRealm): 一个Todo任务管理器, 只有本地数据.
- [ZhihuDaily](https://github.com/mengdd/ZhihuDaily): 知乎日报, 支持离线模式.


## MVVM
自从Google官方推出了[Android Architecture Components](https://developer.android.com/topic/libraries/architecture/)之后, 看起来MVVM也是一种不错的选择.

这是官方的例子: [android-architecture-components](https://github.com/googlesamples/android-architecture-components).

我还正在学习中, 关于这个话题可能以后会单独展开来讲一下, 我先沉淀一下.

目前的心得: 这一套东西也很强大, 就是用起来不太习惯. 要遵循的套路太多, 感觉没有使用MVP的时候那么自由. (可能还是不太熟的缘故吧, 我还是不多说了. ==!)

所以在学习这套模式的时候我突然又怀念起MVP模式, 准备把之前一个烂尾的个人项目重新拯救一把. 就是这个: [GithubClient](https://github.com/mengdd/GithubClient). 这一次准备用个mvp的库玩玩.

## Mosby库的使用和代码分析
Mosby是一个帮你实现MVP或MVI的库.
最近看介绍才发现它的名字是根据How I met your mother这个美剧的主角起的. (我最近才利用生病期间看完这个剧. 觉得真是巧合啊, 注定要用一用了.)

之前都是自己手动实现MVP的, 也没什么难的, 用这个库会帮你解决什么问题呢?
看看Mosby的介绍:
* [Mosby Github Repo](https://github.com/sockeqwe/mosby)
* [Mosby Getting Started](http://hannesdorfmann.com/mosby/getting-started/)
* [Mosby MVP](http://hannesdorfmann.com/mosby/mvp/)

### 使用Mosby的基本步骤:
* View接口继承`MvpView`.
* Presenter: 如果有规定Presenter接口, 接口继承`MvpPresenter<View>`, 其中View是对应的View接口, 实现类继承`MvpBasePresenter<View>`.
如果没有Presenter的接口而直接是实现类也可以, 同样也是实现类继承`MvpBasePresenter<View>`.
* Activity或Fragment实现View接口, 继承`MvpActivity`或`MvpFragment`, 泛型参数类型传入对应的View接口和Presenter类型即可.
* Activity或Fragment实现抽象的`createPresenter()`方法, 在其中创建Presenter的实例.

好了, 所有必须的工作就做完了, mosby的类会处理初始化和实例保存等.
Activity/Fragment中不需要保存presenter的字段, Presenter中也不需要保存View的字段. 这些都在基类中保存了.

### Mosby的实现
关于Mosby的实现可以查看它的类, 里面有详细的注释.
#### 生命周期
* `MvpActivity`中用了`ActivityMvpDelegateImpl`, 在Activity的每一个生命周期回调中做一些事情. 
在`onCreate()`中创建了Presenter, 把它赋值给字段, 并且attachView(); 在`onDestroy()`中detachView()和调用presenter的destroy()来做一些清理工作.
* `MvpFragment`中用了`FragmentMvpDelegateImpl`, 在Fragment的生命周期中做一些事情: 在`onCreate()`中创建Presenter, 赋值给字段; `onViewCreated()`中attachView(); `onDestroyView()`中detachView(); `onDestroy()`中调用presenter的destroy()来做一些清理工作.
所以presenter的初始化, 和view的attach/detach, 以及它们变量的保存都是mosby帮我们处理好了.
* mosby还支持ViewGroup作为View, 它提供了`MvpFrameLayout`, `MvpLinearLayout`和`MvpRelativeLayout`以供继承, Delegate的实现类是`ViewGroupMvpDelegateImpl`, 用到的生命周期主要是`onAttachedToWindow()`和`#onDetachedFromWindow()`.

### Presenter中调用View的方法
* `MvpBasePresenter`的实现没有什么特殊的, 主要是存了一个View的WeakReference. 新版中推荐使用`ifViewAttached(ViewAction<V>)`方法来把判断和执行一次性做了. 原来的`isViewAttached()`和`getView()`已经标记为deprecated了.
关于这样做的原因, 在这里有讨论: https://github.com/sockeqwe/mosby/issues/233.

### 屏幕旋转时的状态保存
mosby是处理了屏幕旋转时的状态保存的, 可以看到初始化`ActivityMvpDelegateImpl`时默认第三个参数是true, 即屏幕旋转时保存状态.
具体做法是通过`PresenterManager`把presenter保存起来.
保存的时候传了activity和一个生成的viewId:
```java
  private P createViewIdAndCreatePresenter() {

    P presenter = delegateCallback.createPresenter();
    if (presenter == null) {
      throw new NullPointerException(
          "Presenter returned from createPresenter() is null. Activity is " + activity);
    }
    if (keepPresenterInstance) {
      mosbyViewId = UUID.randomUUID().toString();
      PresenterManager.putPresenter(activity, mosbyViewId, presenter);
    }
    return presenter;
  }

```
恢复状态的时候需要把之前存的Presenter拿出来还是用activity的实例和viewId:
```java
  @Nullable public static <P> P getPresenter(@NonNull Activity activity, @NonNull String viewId) {
    if (activity == null) {
      throw new NullPointerException("Activity is null");
    }

    if (viewId == null) {
      throw new NullPointerException("View id is null");
    }

    ActivityScopedCache scopedCache = getActivityScope(activity);
    return scopedCache == null ? null : (P) scopedCache.getPresenter(viewId);
  }
```
其中viewId是通过bundle保存和恢复出来的:
```java
  @Override public void onSaveInstanceState(Bundle outState) {
    if (keepPresenterInstance && outState != null) {
      outState.putString(KEY_MOSBY_VIEW_ID, mosbyViewId);
      if (DEBUG) {
        Log.d(DEBUG_TAG,
            "Saving MosbyViewId into Bundle. ViewId: " + mosbyViewId + " for view " + getMvpView());
      }
    }
  }

```
那么问题来了:
* 1.既然我们已经有了一个viewId作为key, 为什么还需要activity来作为查询条件? 
* 2.如果真的需要这个条件, 那么屏幕旋转以后activity都重建了, 如何通过新的activity实例获得之前的Presenter呢?

首先我是在代码中找到了第二个问题的答案, 即两个不同的activity是如何关联起来的:
```java
  static final Application.ActivityLifecycleCallbacks activityLifecycleCallbacks =
      new Application.ActivityLifecycleCallbacks() {
        @Override public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
          if (savedInstanceState != null) {
            String activityId = savedInstanceState.getString(KEY_ACTIVITY_ID);
            if (activityId != null) {
              // After a screen orientation change we map the newly created Activity to the same
              // Activity ID as the previous activity has had (before screen orientation change)
              activityIdMap.put(activity, activityId);
            }
          }
        }

        @Override public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
          // Save the activityId into bundle so that the other
          String activityId = activityIdMap.get(activity);
          if (activityId != null) {
            outState.putString(KEY_ACTIVITY_ID, activityId);
          }
        }
...

        @Override public void onActivityDestroyed(Activity activity) {
          if (!activity.isChangingConfigurations()) {
            // Activity will be destroyed permanently, so reset the cache
            String activityId = activityIdMap.get(activity);
            if (activityId != null) {
              ActivityScopedCache scopedCache = activityScopedCacheMap.get(activityId);
              if (scopedCache != null) {
                scopedCache.clear();
                activityScopedCacheMap.remove(activityId);
              }

              // No Activity Scoped cache available, so unregister
              if (activityScopedCacheMap.isEmpty()) {
                // All Mosby related activities are destroyed, so we can remove the activity lifecylce listener
                activity.getApplication()
                    .unregisterActivityLifecycleCallbacks(activityLifecycleCallbacks);
                if (DEBUG) {
                  Log.d(DEBUG_TAG, "Unregistering ActivityLifecycleCallbacks");
                }
              }
            }
          }
          activityIdMap.remove(activity);
        }
      };

```
通过Bundle存取传递一个activityId, 新创建的activity实例和旧的activity实例就有相同的id. 这个关系存储在`Map<Activity, String> activityIdMap`里.
这样在新的activity中通过map查询到activityId之后, 在`Map<String, ActivityScopedCache> activityScopedCacheMap`中再通过activityId查到了ActivityScopedCache对象, 再用viewId作为key查询到presenter.

看了`onActivityDestroyed()`部分的代码之后也终于明白了第一个问题的答案, 即这样做的原因, 如果只用viewId, 我们是解决了存放和查询, 但是没有解决释放的问题.
因为我们的需求只是在屏幕旋转的情况下保存presenter的实例, 我们仍然需要在activity真的销毁的时候释放对presenter实例的保存.
这里用了`activity.isChangingConfigurations()`的条件来区分activity是真的要销毁, 还是为了屏幕旋转要销毁. 


## 其他
Mosby还支持LCE(Loading-Content-Error)和ViewState, 为开发者省去更多套路化的代码, 还有处理屏幕旋转之后的状态恢复.
有空的时候再写一篇扒一扒吧.


