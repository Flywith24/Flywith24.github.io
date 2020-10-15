---
title: AdroidX 下使用 Activity 和 Fragment 的变化
date: 2020-02-29 00:10:35
categories: 
- 背上 Jetpack
tags: 
- androidx
- Jetpack
- MVVM
image: https://gitee.com/flywith24/Album/raw/master/img/20201015093624.png
---

过去的一段时间，`AndroidX` 软件包下的 `Activity/Fragmet` 的 API 发生了很多变化。让我们看看它们是如何提升Android 的开发效率以及如何适应当下流行的编程规则和模式。

本文中描述的所有功能现在都可以在稳定的 `AndroidX` 软件包中使用，它们在去年均已发布或移至稳定版本。

<!-- more-->


> 原文：[How AndroidX changes the way we work with Activities and Fragments](https://medium.com/@miloszlewandowski/how-androidx-changes-the-way-we-work-with-activities-and-fragments-73b88d157678)
>
> 作者：[Miłosz Lewandowski](https://medium.com/@miloszlewandowski)
>
> 译者：[Flywith24](http://www.yangyunzhao.com/)
> 

### 在构造器中传入布局 ID

从 `AndroidX`  `AppCompat 1.1.0` 和 `Fragment 1.1.0` ( 译者注：AppCompat 包含 Fragment，且 Fragment 包含 Activity，详情见[【整理】Jetpack 主要组件的依赖及传递关系](https://juejin.im/post/5e567ee1518825494466a938) )开始，您可以使用将 `layoutId` 作为参数的构造函数：

``` kotlin
class MyActivity : AppCompatActivity(R.layout.my_activity)
class MyFragmentActivity: FragmentActivity(R.layout.my_fragment_activity)
class MyFragment : Fragment(R.layout.my_fragment)
```

这种方法可以减少 `Activity/Fragment` 中方法重写的数量，并使类更具可读性。 无需在 `Activity` 中重写 `onCreate()` 即可调用 `setContentView()` 方法。 另外，无需手动在`Fragment` 中重写 `onCreateView` 即可手动调用 `Inflater` 来扩展视图。

### 扩展 `Activity/Fragment` 的灵活性

借助 `AndroidX` 新的 API ，可以减少在 `Activity/Fragment` 处理某些功能的情况。通常，您可以获取提供某些功能的对象并向其注册您的处理逻辑，而不是重写 `Activity / Fragment` 中的方法。 这样，您现在可以在屏幕上组成几个独立的类，获得更高的灵活性，复用代码，并且通常在不引入自己的抽象的情况下，对代码结构具有更多控制。 让我们看看这在两个示例中如何工作。

#### 1. OnBackPressedDispatcher

有时，您需要阻止用户返回上一级。 在这种情况下，您需要在 `Activity` 中重写 `onBackPressed()` 方法。 但是，当您使用 `Fragment` 时，没有直接的方法来拦截返回。 在 `Fragment` 类中没有可用的 `onBackPressed()` 方法，这是为了防止同时存在多个 `Fragment` 时发生意外行为。

但是，从 `AndroidX` `Activity 1.0.0` 开始，您可以使用 `OnBackPressedDispatcher` 在您可以访问该 `Activity` 的代码的任何位置（例如，在 `Fragment` 中）注册 `OnBackPressedCallback`。

``` kotlin
class MyFragment : Fragment() {
  override fun onAttach(context: Context) {
    super.onAttach(context)
    val callback = object : OnBackPressedCallback(true) {
      override fun handleOnBackPressed() {
        // Do something
      }
    }
    requireActivity().onBackPressedDispatcher.addCallback(this, callback)
  }
}
```

您可能会在这里注意到另外两个有用的功能：

- `OnBackPressedCallback` 的构造函数中的布尔类型的参数有助于根据当前状态动态 打开/关闭按下的行为
- `addCallback()` 方法的可选第一个参数是 `LifecycleOwner`，以确保仅在您的生命周期感知对象（例如，`Fragment`）至少处于 `STARTED` 状态时才使用回调。

通过使用 `OnBackPressedDispatcher` ，您不仅可以获得在 `Activity` 之外处理返回键的便捷方式。 根据您的需要，您可以在任意位置定义 `OnBackPressedCallback`，使其可复用，或根据应用程序的架构进行任何操作。 您不再需要重写`Activity` 中的 `onBackPressed` 方法，也不必提供自己的抽象的来实现需求的代码。

#### 2. SavedStateRegistry

如果您希望 `Activity` 在终止并重启后恢复之前的状态，则可能要使用 `saved state` 功能。 过去，您需要在 `Activity` 中重写两个方法：`onSaveInstanceState` 和 `onRestoreInstanceState`。 您还可以在 `onCreate` 方法中访问恢复的状态。 同样，在 `Fragment` 中，您可以使用` onSaveInstanceState` 方法（并且可以在 `onCreate`，`onCreateView` 和`onActivityCreated`方法中恢复状态）。

从 `AndroidX`  `SavedState 1.0.0`（它是 `AndroidX` `Activity` 和 `AndroidX`  `Fragment` 内部的依赖。译者注：您不需要单独声明它）开始，您可以访问 `SavedStateRegistry`，它使用了与前面描述的 `OnBackPressedDispatcher` 类似的机制：您可以从 `Activity / Fragment` 中获取 `SavedStateRegistry`，然后 注册您的 `SavedStateProvider`：

``` kotlin
class MyActivity : AppCompatActivity() {

  companion object {
    private const val MY_SAVED_STATE_KEY = "my_saved_state"
    private const val SOME_VALUE_KEY = "some_value"
  }
    
  private lateinit var someValue: String
    
  private val savedStateProvider = SavedStateRegistry.SavedStateProvider {    
    Bundle().apply {
      putString(SOME_VALUE_KEY, someValue)
    }
  }
  
  override fun onCreate(savedInstanceState: Bundle?) {    
    super.onCreate(savedInstanceState)
    savedStateRegistry
      .registerSavedStateProvider(MY_SAVED_STATE_KEY, savedStateProvider)
  }
  
  fun someMethod() {
    someValue = savedStateRegistry
      .consumeRestoredStateForKey(MY_SAVED_STATE_KEY)
      ?.getString(SOME_VALUE_KEY)
      ?: ""
  }
}
```



如您所见，`SavedStateRegistry` 强制您将密钥用于数据。 这样可以防止您的数据被 attach 到同一个 `Activity/Fragment `的另一个 `SavedStateProvider` 破坏。 就像在 `OnBackPressedDispatcher` 中一样，您可以例如将 `SavedStateProvider` 提取到另一个类，通过使用所需的任何逻辑使其与数据一起使用，从而在应用程序中实现清晰的保存状态行为。

此外，如果您在应用程序中使用 `ViewModel`，请考虑使用 `AndroidX`  `ViewModel-SavedState` 使你的`ViewModel` 可以保存其状态。 为了方便起见，从 `AndroidX`  `Activity 1.1.0` 和 `AndroidX` `Fragment 1.2.0` 开始，启用 `SavedState` 的`SavedStateViewModelFactory` 是在获取 `ViewModel` 的所有方式中使用的默认工厂：委托 `ViewModelProvider` 构造函数和 `ViewModelProviders.of()` 方法。

### FragmentFactory

`Fragment` 最常提及的问题之一是不能使用带有参数的构造函数。 例如，如果您使用 `Dagger2` 进行依赖项注入，则无法使用 `Inject` 注解 `Fragment` 构造函数并指定参数。 现在，您可以通过指定 `FragmentFactory` 类来减少 `Fragment` 创建过程中的类似问题。 通过在 `FragmentManager` 中注册 `FragmentFactory`，可以重写实例化 `Fragment` 的默认方法：

``` kotlin
class MyFragmentFactory : FragmentFactory() {

  override fun instantiate(classLoader: ClassLoader, className: String): Fragment {
    // Call loadFragmentClass() to obtain the Class object
    val fragmentClass = loadFragmentClass(classLoader, className)
     
    // Now you can use className/fragmentClass to determine your prefered way 
    // of instantiating the Fragment object and just do it here.
        
    // Or just call regular FragmentFactory to instantiate the Fragment using
    // no arguments constructor
    return super.instantiate(classLoader, className)
  }
}
```

如您所见，该API非常通用，因此您可以执行想要创建 `Fragment` 实例的所有操作。 回到 `Dagger2` 示例，例如，您可以注入`FragmentFactory Provider <Fragment>` 并使用它来获取 `Fragment` 对象。

### 测试 Fragment

从`AndroidX`  `Fragment 1.1.0` 开始，可以使用 `Fragment` 测试组件提供 `FragmentScenario` 类，该类可以帮助在测试中实例化 `Fragment` 并进行单独测试：

``` kotlin
// To launch a Fragment with a user interface:
val scenario = launchFragmentInContainer<FirstFragment>()
        
// To launch a headless Fragment:
val scenario = launchFragment<FirstFragment>()
        
// To move the fragment to specific lifecycle state:
scenario.moveToState(CREATED)

// Now you can e.g. perform actions using Espresso:
onView(withId(R.id.refresh)).perform(click())

// To obtain a Fragment instance:
scenario.onFragment { fragment ->
  ...
}
```

### More Kotlin!

很高兴看到 `-ktx` `AndroidX` 软件包中提供了许多有用的 `Kotlin` 扩展方法，并且定期添加了新的方法。 例如，在`AndroidX` `Fragment-KTX 1.2.0` 中，使用片段化类型的扩展名可用于 `FragmentTransaction` 上的 `replace()` 方法。 将其与 `commit()` 扩展方法结合使用，我们可以获得以下代码：

``` kotlin
// Before
supportFragmentManager
  .beginTransaction()
  .add(R.id.container, MyFragment::class.java, null)
  .commit()

// After
supportFragmentManager.commit {
  replace<MyFragment>(R.id.container)
}
```

### FragmentContainerView

一件小而重要的事情。 如果您将 `FrameLayout` 用作 `Fragment` 的容器，则应改用 `FragmentContainerView` 。 它修复了一些动画 z轴索引顺序问题和窗口插入调度。 从 `AndroidX` `Fragment 1.2.0` 开始可以使用 `FragmentContainerView`。



---

### 关于我

---

我是 [Fly_with24](https://flywith24.gitee.io/)

- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)

- [简书](https://www.jianshu.com/u/3d5ad6043d66)

- [Github](https://github.com/Flywith24)