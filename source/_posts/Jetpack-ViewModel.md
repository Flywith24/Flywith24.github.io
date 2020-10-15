---
title: 即使您不使用MVVM也要了解ViewModel ViewModel的职能边界
date: 2020-03-23 00:10:35
categories: 
- 背上 Jetpack
tags: 
- androidx
- Jetpack
- MVVM
image: https://gitee.com/flywith24/Album/raw/master/img/20201015100650.png
---

Jetpack ViewModel 的职能边界。


<!-- more-->

![目录](https://gitee.com/flywith24/Album/raw/master/img/20200324083031.png)

## 前言



> Android 开发时，我们使用 activity 和 fragment 作为视图控制器， 可能还会使用有一些类可以存储和提供 UI 数据（例如MVP中的 `Presenter` ）



但是 当配置更改时（如旋转屏幕），activity 会重建，但对于 UI 数据的持有者呢？

- 开发者需要重新保存相关的信息并传递给重建的 activity ，否则开发者必须再次获取数据（通过网络请求或本地数据库）
- 由于 UI 数据的持有者的生命周期可能比 activity 长，因此开发者还需要避免出现内存泄漏的问题



如何解决上述问题？ViewModel

**本文重点介绍 ViewModel 的职责（what）以及重点功能的实现原理（how），即使您不使用 `Jetpack MVVM` 架构，也要了解一下 ViewModel**

ViewModel 的原理部分要求您了解 activity 的启动流程，这部分内容网上文章很多，本文不再赘述



<!-- more-->



## ViewModel 的职责

我先上个 [视频](https://www.bilibili.com/video/av97794796/) ，这个小姐姐表述的比文字更形象

[![](https://gitee.com/flywith24/Album/raw/master/img/20200321135602.png)](https://www.bilibili.com/video/av97794796/)



`ViewModel` 主要用于存储 UI 数据以及生命周期感知的数据

![](https://gitee.com/flywith24/Album/raw/master/img/20200321140300.gif)

> 图片来自 [Android Architecture Components: ViewModel](https://android.jlelse.eu/android-architecture-components-viewmodel-e74faddf5b94)

![ViewModel生命周期](https://gitee.com/flywith24/Album/raw/master/img/20200321140641.png)

> `ViewModel` 的生命周期 ，图片来自 [官方文档](https://developer.android.com/topic/libraries/architecture/viewmodel#lifecycle)



### 作为数据持有者

`ViewModel` 能够实时进行配置更改。 这意味着即使在手机旋转后销毁并重新创建 activity 之后，您仍然拥有相同的 `ViewModel` 和相同的数据。 因此：

- 您无需担心 UI 数据持有者的生命周期。 `ViewModel` 将由工厂自动创建，您无需自行创建和销毁
- 数据将始终更新，旋转手机后，您将获得与以前相同的数据。 因此，您无需手动将数据传递给新的 activity 实例或再次调用网络或数据库来获取数据。 



### Fragment 间共享数据

一个 activity 中的两个或更多 fragment 需要相互通信是很常见的。例如您有一个片段，用户在其中从列表中选择一个 item，另一个片段显示了所选 item 的内容。 传统做法两个 fragment 都需要定义一些接口，并且宿主 activity 必须将两者绑定在一起。 此外，两个 fragment 都必须处理另一个 fragment 尚未创建或不可见的情况。



可以通过使用 `ViewModel` 对象解决此问题。 这些 fragment 可以使用 activity 范围内共享一个 `ViewModel` 来处理此通信，如以下示例代码所示：



``` java
public class SharedViewModel extends ViewModel {
    private final MutableLiveData<Item> selected = new MutableLiveData<Item>();

    public void select(Item item) {
        selected.setValue(item);
    }

    public LiveData<Item> getSelected() {
        return selected;
    }
}


public class MasterFragment extends Fragment {
    private SharedViewModel model;

    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        model = new ViewModelProvider(requireActivity()).get(SharedViewModel.class);
        itemSelector.setOnClickListener(item -> {
            model.select(item);
        });
    }
}

public class DetailFragment extends Fragment {

    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        SharedViewModel model = new ViewModelProvider(requireActivity()).get(SharedViewModel.class);
        model.getSelected().observe(getViewLifecycleOwner(), { item ->
           // Update the UI.
        });
    }
}
```

> 由于 两个 fragment 使用的都是 activity 范围的 `ViewModel` （`ViewModelProvider` 构造器传入的 activity ），因此它们获得了相同的 ViewModel 实例，自然其持有的数据也是相同的，这也 **保证了数据的一致性**



这种方法具有以下优点：



- 宿主 activity 无需执行任何操作，也无需了解此通信。

- 除 `SharedViewModel` 外，fragment 不需要彼此了解。 如果其中一个 fragment 消失了，则另一个继续照常工作。

- 每个 fragment 都有其自己的生命周期，并且不受另一个 fragment 的生命周期影响。 如果一个 fragment 替换了另一个 fragment，则 UI 可以继续正常工作而不会出现任何问题。



### 代替 Loader

`CursorLoader` 这样的 Loader 类经常用于使应用程序 UI 中的数据与数据库保持同步。您可以使用 `ViewModel` 和其他一些类来替换 Loader。 使用 `ViewModel` 可将视图控制器与数据加载操作分开，这意味着您在类之间的强引用较少。

在使用 Loader 的一种常见方法中，应用程序可能会使用 `CursorLoader` 来观察数据库的内容。 当数据库中的值更改时，加载程序会自动触发数据的重新加载并更新 UI

![](https://gitee.com/flywith24/Album/raw/master/img/20200321144832.png)

> 图片来自 [官方文档](https://developer.android.com/topic/libraries/architecture/viewmodel#loaders)



`ViewModel` 与 `Room` 和 `LiveData` 一起使用以替换 Loader。 `ViewModel` 确保数据在设备配置更改后仍然存在。 当数据库发生更改时，`Room` 会通知 `LiveData` ，然后 `LiveData` 会使用修改后的数据更新 UI

![](https://gitee.com/flywith24/Album/raw/master/img/20200321144949.png)

> 图片来自 [官方文档](https://developer.android.com/topic/libraries/architecture/viewmodel#loaders)



### 总结

- **ViewModel 可作为 UI 数据的持有者，在 activity/fragment 重建时 ViewModel 中的数据不受影响，同时可以避免内存泄漏**
- **可以通过 ViewModel 来进行 activity 和 fragment ，fragment 和 fragment 之间的通信，无需关心通信的对方是否存在，使用 application 范围的 ViewModel 可以进行全局通信**
- **可以代替 Loader**



## ViewModel 源码分析

分析源码时我们可以不计较细枝末节，只分析主要的逻辑即可。因此我们来思考几个问题，并从源码中寻找答案

- 如何做到 activity 重建后 `ViewModel` 仍然存在？

- 如何做到 fragment 重建后 `ViewModel` 仍然存在？

- 如何控制作用域？（即保证相同作用域获取的 `ViewModel` 实例相同）

- 如何避免内存泄漏？

  

维持我们一贯的风格，我们先来大胆地猜一猜

对于问题1 ：activity 有着 `saveInstanceState` 机制，因此可能通过该机制来处理（**事实证明不是**）

对于问题2：可能 fragment 通过 宿主 activity 或 父 fragment 的帮助来确保 `ViewModel` 实例在重建后仍然存在

对于问题3：实现一个类似单例的效果，相同作用域获取的对象是相同的

对于问题4：避免 `ViewModel` 持有 view 或 context 的引用



首先我们要先了解一下 `ViewModel` 的结构

- `ViewModel`：抽象类，主要有 clear 方法，它是 final 级，不可修改，clear 方法中包含 onClear 钩子，开发者可重写 onClear 方法来自定义数据的清空

- `ViewModelStore`：内部维护一个 HashMap 以管理 `ViewModel`

- `ViewModelStoreOwner`：接口，`ViewModelStore` 的作用域，实现类为 `ComponentActivity` 和 `Fragment`，此外还有 `FragmentActivity.HostCallbacks`

- `ViewModelProvider`：用于创建 `ViewModel`，其构造方法有两个参数，第一个参数传入 `ViewModelStoreOwner` ，确定了 `ViewModelStore` 的作用域，第二个参数为 `ViewModelProvider.Factory`，用于初始化 `ViewModel` 对象，默认为 `getDefaultViewModelProviderFactory()` 方法获取的 factory

  

简单来说 **ViewModelStoreOwner 持有 ViewModelStore 持有 ViewModel**

![](https://gitee.com/flywith24/Album/raw/master/img/20200321152406.png)



### 1. 如何做到 activity 重建后 ViewModel 仍然存在？



在 [【背上Jetpack】绝不丢失的状态 androidx SaveState ViewModel-SaveState 分析](https://juejin.im/post/5e738d12518825495d69cfb9) 中我们提到了 androidx.core.app.ComponentActivity 的引入并探讨了其作为中间层的作用

![](https://gitee.com/flywith24/Album/raw/master/img/20200321155215.png)

我们已经讲过 `SavedStateRegistryOwner` 和 `OnBackPressedDispatcherOwner` 这两种角色，而今天我们来聊一下

`ViewModelStoreOwner` 和 `HasDefaultViewModelProviderFactory` 。其中前者代表着 `ViewModelStore` 的作用域，后者来标记 `ViewModelStoreOwner` 拥有默认的 `ViewModelProvider.Factory`



那么 `ViewModel` 的逻辑肯定就在该类了

`ComponentActivity` 实现了 `ViewModelStoreOwner`  接口，意味着需要重写 `getViewModelStore()` 方法，该方法为 `ComponentActivity`  的 `mViewModelStore` 变量赋值。**activity 重建后 ViewModel 仍然存在，只要保证 activity 重建后 mViewModelStore 变量值不变即可**



顺着这个思路，我们来看一下 `getViewModelStore()` 的实现

```java
public ViewModelStore getViewModelStore() {
    if (mViewModelStore == null) {
        NonConfigurationInstances nc =
                (NonConfigurationInstances) getLastNonConfigurationInstance();
        if (nc != null) {
            //核心，在该位置重置 mViewModelStore
            mViewModelStore = nc.viewModelStore;
        }
        if (mViewModelStore == null) {
            mViewModelStore = new ViewModelStore();
        }
    }
    return mViewModelStore;
}
```



即 `mViewModelStore` 的值由 `getLastNonConfigurationInstance()` 返回的 `NonConfigurationInstances` 对象中的 `viewModelStore` 赋值，如果此时还为空才去 new ViewModelStore 对象。因此我们只需找到 

`getLastNonConfigurationInstance` 中的 `NonConfigurationInstances` 在哪里保存的即可



`getLastNonConfigurationInstance` 为平台 activity 中的方法，返回 `mLastNonConfigurationInstances.activity`

``` java
public Object getLastNonConfigurationInstance() {
    return mLastNonConfigurationInstances != null
            ? mLastNonConfigurationInstances.activity : null;
}
```

那么我们看一下 `mLastNonConfigurationInstances` 的赋值位置

``` java
//省略其他参数
final void attach(NonConfigurationInstances lastNonConfigurationInstances){
	mLastNonConfigurationInstances = lastNonConfigurationInstances;
    //...
}
```



了解过 activity 的启动流程的小伙伴肯定知道，这个 attach 方法是 `ActivityThread` 中的 `performLaunchActivity` 调用的

```java
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    Activity activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent);
    //省略其他参数
    activity.attach(r.lastNonConfigurationInstances);
    r.lastNonConfigurationInstances = null;
    //...
}
```

深入追踪源码我们整理一下调用流程

![](https://gitee.com/flywith24/Album/raw/master/img/20200323114334.png)



由于 `ActivityThread` 中的 `ActivityClientRecord` 不受 activity 重建的影响，所以 activity 重建时 `mLastNonConfigurationInstances` 能够得到上一次的值，使得 `ViewModelStore` 值不变 ，问题1就解决了



### 2. 如何做到 fragment 重建后 ViewModel 仍然存在？

对于问题2，有了上面的思路我们可以认定 fragment 重建后其内部的 `getViewModelStore()` 方法返回的对象是相同的。

``` java
// Fragment.java
public ViewModelStore getViewModelStore() {
    return mFragmentManager.getViewModelStore(this);
}
```



可以看到 `getViewModelStore()` 内部调用的是 `mFragmentManager`（普通fragment 对应 activity 中的 `FragmentManager`，子 fragment 则对应父 fragment 的 `childFragmentManager`）的 `getViewModelStore()` 方法

```java
// FragmentManager.java
private FragmentManagerViewModel mNonConfig;

ViewModelStore getViewModelStore(@NonNull Fragment f) {
    return mNonConfig.getViewModelStore(f);
}
```



**而 FragmentManager 中的 getViewModelStore 使用的是 mNonConfig ，mNonConfig 竟然是个 ViewModel！**



```java
// FragmentManagerViewModel.java
private final HashMap<String, FragmentManagerViewModel> mChildNonConfigs = new HashMap<>();
private final HashMap<String, ViewModelStore> mViewModelStores = new HashMap<>();
```



`FragmentManagerViewModel` 管理着内部的 `ViewModelStore` 和 child 的 `FragmentManagerViewModel` 。因此保证 mNonConfig   值不变即能确保 fragment 中的 `getViewModelStore()`  不变。那么看看 mNonConfig  赋值的位置

```java
// FragmentManager.java
void attachController(@NonNull FragmentHostCallback<?> host, @NonNull FragmentContainer container, @Nullable final Fragment parent) {
    //...
    if (parent != null) {
        // 嵌套 fragment 的情况，有父 fragment
        mNonConfig = parent.mFragmentManager.getChildNonConfig(parent);
    } else if (host instanceof ViewModelStoreOwner) {
        // host 是 FragmentActivity.HostCallbacks
        ViewModelStore viewModelStore = ((ViewModelStoreOwner) host).getViewModelStore();
        mNonConfig = FragmentManagerViewModel.getInstance(viewModelStore);
    } else {
        mNonConfig = new FragmentManagerViewModel(false);
    }
}


// FragmentManagerViewModel.java
static FragmentManagerViewModel getInstance(ViewModelStore viewModelStore) {
    ViewModelProvider viewModelProvider = new ViewModelProvider(viewModelStore,
            FACTORY);
    return viewModelProvider.get(FragmentManagerViewModel.class);
}
```



我们先看 fragment 的直接宿主是 activity （即没有嵌套）的情况，mNonConfig 由`FragmentManagerViewModel.getInstance(viewModelStore)` 赋值，而 getInstance 中使用的是 `ViewModelProvider` 获取 `ViewModel` ，根据我们上面的分析，**只要保证作用域（viewModelStore）相同，即可获取相同的 `ViewModel` 实例**，因此我们需要看一下 host 的 getViewModelStore 方法。经过一番寻找，host 是 `FragmentActivity.HostCallbacks` 



```java
// FragmentActivity.java 内部类
class HostCallbacks extends FragmentHostCallback<FragmentActivity> implements ViewModelStoreOwner, OnBackPressedDispatcherOwner {
    public ViewModelStore getViewModelStore() {
        // 宿主 activity 的 getViewModelStore
    	return FragmentActivity.this.getViewModelStore();
	}
}
```



host 的 getViewModelStore 方法返回的是宿主 activity 的 `getViewModelStore()` ，而 activity 重建后其内部的 `mViewModelStore` 是不变的，因此即使 activity 重建，其内部的 FragmentManager 对象变化，但 FragmentManager 内部的  FragmentManagerViewModel 的实例（`mNonConfig`）不变，mNonConfig.getViewModelStore 不变，fragment 的 `getViewModelStore()` 亦不变，fragment 重建后其内部的 `ViewModel` 仍然存在

对于嵌套 fragment ，mNonConfig 通过 parent.mFragmentManager.getChildNonConfig(parent) 获取

```java
// FragmentManager.java
private FragmentManagerViewModel getChildNonConfig(@NonNull Fragment f) {
    return mNonConfig.getChildNonConfig(f);
}
```

上文提到 `FragmentManagerViewModel` 管理着 mChildNonConfigs Map，因此子 fragment 重置后其内部的 mNonConfig 对象也是相同的

至此问题 2 就解决了



### 3. 如何控制作用域？

对于问题3，我们知道 `ViewModelStoreOwner` 代表着作用域，其内部唯一的方法返回 `ViewModelStore` 对象，也即不同的作用域对应不同的 `ViewModelStore` ，而 `ViewModelStore`  内部维护着 `ViewModel` 的 HashMap ，因此只要保证相同作用域的 `ViewModelStore` 对象相同就能保证相同作用域获取到相同的 `ViewModel` 对象，而问题1我们已经解释了重建时如何保证 `ViewModelStore`  对象不变。

因此问题3也解决了。



### 4. 如何避免内存泄漏？

对于问题4，由于 `ViewModel` 的设计，使得 activity/fragment 依赖它，而 `ViewModel` 不依赖视图控制器。因此只要不让 `ViewModel` 持有 context 或 view 的引用，就不会造成内存泄漏



### 总结

简单的总结一下：

- **activity 重建后 mViewModelStore 通过 ActivityThread 的一系列方法能够保持不变，从而当 activity 重建时 ViewModel 中的数据不受影响**

- **通过宿主 activity 范围内共享的 FragmentManagerViewModel 来存储 fragment 的 ViewModelStore 和子fragment 的 FragmentManagerViewModel ，而 activity 重建后 FragmentManagerViewModel  中的数据不受影响，因此 fragment 内部的 ViewModel 的数据也不受影响**

- **通过同一 ViewModelStoreOwner 获取的 ViewModelStore 相同，从而保证同一作用域通过 ViewModelProvider 获取的ViewModel 对象是相同的**

- **通过单向依赖（视图控制器持有 ViewModel ）来解决内存泄漏的问题**

  

## ViewModel 和 onSaveInstanceState

`ViewModel` 和 `onSaveInstanceState` 的功能有些类似，但它们也有很多差异

![](https://gitee.com/flywith24/Album/raw/master/img/20200323160417.png)

从存储位置上来说，`ViewModel` 是在内存中，因此其读写速度更快，但当进程被系统杀死后，`ViewModel` 中的数据也不存在了。从数据存储的类型上来看，`ViewModel` 适合存储相对较重的数据，例如网络请求到的 list 数据，而 `onSaveInstanceState` 适合存储轻量可序列化的数据

那么我们该如何使用呢？可以使用 `viewmodel-savedstate` 库，详情参考 [【背上Jetpack】绝不丢失的状态 androidx SaveState ViewModel-SaveState 分析](https://juejin.im/post/5e738d12518825495d69cfb9#heading-10)



---
## 关于我
---
我是 [Fly_with24](https://flywith24.gitee.io/)
- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)
- [简书](https://www.jianshu.com/u/3d5ad6043d66)
- [Github](https://github.com/Flywith24)

