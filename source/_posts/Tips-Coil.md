---
title: 【奇技淫巧】新的图片加载库？基于Kotlin协程的图片加载库——Coil
date: 2020-05-18 14:11:46
categories: 
- Tips
tags: 
- Tips
- 奇技淫巧
top: true
---

## 新的图片加载库——Coil

[Coil](https://coil-kt.github.io/coil/) 是 [Instacart](https://www.instacart.com/) 团队研发的新的的图片加载库，它使用了很多高级功能，例如协程，`Okhttp`，`androidx.lifecycle`。`Coil` 还包括一些高级功能，例如图像采样，有效的内存使用以及请求的自动取消/暂停

默认情况下 `Coil` 与 R8 完全兼容，开箱即用，不需要添加额外的规则。如果使用 Proguard ，您可能需要为 [Coroutines](https://github.com/Kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/resources/META-INF/proguard/coroutines.pro), [OkHttp](https://github.com/square/okhttp/blob/master/okhttp/src/main/resources/META-INF/proguard/okhttp3.pro) 和 [Okio](https://github.com/square/okio/blob/master/okio/src/jvmMain/resources/META-INF/proguard/okio.pro) 添加规则



## Coil 的优势

- 快速：`Coil` 进行了很多优化，包括内存和磁盘缓存，对内存中的图像进行采样，重新使用位图，自动暂停/取消请求等等
- 轻量：`Coil` 在您的APK中添加了约 2000 种方法（对于已经使用 `OkHttp` 和 `Coroutines` 的应用程序），与 `Picasso` 相当，远少于 `Glide` 和 `Fresco`
- 易用：`Coil` 的 API 利用 Kotlin 的特性简化了样板代码
- 现代：`Coil` 是 `Kotlin-first`，使用现代化的库，例如 `Coroutines`, `OkHttp`, `Okio`, 以及 `AndroidX Lifecycles`



`Coil` 是以下名称的缩写：**Coroutine Image Loader**



## Artifacts

`Coil` 拥有 5 个 artifact 并发布在 `mavenCentral()`

- `io.coil-kt:coil`：依赖于  `io.coil-kt:coil-base` 并且包含了 `Coil` 的单例和  `ImageView.load` 的扩展函数
- `io.coil-kt:coil-base`：base 库，**不包含** `Coil` 的单例和  `ImageView.load` 的扩展函数，如果使用依赖注入，则可以使用该库
- `io.coil-kt:coil-gif`：引入一系列解码器以支持解码 gif
- `io.coil-kt:coil-svg`：引入一系列解码器以支持 svg
- `io.coil-kt:coil-video`：包括两个 [fetchers](https://coil-kt.github.io/coil/api/coil-base/coil.fetch/-fetcher) ，以支持从 Android 支持的任何视频格式中提取和解码帧



``` groovy
// 普通使用引用
implementation "io.coil-kt:coil:0.11.0"
// 使用依赖注入时或者制作基于 coil 的库引用
implementation "io.coil-kt:coil-base:0.11.0"
```



## Java 8

`Coil` 要求 Java 8，要通过 D8 启用 Java 8 调试，请将以下内容添加到 Gradle 脚本

Gradle (`.gradle`)

``` groovy
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

tasks.withType(org.jetbrains.kotlin.gradle.tasks.KotlinCompile).all {
    kotlinOptions {
        jvmTarget = "1.8"
    }
}
```



Gradle Kotlin DSL (`.gradle.kts`)

```  kotlin
android {
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }
}

tasks.withType<KotlinCompile> {
    kotlinOptions {
        jvmTarget = "1.8"
    }
}
```



## 使用

### ImageView 扩展函数

`io.coil-kt:coil` 提供了 类型安全的 ImageView 扩展函数

在 ImageView 中加载图片，只需调用 load 扩展函数

``` kotlin
// URL
imageView.load("https://www.example.com/image.jpg")

// Resource
imageView.load(R.drawable.image)

// File
imageView.load(File("/path/to/image.jpg"))

// And more...
```

上面的请求等价于：

``` kotlin
val imageLoader = Coil.imageLoader(context)
val request = LoadRequest.Builder(imageView.context)
    .data("https://www.example.com/image.jpg")
    .target(imageView)
    .build()
imageLoader.execute(request)
```



可选的请求配置可以通过 lambda 来操作

``` kotlin
imageView.load("https://www.example.com/image.jpg") {
    crossfade(true)
    placeholder(R.drawable.image)
    transformations(CircleCropTransformation())
}
```



### Image Loaders[¶](https://coil-kt.github.io/coil/#image-loaders)

`ImageLoader` 是执行请求的服务类。 他们处理缓存，数据获取，图像解码，请求管理，bitmap pool，内存管理等。 可以使用 builder 来创建和配置新实例：

``` kotlin
val imageLoader = ImageLoader.Builder(context)
    .availableMemoryPercentage(0.25)
    .crossfade(true)
    .build()
```

imageView.load 使用单例 `ImageLoader` 执行 `LoadRequest` 。 可以使用以下方式访问单例 `ImageLoader`：

``` kotlin
val imageLoader = Coil.imageLoader(context)
```

（可选）您可以创建自己的ImageLoader实例，并通过依赖项注入将它们注入：

``` kotlin
val imageLoader = ImageLoader(context)
```

当您创建单个 `ImageLoader` 并在整个应用程序中共享时，`Coil` 的性能最佳。 这是因为每个 `ImageLoader` 都有自己的内存缓存，bitmap pool 和网络监听



### Requests[¶](https://coil-kt.github.io/coil/#requests)

有两种 Request 类型

- [`LoadRequest`](https://coil-kt.github.io/coil/api/coil-base/coil.request/-load-request/) 是一个生命周期范围的 request，支持 [`Target`](https://coil-kt.github.io/coil/api/coil-base/coil.target/-target/)，[`Transition`](https://coil-kt.github.io/coil/api/coil-base/coil.transition/-transition/) 等等
- [`GetRequest`](https://coil-kt.github.io/coil/api/coil-base/coil.request/-get-request/) 挂起并返回  [`RequestResult`](https://coil-kt.github.io/coil/api/coil-base/coil.request/-request-result/)



如果要加载到自定义 target 中，可以执行 `LoadRequest`

``` kotlin
val request = LoadRequest.Builder(context)
    .data("https://www.example.com/image.jpg")
    .target { drawable ->
        // Handle the result.
    }
    .build()
imageLoader.execute(request)
```

要强制获取图像，请执行GetRequest：

``` kotlin
val request = GetRequest.Builder(context)
    .data("https://www.example.com/image.jpg")
    .build()
val drawable = imageLoader.execute(request).drawable
```



### 单例

如果您使用的是 `io.coil-kt:coil` ，您可以使用以下任意方式设置 `ImageLoader` 的实例



在 Application 中实现 `ImageLoaderFactory`（推荐）

``` kotlin
class MyApplication : Application(), ImageLoaderFactory {

    override fun newImageLoader(): ImageLoader {
        return ImageLoader.Builder(context)
            .crossfade(true)
            .okHttpClient {
                OkHttpClient.Builder()
                    .cache(CoilUtils.createDefaultCache(context))
                    .build()
            }
            .build()
    }
}
```



调用 Coil.setImageLoader

``` kotlin
val imageLoader = ImageLoader.Builder(context)
    .crossfade(true)
    .okHttpClient {
        OkHttpClient.Builder()
            .cache(CoilUtils.createDefaultCache(context))
            .build()
    }
    .build()
Coil.setImageLoader(imageLoader)
```

默认的 `ImageLoader` 可以通过这样取回

``` kotlin
val imageLoader = Coil.imageLoader(context)
```



设置默认的 `ImageLoader` 是可选的。 如果未设置，则 `Coil` 会延迟创建具有默认值的 `ImageLoader`



如果您使用的是 `io.coil-kt:coil-base`，您应创建自己的 `ImageLoader` 实例并通过依赖注入将它注入到 app 中



> 注意：如果设置自定义OkHttpClient，则必须设置缓存实现，否则ImageLoader将没有磁盘缓存。 可以使用CoilUtils.createDefaultCache 创建默认的 Coil 缓存实例



## 支持的数据类型

ImageLoader 支持的数据类型为

- String (mapped to a Uri)
- HttpUrl
- Uri (`android.resource`, `content`, `file`, `http`, and `https` schemes only)
- File
- @DrawableRes Int
- Drawable
- Bitmap



## 预加载

如果要预加载到内存中，执行一个不带 target 的 `LoadRequest`

``` kotlin
val request = LoadRequest.Builder(context)
    .data("https://www.example.com/image.jpg")
    // 可选的，但是设置 ViewSizeResolver 可以通过限制预加载的大小来节省内存
    .size(ViewSizeResolver(imageView))
    .build()
imageLoader.execute(request)
```



如果只想将网络图片预加载到磁盘中，可以为 request 关闭内存缓存

``` kotlin
val request = LoadRequest.Builder(context)
    .data("https://www.example.com/image.jpg")
    .memoryCachePolicy(CachePolicy.DISABLED)
    .build()
imageLoader.execute(request)
```



## 取消请求

`LoadRequest` 会自动取消在以下几种情况下

- 关联的 view detached，

- 关联的 lifecycle destroyed

- 另一个 request  在相同的 view 中开启



此外，每个 `LoadRequest` 返回一个 `RequestDisposable`，可用于检查请求是否在运行中或处理该请求（有效地取消请求并释放其关联资源）

``` kotlin
val disposable = imageView.load("https://www.example.com/image.jpg")

// Cancel the request.
disposable.dispose()
```



`GetRequest` 仅当协程的上下文被取消时才会取消



## 图片采样

假设磁盘上有一个 500x500 的映像，但是只需要以 100x100 的大小将其加载到内存中即可在视图中显示。 `Coil` 会将图像加载到内存中，但是如果您需要 500x500 的图像会怎样呢？ 从磁盘读取还有更好的「质量」，但是图像已经以 100x100 加载到内存中。 理想情况下，当我们从磁盘以 500x500 读取图像时，我们将使用 100x100 图像作为占位符。

这正是 `Coil` 所做的，并且 `Coil` 自动为所有 `BitmapDrawables` 处理此过程。 与 `crossfade(true)` 搭配使用时，可以创建视觉效果，使图像细节看起来像淡入淡出，类似于渐进式 JPEG

![](https://user-gold-cdn.xitu.io/2020/5/15/1721615615eb45ef?w=600&h=1067&f=gif&s=4714987)



## 使用要求

- AndroidX
- Min SDK 14+
- Compile SDK: 29+
- [Java 8+](https://coil-kt.github.io/coil/getting_started/#java-8)



详细内容移步 [官方文档](https://coil-kt.github.io/coil/)



## 关于我

我是 [Flywith24](https://flywith24.gitee.io/)，我的博客内容已经分类整理 [在这里](https://github.com/Flywith24/BlogList)，点击右上角的 Watch 可以及时获取我的文章更新哦 😉



- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)

- [小专栏](https://xiaozhuanlan.com/detail)

- [Github](https://github.com/Flywith24)

  

![](https://user-gold-cdn.xitu.io/2020/6/26/172ee567fb4fbf7e?w=1954&h=624&f=jpeg&s=115362)