---
title: 使用 Navigation + Dynamic Feature Module 实现模块化
date: 2020-05-20 14:11:46
categories: 
- 奇技淫巧
tags: 
- Tips
- 奇技淫巧
image: https://gitee.com/flywith24/Album/raw/master/img/20201015104605.png
---

使用 Navigation + Dynamic Feature Module 实现模块化

<!-- more-->

`androidx navigation 2.3.0` 加入了对 `dynamic feature module` 的导航支持，因此我们利用这个来分离出多个功能 module 来实现模块化

![navigation 2.3.0 更新](https://gitee.com/flywith24/Album/raw/master/img/20200520170621.png)

## 国内基本不用的 dynamic feature module



[Android App Bundle](https://developer.android.com/guide/app-bundle) 是官方 18 年推出的动态发布方案，类似国内各种插件化方案。不过它需要 Google Play Store 支持，这导致在国内无法使用



借着 navigation 组件支持 `dynamic feature module` 间导航的契机，我们可以使用 `dynamic feature module` 来拆分功能模块以实现模块化



传统的拆分方案大概是这样，feature module 之间相互隔离，app module 依赖各个 feature module 间接依赖 base 库，公共库



![传统架构](https://gitee.com/flywith24/Album/raw/master/img/20200520165717.png)





而使用 `dynamic feature module` ，其结构是这样的

![dynamic feature 架构](https://gitee.com/flywith24/Album/raw/master/img/20200520172609.png)



`dynamic feature module` 也可以按需安装，也就是说，它们可能不包含在用户最初下载的 APK 中，而是在运行时安装。而我们可以直接将它们包含到 APK 中



## 使用 dynamic feature module



首先我们在 base lib 中引入依赖

```groovy
dependencies {
    def nav_version = "2.3.0-alpha06"

    api "androidx.navigation:navigation-fragment-ktx:$nav_version"
    api "androidx.navigation:navigation-ui-ktx:$nav_version"
    api "androidx.navigation:navigation-dynamic-features-fragment:$nav_version"
}
```



我们在 app module 中的 res/navigation 目录下创建 main_nav.xml

![main_nav.xml](https://gitee.com/flywith24/Album/raw/master/img/20200520173227.png)



接着我们在 activity_main 中设置默认的 host

![默认 host](https://gitee.com/flywith24/Album/raw/master/img/20200520173336.png)

> 这里不同于正常 navigation 的用法，没有使用 NavHostFragment，而是使用 DynamicNavHostFragment



### 直接跳转 fragment

我们创建 dynamic feature module ，取名为 feature1



![创建 dynamic feature module](https://gitee.com/flywith24/Album/raw/master/img/20200520174302.gif)



![包名前部分需保证与 applicationId 相同](https://gitee.com/flywith24/Album/raw/master/img/20200520174505.png)

> 这里 `dynamic feature module` 的包名前部分要和 applicationId 即 app module 包名相同，否则后续的 include 操作会有问题



![选择加载模式](https://gitee.com/flywith24/Album/raw/master/img/20200520174757.png)



这里我们选择在安装时集成该 module



接着我们在该 module 下创建一个 fragment 取名为 `Feature1OneFragment`

之后我们直接在 main_nav.xml 中引入 该 fragment 并加入 action



![直接引入 fragment](https://gitee.com/flywith24/Album/raw/master/img/20200520175351.png)

接着我们就可以在 app 下的 MainFragment 打开 Feature1OneFragment

![启动 fragment](https://gitee.com/flywith24/Album/raw/master/img/20200520175841.gif)

> 我的 demo 中 feature2 是直接引入 fragment，因此跳转的是 Feature2OneFragment



### 直接跳转 activity

在 feature1 中创建 activity (demo 中为 feature2)

![跳转 activity](https://gitee.com/flywith24/Album/raw/master/img/20200520180201.png)

同样需要指定 moduleName

![启动activity](https://gitee.com/flywith24/Album/raw/master/img/20200520180434.gif)



### 使用 dynamic feature module 内部的 graph

我们可以为 dynamic feature module 单独配置 navigation graph，这样就可以处理 dynamic feature module 内部的跳转了



在 feature1 中创建 feature1_nav.xml ，其中 startDestination 为 Feature1OneFragment

![feature1_nav.xml](https://gitee.com/flywith24/Album/raw/master/img/20200520181021.png)



在 main_nav.xml 我们需要使用另外一种方式来使用该 graph

![include-dynamic](https://gitee.com/flywith24/Album/raw/master/img/20200520181145.png)

我们使用了一个新的标签 `include-dynamic`，同时我们看到了几个没用过的属性

- `graphPackage` 为 `dynamic feature module` 的包名
- `graphResName` 为 `dynamic feature module` 内部 graph 的名字
- `moduleName` 为 module 名

> 注意：这里的 graphPackage 可以省略
>
> 1. 如果 module 的包名没用按照前文的格式配置会导致无法找到 graphId 的异常
> 2. include-dynamic 标签的 id 要与 `feature1_nav.xml` navigation 标签下的 id 一致，或者后者不设置 id



这样从 app module 导航到 feature1 的 startDestination 后便可使用其内部的逻辑进行后续的导航了



![include 跳转](https://gitee.com/flywith24/Album/raw/master/img/20200520182339.gif)



## feature module 间跳转

![暂不支持 deep link](https://gitee.com/flywith24/Album/raw/master/img/20200520182529.png)

Navigation 组件暂不支持 Dynamic include graph 的 deep link



因此我目前也没有找到特别优雅的方式，已知的方案如下

- 反射
- 使用 [ServiceLoader](https://developer.android.com/reference/java/util/ServiceLoader)
- 使用依赖注入



[demo](https://github.com/Flywith24/DynamicFeatureDemo)


## 关于我

我是 [Flywith24](https://flywith24.gitee.io/)，我的博客内容已经分类整理 [在这里](https://github.com/Flywith24/BlogList)，点击右上角的 Watch 可以及时获取我的文章更新哦 😉



- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)

- [小专栏](https://xiaozhuanlan.com/detail)

- [Github](https://github.com/Flywith24)

  

![](https://user-gold-cdn.xitu.io/2020/6/26/172ee567fb4fbf7e?w=1954&h=624&f=jpeg&s=115362)

