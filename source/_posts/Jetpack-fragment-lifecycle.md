---
title: 从源码角度看Fragment的启动流程及生命周期 基于AndroidX Fragment1.2.2
date: 2020-03-10 00:10:35
categories: 
- 背上 Jetpack
tags: 
- androidx
- Jetpack
- MVVM
image: https://gitee.com/flywith24/Album/raw/master/img/20201015094458.png
---

从源码角度看Fragment的启动流程及生命周期 基于AndroidX Fragment1.2.2
<!-- more-->

> 笔者看过不少源码分析类的文章，动辄贴上大段代码，这种方式很容易打断读者的思路，所以很多时候看过这类文章感叹好文好文，却感觉什么都没记住，亦或者默默加入收藏却不知何时能去细心地研读。
>
> 所以本文不会过多介绍源码的细节，更多地是抛砖引玉，如果您看过本文后能够跟着本文的思路自己翻一下源码相信您就不会有我上述的体验了。
>
> 本文默认您已对 fragment 的生命周期有所了解，并清楚fragment的缘起与职责。这部分基础内容可移步 [fragment 官方文档](https://developer.android.com/guide/components/fragments) 
>
> **也即本文不会介绍 “what”，而是介绍 “how” 并且探讨一下 “why”**



这里贴一下 androidx fragment 源码地址

[androidx fragment 官方源码地址](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-master-dev/fragment/)



本文基于 androidx fragment 1.2.2 源码分析

``` groovy
implementation "androidx.fragment:fragment-ktx:1.2.2"
```



本文主要介绍fragment的启动流程，其他内容例如返回栈，会后续更新，敬请关注。欢迎在评论区下讨论。[本文demo](https://github.com/Flywith24/Flywith24-Fragment-Demo)



既然我们都知道 “what”，不妨我们来思考一下 “how”

## 分析前的思考

请大家思考一个问题，我们知道fragment 的生命周期是与其宿主 activity 的生命周期息息相关的，也即 activity 的每次生命周期回调都会引发每个fragment的类似回调。

那么，如果让我们来实现这样的操作，应该怎么做？



>  猜测：在activity每个生命周期的节点，去操作fragment，让其执行相应的生命周期方法。



思路有了，下面进行一些细节的确认。



1. activity 要能操作 fragment，fragment 亦可操作 fragment，所以需要抽象出一个管理 fragment 的模型
2. activity 操作 fragment 的一系列动作，应该是互为可逆一组操作。例如添加 fragment 后，也应能移除 fragment
3. activity 对 fragment 的每组操作不应是单一的，例如可以在一次操作中在 activity 不同位置添加两个 fragment，同时该操作还应满足 2 ，具有可逆性



对于第一条，我们抽象出一个可以管理 fragment 的模型，加入上下级的关系，即 activity 可管理其内部的 fragment，fragment 亦可管理其内部的 fragment。因此 fragment 同时充当着管理者与被管理者两种角色



对于后两条，相信在大学学过数据库的人会想到一种结构：**事务（Transaction）**



>  事务是指一组原子性的操作，这些操作是不可分割的整体，要么全完成，要么全不完成，完成后可以回滚到完成前的状态



因此，fragment 中两个最重要的概念出现了，`FragmentManager` 和 `FragmentTransaction`



`FragmentManager`  封装着对 fragment 操作的各种方法，`addFragment` `removeFragment` 等等，而 `FragmentActivity` 通过 `FragmentController` 来操作 `FragmentManager`  



`FragmentTransaction` 封装对 fragment 容器进行的 fragment 操作，例如在容器1内添加一个 fragment，同时在容器2内替换fragment。



它们均为抽象类，需要具体的实现类。



`FragmentManager` 的实现类为 `FragmentManagerImpl`，其内部逻辑已全部移至 `FragmentManager`  中，是个空实现。

`FragmentTransaction` 的实现类为 `BackStackRecord` ，其内部引用了 `FragmentManager` 的实例 ，同时重写了父类的 四个 `commit` 相关的方法。



## 看似最简单的启动流程



现在让我们看一部分代码，平时在activity中我们是这样填充一个fragment的

```kotlin
 override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        //避免旋转屏幕等场景 fragment 重叠的问题
        if (savedInstanceState == null) {
            supportFragmentManager//步骤1
                .beginTransaction()//步骤2
                .add(R.id.container, BlankFragment.newInstance())//步骤3
                .commitNow()//步骤4
        }
 }
```



- 步骤1，实例化 `FragmentManagerImpl` 对象 (内部经历了一些转换，详情参见源码或查看demo注释)

- 步骤2，实例化 `BackStackRecord`对象，并在构造器中传入 `FragmentManager` 实例

- 步骤3，调用事务方法，对 fragment 容器进行相应的操作，本例表示在 id 为 `container` 容器内添加 `BlankFragment`

- 步骤4，提交事务，交于 `FragmentManager` 处理

  

在 terminal 敲入 `adb shell setprop log.tag.FragmentManager VERBOSE` 可开启`FragmentManager`的日志功能，过滤 `FragmentManager` ，日志如下：


![单fragment启动日志](https://gitee.com/flywith24/Album/raw/master/img/20200324084150.png)


绿色部分为笔者手动添加的log，灰色和蓝色部分为 fragment 源码中的log

根据日志显示的流程，我们的猜测看似是正确的，“在 activity 每个生命周期的节点，去操作 fragment ，让其执行相应的生命周期方法”



其实这里是有干扰的，因为我们是在activity 的 `onCreate` 方法里 创建并提交 `FragmentTransaction` ，如果在 `onResume` 里调用呢？



![单fragment启动日志2](https://gitee.com/flywith24/Album/raw/master/img/20200324084240.png)


WTF！



或许，我们的猜测有问题？看似调用 `commitNow` 后 fragment 的生命流程是自发进行的



那如果我们把调用挪到 `onPause` 呢？

打开 activity 并按下 home 键

![单fragment启动日志-onPause](https://gitee.com/flywith24/Album/raw/master/img/20200324084122.png)



我知道好奇的读者会尝试在 `onStop` 中尝试一下，有惊喜。手动滑稽。



从这几段日志上来看，fragment 在提交事务后会自发进入自己的生命周期流程，而当其宿主 activity  生命周期发生变化时，fragment 的生命周期也跟随变化。

如果这么说比较抽象的话，我们可以看在 onPause 中显示fragment 的日志，当 Fragment 进入 onStart 生命周期后，如果是正常流程应该进入 onResume，但由于按下 home 键 activity进入onStop，fragment 也进入了 onStop 状态



因此，我们将之前的猜测进行扩展：



> 1. 在activity每个生命周期的节点，去操作fragment，让其执行相应的生命周期方法
> 2. FragmentTransaction  被提交后 fragment 会进入自己的生命周期流程，但受 1 约束



那么我们的源码解读就从两个方向入手



## Activity 操作 Fragment 生命周期



activity 是通过 `FragmentController`  操作  `FragmentManager`  进而操作 fragment 的。

具体点就是在 activity 各个生命周期节点通过调用 `FragmentController` 中的各个 `dispatch-` 方法进而调用 `FragmentManager` 中的各个 `dispatch-` 方法



``` java
//FragmentActivity.java
final FragmentController mFragments = FragmentController.createController(new HostCallbacks());

//以下代码省略部分逻辑
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    mFragments.dispatchCreate();
}

@Override
protected void onStart() {
    mFragments.dispatchStart();
}

//onResume 彻底执行完毕的回调
@Override
protected void onPostResume() {
    mFragments.dispatchResume();
}
@Override
protected void onPause() {
    mFragments.dispatchPause();
}

@Override
protected void onStop() {
    mFragments.dispatchStop();
}

@Override
protected void onDestroy() {
    super.onDestroy();
    mFragments.dispatchDestroy();
}
```



这样猜测 1 就被证实了

**activity 会在各个生命周期节点通过 `FragmentController` 间接调用 `FragmentManager`  中的 各种 `dispatch-` 方法，进而影响 fragment 的生命周期**



那么嵌套 fragment 呢？



嵌套 fragment 也应该是宿主使用 `FragmentManager`  中的各种 `dispatch-` 方法，基于这个想法我们可以看一下 `FragmentManager`  中 `dispatch-` 方法的调用

![dispatch-方法的引用](https://gitee.com/flywith24/Album/raw/master/img/20200324084349.png)


可以看到这里有两处调用，第二处为activity 通过 `FragmentController` 间接调用，第一处使用的是 `mChildFragmentManager`



**这里引出 fragment 中另外两个比较重要的概念，`getParentFragmentManager()` 和 `getChildFragmentManager()`**

> 注意：`requireFragmentManager()` 和 `getFragmentManager` 已弃用



`getChildFragmentManager() `获取的是fragment 中的 `mChildFragmentManager`

`getParentFragmentManager()` 获取的是fragment 中的 `mFragmentManager`



`mChildFragmentManager` 为fragment内部的 `fragmentManager`

``` java
// Private fragment manager for child fragments inside of this one.
@NonNull
FragmentManager mChildFragmentManager = new FragmentManagerImpl();
```



`mFragmentManager` 稍显复杂，

1. 如果 fragment 的直接宿主是 activity ，则返回的是 activity 中的`getSupportFragmentManager()` 返回的 `fragmentManager`
2. 如果 fragment 的直接宿主是 fragment，即该 fragment 是其他 fragment 的子 fragment，则返回的是其父 fragment 的 `getChildFragmentManager`



**所以 嵌套fragment 的生命周期是父 fragment 在各个生命周期节点上通过 `mChildFragmentManager` 调用 `dispatch-`  以影响其子 fragment 的生命周期**



这样我们第一部分的解读就告一段落了, 这里点到为止,一些细节需要您自己亲自看看源码



## Fragment 的生命周期自治

在 `看似最简单的启动流程` 一节中我们分别在 activity 的 onCreate ，onResume，onPause 中分别开启并提交事务，来观察 fragment 的生命周期日志。

在没有 activity 干扰的情况下，fragment 的生命周期是自治的。



那么我们继续思考一个问题



Fragment 的生命周期是如何一环扣一环的执行的？



从上面的日志，我们看到很多 “moveto-” 的日志，

**我们可以继续大胆地猜测，一个生命周期节点结束后调用进入另一个生命周期节点的方法**



基于这个猜测，我们确认一些细节



fragment 应该有自己的状态，它可能自己管理内部的状态，也可能会有封装着状态转移的逻辑的专门管理状态的抽象



这里引出另外一个概念 `FragmentStateManager`

`FragmentStateManager` 中持有 fragment 的引用 `mFragment` 以及 `FragmentManager` 的状态 `mFragmentManagerState`

这里fragment的状态值为：

``` java
static final int INITIALIZING = -1;    // Not yet attached.
static final int ATTACHED = 0;         // Attached to the host.
static final int CREATED = 1;          // Created.
static final int ACTIVITY_CREATED = 2; // Fully created, not started.
static final int STARTED = 3;          // Created and started, not resumed.
static final int RESUMED = 4;          // Created started and resumed.
```



`FragmentStateManager` 还封装着 fragment 状态转移的方法，例如：



```java
void activityCreated() {
    if (FragmentManager.isLoggingEnabled(Log.DEBUG)) {
        Log.d(TAG, "moveto ACTIVITY_CREATED: " + mFragment);
    }
    mFragment.performActivityCreated(mFragment.mSavedFragmentState);
}
void start() {
    if (FragmentManager.isLoggingEnabled(Log.DEBUG)) {
        Log.d(TAG, "moveto STARTED: " + mFragment);
    }
    mFragment.performStart();
}
```



fragment 生命周期自治的核心逻辑封装在 `FragmentManager` 中的 `void moveToState(@NonNull Fragment f, int newState)`  内，主要代码为(精简后)：

``` java
void moveToState(@NonNull Fragment f, int newState) {
    FragmentStateManager fragmentStateManager = mFragmentStore.getFragmentStateManager(f.mWho);

    newState = Math.min(newState, fragmentStateManager.computeMaxState());
    if (f.mState <= newState) {
        switch (f.mState) {
            case Fragment.INITIALIZING:
                if (newState > Fragment.INITIALIZING) {
                    fragmentStateManager.attach(mHost, this, mParent);
                }
            case Fragment.ATTACHED:
                if (newState > Fragment.ATTACHED) {
                    fragmentStateManager.create();
                }
            case Fragment.CREATED:
                if (newState > Fragment.INITIALIZING) {
                    fragmentStateManager.ensureInflatedView();
                }
                if (newState > Fragment.CREATED) {
                    fragmentStateManager.createView(mContainer);
                    fragmentStateManager.activityCreated();
                    fragmentStateManager.restoreViewState();
                }
            case Fragment.ACTIVITY_CREATED:
                if (newState > Fragment.ACTIVITY_CREATED) {
                    fragmentStateManager.start();
                }
            case Fragment.STARTED:
                if (newState > Fragment.STARTED) {
                    fragmentStateManager.resume();
                }
        }
    }
}
```



> 注意：这里的switch 没有 break



细心的读者可能发现了，fragment 中的状态怎么只到 resume ，后续的状态呢？

我们可以看一下 `FragmentManager` 中的 `dispatchPause` 方法

``` java
void dispatchPause() {
    dispatchStateChange(Fragment.STARTED);
}
```



为什么 dispatch 了 `STARTED` 的状态？其实刚刚 `moveToState` 方法我精简掉了一部分代码，留下的只有 `f.mState <= newState` 的逻辑，即 **dispatch 的新状态大于等于当前的状态**



而现在dispatch 的新状态比当前状态值小，则走了下面的逻辑，例如当前状态为 RESUMED ，新传递的状态为 STARTED，执行了 `fragmentStateManager.pause();` 



``` java
void moveToState(@NonNull Fragment f, int newState) {
    FragmentStateManager fragmentStateManager = mFragmentStore.getFragmentStateManager(f.mWho);

    newState = Math.min(newState, fragmentStateManager.computeMaxState());
    if (f.mState <= newState) {
    	//省略...   
    }else if (f.mState > newState) {
        switch (f.mState) {
            case Fragment.RESUMED:
                if (newState < Fragment.RESUMED) {
                    fragmentStateManager.pause();
                }
                
            case Fragment.STARTED:
                if (newState < Fragment.STARTED) {
                    fragmentStateManager.stop();
                }
                
            case Fragment.ACTIVITY_CREATED:
                if (newState < Fragment.ACTIVITY_CREATED) {
                    if (isLoggingEnabled(Log.DEBUG)) {
                        Log.d(TAG, "movefrom ACTIVITY_CREATED: " + f);
                    }
                    if (f.mView != null) {
                        if (mHost.onShouldSaveFragmentState(f) && f.mSavedViewState == null) {
                            fragmentStateManager.saveViewState();
                        }
                    }
                    if (mExitAnimationCancellationSignals.get(f) == null) {
                        destroyFragmentView(f);
                    } else {
                        f.setStateAfterAnimating(newState);
                    }
                }
            case Fragment.CREATED:
                if (newState < Fragment.CREATED) {
                    boolean beingRemoved = f.mRemoving && !f.isInBackStack();
                    if (beingRemoved || mNonConfig.shouldDestroy(f)) {
                        makeInactive(fragmentStateManager);
                    } else {
                        if (f.mTargetWho != null) {
                            Fragment target = findActiveFragment(f.mTargetWho);
                            if (target != null && target.getRetainInstance()) {
                                f.mTarget = target;
                            }
                        }
                    }
                    if (mExitAnimationCancellationSignals.get(f) != null) {
                        f.setStateAfterAnimating(newState);
                        newState = Fragment.CREATED;
                    } else {
                        fragmentStateManager.destroy(mHost, mNonConfig);
                    }
                }
            case Fragment.ATTACHED:
                if (newState < Fragment.ATTACHED) {
                    fragmentStateManager.detach(mNonConfig);
                }
        }
    }
}
```



> 注意：这里的switch 还是没有 break



这里有个细节，由于activity没有 onDestroyView 的生命周期，所以 `FragmentController` 中的 `dispatchDestroyView` 是没有调用的

![dispatchOnDestroyView](https://gitee.com/flywith24/Album/raw/master/img/20200324084417.png)


 在 activity 中的 destroy 方法中通过 `fragmentController` 调用了 `dispatchDestroy` 内部调用 `dispatchStateChange(Fragment.INITIALIZING)` ，而此时的fragment 的 mState 为 `ACTIVITY_CREATED`，所以 `moveToState` 方法会走到 `ACTIVITY_CREATED` 的 case 并执行到底



这样 fragment 最简单场景的生命周期就结束了




## 总结

我们做一个总结：activity 和 fragment 会在各个生命周期节点通过被调用 fragment 的 `parentFragmentManager`（或者说父 fragment 的 `childFragmentManager` 和 activity 的 `supportFragmentManager`）中的各种 `dispatch-` 方法以影响子 fragment 的 生命周期，同时子 fragment 也拥有自己生命周期的调用链（从状态A转移至状态B）



不得不说 fragment 的很多 API 并不是很好用，从 androidx fragment 的更新频率也可以看出。比如 fragment 中的 view 和 fragment本身的生命周期是不一致的，存在onDestroyView 但 fragment没有销毁的情况

[Ian Lake](https://medium.com/@ianhlake) 在 [Fragments: Past, Present, and Future (Android Dev Summit '19)](https://www.youtube.com/watch?v=RS1IACnZLy4) 中提到未来官方会将二者合并，届时 fragment 的使用会更加简洁



这里引用 [The Android Lifecycle cheat sheet — part III : Fragments](https://medium.com/androiddevelopers/the-android-lifecycle-cheat-sheet-part-iii-fragments-afc87d4f37fd) 文中的图片 ，和我画的commit `FragmentTransaction`  的脑图（略简陋），帮您更好的理解



![ [The Android Lifecycle cheat sheet — part III : Fragments](https://medium.com/androiddevelopers/the-android-lifecycle-cheat-sheet-part-iii-fragments-afc87d4f37fd)](https://gitee.com/flywith24/Album/raw/master/img/20200324084444.png)


![fragment脑图](https://gitee.com/flywith24/Album/raw/master/img/20200324084515.png)



强烈建议您自己亲自看一看源码，不然就变为我文章开头时说的状态了。

---

## 关于我

---

我是 [Fly_with24](https://flywith24.gitee.io/)

- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)

- [简书](https://www.jianshu.com/u/3d5ad6043d66)

- [Github](https://github.com/Flywith24)