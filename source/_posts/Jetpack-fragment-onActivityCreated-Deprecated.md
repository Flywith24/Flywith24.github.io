---
title: 【Jetpack更新之Fragment】终于动手了，onActivityCreated 被弃用
date: 2020-04-09 00:10:35
categories: 
- Jetpack
tags: 
- androidx
- Jetpack
- MVVM
- Fragment
---

>本系列文章介绍 Jetpack 组件库的更新<br> 
>一直以来， fragment 的 api 都非常难用，官方也承认这一点。一个月前，fragment 中的 `onActivityCreated()` 被弃用了

<!-- more-->


## Fragment

`fragment 1.3.0-alpha02` 中 `onActivityCreated()` 方法被弃用了

![](https://gitee.com/flywith24/Album/raw/master/img/20200421095145.png)

让我们来看一下提交 log 

![](https://gitee.com/flywith24/Album/raw/master/img/20200421092105.png)

简单翻译一下

`onActivityCreated()` 最初的目的是让 fragment 的逻辑与其宿主 activity 创建建立关联，我们不鼓励这种耦合

我们应该传递外部依赖来作为 `FragmentFactory` 参数。view 相关的代码应该放置在 `onViewCreated()` 完成，其他的初始化代码应该在 `onCreate()` 中完成。为了在 `activity onCreate()` 完成后接收回调，可以添加一个 activity 生命周期的 `LifecycleObserver` ，并且接收到 `Lifecycle.State#CREATED` 回调时将其移除


``` kotlin
override fun onAttach(context: Context) {
    super.onAttach(context)
    requireActivity().lifecycle.addObserver(object : DefaultLifecycleObserver {
        override fun onCreate(owner: LifecycleOwner) {
            // 想做啥做点啥
            owner.lifecycle.removeObserver(this)
        }
    })
}
```



## DialogFragment

那么 `DialogFragment` 怎么办？其 `onActivityCreated` 变为可选的

![](https://gitee.com/flywith24/Album/raw/master/img/20200421101629.png)

简单翻译一下



`DialogFragment` 使用 `onActivityCreated`() 帮助创建 dialog。onActivityCreated() 弃用后我们应当寻找一个更好的方式来执行这部分逻辑



关于 view 相关的代码已经转移至 `DialogFragment`

 的 `viewLifecycleOwnerLiveData` ，其他初始化逻辑可以放在 `onGetLayoutInflater`

![](https://gitee.com/flywith24/Album/raw/master/img/20200421103420.png)



我们仍支持为自定义 dialog 在 `onActivityCreated()` 中配置 dialog

## End

查看 `Jetpack fragment` 的变动，不难看出官方正致力于为 fragment 「减负」，将小的，独立的功能从 fragment 中抽离出去，降低耦合，后续文章我们介绍其他的改动



## 关于我

我是 [Fly_with24](https://flywith24.gitee.io/)
- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)

- [简书](https://www.jianshu.com/u/3d5ad6043d66)

- [Github](https://github.com/Flywith24)

  


<center><p> 欢迎关注我的公众号</p></center>

<div align=center><img src="https://gitee.com/flywith24/Album/raw/master/img/20200429102625.jpg"/></div>





