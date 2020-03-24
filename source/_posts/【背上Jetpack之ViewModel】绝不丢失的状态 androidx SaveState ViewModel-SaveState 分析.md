---
title: 【背上Jetpack之ViewModel】绝不丢失的状态 androidx SaveState ViewModel-SaveState 分析
date: 2020-03-19 00:10:35
categories: 
- Jetpack
tags: 
- androidx
- Jetpack
- MVVM

---

## 前言

> 大家都知道 activity 有着一套 `onSaveInstanceState-onRestoreInstanceState` 状态保存机制，旨在「系统资源回收」或「配置发生变化」保存状态，为用户提供更好的体验
>
> 在 androidx 下，提供了 [SavedState](https://developer.android.com/jetpack/androidx/releases/savedstate) 库帮助 activity 和 fragment 处理状态保存和恢复



本文默认您对状态保存机制有一定了解，这部分内容请移步 [Saving UI States](https://developer.android.com/topic/libraries/architecture/saving-states)

此外，关于 android 下的进程管理，推荐 Ian Lake 的 [Who lives and who dies? Process priorities on Android](https://medium.com/androiddevelopers/who-lives-and-who-dies-process-priorities-on-android-cb151f39044f)



本文介绍了 androidx 下 `SavedState` 如何帮助 activity 和 fragment 处理状态的保存和恢复，同时介绍 `viewmodel-savedstate` 库，以及在开发过程中正确使用状态保存的姿势

<!-- more-->

## 软件工程中没有什么是中间层解决不了的



在分析 [SavedState](https://developer.android.com/jetpack/androidx/releases/savedstate) 库之前我们需要简单聊一聊 `ComponentActivity`




androidx activity 1.0.0 时，`ComponentActivity` 成为了 `FragmentActivity` 和 `AppCompatActivity` 的基类。

![androidx activity 1.0.0 ](https://gitee.com/flywith24/Album/raw/master/img/20200318211230.png)



俗话说「百因必有果」，带着强烈的好奇心，我查了一下 ComponentActivity 引入的原因。

![](https://gitee.com/flywith24/Album/raw/master/img/20200318211823.png)

![](https://gitee.com/flywith24/Album/raw/master/img/20200318211806.png)



可以看到 `ComponentActivity` 继承了 androidx.core.app.ComponentActivity(在fragment库中)，并且最初仅实现了`LifecycleOwner` 接口



我们创建的 activity 的继承关系现在变成了这样：

![](https://gitee.com/flywith24/Album/raw/master/img/20200318213053.png)



那么回到最初的问题，为什么要引入 `ComponentActivity` ？其实看看现在 `ComponentActivity` 的类结构答案就很清楚了



![](https://gitee.com/flywith24/Album/raw/master/img/20200318213151.png)



`ComponentActivity` 实现了五个接口，代表着其除了 activity 还充当着五种角色。本着职能单一原则，官方通过建立一个中间层将部分功能分别交于专门的类来负责，OnBackPressedDispatcherOwner 就是我们讲 fragment 返回栈（[【背上Jetpack之OnBackPressedDispatcher】Fragment 返回栈预备篇](https://juejin.im/post/5e6bae35f265da572a0d11ad)）时提到的结构，而其中的 `SavedStateRegistryOwner` 则是我们今天要讲的主角  [SavedState](https://developer.android.com/jetpack/androidx/releases/savedstate) 中的成员



## SavedState

引入 [SavedState](https://developer.android.com/jetpack/androidx/releases/savedstate) 

```groovy
implementation "androidx.savedstate:savedstate:1.0.0"
```



其实您不需要显示地声明，因为 activity 库内部已经引入了。jetpack 组件依赖关系可参考 [【背上Jetpack】Jetpack 主要组件的依赖及传递关系](https://juejin.im/post/5e567ee1518825494466a938)



这是一个很小的库

![](https://gitee.com/flywith24/Album/raw/master/img/20200318215718.png)

![](https://gitee.com/flywith24/Album/raw/master/img/20200318215746.png)

> 图片来自 [Android ViewModels: State persistence — SavedState](https://proandroiddev.com/viewmodels-state-persistence-savedstate-54d015acad82)



###  SavedStateProvider

保存状态的组件，此状态将在以后恢复并使用

``` java
public interface SavedStateProvider {
    @NonNull
    Bundle saveState();
}
```



### SavedStateRegistry

管理 `SavedStateProvider` 列表的组件，此注册表绑定了其所有者的生命周期（即 activity 或 fragment）。每次创建生命周期所有者都会创建一个新的实例

创建注册表的所有者后（例如，在调用 activity 的 `onCreate(savedInstanceState)` 方法之后），将调用其 `performRestore(state)` 方法，以恢复系统杀死其所有者之前保存的任何状态。

``` java
void performRestore(@NonNull Lifecycle lifecycle, @Nullable Bundle savedState) {
    // ...
    if (savedState != null) {
        mRestoredState = savedState.getBundle(SAVED_COMPONENTS_KEY);
    }
    // ...
}
```



每个注册表的 `SavedStateProvider` 都由用于注册它的唯一密钥标识

``` java
private SafeIterableMap<String, SavedStateProvider> mComponents = new SafeIterableMap<>();

public void registerSavedStateProvider(@NonNull String key, @NonNull SavedStateProvider provider) {
    SavedStateProvider previous = mComponents.putIfAbsent(key, provider);
    if (previous != null) {
        throw new IllegalArgumentException("SavedStateProvider with the given key is already registered");
    }
}

public void unregisterSavedStateProvider(@NonNull String key) {
    mComponents.remove(key);
}
```



一旦完成注册，就可以通过`consumeRestoredStateForKey(key)` 来使用特定密钥的还原状态

``` java
public Bundle consumeRestoredStateForKey(@NonNull String key) {
    if (mRestoredState != null) {
        Bundle result = mRestoredState.getBundle(key);
        //调用后就会清空，第二次调用返回null
        mRestoredState.remove(key);
        if (mRestoredState.isEmpty()) {
            mRestoredState = null;
        }
        return result;
    }
    return null;
}
```



> 请注意，此方法检索保存的状态，然后清除其内部引用，这意味着用相同的键调用它两次将在第二次调用中返回 null
>
> 一旦注册表恢复了其保存状态，则由提供者决定是否要求其恢复的数据。 如果没有，下次注册表的所有者被系统杀死时，未使用的还原数据将再次保存到保存状态



已注册的 provider 能够在其所有者被系统杀死之前保存状态。 发生这种情况时，将调用其 `Bundle saveState()` 方法。 对于每个已注册的 `SavedStateProvider`，都可以像这样保存状态。



``` java
savedState.putBundle(savedStateProviderKey, savedStateProvider.saveState());
```



`performSave(outBundle)` 方法的源码如下



``` java
void performSave(@NonNull Bundle outBundle) {
    Bundle components = new Bundle();
    
    // 1.保存未使用的状态
    if (mRestoredState != null) {
        components.putAll(mRestoredState);
    }
    
    // 2. 通过 SavedStateProvider 保存状态
    for (Iterator<Map.Entry<String, SavedStateProvider>> it = mComponents.iteratorWithAdditions(); it.hasNext(); ) {
        Map.Entry<String, SavedStateProvider> entry1 = it.next();
        components.putBundle(entry1.getKey(), entry1.getValue().saveState());
    }
    
    // 3. 将bundle 保存到 outBundle 对象中
    outBundle.putBundle(SAVED_COMPONENTS_KEY, components);
}
```



执行状态保存将所有未使用的状态与注册表提供的状态合并。 此 outBundle 是 activity 的 `onSaveInstanceState` 中传入的 bundle 。



### SavedStateRegistryController

一个包装 `SavedStateRegistry` 并允许通过其2个主要方法对其进行控制的组件：performRestore(savedState) 和 `performSave(outBundle )`。 这两个方法将内部通过 `SavedStateRegistry` 中的方法处理 。



``` java
public final class SavedStateRegistryController {
    private final SavedStateRegistryOwner mOwner;
    private final SavedStateRegistry mRegistry;

    public void performRestore(@Nullable Bundle savedState) {
        // ...
        mRegistry.performRestore(lifecycle, savedState);
    }

    public void performSave(@NonNull Bundle outBundle) {
        mRegistry.performSave(outBundle);
    }
}
```




### SavedStateRegistryOwner

持有 `SavedStateRegistry` 的组件。 默认情况下，androidx 包中的`ComponentActivity` 和 `Fragment` 都实现此接口。

``` java
public interface SavedStateRegistryOwner extends LifecycleOwner {
    @NonNull
    SavedStateRegistry getSavedStateRegistry();
}
```



## Activity 的状态保存

这里我们要明确一件事情，activity 保存的状态究竟都有什么？

这部分内容可以参见 [官方文档](https://developer.android.com/guide/components/activities/activity-lifecycle.html#saras) 

简单来说，**activity 的状态保存分为 view 状态和成员状态**

默认情况下，系统使用 Bundle 实例状态来保存有关 activity 布局中每个 View 对象的信息（例如，输入到 EditText 中的文本值或 recyclerview 的滚动位置）。 因此，如果 activity 实例被销毁并重新创建，则布局状态将恢复为之前的状态，而无需您执行任何代码。（**注意，需要恢复状态的 view 需要配置 id** ）

这部分逻辑在 activity 中的 `onSaveInstanceState` 方法内实现



![onSaveInstanceState ](https://gitee.com/flywith24/Album/raw/master/img/20200319115543.png)




> 不同平台 `onSaveInstanceState`  方法的执行时机稍有不同，android P 之前 `onSaveInstanceState` 执行在 `onStop` 之前，但不限于在 `onPause` 之前或之后。android P 及之后该方法在 `onStop` 后执行



前面我们提到 `ComponentActivity` 实现了 `SavedStateRegistryOwner` ，下面我们来看一看 activity 如何利用该库来实现状态的保存与恢复



``` java
public class ComponentActivity extends androidx.core.app.ComponentActivity implements SavedStateRegistryOwner {

    private final SavedStateRegistryController mSavedStateRegistryController = SavedStateRegistryController.create(this);
  
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mSavedStateRegistryController.performRestore(savedInstanceState);
        // ...
    }
  
    @Override
    protected void onSaveInstanceState(@NonNull Bundle outState) {
        // ...
        //这里先调用父类的 onSaveInstanceState 保存 view 状态
        super.onSaveInstanceState(outState);
        mSavedStateRegistryController.performSave(outState);
    }
  
    @NonNull
    @Override
    public final SavedStateRegistry getSavedStateRegistry() {
        return mSavedStateRegistryController.getSavedStateRegistry();
    }
}
```



其内部持有 `SavedStateRegistryController` 的实例 `mSavedStateRegistryController` ，在 activity 的 `onCreate` 方法中 通过 controller 的 `performRestore` 方法来查询已保存的状态，在 `onSaveInstanceState` 中 使用 controller  的 `performSave` 方法来保存状态



**除了 view 状态和成员状态，activity 还负责保存其内部的 fragment 的状态**。`FragmentActivity` 的 `onSaveInstanceState` 方法有对其内部 fragment 的状态进行保存，并在 onCreate 方法中对已保存的 fragment 进行恢复。这解释了如果操作不当会导致 fragment 重叠的问题



![](https://gitee.com/flywith24/Album/raw/master/img/20200319140343.png)



![](https://gitee.com/flywith24/Album/raw/master/img/20200319140801.png)



## Fragment 的状态保存



androidx fragment 使用 `FragmentStateManager` 来处理 fragment 的状态保存

其内部有四个保存相关的方法

- `saveState`
- `saveBasicState`
- `saveViewState`
- `saveInstanceState`

![FragmentStateManager](https://gitee.com/flywith24/Album/raw/master/img/20200319142729.png)



其调用链为 activity 通过 `FragmentController` 间接 调用 `FragmentManager` 的 `saveAllState`，接着依次调用后面的save 方法



Fragment 的状态保存可分为 view 状态，成员状态，child fragment 状态

关于 view 状态 , `FragmentStateManager` 提供了 `saveViewSate` 方法，它的调用有两处：

1. 在 activity 或父 fragment 触发状态保存时调用，即上述流程
2. 在 fragment 即将进入 `onDestroyView` 生命周期时调用，其位置在 `FragmentManager` moveToState 方法内部，这解释了为什么加入返回栈的 replace 操作在返回时 view 状态可以自动恢复

关于成员状态，由 activity 中的状态机制处理，即上节内容

关于 child  fragment 状态，fragment 的 `onCreate` 方法会调用 `restoreChildFragmentState` 来恢复 child  fragment 的状态，并在 `FragmentStateManager`  中的 `saveBasicState` 方法中 调用 `performSaveInstanceState` 来保存 child  fragment 的状态



## Viewmodel-SavedState



2020-01-22，`ViewModel-SavedState 1.0.0` 正式版发布，02-05 发布了 `2.2.0` 正式版



```groovy
 implementation "androidx.lifecycle:lifecycle-viewmodel-savedstate:2.2.0"
```

> 您不需要手动引入该库，因为 fragment 库以及内部引入该库



`Jetpack MVVM` 下 UI State 通常被 `ViewModel` 持有并存储，因此该模块出现了，配置该模块后，`ViewModel` 对象将通过其构造函数接收 `SavedStateHandle` 对象（键值映射），可让您保存状态并查询已保存的状态。 这些值将在系统终止进程后继续存在，并可以通过同一对象使用。



![ViewModel-SavedState](https://gitee.com/flywith24/Album/raw/master/img/20200319165450.png)

> 图片来自 [Android ViewModels: State persistence — SavedState](https://proandroiddev.com/viewmodels-state-persistence-savedstate-54d015acad82)



### SavedStateHandle

内部持有已保存状态 key-value 的 map，允许读取和写入状态，这些状态在应用进程被杀死后仍然存在

`SavedStateHandle` 通过 `ViewModel` 的构造器传入，下面是其主要的主要的几个方法

- T get(String key)
- MutableLiveData<T> getLiveData(String key)
- void set(String key, T value)

`SavedStateHandle` 还包含 `SavedStateProvider` 的实例，用于帮助 `ViewModel` 的 owner 保存状态



### AbstractSavedStateViewModelFactory

一个实现 `ViewModelFactory.KeyedFactory` 的 `ViewModel Factory`，它会创建一个与实例化的请求的 ViewModel 关联的 `SavedStateHandle` 



```java
public abstract class AbstractSavedStateViewModelFactory extends ViewModelProvider.KeyedFactory {
  
    private final SavedStateRegistry mSavedStateRegistry;
  
    // Default state used when the saved state is empty
    private final Bundle mDefaultArgs;

    @Override
    public final <T extends ViewModel> T create(@NonNull String key, @NonNull Class<T> modelClass) {
        // 读取保存的状态
        Bundle restoredState = mSavedStateRegistry.consumeRestoredStateForKey(key);
      
        // 创建保存状态的 handle
        SavedStateHandle handle = SavedStateHandle.createHandle(restoredState, mDefaultArgs);
        
        // ... 
      
        // 创建 viewModel
        T viewmodel = create(key, modelClass, handle);
      
        // ... 

        return viewmodel;
    }
}
```



### SavedStateViewModelFactory

`AbstractSavedStateViewModelFactory` 的具体实现



``` java
public final class SavedStateViewModelFactory extends AbstractSavedStateVMFactory {

    public SavedStateViewModelFactory(@NonNull Application application,
            @NonNull SavedStateRegistryOwner owner) {
        this(application, owner, null);
    }

    public SavedStateViewModelFactory(@NonNull Application application, @NonNull SavedStateRegistryOwner owner, @Nullable Bundle defaultArgs) {
        mSavedStateRegistry = owner.getSavedStateRegistry();
        mLifecycle = owner.getLifecycle();
        mDefaultArgs = defaultArgs;
        mApplication = application;
        mFactory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
    }
    
	public <T extends ViewModel> T create(@NonNull String key, @NonNull Class<T> modelClass) {
        boolean isAndroidViewModel = AndroidViewModel.class.isAssignableFrom(modelClass);
        Constructor<T> constructor;
        if (isAndroidViewModel) {
            constructor = findMatchingConstructor(modelClass, ANDROID_VIEWMODEL_SIGNATURE);
        } else {
            constructor = findMatchingConstructor(modelClass, VIEWMODEL_SIGNATURE);
        }
        // doesn't need SavedStateHandle
        if (constructor == null) {
            return mFactory.create(modelClass);
        }

        SavedStateHandleController controller = SavedStateHandleController.create(
                mSavedStateRegistry, mLifecycle, key, mDefaultArgs);        
        T viewmodel;
        if (isAndroidViewModel) {
            viewmodel = constructor.newInstance(mApplication, controller.getHandle());
        } else {
            viewmodel = constructor.newInstance(controller.getHandle());
        }
        viewmodel.setTagIfAbsent(TAG_SAVED_STATE_HANDLE_CONTROLLER, controller);
        return viewmodel;
        //...
    }
}
```



### 工作流程



![](https://gitee.com/flywith24/Album/raw/master/img/20200319174431.png)



``` java
ViewModelProvider(this).get(MyViewModel::class.java)
```



在 activity 中创建 ViewModel 实例，传入 this （`SavedStateRegistryOwner` ）作为参数，该参数可以访问其 `SavedStateRegistry`，如果没有传入 factory 会通过 activity 重写的 `getDefaultViewModelProviderFactory` 方法来获取默认的 factory 。然后 factory 将使用保存的状态， 将其包装在 `SavedStateHandle` 中，并将其传递给 ViewModel。 ViewModel 可以读取和写入该 handle



当 activity 的 `onSaveInstanceState(outState)` 方法被调用，其 `SavedStateRegistry` 的 `performSave(outState)` 方法将被执行，其内部的所有 `SavedStateProvider` 的 `saveState` 方法均被执行，一旦执行完毕，`outState` 就包含了已保存的状态



当 app 被重启后，activity 和新的 registry  将被创建，activity 的 `onCreate(savedInstanceState)` 方法会被调用，然后 registry 的 `performRestore(savedInstanceState)` 将被调用以便恢复之前保存的状态



## 状态保存的正确姿势

`ViewModel` 构造器加入 `SavedStateHandle` 参数，并将想要保存的数据使用该 handle 保存

``` kotlin
class WithSavedStateViewModel(private val state: SavedStateHandle) : ViewModel() {
    private val key = "key"
    fun setValue(value: String) = state.set(key, value)
    fun getValue(): LiveData<String> = state.getLiveData(key)
}
```

**无需重写 `onSaveInstanceState/onRestoreInstanceState`  方法**

![](https://gitee.com/flywith24/Album/raw/master/img/20200319231231.png)



![运行示意图](https://gitee.com/flywith24/Album/raw/master/img/20200319231451.png)



[Demo 地址](https://github.com/Flywith24/Flywith24-Jetpack-Demo)



> SavedState 仅适合保存轻量级的数据，重量级操作请考虑持sp，数据库等持久化方案



---
## 关于我
---
我是 [Fly_with24](http://www.yangyunzhao.com)
- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)
- [简书](https://www.jianshu.com/u/3d5ad6043d66)
- [Github](https://github.com/Flywith24)

