---
title: 【背上Jetpack之Fragment】你真的会用Fragment吗？Fragment常见问题以及androidx下Fragment的使用新姿势
date: 2020-03-02 00:10:35
categories: 
- Jetpack
tags: 
- androidx
- Jetpack
- MVVM
---

> 在 `Android Jetpack` 组件中，`fragment`作为视图控制器之一占有很重要的位置。但由于其bug众多，暗坑无数，以至于 Square 有这样一篇博客：[Advocating Against Android Fragments](https://developer.squareup.com/blog/advocating-against-android-fragments/)。github上的 [Fragmentation](https://github.com/YoKeyword/Fragmentation) 有着 9.4k 的star。
>
> 而现在，`androidx fragment` 稳定版已来到 1.2.2，让我们总结一下`fragment`有哪些常见问题以及有哪些使用`fragment`的新姿势

<!-- more-->

## Fragment 常见的问题

- getSupportFragmentManager ， getParentFragmentManager 和 getChildFragmentManager

- FragmentStateAdapter 和 FragmentPagerAdapter

- add 和 replace 

- observe LiveData时传入 this 还是 viewLifecycleOwner

- 使用 simpleName 作为 fragment 的 tag 有何风险？

- 在 BottomBarNavigation 和 drawer 中如何使用Fragment多次添加？

- 返回栈

  

### getSupportFragmentManager , getParentFragmentManager和getChildFragmentManager

> `FragmentManager`是 `androidx.fragment.app`(已弃用的不考虑)下的抽象类，创建用于 添加，移除，替换 `fragment` 的事务（`transaction`）  



首先要确认一件事，`getSupportFragmentManager()`是 `FragmentActivity`下的方法



`getParentFragmentManager` 和 `getChildFragmentManager` 是 `androidx.fragment.app.Fragment` 下的方法，**其中  `androidx.fragment 1.2.0` 后 `getFragmentManager` 与  `requireFragmentManager` 已弃用**



明确了这件事，接下来的就很清晰了

- `getSupportFragmentManager`与 `activity`关联，可以将其视为 `activity` 的 `FragmentManager`
- `getChildFragmentManager` 与 `fragment`关联，可以将其视为`fragment`的`FragmentManager`
- `getParentFragmentManager`情况稍微复杂，正常情况返回的是该`fragment` 依附的`activity`的`FragmentManager`。如果该fragment是另一个`fragment` 的子 `fragment`，则返回的是其父`fragment`的 `getChildFragmentManager`



如果这么说还不明白的话，我们可以做一个实践。

创建一个 `activity`,一个父`fragment` ，一个子`fragment`

``` kotlin
// activity
class MyActivity : AppCompatActivity(R.layout.activity_main) {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        supportFragmentManager.commit {
            add<ParentFragment>(R.id.content)
        }
        Log.i("MyActivity", "supportFragmentManager $supportFragmentManager")
    }
}

class ParentFragment : Fragment(R.layout.fragment_parent) {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        childFragmentManager.commit {
            add<ChildFragment>(R.id.content)
        }
        Log.i("ParentFragment", "parentFragmentManager $parentFragmentManager")
        Log.i("ParentFragment", "childFragmentManager $childFragmentManager")
    }
}

class ChildFragment : Fragment(R.layout.fragment_child) {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        Log.i("ChildFragment", "parentFragmentManager $parentFragmentManager")
        Log.i("ChildFragment", "childFragmentManager $childFragmentManager")
    }
}
```

```
//log
I/MyActivity: supportFragmentManager FragmentManager{825dcef in HostCallbacks{14a13fc}}}
I/ParentFragment: parentFragmentManager FragmentManager{825dcef in HostCallbacks{14a13fc}}}
I/ParentFragment: childFragmentManager FragmentManager{df5de83 in ParentFragment{7cdd800}}}
I/ChildFragment: parentFragmentManager FragmentManager{df5de83 in ParentFragment{7cdd800}}}
I/ChildFragment: childFragmentManager FragmentManager{aba9afb in ChildFragment{5cea718}}}
```

因此

- 在 `activity` 中使用 `ViewPager`，`BottomSheetFragment` 和`DialogFragment` 时，都应使用 `getSupportFragmentManager`

- 在`fragment` 中使用 `ViewPager` 时应该使用`getChildFragmentManager`



错误的在 `fragment` 中使用 `activity` 的 `FragmentManager` 会引发内存泄露。 为什么呢？假如您的fragment中有一些依靠 `ViewPager` 管理的子 `fragment`，并且所有这些 `fragment`  都在 `activity` 中，因为您使用的是`activity` 的`FragmentManager` 。 现在，如果关闭您的父`fragment`，它将被关闭，但不会被销毁，因为所有子`fragment`都处于活动状态，并且它们仍在内存中，从而导致泄漏。 它不仅会泄漏父`fragment`，还会泄漏所有子`fragment`，因为它们都无法从堆内存中清除。 



### FragmentStateAdapter 和 FragmentPagerAdapter

`FragmentPagerAdapter`将整个 `fragment `存储在内存中，如果`ViewPager`中使用了大量  `fragment`，则可能导致内存开销增加。 `FragmentStatePagerAdapter`仅存储片段的`savedInstanceState`，并在失去焦点时销毁所有  `fragment`。



让我们看看常见的两个问题



#### 1. 刷新ViewPager不生效

`ViewPager` 中的 `fragment` 是通过 `activity `或 `fragment`的 `FragmentManager` 管理的，`FragmentManager` 包含了`viewpager`的所有`fragment`的实例

因此，当`ViewPager`没有刷新时，它只是`FragmentManager`仍保留的旧 `fragment` 实例。 您需要找出为什么`FragmentManger`持有`fragment`实例的原因。



#### 2. 在Viewpager中访问当前fragment

这也是我们遇到的一个非常普遍的问题。 如果遇到这种情况，我们一般在 `adapter` 内部创建 `fragment` 的数组列表，或者尝试使用某些标签访问`fragment`。 不过还有另一种选择。 `FragmentStateAdapter` 和`FragmentPagerAdapter`都提供方法`setPrimaryItem`。 可以用来设置当前`fragment`，如下所示：

``` kotlin
  var fragment: ChildFragment? = null
  override fun setPrimaryItem(container: ViewGroup, position: Int, any: Any) {
    if (getChildFragment() != any)
    	fragment = any as ChildFragment
    super.setPrimaryItem(container, position, any)
   }
   fun getChildFragment(): ChildFragment? = fragment

	//use
	mAapter.getChildFragment()
```



### add 和 replace 如何选择？

在我们的`activity`中，我们有一个容器，其中装有`fragment`。



`add`只会将一个`fragment`添加到容器中。 假设您将`FragmentA`和`FragmentB`添加到容器中。 容器将具有`FragmentA`和`FragmentB`，如果容器是`FrameLayout`，则将`fragment`一个添加在另一个之上。



`replace`将简单地替换容器顶部的一个`fragment`，因此，如果我创建了 `FragmentC `并 `replace` 顶部的 `FragmentB`，则`FragmentB`将被从容器中删除（执行`onDestroy`，除非您调用`addToBackStack`，仅执行`onDestroyView`），而`FragmentC`将位于顶部。



那么如何选择呢？ `replace`删除现有`fragment`并添加一个新`fragment`。 这意味着当您按下返回按钮时，将创建被替换的`fragment`，并调用其`onCreateView`。 另一方面，`add`保留现有`fragment`，并添加一个新`fragment`，这意味着现有`fragment`将处于活动状态，并且它们不会处于 “paused” 状态。 因此，按下返回按钮时，现有`fragment`（添加新`fragment`之前的`fragment`）不会调用`onCreateView`。 就`fragment`的生命周期事件而言，在`replace`的情况下将调用`onPause`，`onResume`，`onCreateView`和其他生命周期事件，在`add`的情况下则不会。



如果不需要重新访问当前`fragment`并且不再需要当前`fragment`，请使用`replace`。 另外，如果您的应用有内存限制，请考虑使用`replace`。



### observe LiveData时传入 this 还是 viewLifecycleOwner

`androidx fragment 1.2.0` 起，添加了新的 Lint 检查，以确保您在从 `onCreateView()`、`onViewCreated()` 或 `onActivityCreated()` 观察 `LiveData` 时使用 `getViewLifecycleOwner()`



### 使用 simpleName 作为 fragment 的 tag 有何风险？



一般情况下我们会使用calss的`simpleName` 作为`fragment` 的tag

``` kotlin
supportFragmentManager.commit {
	replace(R.id.content,MyFragment.newInstance("Fragment"),
            MyFragment::class.java.simpleName)
    addToBackStack(null)
}
```



这样做不会出现什么问题，但是...



``` kotlin
val fragment = supportFragmentManager.findFragmentByTag(tag)
```



这样获取到的fragment可能不是想要的结果。



为什么呢？



加入有两个 fragment，经过混淆，它们变成

``` 
com.mypackage.FragmentA → com.mypackage.c.a
com.mypackage.FragmentB → com.mypackage.c.a.a
```

上面是混淆了 full name，如果是simpleName 呢？

```
com.mypackage.FragmentA → a
com.mypackage.FragmentB → a
```

WTF！



**所以在设置tag时尽量用全名或者常量**



### 在 BottomBarNavigation 和 drawer 中如何使用Fragment多次添加？

当我们使用`BottomBarNavigation `和 `NavigationDrawer`时，通常会看到诸如`fragment` 重建或多次添加相同`fragment`之类的问题。



在这种情况下，您可以使用`show / hide` 而不是 `add` 或 `replace`。



### 返回栈

如果您想在`fragment`的一系列跳转中按返回键返回上一个`fragment`，应该在`commit` `transaction`之前调用`addToBackStack`方法



``` kotlin
//使用该扩展 androidx.fragment:fragment-ktx:1.2.0 以上
parentFragmentManager.commit {
	addToBackStack(null)
  	add<SecondFragment>(R.id.content)
}
```



## Fragment的使用新姿势

- fragment-ktx 有哪些好用的扩展函数

- fragment 之间和与 activity 通信

- 使用 FragmentContainerView 作为 fragment 容器

- FragmentFactory 的使用

- Fragment 返回键拦截

- Fragment 使用 ViewBinding

- Fragment 使用 ViewPager2

- 不需要重写 onCreateView 了？

- 使用require_()方法

  


### fragment-ktx 有哪些好用的扩展函数

#### 1. FragmentManagerKt

``` kotlin
//before
supportFragmentManager
    .beginTransaction()
    .add(R.id.content,Fragment1())
    .commit()

//after
supportFragmentManager.commit {
	add<Fragment1>(R.id.content)
}
```

#### 2. FragmentViewModelLazyKt

``` kotlin
//before
//共享范围activity
val mViewMode1l = ViewModelProvider(requireActivity()).get(UpdateAppViewModel::class.java)
//共享范围fragment 内部
val mViewMode1l = ViewModelProvider(this).get(UpdateAppViewModel::class.java)

//after
//共享范围activity
private val mViewModel by activityViewModels<MyViewModel>()
//共享范围fragment 内部
private val mViewModel by viewModel<MyViewModel>()
```



> **注意：ViewModelProviders.of(this).get(MyViewModel.class); 的方式已弃用** 
>
> **`lifecycle-extensions` 依赖包已弃用**



### fragment 之间和与 activity 通信



fragment 和 fragment之间，fragment 和 activity 之间的通信有很多方法，android jetpack 推荐我们使用 ViewModel + LiveData 处理

同一个activity 内的 fragment 之间通信，可以使用作用范围为activity的ViewModel，activity与 fragment通信同理。详情可移步 [Android官方应用架构指南](https://developer.android.com/jetpack/docs/guide)



### 使用 FragmentContainerView 作为 fragment 容器



过去我们使用 `FrameLayout` 作为 `Fragment` 的容器，在 `AndroidX Fragment 1.2.0` 后，可以使用 `FragmentContainerView ` 代替 `Fragment` 。



它修复了一些动画 z轴索引顺序问题和窗口插入调度，这意味着两个`fragment`之间的退出和进入过渡不会互相重叠。使用`FragmentContainerView`将先开启退出动画然后才是进入动画。



`FragmentContainerView `  是专门为 fragment设计的自定义View，它继承自 FrameLayout

`android:name` 属性允许您添加`fragment`，`android:tag` 属性可以为`fragment`设置tag



``` xml
 <androidx.fragment.app.FragmentContainerView
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:id="@+id/fragment_container_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:name="com.example.MyFragment"
        android:tag="my_tag">
 </androidx.fragment.app.FragmentContainerView>
```



### FragmentFactory 的使用



过去，我们只能使用其默认的空构造函数实例化Fragment实例。 这是因为在某些情况下，例如配置更改和应用程序的流程重新创建，系统需要重新初始化。 如果不是默认的构造方法，系统将不知道如何重新初始化Fragment实例。

创建FragmentFactory来解决此限制。 通过向其提供实例化Fragment所需的必要参数/依赖关系，它可以帮助系统创建Fragment实例。



过去我们实例化fragment并传递参数会使用类似下面的代码

``` kotlin
class MyFragment : Fragment() {
    private lateinit var arg: String
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        arguments?.getString(ARG) ?: ""
    }
    companion object {
        fun newInstance(arg: String) =
            MyFragment().apply {
                arguments = Bundle().apply {
                    putString(ARG, arg)
                }
            }
    }
}

//use
val fragment = MyFragment.newInstance("my argument")
```



如果您的Fragment有一个非空的构造函数，则需要创建一个FragmentFactory来处理它的初始化。

``` kotlin
class MyFragmentFactory(private val arg: String) : FragmentFactory() {
    override fun instantiate(classLoader: ClassLoader, className: String): Fragment {
        if (className == MyFragment::class.java.name) {
            return MyFragment(arg)
        }
        return super.instantiate(classLoader, className)
    }
}
```



`fragment`由`FragmentManager` 管理，因此很自然，`FragmentFactory`需要添加到`FragmentManager`才能使用。



那么什么时候把`FragmentFactory` 添加到`FragmentManager`呢？



**父类调用 `Activity#onCreate()`  和 `Fragment#onCreate()`之前**



``` kotlin
class HostActivity : AppCompatActivity() {
    private val customFragmentFactory = CustomFragmentFactory(Dependency())

    override fun onCreate(savedInstanceState: Bundle?) {
        supportFragmentManager.fragmentFactory = customFragmentFactory
        super.onCreate(savedInstanceState)
        // ...
    }
}

class ParentFragment : Fragment() {
    private val customFragmentFactory = CustomFragmentFactory(Dependency())

    override fun onCreate(savedInstanceState: Bundle?) {
        childFragmentManager.fragmentFactory = customFragmentFactory
        super.onCreate(savedInstanceState)
        // ...
    }
}
```



如果您的`Fragment`具有默认的空构造函数，则无需使用`FragmentFactory`。 但是，如果您的`Fragment`在其构造函数中接受参数，则必须使用`FragmentFactory`，否则将抛出`Fragment.InstantiationException`，因为将使用的默认`FragmentFactory`将不知道如何实例化`Fragment`的实例。



### Fragment 返回键拦截



有时候，您需要阻止用户返回上一级。 在这种情况下，您需要在 `Activity` 中重写 `onBackPressed()` 方法。 但是，当您使用 `Fragment` 时，没有直接的方法来拦截返回。 在 `Fragment` 类中没有可用的 `onBackPressed()` 方法，这是为了防止同时存在多个 `Fragment` 时发生意外行为。

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



### Fragment 使用 ViewBinding



`Android Studio 3.6.0` 后提供了 `ViewBindind`的支持，完整使用流程参见 [[译]深入研究ViewBinding 在 include, merge, adapter, fragment, activity 中使用](https://juejin.im/post/5e4806f3e51d4526c550a2ef)



``` kotlin
class HomeFragment : Fragment() {
    private var _binding: FragmentHomeBinding? = null
    private val binding get() = _binding!!

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        _binding = FragmentHomeBinding.inflate(inflater, container, false)
        return binding.root
    }
    override fun onDestroyView() {
        _binding = null
    }
}
```



### Fragment 使用 ViewPager2



`ViewPager`使用了三个`adapter`的抽象类，而`ViewPager2`中只有两个

- ViewPager 中使用 `PagerAdaper`，ViewPager2 中使用 **`Recyclerview.Adapter`**
- ViewPager 中使用 `FragmentPagerAdapter` ，ViewPager2中使用 **`FragmentStateAdapter`**
- ViewPager 中使用 `FragmentStatePagerAdapter` ，ViewPager2中使用 **`FragmentStateAdapter`**



``` kotlin
// A simple ViewPager adapter class for paging through fragments
class ScreenSlidePagerAdapter(fm: FragmentManager) : FragmentStatePagerAdapter(fm) {
    override fun getCount(): Int = NUM_PAGES

    override fun getItem(position: Int): Fragment = ScreenSlidePageFragment()
}

// An equivalent ViewPager2 adapter class
class ScreenSlidePagerAdapter(fa: FragmentActivity) : FragmentStateAdapter(fa) {
    override fun getItemCount(): Int = NUM_PAGES

    override fun createFragment(position: Int): Fragment = ScreenSlidePageFragment()
}
```



使用 `TabLayout`的变化，`TabLayout` 已从`ViewPager2`中解耦，如果使用`TabLayout`，需要引入依赖

``` groovy
implementation "com.google.android.material:material:1.1.0"
```



对于`ViewPager2` ，`TabLayout`布局应与`ViewPager2`在同一级别



``` xml
<!-- A ViewPager element with a TabLayout -->
<androidx.viewpager.widget.ViewPager
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/pager"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tab_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

</androidx.viewpager.widget.ViewPager>

<!-- A ViewPager2 element with a TabLayout -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tab_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/pager"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

</LinearLayout>
```



使用`ViewPager`时，`TabLayout`与`ViewPager`联动需要调用 `setupWithViewPager`，并重写`getPageTitle`方法，而`ViewPager2`改为使用`TabLayoutMediator`对象



``` kotlin
// Integrating TabLayout with ViewPager
class CollectionDemoFragment : Fragment() {
    ...
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        val tabLayout = view.findViewById(R.id.tab_layout)
        tabLayout.setupWithViewPager(viewPager)
    }
    ...
}

class DemoCollectionPagerAdapter(fm: FragmentManager) : FragmentStatePagerAdapter(fm) {

    override fun getCount(): Int  = 4

    override fun getPageTitle(position: Int): CharSequence {
        return "OBJECT ${(position + 1)}"
    }
    ...
}

// Integrating TabLayout with ViewPager2
class CollectionDemoFragment : Fragment() {
    ...
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        val tabLayout = view.findViewById(R.id.tab_layout)
        TabLayoutMediator(tabLayout, viewPager) { tab, position ->
            tab.text = "OBJECT ${(position + 1)}"
        }.attach()
    }
    ...
}
```



### 不需要重写 onCreateView 了？



`androidx fragment 1.1.0` 后，您可以使用将 `layoutId` 作为参数的构造函数，这样就无需重写 `onCreateView` 方法了

``` kotlin
class MyActivity : AppCompatActivity(R.layout.my_activity)
class MyFragmentActivity: FragmentActivity(R.layout.my_fragment_activity)
class MyFragment : Fragment(R.layout.my_fragment)
```



### 使用require_()方法



`androidx fragment 1.2.2` 起，新增了一项lint检查，`fragment` 建议使用关联的`require_()`方法获取更多描述性错误消息，而不是使用`checkNotNull(get_())`，`requireNonNull(get_())` 或`get()！` 适用于所有包含 get 和 require Fragment API



例如：使用 `requireActivity()` 替代 `getActivity()`



