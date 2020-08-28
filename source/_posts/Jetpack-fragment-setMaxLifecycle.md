---
title: 【Jetpack更新之Fragment】setMaxLifecycle 上位，setUserVisibleHint 被弃用
date: 2020-04-29 00:10:35
categories: 
- Jetpack
tags: 
- androidx
- Jetpack
- MVVM
- Fragment
---

很多情况下，fragment 的生命周期上限应该低于 FragmentManager/Activity。例如，`ViewPager` 屏幕外的界面不应被 `resumed`



理想状态下，可以通过以下 API 实现

``` kotlin
supportFragmentManager
	  .beginTransaction()
      .setMaxLifecycle(fragment, Lifecycle.State.RESUMED)
      .commit()
```



将最大生命周期设置为 `Lifecycle.State.RESUMED` 将有效地消除限制（因为这是最高生命周期状态）



这将允许废弃 `setUserVisibleHint()` API

<!-- more-->


## setMaxLifecycle 出现始末

该功能应如何实现的？我们沿着 `commit log` 来理一下官方的思路



将 `BackStackRecord` 的部分逻辑转移至父类 `FragmentTransaction` 中



![](https://gitee.com/flywith24/Album/raw/master/img/20200423094356.png)

![](https://gitee.com/flywith24/Album/raw/master/img/20200423101522.png)



在 `FragmentTransaction` 中添加 `setMaxLifecycle` API

![](https://gitee.com/flywith24/Album/raw/master/img/20200423101908.png)

![](https://gitee.com/flywith24/Album/raw/master/img/20200423102052.png)



保存 fragment `maxState`

![](https://gitee.com/flywith24/Album/raw/master/img/20200423103245.png)

![](https://gitee.com/flywith24/Album/raw/master/img/20200423103057.png)



弃用 `setUserVisibleHint`

![](https://gitee.com/flywith24/Album/raw/master/img/20200423103528.png)



`FragmentPagerAdapter` 构造器新增参数，使用 `setMaxLifecycle()` API 确保 fragment `resumed` 时对用户可见



![](https://gitee.com/flywith24/Album/raw/master/img/20200423104355.png)

![](https://gitee.com/flywith24/Album/raw/master/img/20200423104634.png)



弃用 `FragmentStatePagerAdapter` 原来的单参构造器，推荐使用新的构造



![](https://gitee.com/flywith24/Album/raw/master/img/20200423105038.png)

![](https://gitee.com/flywith24/Album/raw/master/img/20200423105144.png)



**随着 `ViewPager2 1.0.0` 正式版发布，与 `ViewPager` 交互的`FragmentPagerAdapter` 和 `FragmentStatePagerAdapter` 被弃用了**

![](https://gitee.com/flywith24/Album/raw/master/img/20200423111201.png)



至此我们捋顺了 `setMaxLifecycle` 的出现，`setUserVisibleHint` 的弃用以及与`ViewPager` 相关的 `FragmentPagerAdapter` 和 `FragmentStatePagerAdapter` 的弃用



## setMaxLifecycle 内部逻辑

接下来我们看看 `setMaxLifecycle`  是如何发挥作用的

首先我们要研究一下 fragment 的状态管理，为了更好的管理 fragment 的状态，官方添加了 `FragmentStateManager` 类来专门管理 fragment 的状态，职能单一原则哈



![](https://gitee.com/flywith24/Album/raw/master/img/20200423113312.png)

![](https://gitee.com/flywith24/Album/raw/master/img/20200423113509.png)



接着在该类中添加了计算 fragment 最大生命周期的方法 `computeMaxState()` 



![](https://gitee.com/flywith24/Album/raw/master/img/20200423114557.png)

![](https://gitee.com/flywith24/Album/raw/master/img/20200423114615.png)



后来该方法改名为 `computeExpectedState()` 并加入了 `moveToExpectedState()` 方法



![](https://gitee.com/flywith24/Album/raw/master/img/20200423115055.png)



`computeExpectedState()`  方法会根据 fragment `mMaxState` 计算 fragment 应该所处的生命周期



![](https://gitee.com/flywith24/Album/raw/master/img/20200423115521.png)



而 fragment 的 `mMaxState` 是通过 `FragmentManager` 的 `setMaxLifecycle()` 方法设置的 ，而该方法是 `BackStackRecord` 执行 OP 时调用的，而 OP 值正是通过 `FragmentTransaction` 的 `setMaxLifecycle()` 设置的



![](https://gitee.com/flywith24/Album/raw/master/img/20200423115744.png)

![](https://gitee.com/flywith24/Album/raw/master/img/20200423115928.png)



至此，我们理清了 `setMaxLifecycle()` 的内部逻辑



## 总结

我们可以看到官方为了使 fragment 能够在正确的生命周期上，引入了 `setMaxLifecycle()` 方法，同时为了更好的管理 fragment 的状态，抽象出了 `FragmentStateManager` 。*更少的代码，更少的职责*，fragment 的内部逻辑会越来越清晰



- 关于如何迁移至 ViewPager2 ，请移步 [官方视频](https://www.bilibili.com/video/BV1uJ411C7S4?from=search&seid=7755962876168902731)

- 关于新的 API 下懒加载实现，请移步 [Androidx 下 Fragment 懒加载的新实现](https://juejin.im/post/5e232d01e51d455801624c06)

  



## 关于我

我是 [Fly_with24](https://flywith24.gitee.io/)
- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)

- [简书](https://www.jianshu.com/u/3d5ad6043d66)

- [Github](https://github.com/Flywith24)

  


<center><p> 欢迎关注我的公众号</p></center>

<div align=center><img src="https://gitee.com/flywith24/Album/raw/master/img/20200429102625.jpg"/></div>




