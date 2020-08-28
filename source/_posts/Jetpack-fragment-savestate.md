---
title: 【Jetpack更新之Fragment】setRetainInstance 被弃用，那么 fragment 是如何保存状态的？
date: 2020-04-30 00:10:35
categories: 
- Jetpack
tags: 
- androidx
- Jetpack
- MVVM
- Fragment
---

我们都知道 fragment 中的 `setRetainInstance` 用于控制是否在 activity 保留 fragment 实例，具体内容可参见 [WanAndroid 的每日一问：Fragment 是如何被存储与恢复的？](https://www.wanandroid.com/wenda/show/12574)



但是该方法已于 `androidx fragment 1.3.0-alpha01` 弃用了


<!-- more-->


![](https://gitee.com/flywith24/Album/raw/master/img/20200422093129.png)



老规矩，我们查看一下 commit log



![](https://gitee.com/flywith24/Album/raw/master/img/20200422094533.png)



简单概况一下

`SetRetainInstance` 尝试在 activity 重建时保存状态。但它带来了很多副作用。

随着 `ViewModel` 的引入，开发者拥有一个特定的 API，用于保留与 Activity，Fragments 和 Navigation 相关联的状态。 这使开发者可以使用正常的，不需要保留 fragment ，从而在保存单个需要的属性时避免了常见的泄漏源，并且可以销毁保存的状态（即 `ViewModel` 的构造器和 `onCleared` 回调）



详情可参见 [【背上Jetpack】绝不丢失的状态 androidx SaveState ViewModel-SaveState 分析](https://juejin.im/post/5e738d12518825495d69cfb9) 和 [【背上Jetpack之ViewModel】即使您不使用MVVM也要了解ViewModel ——ViewModel 的职能边界](https://juejin.im/post/5e786d415188255e00661a4e)



从这个改动可以看出官方正致力于保证逻辑的单一性，状态保存交给 `ViewModel` ，减少这种特殊的例外情况，从而消除一些不符合预期的问题




## 关于我

我是 [Fly_with24](https://flywith24.gitee.io/)
- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)

- [简书](https://www.jianshu.com/u/3d5ad6043d66)

- [Github](https://github.com/Flywith24)

  


<center><p> 欢迎关注我的公众号</p></center>

<div align=center><img src="https://gitee.com/flywith24/Album/raw/master/img/20200429102625.jpg"/></div>


