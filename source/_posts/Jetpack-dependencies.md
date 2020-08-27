---
title: 【背上Jetpack】Jetpack 主要依赖关系
date: 2020-03-01 00:10:35
categories: 
- Jetpack
tags: 
- androidx
- Jetpack
- MVVM
---

> 在学习和使用 `jetpack` 组件时，总是被其 gradle 依赖搞的晕头转向，故在此整理 `jetpack` 主要组件的依赖，及传递关系

<!-- more-->

- [`jetpcak` 组件源码地址](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-master-dev)
- 查询 `jetpcak` 组件 版本: [Google's Maven Repository](https://maven.google.com/web/index.html)
- 查看依赖树：在项目根目录下执行`./gradlew :app:dependencies`


**可直接跳过后续内容前往最后一节查看总结**

关于 `androidx`下 `fragment/activity`的变化，可查看该文, [【译】AdroidX下使用Activity和Fragment的变化](https://juejin.im/post/5e5a0c316fb9a07cd248d29e)

### Appcompat

#### 引入
``` groovy
dependencies {
    def appcompat_version = "1.1.0"

    implementation "androidx.appcompat:appcompat:$appcompat_version"
    // For loading and tinting drawables on older versions of the platform
    implementation "androidx.appcompat:appcompat-resources:$appcompat_version"
}
```

#### 依赖树

``` groovy
+--- androidx.appcompat:appcompat:1.1.0
|    +--- androidx.annotation:annotation:1.1.0
|    +--- androidx.core:core:1.1.0
|    |    +--- androidx.annotation:annotation:1.1.0
|    |    +--- androidx.lifecycle:lifecycle-runtime:2.0.0 -> 2.1.0
|    |    |    +--- androidx.lifecycle:lifecycle-common:2.1.0
|    |    |    |    \--- androidx.annotation:annotation:1.1.0
|    |    |    +--- androidx.arch.core:core-common:2.1.0
|    |    |    |    \--- androidx.annotation:annotation:1.1.0
|    |    |    \--- androidx.annotation:annotation:1.1.0
|    |    +--- androidx.versionedparcelable:versionedparcelable:1.1.0
|    |    |    +--- androidx.annotation:annotation:1.1.0
|    |    |    \--- androidx.collection:collection:1.0.0 -> 1.1.0
|    |    |         \--- androidx.annotation:annotation:1.1.0
|    |    \--- androidx.collection:collection:1.0.0 -> 1.1.0 (*)
|    +--- androidx.cursoradapter:cursoradapter:1.0.0
|    |    \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    +--- androidx.fragment:fragment:1.1.0
|    |    +--- androidx.annotation:annotation:1.1.0
|    |    +--- androidx.core:core:1.1.0 (*)
|    |    +--- androidx.collection:collection:1.1.0 (*)
|    |    +--- androidx.viewpager:viewpager:1.0.0
|    |    |    +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |    |    +--- androidx.core:core:1.0.0 -> 1.1.0 (*)
|    |    |    \--- androidx.customview:customview:1.0.0
|    |    |         +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |    |         \--- androidx.core:core:1.0.0 -> 1.1.0 (*)
|    |    +--- androidx.loader:loader:1.0.0
|    |    |    +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |    |    +--- androidx.core:core:1.0.0 -> 1.1.0 (*)
|    |    |    +--- androidx.lifecycle:lifecycle-livedata:2.0.0
|    |    |    |    +--- androidx.arch.core:core-runtime:2.0.0
|    |    |    |    |    +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |    |    |    |    \--- androidx.arch.core:core-common:2.0.0 -> 2.1.0 (*)
|    |    |    |    +--- androidx.lifecycle:lifecycle-livedata-core:2.0.0
|    |    |    |    |    +--- androidx.lifecycle:lifecycle-common:2.0.0 -> 2.1.0 (*)
|    |    |    |    |    +--- androidx.arch.core:core-common:2.0.0 -> 2.1.0 (*)
|    |    |    |    |    \--- androidx.arch.core:core-runtime:2.0.0 (*)
|    |    |    |    \--- androidx.arch.core:core-common:2.0.0 -> 2.1.0 (*)
|    |    |    \--- androidx.lifecycle:lifecycle-viewmodel:2.0.0 -> 2.1.0
|    |    |         \--- androidx.annotation:annotation:1.1.0
|    |    +--- androidx.activity:activity:1.0.0
|    |    |    +--- androidx.annotation:annotation:1.1.0
|    |    |    +--- androidx.core:core:1.1.0 (*)
|    |    |    +--- androidx.lifecycle:lifecycle-runtime:2.1.0 (*)
|    |    |    +--- androidx.lifecycle:lifecycle-viewmodel:2.1.0 (*)
|    |    |    \--- androidx.savedstate:savedstate:1.0.0
|    |    |         +--- androidx.annotation:annotation:1.1.0
|    |    |         +--- androidx.arch.core:core-common:2.0.1 -> 2.1.0 (*)
|    |    |         \--- androidx.lifecycle:lifecycle-common:2.0.0 -> 2.1.0 (*)
|    |    \--- androidx.lifecycle:lifecycle-viewmodel:2.0.0 -> 2.1.0 (*)
|    +--- androidx.appcompat:appcompat-resources:1.1.0
|    |    +--- androidx.annotation:annotation:1.1.0
|    |    +--- androidx.core:core:1.0.1 -> 1.1.0 (*)
|    |    +--- androidx.vectordrawable:vectordrawable:1.1.0
|    |    |    +--- androidx.annotation:annotation:1.1.0
|    |    |    +--- androidx.core:core:1.1.0 (*)
|    |    |    \--- androidx.collection:collection:1.1.0 (*)
|    |    +--- androidx.vectordrawable:vectordrawable-animated:1.1.0
|    |    |    +--- androidx.vectordrawable:vectordrawable:1.1.0 (*)
|    |    |    +--- androidx.interpolator:interpolator:1.0.0
|    |    |    |    \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |    |    \--- androidx.collection:collection:1.1.0 (*)
|    |    \--- androidx.collection:collection:1.0.0 -> 1.1.0 (*)
|    +--- androidx.drawerlayout:drawerlayout:1.0.0
|    |    +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
|    |    +--- androidx.core:core:1.0.0 -> 1.1.0 (*)
|    |    \--- androidx.customview:customview:1.0.0 (*)
|    \--- androidx.collection:collection:1.0.0 -> 1.1.0 (*)

```

#### 传递依赖

`androidx.annotation:annotation:1.1.0`

`androidx.core:core:1.1.0`

` androidx.cursoradapter:cursoradapter:1.0.0`

`androidx.fragment:fragment:1.1.0`

` androidx.appcompat:appcompat-resources:1.1.0`

`androidx.drawerlayout:drawerlayout:1.0.0`

`androidx.collection:collection:1.0.0`


>**`appcompat` 中默认引入了 `fragment` 库，如果想使用更新版本的 `fragment` 库，可以单独引用**

> [appcompat build.gradle 源码地址](https://android.googlesource.com/platform/frameworks/support/+/b0b973c9bc89f9ad3efe3c06e41b72957f032a99/appcompat/build.gradle)

### Fragment

#### 引入

``` groovy
dependencies {
    def fragment_version = "1.2.2"

    // Java language implementation
    implementation "androidx.fragment:fragment:$fragment_version"
    // Kotlin
    implementation "androidx.fragment:fragment-ktx:$fragment_version"
    // Testing Fragments in Isolation
    implementation "androidx.fragment:fragment-testing:$fragment_version"
}
```

> ⚠️ Note: The Kotlin dependant libraries of this version (fragment-ktx,fragment-testing) target Java 8 programming language bytecode. [Please read Use Java 8 language features to learn how to use it in your project.](https://developer.android.com/studio/write/java8-support)

#### 依赖树

``` groovy
 androidx.fragment:fragment-ktx:1.2.2
     +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.50 -> 1.3.61 (*)
     +--- androidx.fragment:fragment:[1.2.2] -> 1.2.2
     |    +--- androidx.annotation:annotation:1.1.0
     |    +--- androidx.core:core:1.1.0
     |    |    +--- androidx.annotation:annotation:1.1.0
     |    |    +--- androidx.lifecycle:lifecycle-runtime:2.0.0 -> 2.2.0
     |    |    |    +--- androidx.lifecycle:lifecycle-common:2.2.0
     |    |    |    |    \--- androidx.annotation:annotation:1.1.0
     |    |    |    +--- androidx.arch.core:core-common:2.1.0
     |    |    |    |    \--- androidx.annotation:annotation:1.1.0
     |    |    |    \--- androidx.annotation:annotation:1.1.0
     |    |    +--- androidx.versionedparcelable:versionedparcelable:1.1.0
     |    |    |    +--- androidx.annotation:annotation:1.1.0
     |    |    |    \--- androidx.collection:collection:1.0.0 -> 1.1.0
     |    |    |         \--- androidx.annotation:annotation:1.1.0
     |    |    \--- androidx.collection:collection:1.0.0 -> 1.1.0 (*)
     |    +--- androidx.collection:collection:1.1.0 (*)
     |    +--- androidx.viewpager:viewpager:1.0.0
     |    |    +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
     |    |    +--- androidx.core:core:1.0.0 -> 1.1.0 (*)
     |    |    \--- androidx.customview:customview:1.0.0
     |    |         +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
     |    |         \--- androidx.core:core:1.0.0 -> 1.1.0 (*)
     |    +--- androidx.loader:loader:1.0.0
     |    |    +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
     |    |    +--- androidx.core:core:1.0.0 -> 1.1.0 (*)
     |    |    +--- androidx.lifecycle:lifecycle-livedata:2.0.0
     |    |    |    +--- androidx.arch.core:core-runtime:2.0.0 -> 2.1.0
     |    |    |    |    +--- androidx.annotation:annotation:1.1.0
     |    |    |    |    \--- androidx.arch.core:core-common:[2.1.0] -> 2.1.0 (*)
     |    |    |    +--- androidx.lifecycle:lifecycle-livedata-core:2.0.0 -> 2.2.0
     |    |    |    |    +--- androidx.lifecycle:lifecycle-common:2.2.0 (*)
     |    |    |    |    +--- androidx.arch.core:core-common:2.1.0 (*)
     |    |    |    |    \--- androidx.arch.core:core-runtime:2.1.0 (*)
     |    |    |    \--- androidx.arch.core:core-common:2.0.0 -> 2.1.0 (*)
     |    |    \--- androidx.lifecycle:lifecycle-viewmodel:2.0.0 -> 2.2.0
     |    |         \--- androidx.annotation:annotation:1.1.0
     |    +--- androidx.activity:activity:1.1.0
     |    |    +--- androidx.annotation:annotation:1.1.0
     |    |    +--- androidx.core:core:1.1.0 (*)
     |    |    +--- androidx.lifecycle:lifecycle-runtime:2.2.0 (*)
     |    |    +--- androidx.lifecycle:lifecycle-viewmodel:2.2.0 (*)
     |    |    +--- androidx.savedstate:savedstate:1.0.0
     |    |    |    +--- androidx.annotation:annotation:1.1.0
     |    |    |    +--- androidx.arch.core:core-common:2.0.1 -> 2.1.0 (*)
     |    |    |    \--- androidx.lifecycle:lifecycle-common:2.0.0 -> 2.2.0 (*)
     |    |    \--- androidx.lifecycle:lifecycle-viewmodel-savedstate:1.0.0 -> 2.2.0
     |    |         +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
     |    |         +--- androidx.savedstate:savedstate:1.0.0 (*)
     |    |         +--- androidx.lifecycle:lifecycle-livedata-core:2.2.0 (*)
     |    |         \--- androidx.lifecycle:lifecycle-viewmodel:2.2.0 (*)
     |    +--- androidx.lifecycle:lifecycle-livedata-core:2.2.0 (*)
     |    +--- androidx.lifecycle:lifecycle-viewmodel:2.2.0 (*)
     |    \--- androidx.lifecycle:lifecycle-viewmodel-savedstate:2.2.0 (*)
     +--- androidx.activity:activity-ktx:1.1.0
     |    +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.50 -> 1.3.61 (*)
     |    +--- androidx.activity:activity:[1.1.0] -> 1.1.0 (*)
     |    +--- androidx.core:core-ktx:1.1.0
     |    |    +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.31 -> 1.3.61 (*)
     |    |    +--- androidx.annotation:annotation:1.1.0
     |    |    \--- androidx.core:core:1.1.0 (*)
     |    +--- androidx.lifecycle:lifecycle-runtime-ktx:2.2.0
     |    |    +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.50 -> 1.3.61 (*)
     |    |    +--- org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.0
     |    |    |    +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.50 -> 1.3.61 (*)
     |    |    |    \--- org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.0
     |    |    |         +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.50 -> 1.3.61 (*)
     |    |    |         \--- org.jetbrains.kotlin:kotlin-stdlib-common:1.3.50 -> 1.3.61
     |    |    +--- androidx.lifecycle:lifecycle-runtime:2.2.0 (*)
     |    |    \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
     |    \--- androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0
     |         +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.50 -> 1.3.61 (*)
     |         +--- org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.0 (*)
     |         \--- androidx.lifecycle:lifecycle-viewmodel:2.2.0 (*)
     +--- androidx.core:core-ktx:1.1.0 (*)
     +--- androidx.collection:collection-ktx:1.1.0
     |    +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.20 -> 1.3.61 (*)
     |    \--- androidx.collection:collection:1.1.0 (*)
     +--- androidx.lifecycle:lifecycle-livedata-core-ktx:2.2.0
     |    +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.50 -> 1.3.61 (*)
     |    \--- androidx.lifecycle:lifecycle-livedata-core:2.2.0 (*)
     \--- androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0 (*)
```

#### 传递依赖

`org.jetbrains.kotlin:kotlin-stdlib:1.3.50`

`androidx.activity:activity-ktx:1.1.0`

`androidx.core:core-ktx:1.1.0`

`androidx.collection:collection-ktx:1.1.0`

`androidx.lifecycle:lifecycle-livedata-core-ktx:2.2.0`

`androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0`

> **`fragment` 库默认引入了 `activity` `core-ktx` `lifecycle-livedata-core-ktx` `lifecycle-viewmodel-ktx` 库**  
> 
> [fragment build.grdle 源码地址](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-master-dev/fragment/fragment-ktx/build.gradle)

### Activity

#### 引入
``` groovy
dependencies {
    def activity_version = "1.1.0"

    // Java language implementation
    implementation "androidx.activity:activity:$activity_version"
    // Kotlin
    implementation "androidx.activity:activity-ktx:$activity_version"
}
```


> ⚠️ Note: The Kotlin dependant libraries of this version (activity-ktx) target Java 8 programming language bytecode. [Please read Use Java 8 language features to learn how to use it in your project.](https://developer.android.com/studio/write/java8-support)

#### 依赖树

``` groovy
androidx.activity:activity-ktx:1.1.0
     +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.50 -> 1.3.61 (*)
     +--- androidx.activity:activity:[1.1.0] -> 1.1.0
     |    +--- androidx.annotation:annotation:1.1.0
     |    +--- androidx.core:core:1.1.0
     |    |    +--- androidx.annotation:annotation:1.1.0
     |    |    +--- androidx.lifecycle:lifecycle-runtime:2.0.0 -> 2.2.0
     |    |    |    +--- androidx.lifecycle:lifecycle-common:2.2.0
     |    |    |    |    \--- androidx.annotation:annotation:1.1.0
     |    |    |    +--- androidx.arch.core:core-common:2.1.0
     |    |    |    |    \--- androidx.annotation:annotation:1.1.0
     |    |    |    \--- androidx.annotation:annotation:1.1.0
     |    |    +--- androidx.versionedparcelable:versionedparcelable:1.1.0
     |    |    |    +--- androidx.annotation:annotation:1.1.0
     |    |    |    \--- androidx.collection:collection:1.0.0
     |    |    |         \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
     |    |    \--- androidx.collection:collection:1.0.0 (*)
     |    +--- androidx.lifecycle:lifecycle-runtime:2.2.0 (*)
     |    +--- androidx.lifecycle:lifecycle-viewmodel:2.2.0
     |    |    \--- androidx.annotation:annotation:1.1.0
     |    +--- androidx.savedstate:savedstate:1.0.0
     |    |    +--- androidx.annotation:annotation:1.1.0
     |    |    +--- androidx.arch.core:core-common:2.0.1 -> 2.1.0 (*)
     |    |    \--- androidx.lifecycle:lifecycle-common:2.0.0 -> 2.2.0 (*)
     |    \--- androidx.lifecycle:lifecycle-viewmodel-savedstate:1.0.0
     |         +--- androidx.annotation:annotation:1.0.0 -> 1.1.0
     |         +--- androidx.savedstate:savedstate:1.0.0 (*)
     |         +--- androidx.lifecycle:lifecycle-livedata-core:2.2.0
     |         |    +--- androidx.lifecycle:lifecycle-common:2.2.0 (*)
     |         |    +--- androidx.arch.core:core-common:2.1.0 (*)
     |         |    \--- androidx.arch.core:core-runtime:2.1.0
     |         |         +--- androidx.annotation:annotation:1.1.0
     |         |         \--- androidx.arch.core:core-common:[2.1.0] -> 2.1.0 (*)
     |         \--- androidx.lifecycle:lifecycle-viewmodel:2.2.0 (*)
     +--- androidx.core:core-ktx:1.1.0
     |    +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.31 -> 1.3.61 (*)
     |    +--- androidx.annotation:annotation:1.1.0
     |    \--- androidx.core:core:1.1.0 (*)
     +--- androidx.lifecycle:lifecycle-runtime-ktx:2.2.0
     |    +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.50 -> 1.3.61 (*)
     |    +--- org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.0
     |    |    +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.50 -> 1.3.61 (*)
     |    |    \--- org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.0
     |    |         +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.50 -> 1.3.61 (*)
     |    |         \--- org.jetbrains.kotlin:kotlin-stdlib-common:1.3.50 -> 1.3.61
     |    +--- androidx.lifecycle:lifecycle-runtime:2.2.0 (*)
     |    \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
     \--- androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0
          +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.50 -> 1.3.61 (*)
          +--- org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.0 (*)
          \--- androidx.lifecycle:lifecycle-viewmodel:2.2.0 (*)

```

#### 依赖传递
`org.jetbrains.kotlin:kotlin-stdlib:1.3.50`

`androidx.core:core-ktx:1.1.0`

`androidx.lifecycle:lifecycle-runtime-ktx:2.2.0`

`androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0`

> [activity build.gradle 源码地址](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-master-dev/activity/activity-ktx/build.gradle)

### Core

#### 引入

``` groovy
dependencies {
    def core_version = "1.2.0"

    // Java language implementation
    implementation "androidx.core:core:$core_version"
    // Kotlin
    implementation "androidx.core:core-ktx:$core_version"

    // To use RoleManagerCompat
    implementation "androidx.core:core-role:1.0.0-alpha01"
}
```

#### 依赖树
``` groovy
androidx.core:core-ktx:1.2.0
     +--- org.jetbrains.kotlin:kotlin-stdlib:1.3.41
     |    +--- org.jetbrains.kotlin:kotlin-stdlib-common:1.3.41
     |    \--- org.jetbrains:annotations:13.0
     +--- androidx.annotation:annotation:1.1.0
     \--- androidx.core:core:1.2.0
          +--- androidx.annotation:annotation:1.1.0
          +--- androidx.lifecycle:lifecycle-runtime:2.0.0
          |    +--- androidx.lifecycle:lifecycle-common:2.0.0
          |    |    \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
          |    +--- androidx.arch.core:core-common:2.0.0
          |    |    \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
          |    \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
          +--- androidx.versionedparcelable:versionedparcelable:1.1.0
          |    +--- androidx.annotation:annotation:1.1.0
          |    \--- androidx.collection:collection:1.0.0
          |         \--- androidx.annotation:annotation:1.0.0 -> 1.1.0
          \--- androidx.collection:collection:1.0.0 (*)

```

### Lifecycle

#### 引入
``` groovy
dependencies {
    def lifecycle_version = "2.2.0"
    def arch_version = "2.1.0"

    // ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
    // LiveData
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:$lifecycle_version"
    // Lifecycles only (without ViewModel or LiveData)
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:$lifecycle_version"

    // Saved state module for ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel-savedstate:$lifecycle_version"

    // Annotation processor
    kapt "androidx.lifecycle:lifecycle-compiler:$lifecycle_version"
    // alternately - if using Java8, use the following instead of lifecycle-compiler
    implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"

    // optional - helpers for implementing LifecycleOwner in a Service
    implementation "androidx.lifecycle:lifecycle-service:$lifecycle_version"

    // optional - ProcessLifecycleOwner provides a lifecycle for the whole application process
    implementation "androidx.lifecycle:lifecycle-process:$lifecycle_version"

    // optional - ReactiveStreams support for LiveData
    implementation "androidx.lifecycle:lifecycle-reactivestreams-ktx:$lifecycle_version"

    // optional - Test helpers for LiveData
    testImplementation "androidx.arch.core:core-testing:$arch_version"
}
```

> - ⚠️ **`lifecycle-extensions` 已废弃，如果使用 `LifecycleService ` 请依赖 `lifecycle-service`；如果使用 `ProcessLifecycleOwner` 请依赖 `lifecycle-process`。`lifecycle-extensionsl`不会有2.3.0版本**
> - **2.1.0 后 `ViewModelProviders.of()` 被废弃。您可以在 `FragmentActivity` 或者 `Fragment` 使用 `ViewModelProvider(ViewModelStoreOwner)` 构造器来实现相同的功能。（`Fragment` 库 1.2.0以上）**

### Navigation

#### 引入

``` groovy
dependencies {
  def nav_version = "2.3.0-alpha02"

  // Java language implementation
  implementation "androidx.navigation:navigation-fragment:$nav_version"
  implementation "androidx.navigation:navigation-ui:$nav_version"

  // Kotlin
  implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
  implementation "androidx.navigation:navigation-ui-ktx:$nav_version"

  // Dynamic Feature Module Support
  implementation "androidx.navigation:navigation-dynamic-features-fragment:$nav_version"

  // Testing Navigation
  androidTestImplementation "androidx.navigation:navigation-testing:$nav_version"
}
```

### Paging

#### 引入

``` groovy
dependencies {
  def paging_version = "2.1.1"

  implementation "androidx.paging:paging-runtime:$paging_version" // For Kotlin use paging-runtime-ktx

  // alternatively - without Android dependencies for testing
  testImplementation "androidx.paging:paging-common:$paging_version" // For Kotlin use paging-common-ktx

  // optional - RxJava support
  implementation "androidx.paging:paging-rxjava2:$paging_version" // For Kotlin use paging-rxjava2-ktx
}
```

### Room

#### 引入

``` groovy
dependencies {
  def room_version = "2.2.4"

  implementation "androidx.room:room-runtime:$room_version"
  annotationProcessor "androidx.room:room-compiler:$room_version" // For Kotlin use kapt instead of annotationProcessor

  // optional - Kotlin Extensions and Coroutines support for Room
  implementation "androidx.room:room-ktx:$room_version"

  // optional - RxJava support for Room
  implementation "androidx.room:room-rxjava2:$room_version"

  // optional - Guava support for Room, including Optional and ListenableFuture
  implementation "androidx.room:room-guava:$room_version"

  // Test helpers
  testImplementation "androidx.room:room-testing:$room_version"
}
```

> ⚠️ Note: For Kotlin-based apps, make sure you use kapt instead of annotationProcessor. You should also add the kotlin-kapt plugin.

### 总结

#### 版本说明
> `androidx`库遵循严格的语义版本控制。版本字符串（例如 1.0.1-beta02）包含三个数字，分别代表 major 级别、minor 级别和问题修复级别。预发布版本也有一个后缀，用于指定预发布阶段（Alpha 版、Beta 版、候选版本）和版本号（01、02 等）。

库的每个版本都要经历三个预发布阶段，才能成为稳定版本。各预发布阶段的标准如下：

**Alpha 版**
- Alpha 版功能稳定，但功能可能不完整。
- 在版本处于 Alpha 版状态时，可以添加、移除或更改 API。


**Beta 版**

- Beta 版功能稳定，并且具有功能完整的 API Surface。
它们可以投入实际使用，但可能包含错误。
- Beta 版无法使用实验性编译器功能（例如 @UseExperimental）。
- 其他库的依赖项必须为 Beta 版、RC 版或稳定版。不允许使用 Alpha 版依赖项。

**候选版本 (RC)**

- 候选版本是未来的稳定版。
- 此版本可能包含在最后一刻提供的重要修复。
- 此版本的 API Surface 无法更改。
- 其他库的依赖项只能是 RC 版或稳定版。

一个库可以同时具有多个版本。每个版本都具有不同的发布阶段。例如，虽然 androidx.activity 的稳定版可以是 1.0.0，但也可能还有 1.1.0-beta02 版本以及 2.0.0-alpha01 版本。

#### kotlin 协程的使用

> `ViewModel` `LiveData` `Activity` `Fragment` `Service`等均可使用协程

- 对于 `ViewModelScope`，请使用 `androidx.lifecycle:lifecycle-viewmodel-ktx:2.1.0-beta01` 或更高版本。
- 对于 `LifecycleScope`，请使用 `androidx.lifecycle:lifecycle-runtime-ktx:2.2.0-alpha01` 或更高版本。
- 对于 `liveData`，请使用 `androidx.lifecycle:lifecycle-livedata-ktx:2.2.0-alpha01` 或更高版本。 

``` kotlin
class MyViewModel: ViewModel() {
    init {
        viewModelScope.launch {
            // Coroutine that will be canceled when the ViewModel is cleared.
        }
    }
}
```

``` kotlin 
class MyFragment : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        viewLifecycleOwner.lifecycleScope.launch {
            val params = TextViewCompat.getTextMetricsParams(textView)
            val precomputedText = withContext(Dispatchers.Default) {
                PrecomputedTextCompat.create(longTextContent, params)
            }
            TextViewCompat.setPrecomputedText(textView, precomputedText)
        }
    }
}
```

``` kotlin 
val user: LiveData<User> = liveData {
    val data = database.loadUser() // loadUser is a suspend function.
    emit(data)
}
    
```

#### 依赖关系
- 带有`-ktx`的库拥有 `kotlin` 的特性，有很多常用的扩展函数

- 带有`-ktx`的库依赖 java 版本， 带有`-ktx`和 java 版本各自单独使用即可

- `appcompat` 库包含 `fragmnet`,`fragment` 包含 `activity` ，当你引入`androidx` `appcompat` 库便可以使用`androidx` `fragment` 和 `androidx` `activity`

- 受限于`appcompat` 稳定版的更新速度，您可以选择单独使用`androidx` `fragment`/`activity`

- `AppCompatActivity` 继承自 `FragmentActivity`， `AppCompatActivity`可以直接使用 `fragment`

- `DataBinding` `ViewBinding` 依赖于 `android build gradle` 插件 ,无需引入其他依赖

- `androidx` `fragment`/`activity`下均依赖的`ViewModel`和 `LiveData`，您可以直接使用

- 使用`ViewModel`和 `LiveData`完整功能需要单独引入，它们都是`lifecycle`大家族下的

- `lifecycle-extensions` 已废弃，不要使用它了

- 如果想使用实现`LifecycleOwner` 的 `Service` ，需要引入 `lifecycle-service`


---
### 关于我
---
我是 [Fy_with24](http://www.yangyunzhao.com)
- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)
- [简书](https://www.jianshu.com/u/3d5ad6043d66)
- [Github](https://github.com/Flywith24)

