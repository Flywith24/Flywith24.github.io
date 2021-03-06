---
title: 使用 ProcessLifecycle 优雅地监听应用前后台切换
date: 2020-07-01 14:11:46
categories:   
- 奇技淫巧
tags: 
- Tips
- 奇技淫巧
image: https://gitee.com/flywith24/Album/raw/master/img/20201015104945.png
---

使用 ProcessLifecycle 优雅地监听应用前后台切换

<!-- more-->

## 前言

很高兴见到你，又来到了「奇技淫巧」系列，本系列介绍一些「骚操作」，可能不适合用于生产，但可以开拓思路

前些天在群里看到有人讨论通过维护 activity 栈来监听程序前后台切换的问题。其实单纯监听程序的前后台切换完全不需要维护 activity 栈，而现在比较主流的做法是使用 `registerActivityLifecycleCallbacks`。而今天我来介绍一下使用 ProcessLifecycleOwner 来实现这一功能



## lifecycle-process 库



Android Jetpack Lifecycle 组件有一个可选库：lifecycle-process，它可以为整个 app 进程提供一个 ProcessLifecycleOwner

![lifecycle-process 引入](https://gitee.com/flywith24/Album/raw/master/img/20200701110431.png)

该库十分简单，只有四个文件

![lifecycle-process](https://gitee.com/flywith24/Album/raw/master/img/20200701110904.png)



`ProcessLifecycleOwnerInitializer` 借助 ContentProvider 拿到 Context，用于初始化操作

![init](https://gitee.com/flywith24/Album/raw/master/img/20200701112017.png)



`EmptyActivityLifecycleCallbacks` 为 `Application.ActivityLifecycleCallbacks` 的实现类，内部为空实现



![EmptyActivityLifecycleCallbacks](https://gitee.com/flywith24/Album/raw/master/img/20200701112219.png)



`LifecycleDispatcher` 通过 ReportFragment 来 hook 宿主的生命周期事件

![](https://gitee.com/flywith24/Album/raw/master/img/20200701113143.png)



核心逻辑都在 ProcessLifecycleOwner 中

![ProcessLifecycleOwner ](https://gitee.com/flywith24/Album/raw/master/img/20200701113324.png)

该类提供了整个 app 进程的 lifecycle

可以将其视为所有 activity 的 LifecycleOwner ，其中 [Lifecycle.Event.ON_CREATE](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.Event#ON_CREATE) 只会分发一次，而 [Lifecycle.Event.ON_DESTROY](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.Event#ON_DESTROY) 则永远不会分发

其它的生命周期事件将按以下规则分发：

`ProcessLifecycleOwner` 会分发 [Lifecycle.Event.ON_START](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.Event#ON_START) 和 [Lifecycle.Event.ON_RESUME](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.Event#ON_RESUME) 事件（在第一个 activity 移动到这些事件时）

[Lifecycle.Event.ON_PAUSE](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.Event#ON_PAUSE) 与 [Lifecycle.Event.ON_STOP](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.Event#ON_STOP) 会在最后一个 activity 移动到这些状态后 **延迟** 分发，该延迟足够长，以确保由于配置更改等操作重建 activity 后不会分发任何事件

对于监听应用在前后台切换且不需要毫秒级的精度的场景，这十分有用



## ProcessLifecycleOwner  源码解析

根据上图我们得知 `ProcessLifecycleOwner`  实现了 LifecycleOwner 接口

由于在 `ProcessLifecycleOwnerInitializer` 中初始化时传入了 Context，因此 `ProcessLifecycleOwner`  在 attach 方法中借助 Context 拿到了 Application 实例，并调用了 `registerActivityLifecycleCallbacks`

``` java
void attach(Context context) {
    mHandler = new Handler();
    mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE);
    Application app = (Application) context.getApplicationContext();
    app.registerActivityLifecycleCallbacks(new EmptyActivityLifecycleCallbacks() 
        @RequiresApi(29)
        @Override
        public void onActivityPreCreated(@NonNull Activity activity,
                @Nullable Bundle savedInstanceState) {
            //我们需要 ProcessLifecycleOwner 刚好在第一个 activity 的 LifecycleOwner started/resumed 之前获取 ON_START 和 ON_RESUME。
            //activity 的 LifecycleOwner 通过在 onCreate() 中添加 activity 注册的 callback 来获取 started/resumed 状态。
            //通过在 onActivityPreCreated() 中添加我们自己的 activity 注册的 callback，我们首先获得了回调，同时与 Activity 的 onStart()/ onResume()回调相比仍具有正确的相对顺序
  		   
            activity.registerActivityLifecycleCallbacks(new EmptyActivityLifecycl
                @Override
                public void onActivityPostStarted(@NonNull Activity activity) {
                    activityStarted();
                }
                @Override
                public void onActivityPostResumed(@NonNull Activity activity) {
                    activityResumed();
                }
            });
        }
        @Override
        public void onActivityCreated(Activity activity, Bundle savedInstanceStat
            //仅在API 29 之前使用 ReportFragment，在此之后，我们可以使用在 onActivityPreCreated() 中注册的 onActivityPostStarted 和 onActivityPostResumed 回调
            if (Build.VERSION.SDK_INT < 29) {
                ReportFragment.get(activity).setProcessListener(mInitializationLi
            }
        }
        @Override
        public void onActivityPaused(Activity activity) {
            activityPaused();
        }
        @Override
        public void onActivityStopped(Activity activity) {
            activityStopped();
        }
    });
}
```

内部维护了 Started 和 Resumed 的数量

```java
private int mStartedCounter = 0;
private int mResumedCounter = 0;
private boolean mPauseSent = true;
private boolean mStopSent = true;
```

并在 activityStarted 和 activityResumed 方法中对 这两个数值进行 ++，并更改 lifecycle 状态

```java
void activityStarted() {
    mStartedCounter++;
    if (mStartedCounter == 1 && mStopSent) {
        mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_START);
        mStopSent = false;
    }
}
void activityResumed() {
    mResumedCounter++;
    if (mResumedCounter == 1) {
        if (mPauseSent) {
            mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_RESUME);
            mPauseSent = false;
        } else {
            mHandler.removeCallbacks(mDelayedPauseRunnable);
        }
    }
}
```

在 activityPaused 和 activityStopped 方法对这两个数值进行 --

```java
void activityPaused() {
    mResumedCounter--;
    if (mResumedCounter == 0) {
        mHandler.postDelayed(mDelayedPauseRunnable, TIMEOUT_MS);
    }
}
void activityStopped() {
    mStartedCounter--;
    dispatchStopIfNeeded();
}
```

而在这里我们看到了上文提到的延迟操作

```java
// 使用 handler 进行延迟操作
mHandler.postDelayed(mDelayedPauseRunnable, TIMEOUT_MS);

// 延迟 700 ms
static final long TIMEOUT_MS = 700; //mls

private Runnable mDelayedPauseRunnable = new Runnable() {
    @Override
    public void run() {
        // 根据需要分发事件
        dispatchPauseIfNeeded();
        dispatchStopIfNeeded();
    }
};

void dispatchPauseIfNeeded() {
    if (mResumedCounter == 0) {
        mPauseSent = true;
        mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_PAUSE);
    }
}
void dispatchStopIfNeeded() {
    if (mStartedCounter == 0 && mPauseSent) {
        mRegistry.handleLifecycleEvent(Lifecycle.Event.ON_STOP);
        mStopSent = true;
    }
}
```



源码就解析到这里，接下来我们看看如何使用吧

## 使用

首先引入该库

```groovy
implementation "androidx.lifecycle:lifecycle-process:2.3.0-alpha05"
```

由于我们要自定义 lifecycleObserver，因此还需引入

```groovy
implementation "androidx.lifecycle:lifecycle-common-java8:2.3.0-alpha05"
```



首先创建 `ProcessLifecycleObserver` 类，实现 `DefaultLifecycleObserver` 接口，在相应的生命周期中打印 log

接着在自定义 Application 中加入

![](https://gitee.com/flywith24/Album/raw/master/img/20200701121028.png)



这样便完成了！



![演示](https://user-gold-cdn.xitu.io/2020/7/1/173089686a09e8f3?w=1854&h=999&f=gif&s=2823298)



[Demo 在这里](https://github.com/Flywith24/ProcessLifecycle-Demo)




## 系列文章

- [【奇技淫巧】AndroidStudio Nexus3.x搭建Maven私服遇到问题及解决方案](https://juejin.im/post/5e481a28f265da570b3f235c)


- [【奇技淫巧】什么？项目里gradle代码超过200行了！你可能需要 Kotlin+buildSrc Plugin](https://juejin.im/post/5e22c2ce6fb9a02ff67d41c3)


- [【奇技淫巧】gradle依赖查找太麻烦？这个插件可能帮到你](https://juejin.im/post/5e481a28f265da570b3f235c)


- [【奇技淫巧】Android组件化不使用 Router 如何实现组件间 activity 跳转](https://juejin.im/post/5e967f35f265da47d77cd4c3)


- [【奇技淫巧】新的图片加载库？基于Kotlin协程的图片加载库——Coil](https://juejin.im/post/5ebdfb0b6fb9a0436153db22)


- [【奇技淫巧】使用 Navigation + Dynamic Feature Module 实现模块化](https://juejin.im/post/5ec50ae46fb9a047a862124f)


- [【奇技淫巧】除了 buildSrc 还能这样统一配置依赖版本？巧用 includeBuild](https://juejin.im/post/5ecde219e51d457841190d08)


- [【奇技淫巧】巧用 kotlin 扩展函数和 typealias 封装 带网络状态和解决「粘性」事件的 LiveData](https://juejin.im/post/5ed9c92ce51d45789b35afa9)



## 关于我

我是 [Flywith24](https://flywith24.gitee.io/)，我的博客内容已经分类整理 [在这里](https://github.com/Flywith24/BlogList)，点击右上角的 Watch 可以及时获取我的文章更新哦 😉



- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)

- [小专栏](https://xiaozhuanlan.com/detail)

- [Github](https://github.com/Flywith24)

  

![](https://user-gold-cdn.xitu.io/2020/6/26/172ee567fb4fbf7e?w=1954&h=624&f=jpeg&s=115362)