---
title: 背上Jetpack之LiveData】ViewModel的左膀右臂 数据驱动真的香
date: 2020-03-31 00:10:35
categories: 
- Jetpack
tags: 
- androidx
- Jetpack
- MVVM
---

## 前言

> 之前我们讨论过 [ViewModel 的职能边界](https://juejin.im/post/5e786d415188255e00661a4e) ，得益于 ViewModel 的生命周期更长，我们可以在 activity 重建后将数据传递给 activity ，也可以避免内存泄漏。但是如果不是每次需要就获取数据，而是当每次有新数据时通知我们，应该怎么办？

本文介绍 `LiveData` ，一个 **生命周期感知的，可观察的，数据持有者**。同时还会简单分析 `LiveData` 的源码实现

<!-- more-->



## 我们都是 Adapter

在谈 `LiveData` 前我们来思考一个问题

**Android 开发（亦或者说前端开发）的本质工作内容是什么？**

对于应用层 app 开发者，开发者的工作主要工作就是 Adapter 

什么是 Adapter ，下图可能比较直观

![Adapter](https://gitee.com/flywith24/Album/raw/master/img/20200330153727.png)

> 图片来自 google image



我们的工作本质是 **将数据转换成 UI** 

数据可能来自网络，来自本地数据库，来自内存，而 UI 可能是 activity 或 fragment。



## 理想的数据模型

上面我们提到 Android 开发者的核心工作就是将数据转换为 UI 。这个过程比较理想的状态是：当数据发生变化时，UI 跟随变化。我们还可以进一步展开：当 UI 对用户可见时，数据发生变化时 UI 跟随变化；当 UI 对用户不可见时，我们希望数据变化时什么都不做，当 UI 再次对用户可见时根据最新的数据进行 UI 的处理。

而 `LiveData` 就是我们理想中的数据模型

![LiveData](https://gitee.com/flywith24/Album/raw/master/img/20200330160205.png)

> 图片来自 [Android Dev Summit '18-Fun with LiveData](https://www.youtube.com/watch?v=2rO4r-JOQtA&list=PLWz5rJ2EKKc_dskHzXdKHB2ZvAlMyFwZe&index=2&t=0s)



 LiveData 可以三个关键词概括

- lifecycle-aware

- observable

- data holder

  

### observable

Android 中不同的组件有着不同的生命周期，不同的存活时间

![ViewModel](https://gitee.com/flywith24/Album/raw/master/img/20200321140641.png)



因此我们不会在 `ViewModel` 中持有 `Activity` 的引用，因为这会导致当 `Activity` 重建时内存泄漏，甚至出现空指针的情况



![observable](https://gitee.com/flywith24/Album/raw/master/img/20200330161100.png)



通常我们会在 `Activity` 中持有 `ViewModel` 的引用，那么如何进行二者间的通信，如何向 `Activity` 发送 `ViewModel` 中的数据？

答案是让 `Activity` 观察 `ViewModel`

![](https://gitee.com/flywith24/Album/raw/master/img/20200330161411.png)

`LiveData` 是 `observable`



### lifecycle-aware

当观察者观察着某个数据时，该数据必须保留对观察者的引用才能调用它，为了解决这个问题，`LiveData` 被设计成可感知生命周期

![](https://gitee.com/flywith24/Album/raw/master/img/20200330162126.png)



当 activity / fragment 被销毁后，它会自动的取消订阅



### data holder

`LiveData` 仅持有 **单个且最新** 的数据

![data holder](https://gitee.com/flywith24/Album/raw/master/img/20200330214220.gif)



上图中，最右侧是在 `ViewModel` 中的 `LiveData`，左侧为观察这个 `LiveData` 的 activity / fragment 。一旦我们为 `LiveData` 设值，该值会传递到 activity。简而言之，`LiveData` 值改变，activity 收到最新的值的变化。但是当观察者不再处于活动状态（STARTED 到 RESUMED ），数据 C 不会被发送到 activity 。当 activity 回到前台，它将收到最新的值，数据 D。**LiveData 仅持有单个且最新的数据**。当 activity 执行销毁流程时，此时的数据 E 也不会产生任何影响



## Transformations

`LiveData` 提供 两种 transformation ，`map` 和 `switch map`。开发者也可以创建自定义的  `MediatorLiveData` 

我们都知道 `LiveData` 可以为 View 和 ViewModel 提供通信，但如果有一个第三方组件（例如 repository ）也持有 `LiveData`。那么它应该如何在 `ViewModel` 中订阅？该组件并没有 lifecycle 

![](https://gitee.com/flywith24/Album/raw/master/img/20200331090310.png)

一旦我们的应用愈发复杂，repository 可能会观察数据源

![](https://gitee.com/flywith24/Album/raw/master/img/20200331090412.png)

那么 view 如何获取 repository 中的 `LiveData`？



### 一对一的静态转换（map）

![one-to-one static transformation](https://gitee.com/flywith24/Album/raw/master/img/20200331090941.png)

在上面的示例中，`ViewModel` 仅将数据从 repository 转发到 view，然后将其转换为 UI Model。 每当 repository 中有新数据时，`ViewModel` 只需 `map` 

``` kotlin
class MainViewModel {
  val viewModelResult = Transformations.map(repository.getDataForUser()) { data ->
     convertDataToMainUIModel(data)
  }
}
```

第一个参数为 `LiveData` 源（来自 repository ），第二个参数是一个转换函数。

``` kotlin
// 这里的转换为将 X 转换为 Y
inline fun <X, Y> LiveData<X>.map(crossinline transform: (X) -> Y): LiveData<Y> =
        Transformations.map(this) { transform(it) }
```



### 一对一的动态转换（switchMap）

假如您正在观察一个提供用户的用户管理器，并且需要提供用户的 id 才能开始观察 repository 

![](https://gitee.com/flywith24/Album/raw/master/img/20200331091954.png)

您不能将其写到 `ViewModel` 初始化的过程中，因为此时用户的 id 还不可用

这时 `switchMap` 就派上用场了

``` kotlin
class MainViewModel {
  val repositoryResult = Transformations.switchMap(userManager.userId) { userId ->
     repository.getDataForUser(userId)
  }
}
```

`switchMap` 在内部使用 `MediatorLiveData`，因此了解它非常重要，因为当您要组合多个 `LiveData` 源时需要使用它

```kotlin
// 这里的转换为将 X 转换为 LiveData<Y>
inline fun <X, Y> LiveData<X>.switchMap(
    crossinline transform: (X) -> LiveData<Y>
): LiveData<Y> = Transformations.switchMap(this) { transform(it) }
```



### 一对多依赖（MediatorLiveData）

`MediatorLiveData` 允许您将一个或多个数据源添加到单个可观察的 `LiveData` 中

``` kotlin
val liveData1: LiveData<Int> = ...
val liveData2: LiveData<Int> = ...

val result = MediatorLiveData<Int>()

result.addSource(liveData1) { value ->
    result.setValue(value)
}
result.addSource(liveData2) { value ->
    result.setValue(value)
}
```



在上面的例子中，当任何一个数据源变化时，result 会更新。

> 注意：数据并不是合并，MediatorLiveData 只是处理通知



为了实现示例中的转换，我们需要将两个不同的 LiveData 组合为一个

![](https://gitee.com/flywith24/Album/raw/master/img/20200331094815.png)

> 图片来自 [LiveData beyond the ViewModel — Reactive patterns using Transformations and MediatorLiveData](https://medium.com/androiddevelopers/livedata-beyond-the-viewmodel-reactive-patterns-using-transformations-and-mediatorlivedata-fda520ba00b7)



使用 `MediatorLiveData` 合并数据的一种方法是添加源并以其他方法设置值：

``` kotlin
fun blogpostBoilerplateExample(newUser: String): LiveData<UserDataResult> {

    val liveData1 = userOnlineDataSource.getOnlineTime(newUser)
    val liveData2 = userCheckinsDataSource.getCheckins(newUser)

    val result = MediatorLiveData<UserDataResult>()

    result.addSource(liveData1) { value ->
        result.value = combineLatestData(liveData1, liveData2)
    }
    result.addSource(liveData2) { value ->
        result.value = combineLatestData(liveData1, liveData2)
    }
    return result
}
```

数据的实际组合是在 `combineLatestData` 方法中完成的

``` kotlin
private fun combineLatestData(
        onlineTimeResult: LiveData<Long>,
        checkinsResult: LiveData<CheckinsResult>
): UserDataResult {

    val onlineTime = onlineTimeResult.value
    val checkins = checkinsResult.value

    // Don't send a success until we have both results
    if (onlineTime == null || checkins == null) {
        return UserDataLoading()
    }

    // TODO: Check for errors and return UserDataError if any.

    return UserDataSuccess(timeOnline = onlineTime, checkins = checkins)
}
```

检查值是否准备好并发出结果（加载中，失败或成功）



## LiveData 的错误用法

### 错误地使用 var LiveData

``` kotlin
var lateinit randomNumber: LiveData<Int>

fun onGetNumber() {
   randomNumber = Transformations.map(numberGenerator.getNumber()) {
       it
   }
}
```

这里有一个重要的问题需要理解：转换会在调用时（`map` 和 `switchMap`）会创建一个新的 `LiveData`。 在此示例中，randomNumber 公开给 View ，但是每次用户单击按钮时都会对其进行重新赋值。 观察者只会在订阅时收到分配给 var 的 `LiveData` 更新的信息

``` kotlin
// 只会收到第一次分配的值
viewmodel.randomNumber.observe(this, Observer { number ->
    numberTv.text = resources.getString(R.string.random_text, number)
})
```

如果 viewmodel.randomNumber `LiveData` 实例发生更改，这里永远不会回调。而且这里泄漏了之前的 `LiveData` ，这些 `LiveData` 不会再发送更新



![](https://gitee.com/flywith24/Album/raw/master/img/20200331111630.gif)



一言以蔽之，**不要在 var 中使用 Livedata**

正确示例见  [demo](https://github.com/Flywith24/Flywith24-Jetpack-Demo/tree/master/demo_livedata)



### LiveData 粘性事件

一般来说我们使用 LiveData 持有 UI 数据和状态，但是如果通过它来发送事件，可能会出现一些问题。这些问题及解决方案 [在这](https://juejin.im/post/5b2b1b2cf265da5952314b63)



### fragment 中错误地传入 LifecycleOwner

`androidx fragment 1.2.0` 起，添加了新的 Lint 检查，以确保您在从 onCreateView()、onViewCreated() 或 onActivityCreated() 观察 `LiveData` 时使用 getViewLifecycleOwner()



![](https://gitee.com/flywith24/Album/raw/master/img/20200331152505.png)



![bug](https://gitee.com/flywith24/Album/raw/master/img/20200331205402.gif)



如图，我们有一个 fragment ，onCreate 观察 `LiveData`，通过正常的生命周期创建了 View ，接着进入了 resume 状态。此时你使用了 `LiveData`，UI 将开始展示它。之后，用户点击了按钮，由于跳转了另一个 fragment，所以要 detach 该 fragment，一旦 fragment stop 我们就不需要其中的 view 了，因此 destroyView 。之后用户点击了返回按钮回到了上一个 fragment，由于我们已经 destroyView，因此我们需要创建一个新的 view ，接着进入正常的生命周期，但此时，出现了一个 bug 。这个新 View 不会恢复 `LiveData` 的状态，因为我们使用的是 fragment 的 lifecycle observe 的 `LiveData`



我们有两种选择，在 onCreate 或者在 onCreateView 中使用 fragment 的 lifecycle observe LiveData

![](https://gitee.com/flywith24/Album/raw/master/img/20200331171844.png)

前者的优点是一次注册，缺点是当 recreate 时有bug；后者优点是能够解决 recreate 的 bug，但会导致重复注册



**该问题的核心是 fragment 拥有两个生命周期：fragment 自身和 fragment 内部 view 的生命周期**



`androidx fragment 1.0` 和 `support library 28` 了 viewLifecycle



**因此，当需要观察 view 相关的 LiveData ，可以在 onCreateView()、onViewCreated() 或 onActivityCreated()  中 LiveData observe 方法中传入 viewLifecycleOwner 而不是传入 this**



## 源码结构

首先来看 `LiveData` 主要的源码结构

- LiveData 
- MutableLiveData
- Observer

### LiveData

`LiveData` 是可以在给定生命周期内观察到的数据持有者类。 这意味着可以将一个`Observer` 与 `LifecycleOwner` 成对添加，并且只有在配对的 `LifecycleOwner` 处于活动状态时，才会向该观察者通知有关包装数据的修改。 如果 LifecycleOwner 的状态为 `Lifecycle.State.STARTED` 或 `Lifecycle.State.RESUMED`，则将其视为活动状态。 通过 `observeForever`（Observer）添加的观察者被视为始终处于活动状态，因此将始终收到有关修改的通知。 对于那些观察者，需要手动调用 `removeObserver`（Observer）

如果相应的生命周期移至 `Lifecycle.State.DESTROYED` 状态，则添加了生命周期的观察者将被自动删除。 这对于 activity 和 fragment 可以安全地观察 `LiveData` 而不用担心泄漏

此外，`LiveData` 具有 onActive() 和 onInactive() 方法，以便在活动观察者的数量在 0 到 1 之间变化时得到通知。这使 `LiveData` 在没有任何活动观察者的情况下可以释放大量资源。

主要方法有：

- T getValue() 获取LiveData 包装的数据
- observe(LifecycleOwner owner, Observer<? super T> observer) 设置观察者（主线程调用）
- setValue(T value)  设值（主线程调用），可见性为 protected 无法直接使用
- postValue(T value) 设置（其他线程调用），可见性为 protected 无法直接使用



### MutableLiveData

`LiveData` 实现类，公开了 `setValue` 和 `postValue` 方法

### Observer

接口，内部只有 onChanged(T t) 方法，在数据变化时该方法会被调用



## 源码分析

我们通过源码来看看 `LiveData` 如何实现它的特性的

- 1. 如何控制在 activity 或 fragment 活动状态时接收回调，否则不接收？
- 2. 如何在 activity 或 fragment 销毁时自动取消注册观察者？
- 3. 如何保证 `LiveData` 持有最新的数据？



我们查看 `LiveData` 的 observe 方法

```java
// LiveData.java
@MainThread
public void observe(LifecycleOwner owner, Observer<? super T> observer) {
    if (owner.getLifecycle().getCurrentState() == DESTROYED) {
        // 如果 owner 已经是 DESTROYED 状态，则忽略
        return;
    }
    // 使用 LifecycleBoundObserver 包装 owner 和 observer
    LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
    ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
    // 如果已经添加过直接 return 
    if (existing != null) {
        return;
    }
   
    owner.getLifecycle().addObserver(wrapper);
}

// LifecycleBoundObserver.java
class LifecycleBoundObserver extends ObserverWrapper implements LifecycleEventObserver {
    @NonNull
    final LifecycleOwner mOwner;
    LifecycleBoundObserver(LifecycleOwner owner, Observer<? super T> observer) {
        super(observer);
        mOwner = owner;
    }
}
```



通过源码我们知道，当我们调用 observe 方法时，内部是通过 `LifecycleBoundObserver` 将 owner 和 observer 包裹起来并通过 `addObserver` 方法添加观察者的，因而当数据变化时，会调用 `LifecycleBoundObserver` 的 `onStateChanged` 方法

```java
// LiveData.LifecycleBoundObserver.java
@Override
public void onStateChanged(@NonNull LifecycleOwner source,
        @NonNull Lifecycle.Event event) {
    if (mOwner.getLifecycle().getCurrentState() == DESTROYED) {
        // 自动移除观察者，问题 2 得到解释
        removeObserver(mObserver);
        return;
    }
    activeStateChanged(shouldBeActive());
}
```

当什么周期所有者处于 `DESTROYED` 状态时，会调用 `removeObserver` 方法，因此问题 2  得到解释



我们继续向下看，`activeStateChanged` 方法调用时传入了 `shouldBeActive()`

```java
@Override
boolean shouldBeActive() {
    // 至少是 STARTED 状态 返回 true
    return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
}

void activeStateChanged(boolean newActive) {
    if (newActive == mActive) {
        // 与上次值相同，则直接 return （两次均为活动状态或均为非活动状态）
        return;
    }
    mActive = newActive;
    boolean wasInactive = LiveData.this.mActiveCount == 0;
    // 根据 mActive 修改活动状态观察者的数量（加 1 或减 1 ）
    LiveData.this.mActiveCount += mActive ? 1 : -1;
    if (wasInactive && mActive) {
        onActive();
    }
    if (LiveData.this.mActiveCount == 0 && !mActive) {
        onInactive();
    }
    if (mActive) {
        // 如果是活动状态，则发送数据，问题 1 得到解释
        dispatchingValue(this);
    }
}
```

这里牵扯了 `Lifecycle State` 比较的知识，[详情在这](https://juejin.im/post/5e8348bef265da47e02a6ce2#heading-11)

只有 `STARTED` 和 `RESUMED` 状态 `shouldBeActive()` 才返回 true，至此问题 1 得到解释

`dispatchingValue` 方法内部调用了 `considerNotify` 方法

```java
private void considerNotify(ObserverWrapper observer) {
    if (!observer.mActive) {
        return;
    }
    // 再次判断生命周期所有者状态
    if (!observer.shouldBeActive()) {
        observer.activeStateChanged(false);
        return;
    }
    // 比较版本号
    if (observer.mLastVersion >= mVersion) {
        return;
    }
    observer.mLastVersion = mVersion;
    // 调用我们传入的 mObserver 的 onChanged 方法
    observer.mObserver.onChanged((T) mData);
}
```



可以看到 `considerNotify` 中比较了 observer 的版本号，如果是最新的数据，直接 return

而 `mVersion` 在 `setValue` 方法中 进行更改

```java
@MainThread
protected void setValue(T value) {
    // 每次设置对 mVersion 进行++
    mVersion++;
    mData = value;
    dispatchingValue(null);
}
```

因此 `LiveData` 每次都持有最新的数据，问题 3 得到解释



## 总结

回到本文开头的思考，Android 开发者的主要工作是将数据转换成 UI ，而 `LiveData` 本质上是一种「数据驱动」，即通过改变状态数据，来驱动视图树中绑定了相应状态数据的控件重新发生绘制。Flutter 和未来的  Jetpack Compose 采用的都是这种机制。使用 ViewModel + LiveData，可以 **安全地在订阅者的生命周期内分发正确的数据**，使开发者不知不觉中完成了 `UI -> ViewModel -> Data` 的单向依赖。



**所谓架构，很多时候不是使用它能做什么，更多的是不要做什么，使用它时开发者能够得到约束，以便产出更健壮的代码**



各位小伙伴如果有什么想法欢迎在评论区留言



---
## 关于我
---
我是 [Fly_with24](https://flywith24.gitee.io/)
- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)
- [简书](https://www.jianshu.com/u/3d5ad6043d66)
- [Github](https://github.com/Flywith24)

