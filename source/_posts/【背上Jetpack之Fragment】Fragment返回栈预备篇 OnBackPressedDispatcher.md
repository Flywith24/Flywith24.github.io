---
title: 【背上Jetpack】Fragment返回栈预备篇 OnBackPressedDispatcher
date: 2020-03-14 00:10:35
categories: 
- Jetpack
tags: 
- androidx
- Jetpack
- MVVM
---

> 这两天在准备写 fragment 返回栈的文章，但是发现必须先介绍一下 OnBackPressedDispatcher ，所以这是一篇介绍 what 的文章，喜欢一手资料的可以移步 [官方文档](https://developer.android.google.cn/reference/kotlin/androidx/activity/OnBackPressedDispatcher)


<!-- more-->

## 系列文章

> [【背上Jetpack】Jetpack 主要组件的依赖及传递关系](https://juejin.im/post/5e567ee1518825494466a938)<br>
>
> [【背上Jetpack】AdroidX下使用Activity和Fragment的变化](https://juejin.im/post/5e5a0c316fb9a07cd248d29e)<br>
>
>[【背上Jetpack之Fragment】你真的会用Fragment吗？Fragment常见问题以及androidx下Fragment的使用新姿势](https://juejin.im/post/5e5cd8686fb9a07cbc269d10)<br>
>
> [【背上Jetpack之Fragment】从源码角度看 Fragment 生命周期 AndroidX Fragment1.2.2源码分析](https://juejin.im/post/5e67523551882549003d2c4f)



## When

`OnBackPressedDispatcher` 在 `androidx activity 1.0.0` 加入，旨在处理返回逻辑。您不仅可以获得在 `Activity` 之外处理返回键的便捷方式。 根据您的需要，您可以在任意位置定义 `OnBackPressedCallback`，使其可复用，或根据应用程序的架构进行任何操作。 您不再需要重写`Activity` 中的 `onBackPressed` 方法，也不必提供自己的抽象的来实现需求的代码。


![OnBackPressedDispatcher](https://user-gold-cdn.xitu.io/2020/3/14/170d4a399942581f?w=1778&h=1246&f=png&s=339228)

## What

`ComponentActivity` 是 `FragmentActivity` 和 `AppCompatActivity` 的基类，它使您可以通过使用其 `OnBackPressedDispatcher`（可以通过调用 `getOnBackPressedDispatcher()` ）来控制返回按钮的行为。




![ComponentActivity-onBackPressed](https://user-gold-cdn.xitu.io/2020/3/14/170d4a9d225a9575?w=2168&h=1150&f=png&s=238889)



`OnBackPressedDispatcher` 控制如何将返回按钮事件分配给一个或多个`OnBackPressedCallback` 对象。 `OnBackPressedCallback` 的构造函数将布尔值用于初始启用状态。 仅当启用了回调（即 `isEnabled()` 返回true）时，调度程序才会调用回调的`handleOnBackPressed()` 来处理返回按钮事件。 您可以通过调用 `setEnabled()` 来更改启用状态。



回调是通过 `addCallback` 方法添加的。 强烈建议使用采用 LifecycleOwner 的`addCallback()` 方法。 这样可以确保仅在 LifecycleOwner 为 Lifecycle.State.STARTED 时才添加`OnBackPressedCallback`。 当关联的 LifecycleOwner 被销毁时，该 activity 会删除已注册的回调，以防止内存泄漏，并使其适用于寿命比该 `activity` 短的 `fragment` 或其他生命周期所有者。



下面是一个示例

```kotlin
class MyFragment : Fragment() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        //此 callback 仅当 MyFragment 至少是 Started 状态下调用
        val callback = requireActivity().onBackPressedDispatcher.addCallback(this) {
            //拦截返回事件
        }

        //此 callback 可以在这里或者上面的 lambda 中开启和关闭
    }
    ...
}
```



您可以通过 `addCallback()` 提供多个回调。 这样做时，将按照添加回调的相反顺序调用回调，即最后添加的回调是第一个给予处理返回按钮事件的机会的回调。 例如，如果您依次添加了三个分别名为1、2和3的回调，则将分别以3、2和1的顺序调用它们。



回调遵循“责任链”模式。 仅当未启用前一个回调时，才调用链中的每个回调。 这意味着在前面的示例中，仅当未启用回调3时，才会调用回调2。 仅当未启用回调2时，才调用回调1，依此类推。



请注意，通过 `addCallback()` 添加回调时，直到 LifecycleOwner 进入Lifecycle.State.STARTED 状态，才将回调添加到责任链中。



强烈建议更改 `OnBackPressedCallback` 的启用状态以进行临时更改(即更改 isEnabled 的值)，因为它可以保持上述顺序，如果您在多个不同的嵌套生命周期所有者上注册了回调，这尤其重要。



但是，如果要完全删除 `OnBackPressedCallback`，则应调用 remove()。 但是，这通常不是必需的，因为在销毁关联的 `LifecycleOwner` 时会自动删除其回调。



## Activity onBackPressed()



如果您使用 `onBackPressed()` 处理返回按钮事件，建议您改用 `OnBackPressedCallback` 。 但是，如果您无法进行此更改，则适用以下规则：

- 当您调用 `super.onBackPressed()` 时，将通过 `addCallback` 注册的所有回调。

- 无论 `OnBackPressedCallback` 的任何注册实例，始终会调用 `onBackPressed`。



## Demo

关于 fragment 返回栈的 demo 已经写好了，感兴趣的小伙伴可以 [在这](https://github.com/Flywith24/Flywith24-Fragment-Demo) 找到它。



我们下一篇再见。



---

## 关于我

---

我是 [Fly_with24](https://flywith24.gitee.io/)

- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)

- [简书](https://www.jianshu.com/u/3d5ad6043d66)

- [Github](https://github.com/Flywith24)