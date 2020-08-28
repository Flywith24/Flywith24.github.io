---
title: 【Jetpack更新之Fragment】1.3.0-alpha04 来袭，Fragment 间通信的新姿势
date: 2020-04-30 00:10:35
categories: 
- Jetpack
tags: 
- androidx
- Jetpack
- MVVM
- Fragment
---

## 前言

`fragment 1.3.0-alpha04` 发布了，其中有很多变动，其中提供了 fragment 间传递数据的新方式

![1.3.0-alpha04 更新](https://gitee.com/flywith24/Album/raw/master/img/20200430090529.png)

<!-- more-->

### API 更改

首先我们介绍一下 API 更改

- `startActivityForResult()`/`onActivityResult()` 和 `requestPermissions()`/`onRequestPermissionsResult()` 弃用
- `prepareCall()` 重命名为 `registerForActivityResult()` 
- `target fragment API` 被弃用



### Activity Result API 上位

由于官方提供了 **Activity Result API** 来替换 **onActivityResult**  机制，因此 fragment 的  `startActivityForResult()`/`onActivityResult()` 和 `requestPermissions()`/`onRequestPermissionsResult()` 方法被标记弃用了



**Activity Result API** 详情可参考 [秉心说](https://juejin.im/user/586eff908d6d81005879507d) 的 [是时候丢掉 onActivityResult 了 ！](https://juejin.im/post/5e80cb1ee51d45471654fae7)

文章介绍的很详尽，这里不再赘述



### prepareCall 重命名

值得注意的地方是 `prepareCall()` 被命名为 `registerForActivityResult()` 

> 注意：在版本处于 Alpha 版状态时，可以添加、移除或更改 API。因此 Alpha 版本不适合在生产上使用

![来自我的另一篇博客](https://gitee.com/flywith24/Album/raw/master/img/20200430091418.png)



### target fragment API 被弃用

其实 `target fragment API` 早已被弃用

![setTargetFragment 被弃用](https://gitee.com/flywith24/Album/raw/master/img/20200430092548.png)

`target fragment` 需要直接访问另一个 fragment 的实例，这是十分危险的，因为你不知道目标 fragment 处于什么状态。而且 `target fragment` 不支持 Navigation



![弃用 target fragment API](https://gitee.com/flywith24/Album/raw/master/img/20200430092824.png)

那么，fragment 之间传递数据更干净的方式是什么呢？



### fragment 之间传递数据的新方式

前文提到，在相同的 FragmentManager 中可以使用 target fragment API 来在 fragment 间传递数据，但这种方式需要直接访问目标fragment 的实例，这很危险，因为目标 fragment 的状态是未知的



因此官方提供了这样的 API，它允许在一个 fragment 上设置结果，并将该结果在 fragment 的适当的生命周期中使用。

这种传递数据的方式适用于 DialogFragment ，Navigation 中的 fragment

此更改还包括 -ktx 扩展功能以确保 kotlin 用户可以将 FragmentResultListener 作为 lambda 传递

![FragmentA 源码](https://gitee.com/flywith24/Album/raw/master/img/20200430105343.png)

![FragmentB 源码](https://gitee.com/flywith24/Album/raw/master/img/20200430105411.png)

 

![demo](https://gitee.com/flywith24/Album/raw/master/img/20200430105045.gif)



## 源码分析

老规矩，我们沿着官方的 commit log 来看看官方实现该功能的思路

首先，添加了 FragmentResultOwner 这样的的抽象，用于处理 fragment result，其内部有两个方法

- setResult
- setResultListener

前者用于发送数据，后者用于接收数据

![FragmentResultOwner](https://gitee.com/flywith24/Album/raw/master/img/20200430110206.png)

而其实现类为 FragmentManager

![FragmentManager implement FragmentResultOwner ](https://gitee.com/flywith24/Album/raw/master/img/20200430110412.png)

我们来看看 FragmentManager 两个方法的具体实现

![](https://gitee.com/flywith24/Album/raw/master/img/20200430111003.png)



``` java
public final void setFragmentResultListener(@NonNull final String requestKey,
        @NonNull final LifecycleOwner lifecycleOwner,
        @Nullable final FragmentResultListener listener) {
    // 设置的 listener 为空时将 requestKey 对应的 listener 移除
    if (listener == null) {
        mResultListeners.remove(requestKey);
        return;
    }
    
    // 当fragment 处于DESTROYED 状态时 直接 return ，避免了异常
    final Lifecycle lifecycle = lifecycleOwner.getLifecycle();
    if (lifecycle.getCurrentState() == Lifecycle.State.DESTROYED) {
        return;
    }
    
    // 观察生命周期，fragment started 后接收回调，destroyed 移除回调
    LifecycleEventObserver observer = new LifecycleEventObserver() {
        @Override
        public void onStateChanged(@NonNull LifecycleOwner source,
                @NonNull Lifecycle.Event event) {
            if (event == Lifecycle.Event.ON_START) {
                // once we are started, check for any stored results
                Bundle storedResult = mResults.get(requestKey);
                if (storedResult != null) {
                    // if there is a result, fire the callback
                    listener.onFragmentResult(requestKey, storedResult);
                    // and clear the result
                    setFragmentResult(requestKey, null);
                }
            }
            if (event == Lifecycle.Event.ON_DESTROY) {
                lifecycle.removeObserver(this);
                mResultListeners.remove(requestKey);
            }
        }
    };
    lifecycle.addObserver(observer);
    mResultListeners.put(requestKey, new LifecycleAwareResultListener(lifecycle, listener));
}
```

以上便是这部分的源码

> 这里要注意一点的是 `fragment result api` 是基于同一 `FragmentManager` 的



## 总结

官方一直致力于将 fragment 的 api 变得更好用

[Ian Lake](https://medium.com/@ianhlake) 在 [Fragments: Past, Present, and Future (Android Dev Summit '19)](https://www.youtube.com/watch?v=RS1IACnZLy4) 中提到了 fragment 间通信的问题，未来 fragment 会整合 fragment 自身和其内部 view 的生命周期，提供同一 FragmentManager 多返回栈的支持



看到 `fragment result API` ，我突然有个想法，如果将其应用到 Navigation 中是否是解决 Navigation 跳转返回后状态重置的一个方法呢？

各位小伙伴有什么想法欢迎评论区留言




## 关于我

我是 [Fly_with24](https://flywith24.gitee.io/)
- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)

- [简书](https://www.jianshu.com/u/3d5ad6043d66)

- [Github](https://github.com/Flywith24)

  


<center><p> 欢迎关注我的公众号</p></center>

<div align=center><img src="https://gitee.com/flywith24/Album/raw/master/img/20200429102625.jpg"/></div>


