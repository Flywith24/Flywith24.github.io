---
title: 【背上Jetpack之DataBinding】数据驱动魔法师 何时迎来翻身日？
date: 2020-03-14 00:10:35
categories: 
- Jetpack
tags: 
- androidx
- Jetpack
- MVVM
---

> [LiveData 篇](https://juejin.im/post/5e834bb5f265da480d61668d) 我们提到 Android 开发的主要工作内容是将数据转换为 UI ，同时我们也介绍了数据驱动 UI 的思想，使用 ViewModel + LiveData，可以安全地在订阅者的生命周期内分发正确的数据。但是 activity 和 fragment 充斥着大量的模板代码，铺天盖地的 findViewById，以及各种 set （根据数据设置 UI）。如果能够消灭掉这些模板代码就好了

<!-- more-->

![他来了他来了](https://gitee.com/flywith24/Album/raw/master/img/20200402101708.gif)

他来了他来了，他欢快地走来了

![DataBinding](https://gitee.com/flywith24/Album/raw/master/img/20200402101727.png)

然而，很多开发者对 DataBinding 存在偏见，「DataBinding 不是个好东西，在声明式编程中书写 UI 逻辑，既不可调试，也不便于察觉和追踪，万一出现问题就麻烦了。」

 <img src="https://gitee.com/flywith24/Album/raw/master/img/20200402165235.png" style="zoom:200%;" />

本文主要介绍 DataBinding 的解决的问题以及其背后的逻辑，带您对 DataBinding 有一个感性的认识。本文末尾会对各个 findViewById 的替代方案进行对比



DataBinding 的相关资源

- [官方文档](https://developer.android.com/topic/libraries/data-binding)
- [codelab](https://codelabs.developers.google.com/codelabs/android-databinding/#0)
- [官方示例](https://github.com/android/databinding-samples)



<h2 style='color: inherit; line-height: inherit; padding: 0px; margin: 1.6em 0px; font-weight: bold; border-bottom: 2px solid #9370DB; font-size: 1.3em;'><span style='font-size: inherit; line-height: inherit; margin: 0px; display: inline-block; font-weight: normal; background: #9370DB; color: rgb(255, 255, 255); padding: 3px 10px 1px; border-top-right-radius: 3px; border-top-left-radius: 3px; margin-right: 3px;'> 数据驱动魔法师</span></h2>

DataBinding 允许使用声明性格式而不是通过编程方式将布局中的 UI 组件与数据源绑定

``` java
// before
TextView textView = findViewById(R.id.sample_text);
textView.setText(viewModel.getUserName());
```

``` xml
<TextView
    android:text="@{viewmodel.userName}" />
```

通过在布局文件中绑定组件，您可以删除 activity 中的许多设置 UI 调用，从而使它们更易于维护。 这也可以提高应用程序的性能，并有助于防止内存泄漏和空指针异常



> 如果仅替换 findViewById 而不需要数据的绑定，可以使用 ViewBinding，它使用起来更简单，性能也更好。
>
> 使用方法参见 [[译]深入研究ViewBinding 在 include, merge, adapter, fragment, activity 中使用](https://juejin.im/post/5e4806f3e51d4526c550a2ef)



<h2 style='color: inherit; line-height: inherit; padding: 0px; margin: 1.6em 0px; font-weight: bold; border-bottom: 2px solid #9370DB; font-size: 1.3em;'><span style='font-size: inherit; line-height: inherit; margin: 0px; display: inline-block; font-weight: normal; background: #9370DB; color: rgb(255, 255, 255); padding: 3px 10px 1px; border-top-right-radius: 3px; border-top-left-radius: 3px; margin-right: 3px;'> DataBinding 基础</span></h2>

详细内容参见 [官方文档](https://developer.android.com/topic/libraries/data-binding) ，这里只简单介绍 DataBinding

### DataBinding 引入

app build.gradle 中加入 

```groovy
android {
    ...
    dataBinding {
        enabled = true
    }
}

// Android Studio 4.0
android {
    ...
    buildFeatures {
        dataBinding = true
    }
}

```




> 必须在 app module 中声明，声明后其他子 module 可直接使用 DataBinding



使用 DataBinding 无需开发者手动引入库，android build gradle plugin 内部已经引入了

![](https://gitee.com/flywith24/Album/raw/master/img/20200402115116.png)



DataBinding 中使用了注解，因此在构建速度上比 ViewBinding 差些（不过功能这么强大要啥自行车）



### 布局



DataBinding 布局文件略有不同，它们以 layout 的根标记开始，后跟一个 data 元素和一个 view 根元素。 view 元素是您的根将位于非绑定布局文件中的元素。 以下代码显示了一个示例布局文件：

``` xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">
    <data>
        <variable
            name="viewmodel"
            type="com.myapp.data.ViewModel" />
    </data>
    <ConstraintLayout... /> <!-- UI layout's root element -->
</layout>
```



### 生成绑定类

DataBinding 会为每个在布局声明 layout 标签的 xml 布局文件生成一个绑定类。 默认情况下，类的名称基于布局文件的名称。 上面的布局文件名是 activity_main.xml，因此相应的生成类是 ActivityMainBinding。 此类包含从布局属性（例如，viewmodel 变量）到布局视图的所有绑定，并且知道如何为绑定表达式分配值



### 配置绑定

``` java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // before
    // setContentView(R.layout.activity_main)
    
    // after
    val binding : ActivityMainBinding =
    DataBindingUtil.setContentView(this, R.layout.activity_main)
}
```



<h2 style='color: inherit; line-height: inherit; padding: 0px; margin: 1.6em 0px; font-weight: bold; border-bottom: 2px solid #9370DB; font-size: 1.3em;'><span style='font-size: inherit; line-height: inherit; margin: 0px; display: inline-block; font-weight: normal; background: #9370DB; color: rgb(255, 255, 255); padding: 3px 10px 1px; border-top-right-radius: 3px; border-top-left-radius: 3px; margin-right: 3px;'> 使用DataBinding 解决的问题及实现原理</span></h2>



不知道你是否有这些烦恼：activity 和 fragment 中有着大量的模板代码，即使使用 ButterKnife 等工具写起代码来也很繁琐。而且 View id 与 View 的类型不匹配时，只有在运行期才能发现；旋转屏幕后如果新的布局中不存在之前 id 的 view ，可能还导致空指针异常；项目中使用各类 bus 通知 UI 刷新，但是有时 UI 的显示并不符合预期，而排查起来特别困难，因为数据源很多...



不要慌，DataBinding 可以解决以下问题



- 替换 findViewById ，减少模板代码

- 解决类型安全问题

- 解决空安全问题

- 保证了数据的一致性

  

### 魔法的背后

com.android.tools.build:gradle 插件中封装了 DataBinding 的魔法



![](https://gitee.com/flywith24/Album/raw/master/img/20200402162010.png)



查看 com.android.tools.build:gradle:3.6.2 的源码，找到 DataBinding 配置项的类 DataBindingOptions

``` java
// DataBindingOptions.java
@Override
public boolean isEnabled() {
    // DataBinding 是否开启，对应上面在 build.gradle 中的配置
    return enabled;
}
```

它的调用者很多，在 TaskManager 中的 createDataBindingTasksIfNecessary

```java
// TaskManager 
protected void createDataBindingTasksIfNecessary(@NonNull VariantScope scope) {
    // 是否开启 DataBinding
    boolean dataBindingEnabled = extension.getDataBinding().isEnabled();
    boolean viewBindingEnabled = extension.getViewBinding().isEnabled();
    if (!dataBindingEnabled && !viewBindingEnabled) {
        // DataBinding 和 ViewBinding 均未开启则直接 return
        return;
    }
    createDataBindingMergeBaseClassesTask(scope);
    createDataBindingMergeArtifactsTask(scope);
    //...
    // 构建 DataBinding 相应绑定类
    taskFactory.register(new DataBindingGenBaseClassesTask.CreationAction(scope));
}
// CreationAction
override fun handleProvider(taskProvider: TaskProvider<out DataBindingGenBaseClassesTask>) {
    variantScope.artifacts.producesDir(
        // DATA_BINDING_BASE_CLASS_SOURCE_OUT
        InternalArtifactType.DATA_BINDING_BASE_CLASS_SOURCE_OUT,
        BuildArtifactsHolder.OperationType.INITIAL,
        taskProvider,
        DataBindingGenBaseClassesTask::sourceOutFolder
	)
}
```



可以看到生成 DataBinding 绑定类的 task 为 DataBindingGenBaseClassesTask，而InternalArtifactType.DATA_BINDING_BASE_CLASS_SOURCE_OUT 则对应着 build 目录生成的 DataBinding 类的

data_binding_base_class_source_out 目录

![](https://gitee.com/flywith24/Album/raw/master/img/20200402161451.png)

这里可以简单看一下，感兴趣的小伙伴可以自己查看源码



### DataBinding 如何解决上述问题的

我们可以查看 DataBinding 生成的绑定类

```java
public final class FragmentSingleChildBinding implements ViewBinding {
  // NonNull 注解标记
  // 如果存在不同配置的不同布局文件（如横竖屏）且该控件不是存在于所有布局，该处使用 Nullable注解标记
  @NonNull
  public final MaterialButton button;
    
  // 省略...
    
  @NonNull
  public static FragmentSingleChildBinding bind(@NonNull View rootView) {
    String missingId;
    missingId: {
      //其内部也是使用 findViewById
      MaterialButton button = rootView.findViewById(R.id.button);
      if (button == null) {
        missingId = "button";
        break missingId;
      }
      return new FragmentSingleChildBinding((MaterialButton) rootView, button);
    }
    throw new NullPointerException("Missing required view with ID: ".concat(missingId));
  }
}
```



- Binding 类内部的也是使用 findViewById ，因此 DataBinding 可以代替 findViewById ，并且减少模板代码

- View 控件变量类型是固定的，因此不会出现类型安全问题

- View 控件变量由空/非空注解修饰，（如果为 Nullable java 中会有 lint 警告，而 kotlin 直接调用时无法通过编译的）因此 不会出现空安全问题
- 通过声明式的配置，UI 完全来自唯一可信的数据源配置，保证了数据的一致性



> 注意：以上分析同样适用于 ViewBinding



<h2 style='color: inherit; line-height: inherit; padding: 0px; margin: 1.6em 0px; font-weight: bold; border-bottom: 2px solid #9370DB; font-size: 1.3em;'><span style='font-size: inherit; line-height: inherit; margin: 0px; display: inline-block; font-weight: normal; background: #9370DB; color: rgb(255, 255, 255); padding: 3px 10px 1px; border-top-right-radius: 3px; border-top-left-radius: 3px; margin-right: 3px;'> 感受魔法的魅力</span></h2>

这里简单展示一下 DataBinding 的「魔法」



### 基本操作

``` xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
      
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView
           android:id="@+id/firstName"
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           />
       <TextView
           android:id="@+id/lastName"
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           />
   </LinearLayout>
</layout>
```



``` java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

		// Before Data Binding
		//setContentView(R.layout.activity_main);
        
		//TextView firstName = (TextView) findViewById(R.id.firstName);
		//TextView lastName = (TextView) findViewById(R.id.lastName);

		//firstName.setText("xxx");
		//lastName.setText("xxx");


		// After Data Binding
        ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);

        binding.firstName.setText("xxx");
        binding.lastName.setText("xxx");
    }
}
```

上面展示了 DataBinding 的基础操作（单纯的替换 findViewById），如果仅使用 DataBinding 这部分功能，可以考虑使用 ViewBinding

![](https://gitee.com/flywith24/Album/raw/master/img/20200409114053.png)

### 绑定数据



在之前的布局的基础上绑定数据

``` xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView 
           android:id="@+id/firstName"      
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView 
           android:id="@+id/lastName"      
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```

``` java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
		binding.user = new User("xxx","xxx");
    }
}
```



这种方式也可以用在 recyclerview adapter 中，adapter 中的代码大大减少



### Binding Adapter

您可能会好奇配置 `android:text="@{user.firstName}` 后内部发生了什么

DataBinding 中使用 `Binding Adapter` 来处理，它主要处理「属性」和「事件」，前者如  `setText()` ，后者如 `setOnClickListener()`。上面的 `android:text` 实际上调用的是下面的方法

![](https://gitee.com/flywith24/Album/raw/master/img/20200409145551.png)



DataBinding 中提供了很多 `Binding Adapter`

![](https://gitee.com/flywith24/Album/raw/master/img/20200410160031.png)



如果官方提供的 `Binding Adapter` 不满足您的需求，您还可以自定义 `Binding Adapter`

```java
@BindingAdapter({"imageUrl", "error"})
public static void loadImage(ImageView view, String url, Drawable error) {
  Glide.with(view).load(url).error(error).into(view);
}
```

``` xml
<ImageView app:imageUrl="@{venue.imageUrl}" app:error="@{@drawable/venueError}" />
```



### DadaBinding + LiveData

要将 LiveData 与 DataBinding 一起使用，需要指定生命周期所有者来定义 LiveData 对象的范围

``` java
class ViewModelActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        UserBinding binding = DataBindingUtil.setContentView(this, R.layout.user);

        binding.setLifecycleOwner(this);
    }
}
```



### 双向绑定

使用单向 DataBinding，可以在属性上设置一个值，并设置一个对该属性的更改做出反应的监听器：

``` xml
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@{viewmodel.rememberMe}"
    android:onCheckedChanged="@{viewmodel.rememberMeChanged}"
/>
```

使用双向绑定可以简化该过程

``` xml
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@={viewmodel.rememberMe}"
/>
```



使用 `@={}` 接收对该属性的数据更改，并同时监听用户更新（注意，这里有 `=` ）



那么究竟什么是双向绑定呢？

所谓的「数据驱动」就是数据驱动视图的变化，而 DataBinding 的单向绑定就是如此。反过来讲，有些时候我们需要视图来驱动数据的变化（例如当我们在 EditText 上输入了文字，我们希望对应的 ViewModel 的 LiveData 的值能够及时响应该变化）

![](https://gitee.com/flywith24/Album/raw/master/img/20200409161036.jpg)



![](https://gitee.com/flywith24/Album/raw/master/img/20200409110254.gif)



如图，绿色部分为独立的 fragment ，内部存在两个 TextView，用于显示外部 fragment EditText 输入的文字



如果实现上述功能，传统做法可能是使用 activity 级别的 ViewModel 进行两个 fragment 之间的通信，通过监听 EditText 文字的变化改变 ViewModel 中 LiveData 的值，并在绿色 fragment 中观察 LiveData 并显示到 TextView 中

``` kotlin
class NormalViewModel : ViewModel() {
    val firstName = MutableLiveData<String>()
    val lastName = MutableLiveData<String>()
}

class NormalDetailFragment : Fragment(R.layout.fragment_normal_detail) {
    private val mViewModel by activityViewModels<NormalViewModel>()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        mViewModel.firstName.observe(viewLifecycleOwner) {
            tvFirstName.text = it
        }
        mViewModel.lastName.observe(viewLifecycleOwner) {
            tvLastName.text = it
        }
    }
}

class NormalFragment : Fragment(R.layout.fragment_normal) {
    private val mViewModel by activityViewModels<NormalViewModel>()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        etFirstName.addTextChangedListener {
            mViewModel.firstName.value = it.toString()
        }
        etLastName.addTextChangedListener {
            mViewModel.lastName.value = it.toString()
        }
    }
}
```

得益于 kotlin ，上面的代码以及很简洁了，如果使用 java 代码片段只会更长。

不过使用 DataBinding，还可以更简洁



``` xml
<com.google.android.material.textview.MaterialTextView
    android:id="@+id/tvFirstName"
    android:text="@{vm.firstName}"/>
<com.google.android.material.textview.MaterialTextView
    android:id="@+id/tvLastName"
    android:text="@{vm.lastName}"/>
```

``` xml
<com.google.android.material.textfield.TextInputEditText
    android:id="@+id/etFirstName"
    android:text="@={vm.firstName}"/>
<com.google.android.material.textfield.TextInputEditText
    android:id="@+id/etLastName"
    android:text="@={vm.lastName}"/>
```

只需配置好双向绑定（EditText 驱动 ViewModel 的 LiveData 的值变化，ViewModel 再驱动 TextView 显示数据），并在 fragment 通过固定的模板代码设置好 ViewModel 即可



![](https://gitee.com/flywith24/Album/raw/master/img/20200409112128.jpg)



这里的魔法还是来自 Binding Adapter



``` java
// TextViewBindingAdapter.java
@BindingAdapter(value = {"android:beforeTextChanged", "android:onTextChanged",
        "android:afterTextChanged", "android:textAttrChanged"}, requireAll = false)
public static void setTextWatcher(TextView view, final BeforeTextChanged before,
        final OnTextChanged on, final AfterTextChanged after,
        final InverseBindingListener textAttrChanged) {
    final TextWatcher newValue;
    if (before == null && after == null && on == null && textAttrChanged == null) {
        newValue = null;
    } else {
        newValue = new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {
                if (before != null) {
                    before.beforeTextChanged(s, start, count, after);
                }
            }
            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                if (on != null) {
                    on.onTextChanged(s, start, before, count);
                }
                if (textAttrChanged != null) {
                    //通知发生变化
                    textAttrChanged.onChange();
                }
            }
            @Override
            public void afterTextChanged(Editable s) {
                if (after != null) {
                    after.afterTextChanged(s);
                }
            }
        };
    }
    final TextWatcher oldValue = ListenerUtil.trackListener(view, newValue, R.id.textWatcher);
    if (oldValue != null) {
        view.removeTextChangedListener(oldValue);
    }
    if (newValue != null) {
        view.addTextChangedListener(newValue);
    }
}
```

这里使用 InverseBindingListener （调用 `textAttrChanged.onChange()`）来通知 LiveData 数据发生变化



而变化后的值 通过 @InverseBindingAdapter 注解标记的方法处理，这里的 event 与上面的标记匹配（`android:textAttrChanged`）

``` java
// TextViewBindingAdapter.java
@InverseBindingAdapter(attribute = "android:text", event = "android:textAttrChanged")
public static String getTextString(TextView view) {
    return view.getText().toString();
}
```



view 层变化通知数据变化，数据变化再通知 view 层变化，仿佛是个套娃



![](https://gitee.com/flywith24/Album/raw/master/img/20200409163420.jpg)



因此避免这种死循环十分重要，setText 方法判断了新旧值是否相等来避免死循环



![](https://gitee.com/flywith24/Album/raw/master/img/20200409163541.png)



<h2 style='color: inherit; line-height: inherit; padding: 0px; margin: 1.6em 0px; font-weight: bold; border-bottom: 2px solid #9370DB; font-size: 1.3em;'><span style='font-size: inherit; line-height: inherit; margin: 0px; display: inline-block; font-weight: normal; background: #9370DB; color: rgb(255, 255, 255); padding: 3px 10px 1px; border-top-right-radius: 3px; border-top-left-radius: 3px; margin-right: 3px;'> 总结</span></h2>

DataBinding 主要提供两部分功能

- 替换 findViewById ，如果只用这部分功能可以使用 ViewBinding
- 进行 data 和 UI 的绑定，使用「数据驱动」的思想解决了视图的一致性问题

### 各种 findViewById 替代方案对比

- findViewById 
- Butterknife
- Kotlin Synthetics
- Data Binding
- View Binding

#### findViewById

![](https://gitee.com/flywith24/Album/raw/master/img/20200409170514.png)

findViewById 有两个问题

1. 当不能在 Activity/Fragment/ViewGroup 中定位到指定 id 的 View，会在运行期间崩溃，即非空安全
2. 如果某个 view 为 TextView 类型，而在使用中将其指定为其他类型不会在编译器报错，即非类型安全



在 compileSdk 的 API 级别 26 中，对该方法的定义稍作更改以消除强制类型转换问题

![](https://gitee.com/flywith24/Album/raw/master/img/20200409171139.png)



现在，开发人员无需在代码中手动转换 view 类型。 如果您引用 id 指向类型 TextView 的 View 并将其指定为 Button，则 Android SDK 会尝试查找具有提供的 id 的 Button，并且它将返回 Null，因为它将无法找到它

但是在 Kotlin 中，您仍然需要提供诸如 findViewById<TextView>(R.id.txtUsername) 之类的类型。 如果您不检查视图是否具有 null 安全，则可能出现 NullPointerException，但是此方法不会像以前那样抛出ClassCastException

#### Butterknife

[Butterknife](https://github.com/JakeWharton/butterknife) 是 [Jake Wharton](https://medium.com/u/8ddd94878165?source=post_page-----98b8ef5b9249----------------------) 大神写的替代 findViewById 的库，该库使用注解处理并生成 findViewById 代码

![](https://gitee.com/flywith24/Album/raw/master/img/20200409171316.png)



它具有与 findViewById 几乎相似的问题。 但是，它在运行时添加了null 安全检查以避免 NullPointerException

![](https://gitee.com/flywith24/Album/raw/master/img/20200409171554.jpg)



由于 DataBinding 和 ViewBinding 的出现，沃神已经宣布弃用该库



#### Kotlin Synthetics

Kotlin 引入的最大功能之一是 Kotlin 扩展方法。 在它的帮助下，Kotlin Synthetics 诞生了。 Kotlin Synthetics 通过自动生成的 Kotlin 扩展方法，使开发人员可以从 xml 布局直接访问其内部的 view

![](https://gitee.com/flywith24/Album/raw/master/img/20200409171843.png)



Kotlin Synthetics 第一次调用 findViewById 方法，然后默认情况下将 view 实例缓存在 HashMap 中。 可以通过Gradle 设置将此缓存配置更改为 SparseArray 或不缓存

总体而言，Kotlin Synthetics 是一种很好的选择，因为它类型安全，并且通过 Kotlin 的 ？进行空检查。 它不需要开发人员的额外代码。 但这仅适用于 Kotlin 项目



但是，在使用 Kotlin Synthetics 时遇到了一个小问题。 例如，如果将内容视图设置为布局，然后使用仅存在于其他布局中的 id ，则 IDE 可让您自动完成并添加新的 import 语句。 除非您专门检查以确保其 import 语句仅导入正确的 view，否则没有安全的方法来验证这不会导致运行时问题



#### DataBinding

DataBinding 在功能上比其他方法优越得多，因为它不仅为您提供类型安全和空安全的 view 引用，而且还允许您直接在 xml 布局内使用数据驱动视图变化

![](https://gitee.com/flywith24/Album/raw/master/img/20200409172613.png)



#### ViewBinding

最近在 Android Studio 3.6 中引入的 ViewBinding 是 DataBinding 库的子集。 由于不需要注解处理，因此可以缩短构建时间。详细的使用可以参见 [这篇文章](https://juejin.im/post/5e4806f3e51d4526c550a2ef)



|            | findViewById | Butterknife | Kotlin Synthetics | DataBinding | ViewBinding |
| :--------: | :----------: | :---------: | :---------------: | :---------: | :---------: |
| 一直空安全 |      ❌       |    部分     |       部分        |      ✔️      |      ✔️      |
|  类型安全  |      ❌       |      ❌      |         ✔️         |      ✔️      |      ✔️      |
|  样板代码  |     <font color=#CA0C16 >多</font>       |      <font color=#0DD613 >少</font>        |        <font color=#0DD613 >少</font>           |    <font color=#B77441 >中等</font>     |     <font color=#0DD613 >少</font>        |
|  构建时间  |      ✔️       |      ❌      |         ✔️         |      ❌      |      ✔️      |
|  支持语音  | java/kotlin  | java/kotlin |      kotlin       | java/kotlin | java/kotlin |

<h2 style='color: inherit; line-height: inherit; padding: 0px; margin: 1.6em 0px; font-weight: bold; border-bottom: 2px solid #9370DB; font-size: 1.3em;'><span style='font-size: inherit; line-height: inherit; margin: 0px; display: inline-block; font-weight: normal; background: #9370DB; color: rgb(255, 255, 255); padding: 3px 10px 1px; border-top-right-radius: 3px; border-top-left-radius: 3px; margin-right: 3px;'> 关于我</span></h2>

我是 [Fly_with24](https://flywith24.gitee.io/)
- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)
- [简书](https://www.jianshu.com/u/3d5ad6043d66)
- [Github](https://github.com/Flywith24)

