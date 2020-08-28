---
title: 【奇技淫巧】Android组件化不使用 Router 如何实现组件间 activity 跳转
date: 2020-04-15 14:11:46
categories: 
- Tips
tags: 
- Tips
- 奇技淫巧
top: true
---

## 前言

越来越多的项目使用了组件化，组件之间的通信是一个比较重要的问题。`ARouter` 等路由方案为我们提供了解决办法。那么如果不使用 Router 如何实现组件间的界面跳转呢？

<!-- more-->

## 万能的 `setClassName`

从一个 Activity 跳转到另一个Activity 的最直接方法如下：

``` kotlin
val intent = Intent(this, TestActivity::class.java)
startActivity(intent)
```



但是，采用这种方法，当原 activity 位于一个 module（例如 `FeatureA` ）中，而目标 activity 位于另一个 module（`FeatureB`）中时，该怎么办？

![](https://gitee.com/flywith24/Album/raw/master/img/20200415110401.png)



我们可以使用 Intent 的 `setClassName` 方法

``` kotlin
val intent = Intent()
intent.setClassName(this, “com.flywith24.demo.TestActivity”)
startActivity(intent)
```



但是这种方式硬编码目标 activity 的完整类名，如果 activity 的类名被更改或者移动，而且没有更改硬编码，则编译可以通过，但是运行时崩溃



如果可以自动生成 activity 完整类名就好了



## 使用插件

我们知道 activity 作为 Android 的组件之一需要在 Manifest 文件中声明



``` xml
<activity android:name=”com.flywith24.demo.MainActivity” />

<activity android:name=”com.flywith24.demo.TestActivity” />
```



如果我们的数据是从 Manifest 中获得的，那么就解决了硬编码的问题了



有这样一个[插件](https://github.com/gaelmarhic/Quadrant) ，在 build 时会将所有在 Manifest 中声明的 activity 的完整类名以静态常量的形式罗列到一个静态类中



``` kotlin
object QuadrantConstants {
  
  const val MAIN_ACTIVITY: String = "com.gaelmarhic.quadrant.MainActivity"

  const val SECONDARY_ACTIVITY: String = "com.gaelmarhic.quadrant.SecondaryActivity"

  const val TERTIARY_ACTIVITY: String = "com.gaelmarhic.quadrant.TertiaryActivity"
}
```



这样在使用时就避免了硬编码



``` kotlin
val intent = Intent()
intent.setClassName(context, QuadrantConstants.MAIN_ACTIVITY)
startActivity(intent)
```



## 使用依赖注入

组件化中 `app` module 会依赖所有的功能 module ，因此如果我们使用依赖注入在 `app` 中将所有的目标 activity 的完整类名声明出来，也能达到解决硬编码的问题



这里以 [koin](https://insert-koin.io/) 为例



``` kotlin
class MyApplication : Application() {
    val myModule = module {
        single { Feature2Activity::class.java.name }
    }

    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidContext(this@MyApplication)
            modules(myModule)
        }
    }
}
```



这样通过 get() 方法即可拿到 `Feature2Activity` 的完整类名



``` kotlin
val intent = Intent()
    .setClassName(this@Feature1Activity, get())
    .putExtra("key", "value")
startActivity(intent)
```



## Demo

[Demo 地址](https://github.com/Flywith24/MultitableModuleApp)



各位有什么想法欢迎在评论区留言



## 关于我

我是 [Flywith24](https://flywith24.gitee.io/)，我的博客内容已经分类整理 [在这里](https://github.com/Flywith24/BlogList)，点击右上角的 Watch 可以及时获取我的文章更新哦 😉



- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)

- [小专栏](https://xiaozhuanlan.com/detail)

- [Github](https://github.com/Flywith24)

  

![](https://user-gold-cdn.xitu.io/2020/6/26/172ee567fb4fbf7e?w=1954&h=624&f=jpeg&s=115362)

