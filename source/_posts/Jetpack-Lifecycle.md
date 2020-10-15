---
title: 万物基于Lifecycle 默默无闻大作用
date: 2020-03-30 00:10:35
categories: 
- 背上 Jetpack
tags: 
- androidx
- Jetpack
- MVVM
image: https://gitee.com/flywith24/Album/raw/master/img/20201015101301.png
---
Jetpack Lifecycle 介绍。
<!-- more-->
>  Android 中有一个比较重要的概念：「生命周期」。刚毕业去面试，总会被问到「四大组件的生命周期」这类的问题。17年的 IO 大会上，Google 推出了 Lifecycle-Aware Components（生命周期感知组件），帮助开发者组织更好，更轻量，易于维护的代码


本文介绍 `Lifecycle` 的职责以及简单分析 lifecycle 如何感知 activity 和 fragment ，帮助您对 `Lifecycle` 有一个感性的认识



## 万物基于 **Lifecycle** 

### 手动管理生命周期的痛苦你不懂

![lifecycles](https://user-gold-cdn.xitu.io/2020/3/30/1712bbbbf69f3b1e?w=2407&h=5062&f=jpeg&s=2270014)



>  鲁迅曾说过：万物基于 Lifecycle

![](https://gitee.com/flywith24/Album/raw/master/img/20200331214314.jpeg)

哦不对

![](https://gitee.com/flywith24/Album/raw/master/img/20200331214054.jpeg)



Android 中的视图控制器就有这么多生命周期的情况，所以处理好生命周期十分重要，否则会导致内存泄漏甚至是程序崩溃。这里引用 [官方文档](https://developer.android.com/topic/libraries/architecture/lifecycle) 的例子



``` java
class MyLocationListener {
    public MyLocationListener(Context context, Callback callback) {
        // ...
    }

    void start() {
        // 连接系统的定位服务
    }

    void stop() {
        // 与系统的定位服务断开连接
    }
}

class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    @Override
    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, (location) -> {
            // 更新 UI
        });
    }

    @Override
    public void onStart() {
        super.onStart();
        myLocationListener.start();
        //管理其他需要响应 activity 生命周期的组件
    }

    @Override
    public void onStop() {
        super.onStop();
        myLocationListener.stop();
        //管理其他需要响应 activity 生命周期的组件
    }
}
```



此示例看起来不错，在实际的应用程序中，您仍然会响应生命周期的当前状态而进行过多的调用来管理 UI 和其他组件。 管理多个组件会在生命周期方法中放置大量代码，例如 onStart() 和 onStop()，这使它们难以维护



而且，不能保证组件在 activity 或 fragment 停止之前就已启动。 如果我们需要执行长时间运行的操作（例如onStart() 中的某些配置检查），则可能会导致争用情况，其中onStop() 方法在 onStart() 之前完成，从而使组件的生存期超过了所需的生存期。

```java
class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, location -> {
            // 更新 UI
        });
    }

    @Override
    public void onStart() {
        super.onStart();
        Util.checkUserStatus(result -> {
            // 如果在 activity 停止后调用此回调怎么办？
            if (result) {
                myLocationListener.start();
            }
        });
    }

    @Override
    public void onStop() {
        super.onStop();
        myLocationListener.stop();
    }
}
```

如果有所有的组件，都能感知外部的生命周期，能在相应的时机释放资源，并且在错过生命周期时能及时叫停异步的任务就好了，

![](https://gitee.com/flywith24/Album/raw/master/img/20200326095005.gif)



我们不妨先思考一下，如果实现这样的想法，应该如何做



### 按照惯例的思考

首先我们先来整理一下我们的需求

- 内部组件能够感知外部的生命周期
- 能够统一地管理，做到一处修改，处处生效
- 能够及时叫停错过的任务



针对需求1，可以用观察者模式，内部组件能够在外部生命周期变化时做出相应

针对需求2，可以将依赖组件的代码移出生命周期方法内，然后移入组件本身，这样只需修改组件内部逻辑即可

针对需求3，可以在合适的时机移除观察者



### 观察者模式

关于开发者模式，我第一次比较详细的了解是在 [扔物线](https://juejin.im/user/552f20a7e4b060d72a89d87f) 的 [给 Android 开发者的 RxJava 详解](https://gank.io/post/560e15be2dca930e00da1083)。



> 观察者模式面向的需求是：A 对象（观察者）对 B 对象（被观察者）的某种变化高度敏感，需要在 B 变化的一瞬间做出反应。举个例子，新闻里喜闻乐见的警察抓小偷，警察需要在小偷伸手作案的时候实施抓捕。在这个例子里，警察是观察者，小偷是被观察者，警察需要时刻盯着小偷的一举一动，才能保证不会漏过任何瞬间。程序的观察者模式和这种真正的『观察』略有不同，观察者不需要时刻盯着被观察者（例如 A 不需要每过 2ms 就检查一次 B 的状态），而是采用**注册**(Register)**或者称为**订阅**(Subscribe)**的方式，告诉被观察者：我需要你的某某状态，你要在它变化的时候通知我。 Android 开发中一个比较典型的例子是点击监听器 `OnClickListener` 。对设置 `OnClickListener` 来说， `View` 是被观察者， `OnClickListener` 是观察者，二者通过 `setOnClickListener()` 方法达成订阅关系。订阅之后用户点击按钮的瞬间，Android Framework 就会将点击事件发送给已经注册的 `OnClickListener` 。采取这样被动的观察方式，既省去了反复检索状态的资源消耗，也能够得到最高的反馈速度。当然，这也得益于我们可以随意定制自己程序中的观察者和被观察者，而警察叔叔明显无法要求小偷『你在作案的时候务必通知我』。

OnClickListener 的模式大致如下图：

![](https://gitee.com/flywith24/Album/raw/master/img/20200326103024.jpg)

> 上述描述及图片均来自 [给 Android 开发者的 RxJava 详解](https://gank.io/post/560e15be2dca930e00da1083)



**因此在生命周期组件的生命周期发生变化时告诉观察者，内部组件即可感知外部的生命周期**



### 引入 Lifecycle 后

```java
public class MyObserver implements LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void connectListener() {
        ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void disconnectListener() {
        ...
    }
}

myLifecycleOwner.getLifecycle().addObserver(new MyObserver());
```



## 源码结构



![](https://gitee.com/flywith24/Album/raw/master/img/20200326134237.png)

这是 [Lifecycle](https://developer.android.com/reference/androidx/lifecycle/Lifecycle)  的结构，抽象类，其内部有两个枚举，分别代表着「事件」和「状态」，此外还有三个方法，添加/移除观察者，获取当前状态

> **注意，这里 State 中的枚举顺序是有意义的，后文详细介绍**



其实现类为  [LifecycleRegistry](https://developer.android.com/reference/androidx/lifecycle/LifecycleRegistry) ，可以处理多个观察者

![LifecycleRegistry](https://gitee.com/flywith24/Album/raw/master/img/20200326135029.png)



其内部持有当前的状态 mState ，LifecycleOwner 以及观察者的自定义列表，同时重写了父类的添加/删除观察者的方法



![LifecycleOwner](https://gitee.com/flywith24/Album/raw/master/img/20200326135636.png)



[LifecycleOwner](https://developer.android.com/reference/androidx/lifecycle/LifecycleOwner) ，具有 Android 的生命周期，定制组件可以使用这些事件来处理生命周期更改，而无需在 Activity 或 Fragment 中实现任何代码



[LifecycleObserver](https://developer.android.com/reference/androidx/lifecycle/LifecycleObserver) ，将一个类标记为 `LifecycleObserver`。 它没有任何方法，而是依赖于 OnLifecycleEvent 注解的方法



[LifecycleEventObserver](https://developer.android.com/reference/androidx/lifecycle/LifecycleEventObserver) ，可以接收任何生命周期更改并将其分派给接收方。

**如果一个类实现此接口并同时使用 OnLifecycleEvent，则注解将被忽略**



[DefaultLifecycleObserver](https://developer.android.com/reference/androidx/lifecycle/DefaultLifecycleObserver) ，用于监听 [LifecycleOwner](https://developer.android.com/reference/androidx/lifecycle/LifecycleOwner) 状态更改的回调接口。

如果一个类同时实现了此接口和 [LifecycleEventObserver](https://developer.android.com/reference/androidx/lifecycle/LifecycleEventObserver)，则将首先调用`DefaultLifecycleObserver` 的方法，然后再调用LifecycleEventObserver.onStateChanged（LifecycleOwner，Lifecycle.Event）



> 注意：使用 [DefaultLifecycleObserver](https://developer.android.com/reference/androidx/lifecycle/DefaultLifecycleObserver) 需引入
>
>  implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"



## 简单的源码分析

### activity 生命周期处理

首先我们还是来看 **androidx.activity.ComponentActivity** ，这个类我们这个系列的文章里提到多次，第一次提及是在 [【背上Jetpack】绝不丢失的状态 androidx SaveState ViewModel-SaveState 分析](https://juejin.im/post/5e738d12518825495d69cfb9#heading-2) ，感兴趣的小伙伴可以看看。

![ComponentActivity](https://gitee.com/flywith24/Album/raw/master/img/20200327104836.png)

其实现的接口大多数我们都已经探讨过了，今天我们来看看 LifecycleOwner

> ActivityResultCaller 为 activity 1.2.0-alpha02 推出的，旨在统一 onActivityResult ，这里暂时不讨论它

![](https://gitee.com/flywith24/Album/raw/master/img/20200327105352.png)



既然实现了 `LifecycleOwner` 接口，必定重写 getLifecycle() 方法

``` java
// androidx.activity.ComponentActivity.java
private final LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);

@Override
public Lifecycle getLifecycle() {
    return mLifecycleRegistry;
}
```

其返回的 `Lifecycle` 为 实现类 `LifecycleRegistry` 的实例

而 activity 操作生命周期是通过 `ReportFragment` 处理的



```java
// androidx.activity.ComponentActivity.java
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ReportFragment.injectIfNeededIn(this);
    //...
}

// ReportFragment
public static void injectIfNeededIn(Activity activity) {
    if (Build.VERSION.SDK_INT >= 29) {
        // api 29 及以上 直接注册正确的生命周期回调
        activity.registerActivityLifecycleCallbacks(
                new LifecycleCallbacks());
    }
    android.app.FragmentManager manager = activity.getFragmentManager();
    if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
        manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
        manager.executePendingTransactions();
    }
}
```

![](https://gitee.com/flywith24/Album/raw/master/img/20200327113340.png)

```java
// ReportFragment.java
static void dispatch(@NonNull Activity activity, @NonNull Lifecycle.Event event) {
    if (activity instanceof LifecycleRegistryOwner) {
        ((LifecycleRegistryOwner) activity).getLifecycle().handleLifecycleEvent(event);
        return;
    }
    if (activity instanceof LifecycleOwner) {
        Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
        if (lifecycle instanceof LifecycleRegistry) {
            ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
        }
    }
}

private void dispatch(@NonNull Lifecycle.Event event) {
    if (Build.VERSION.SDK_INT < 29) {
        dispatch(getActivity(), event);
    }
}

@Override
public void onActivityCreated(Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    dispatch(Lifecycle.Event.ON_CREATE);
}
@Override
public void onStart() {
    super.onStart();
    dispatch(Lifecycle.Event.ON_START);
}
@Override
public void onResume() {
    super.onResume();
    dispatch(Lifecycle.Event.ON_RESUME);
}
@Override
public void onPause() {
    super.onPause();
    dispatch(Lifecycle.Event.ON_PAUSE);
}
@Override
public void onStop() {
    super.onStop();
    dispatch(Lifecycle.Event.ON_STOP);
}
@Override
public void onDestroy() {
    super.onDestroy();
    dispatch(Lifecycle.Event.ON_DESTROY);
}
```

```java
// LifecycleCallbacks
static class LifecycleCallbacks implements Application.ActivityLifecycleCallbacks {
    @Override
    public void onActivityPostCreated(@NonNull Activity activity,
            @Nullable Bundle savedInstanceState) {
        dispatch(activity, Lifecycle.Event.ON_CREATE);
    }

    @Override
    public void onActivityPostStarted(@NonNull Activity activity) {
        dispatch(activity, Lifecycle.Event.ON_START);
    }

    @Override
    public void onActivityPostResumed(@NonNull Activity activity) {
        dispatch(activity, Lifecycle.Event.ON_RESUME);
    }
    @Override
    public void onActivityPrePaused(@NonNull Activity activity) {
        dispatch(activity, Lifecycle.Event.ON_PAUSE);
    }

    @Override
    public void onActivityPreStopped(@NonNull Activity activity) {
        dispatch(activity, Lifecycle.Event.ON_STOP);
    }

    @Override
    public void onActivityPreDestroyed(@NonNull Activity activity) {
        dispatch(activity, Lifecycle.Event.ON_DESTROY);
    }
	//...
}
```



在 activity 的 onCreate 方法中，调用了 `ReportFragment` 中的静态方法 `injectIfNeededIn()` 。而其内部，**如果 api 29 及以上的设备上直接注册正确的生命周期回调，低版本通过启动 ReportFragment ，借助 fragment 各个生命周期来处理生命周期回调**

### fragment 生命周期处理

在 fragment 内部，每个生命周期节点调用 `handleLifecycleEvent` 方法

```java
// Fragment.java
public Fragment() {
    initLifecycle();
}

private void initLifecycle() {
    mLifecycleRegistry = new LifecycleRegistry(this);
}

@Override
public Lifecycle getLifecycle() {
    return mLifecycleRegistry;
}

void performCreate(Bundle savedInstanceState) {
    onCreate(savedInstanceState);
	mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE);    
}

void performStart() {
    onStart();
    mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_START);
}

void performResume() {
    onResume();
    mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_RESUME);    
}

void performPause() {
    mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_PAUSE);
    onPause();
}

void performStop() {
    mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_STOP);
    onStop();
}

void performDestroy() {
    mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_DESTROY);
    onDestroy();
}
```



### Lifecycle State 大小比较

`Lifecycle.State`  中有一个 `isAtLeast` 方法，用于判断当前状态是否不小于传入的状态

```java
// Lifecycle.State
public boolean isAtLeast(@NonNull State state) {
    return compareTo(state) >= 0;
}
```

**枚举的 compareTo 方法其实是比较的枚举声明的顺序**

而 State 的顺序为 DESTROYED -> INITIALIZED -> CREATED -> STARTED -> RESUMED



> 如果传入的 state 为 STARTED，则当前状态为 STARTED 或 RESUMED 时返回 true ，否则返回 false
>
> LiveData 篇会用到这个知识点



<h2 style='color: inherit; line-height: inherit; padding: 0px; margin: 1.6em 0px; font-weight: bold; border-bottom: 2px solid #9370DB; font-size: 1.3em;'><span style='font-size: inherit; line-height: inherit; margin: 0px; display: inline-block; font-weight: normal; background: #9370DB; color: rgb(255, 255, 255); padding: 3px 10px 1px; border-top-right-radius: 3px; border-top-left-radius: 3px; margin-right: 3px;'> 关于我</span></h2>


我是 [Fly_with24](https://flywith24.gitee.io/)
- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)
- [简书](https://www.jianshu.com/u/3d5ad6043d66)
- [Github](https://github.com/Flywith24)
