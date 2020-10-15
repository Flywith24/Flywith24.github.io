---
title: 从源码的角度看 Fragment 返回栈
date: 2020-03-16 00:10:35
categories: 
- 背上 Jetpack
tags: 
- androidx
- Jetpack
- MVVM
image: https://gitee.com/flywith24/Album/raw/master/img/20201015095213.png
---

从源码的角度看 Fragment 返回栈。

<!-- more-->

## 前言

> [上一篇](https://juejin.im/post/5e6bae35f265da572a0d11ad) 我们介绍了 `OnBackPressedDispather` ，那么今天我们来正式地从源码的角度看看 fragment 的返回栈吧。由于其主流程和生命周期差不多，因此本文将详细地分析返回栈相关的源码，并插入大量源码。建议将生命周期流程熟悉后阅读本文。文末提供单返回栈和多返回栈的 [demo](https://github.com/Flywith24/Flywith24-Fragment-Demo)



如果您对 activity 对任务栈和返回栈不是很了解，可以移步  [Tasks and the Back Stack](https://medium.com/androiddevelopers/tasks-and-the-back-stack-dbb7c3b0f6d4)



## 小问号你是否有很多朋友？

在分析源码之前，我们先来思考几个问题。

- 返回栈中的元素是什么？
- 谁来管理 fragment 的返回栈？
- 如何返回？



### 返回栈中的元素是什么？

返回栈，顾名思义，是一个栈结构。所以我们要搞清楚，这个栈结构到底存的是什么。



我们都知道，使用 fragment 的返回栈需要调用 `addToBackStack("")` 方法



在 [从源码角度看 Fragment 生命周期](https://juejin.im/post/5e67523551882549003d2c4f) 一文中，我们提到了 FragmentTransaction ，它是一个「事务」的模型，事务可以回滚到之前的状态。所以当触发返回操作时，就是将之前提交的事务进行回滚。

`FragmentTransaction` 的实现类为 `BackStackRecord` ，所以 **fragment 的返回栈其实存放的就是 BackStackRecord** 



作为返回栈的元素，BackStackRecord 实现了FragmentManager.BackStackEntry 接口



![BackStackRecord](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf0e912bd4?w=748&h=171&f=png&s=27904)



从 `BackStackRecord` 的定义我们可以发现 `BackStackRecord` 有三种身份

- 继承了 `FragmentTransaction`，即是事务，保存了整个事务的全部操作
- 实现了 `FragmentManager.BackStackEntry` ，作为回退栈的元素
- 实现了`OpGenerator` ，可以生成 `BackStackRecord` 列表，后文详细介绍



### 谁来管理 fragment 的返回栈？

我们已经知道 fragment 的返回栈其实存放的是 BackSrackRecord , 那么谁来管理 fragment 的返回栈？



`FragmentManager` 用于管理 fragment ，所以 **fragment 返回栈也应该由 FragmentManager 管理**



``` java
//FragmentManager.java
ArrayList<BackStackRecord> mBackStack;
```



其实触发 fragment 的返回逻辑有两种途径

- 开发主动调用 fragment 的返回方法

- 用户按返回键触发

  

后文我们会从这两个角度分析一下 fragment 中的返回栈逻辑究竟是怎样的



### 如何返回？

我们已经知道返回栈中的元素是 `BackStackRecord` ，也清楚了是 `FragmentManager` 来管理返回栈。那么如果让我们来实现「返回」逻辑，应该如何做？



首先我们要清楚所谓的「返回」是对事务的回滚，即 **对 commit 事务的内部逻辑执行相应的「逆操作」**。

例如

addFragment←→removeFragment

showFragment←→hideFragment

attachFragment←→detachFragment



有的小伙伴可能会疑惑 replace 呢？

`expandReplaceOps` 方法会把 replace 替换(目标 fragment 已经被 add )成相应的 remove 和 add 两个操作，或者(目标 fragment 没有被 add )只替换成 add 操作



## popBackStack 系列方法



`FragmentManager` 中提供了`popBackStack` 系列方法



![popBackStack系列方法](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf0f7fecec?w=523&h=207&f=png&s=41693)



是否觉得很眼熟？提交事务也有类似的api，commit 系列方法



![commit系列方法](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf0f5fe236?w=425&h=116&f=png&s=19768)



这里分别提供了同步和异步的方法，可能有读者会疑惑，同样是对事务的操作，一个为提交，一个为回滚，为什么一个封装到了 `FragmentManager` 中，一个却在 `FragmentTransaction` 中。既然都是对事务的操作，应该都放在FragmentManager 中。我认为可能为了api使用的方便，使得 `FragmentManager` 开启事务的链式调用一气呵成。各位有什么想法欢迎在评论区留言。



这里主要介绍一下 popBackStack(String name, int flag)



name 为 addToBackStack(String name) 的参数，通过 name 能找到回退栈的特定元素，flag可以为 0 或者`FragmentManager.POP_BACK_STACK_INCLUSIVE`，0 表示只弹出该元素以上的所有元素，`POP_BACK_STACK_INCLUSIVE` 表示弹出包含该元素及以上的所有元素。这里说的弹出所有元素包含回退这些事务。如果这么说比较抽象的话，看图



```kotlin
//flag 传入0，弹出 ♥2 上的所有元素
childFragmentManager.popBackStack("♥", 0)
```



![flag为0](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf111e4eca?w=544&h=968&f=gif&s=332027)





```kotlin
//flag 为 POP_BACK_STACK_INCLUSIVE 弹出包括该元素及及以上的元素
childFragmentManager.popBackStack("♥",  androidx.fragment.app.FragmentManager.POP_BACK_STACK_INCLUSIVE)
```



![flag为1](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf11573c7e?w=544&h=968&f=gif&s=307357)



## 走进源码

### 1. popBackStack() 逻辑



在分析返回栈源码之前我们回顾一下 FragmentManager 提交事务到 fragment 各个生命周期的流程



![异步](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf128028ab?w=1137&h=1745&f=png&s=148227)

![commitNow](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf3c14d698?w=1078&h=1232&f=png&s=120922)



下面我们看看 popBackStack 的源码



![popBackStack源码](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf3e264529?w=1111&h=744&f=png&s=152765)



等等，这个 enqueueAction 有些眼熟...



![commit](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf4261e2a2?w=582&h=144&f=png&s=16887)

![commitInternal](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf43057f36?w=792&h=404&f=png&s=74980)



看来提交事务和回滚事务的流程基本是相同的，只是传递的 action 不同



![enqueueAction](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf463c7504?w=818&h=722&f=png&s=108117)


![OpGenerator](https://user-gold-cdn.xitu.io/2020/3/16/170df028ee01dd57?w=1081&h=713&f=png&s=131939)


由源码可知，`OpGenerator` 是一个接口，其内只有一个 `generateOps` 方法，用于生成事务列表以及对应的该事务是否是弹出的。有两个实现类

![OpGenerator实现类](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf5579b571?w=1319&h=139&f=png&s=46323)



由此可见 commit 调用的为 `BackStackRecord` 的 `generateOps` 方法，`popBackStack` 调用的是 `PopBackStackState` 中的  `generateOps` 



前者的逻辑很简单，向 records list 中添加数据， isRecordPop list 全部传入 false

```java
records.add(this);
isRecordPop.add(false);
```



后者的逻辑稍微复杂些，其内部调用了 `popBackStackState` 方法

如果是 `popBackStack` 方法 ，则将 `FragmentManager` 的返回栈列表（`mBackStack`）的栈顶移除， `isRecordPop` list 全部传入 true

```java
int last = mBackStack.size() - 1;
records.add(mBackStack.remove(last));
isRecordPop.add(true);
```



如果传入的 name 或 id 有值，且 flag 为 0，则找到返回栈（倒序遍历）中第一个符合 name 或 id 的位置，并将该位置上方的所有 `BackStackRecord` 并添加到 `record` list 中，同时 `isRecordPop` list 全部传入 true

```java
index = mBackStack.size() - 1;
while (index >= 0) {
    BackStackRecord bss = mBackStack.get(index);
    if (name != null && name.equals(bss.getName())) {
        break;
    }
    if (id >= 0 && id == bss.mIndex) {
        break;
    }
    index--;
}

for (int i = mBackStack.size() - 1; i > index; i--) {
  records.add(mBackStack.remove(i));
  isRecordPop.add(true);
}
```



如果传入的 name 或 id 有值，且 flag 为 `POP_BACK_STACK_INCLUSIVE`，则在上一条获取位置的基础上继续遍历，直至栈底或者遇到不匹配的跳出循环，接着出栈所有 `BackStackRecord`

```java
//index 操作与上方相同，先找到返回栈（倒序遍历）中第一个符合 name 或 id 的位置
if ((flags & POP_BACK_STACK_INCLUSIVE) != 0) {
    index--;
    // 继续遍历 mBackStack 直至栈底或者遇到不匹配的跳出循环
    while (index >= 0) {
        BackStackRecord bss = mBackStack.get(index);
        if ((name != null && name.equals(bss.getName()))
                || (id >= 0 && id == bss.mIndex)) {
            index--;
            continue;
        }
        break;
    }
}
//后续出栈逻辑与上方相同
```



可以配合上面的动图理解



入栈和出栈后续的逻辑大体是相同的，只是根据 isPop 的正负出现了分支，出栈调用的是 executePopOps



![](https://user-gold-cdn.xitu.io/2020/3/16/170df06c04191fff?w=972&h=682&f=png&s=132928)




上文我们有提到，「返回」逻辑实际上就是执行提交事务内部操作逻辑的「逆操作」

那么接下的逻辑就很清晰了，根据不同的 mCmd 执行相应的逆操作



``` java
void executePopOps(boolean moveToState) {
    for (int opNum = mOps.size() - 1; opNum >= 0; opNum--) {
        final Op op = mOps.get(opNum);
        Fragment f = op.mFragment;
        switch (op.mCmd) {
            case OP_ADD:
                mManager.removeFragment(f);
                break;
            case OP_REMOVE:
                mManager.addFragment(f);
                break;
            case OP_HIDE:
                mManager.showFragment(f);
                break;
            case OP_SHOW:
                mManager.hideFragment(f);
                break;
            case OP_DETACH:
                mManager.attachFragment(f);
                break;
            case OP_ATTACH:
                mManager.detachFragment(f);
                break;
            case OP_SET_PRIMARY_NAV:
                mManager.setPrimaryNavigationFragment(null);
                break;
            case OP_UNSET_PRIMARY_NAV:
                mManager.setPrimaryNavigationFragment(f);
                break;
            case OP_SET_MAX_LIFECYCLE:
                mManager.setMaxLifecycle(f, op.mOldMaxState);
                break;
            default:
                throw new IllegalArgumentException("Unknown cmd: " + op.mCmd);
        }
        if (!mReorderingAllowed && op.mCmd != OP_REMOVE && f != null) {
            mManager.moveFragmentToExpectedState(f);
        }
    }
    if (!mReorderingAllowed && moveToState) {
        mManager.moveToState(mManager.mCurState, true);
    }
}
```



后面的逻辑就完全一样了



![popBackStack](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf65e331c7?w=1137&h=1585&f=png&s=127564)



### 2. fragment 是怎样拦截 activity 的返回逻辑的？



在 [【背上Jetpack之OnBackPressedDispatcher】Fragment 返回栈预备篇](https://juejin.im/post/5e6bae35f265da572a0d11ad) 一文中我们介绍了 `OnBackPressedDispatcher` 



![ComponetActivity](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf6fd444f6?w=1138&h=544&f=png&s=96482)

![](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf70da7341?w=642&h=512&f=png&s=71452)



activity 的 `onBackPressed` 的逻辑主要分为两部分，判断所有注册的 `OnBackPressedCallback` 是否有 enabled 的，如果有则拦截，不执行后续逻辑；



![fragment 拦截返回逻辑](https://user-gold-cdn.xitu.io/2020/3/16/170df0d07750776b?w=801&h=765&f=png&s=137606)


否则着执行 mFallbackOnBackPressed.run() ，其内部逻辑为调用 ComponentActivity 父类的 `onBackPressed` 方法



![](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf7443eb77?w=715&h=275&f=png&s=39938)



**所以我们只需看 mOnBackPressedCallbacks（ArrayDeque<OnBackPressedCallback） 是怎样被添加的以及 isEnabled 何时赋值为 true**



经过查找我们发现它是在 FragmentManager 的 attachController 调用 `addCallback`

``` java
 mOnBackPressedDispatcher.addCallback(owner,mOnBackPressedCallback)
```

进而执行了


![](https://user-gold-cdn.xitu.io/2020/3/16/170df099d16ae502?w=1000&h=425&f=png&s=75605)
![](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf7c970039?w=822&h=233&f=png&s=41860)



而 `mOnBackPressedCallback` 在初始化时 enabled 赋值为 false 

![mOnBackPressedCallback](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf8362798b?w=954&h=194&f=png&s=42041)



`isEnadbled` 会在返回栈数量大于 0 且其 mParent 为 `PrimaryNavigation` 时赋值为true



![](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf9805eb1e?w=796&h=555&f=png&s=94237)



而返回栈（`mBackStack`）的赋值在 `BackStackRecord` 的 `generateOps` 方法中，且是否添加到返回栈由 `mAddToBackStack` 这个布尔类型的属性控制



![](https://user-gold-cdn.xitu.io/2020/3/16/170df0f5d77e3c7f?w=891&h=544&f=png&s=94633)

![](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf9a0dfa56?w=548&h=215&f=png&s=28095)



**mAddToBackStack 的赋值在 addToBackStack 方法中，这也解释了为何调用 addToBackStack 方法就能将事务加入返回栈**

![](https://user-gold-cdn.xitu.io/2020/3/15/170deeaf9eb75bde?w=841&h=306&f=png&s=47458)



>  我们来总结一下，fragment 拦截 activity 返回栈是通过 `OnBackPressedDispatcher` 实现的，如果开启事务调用了 `addToBackStack` 方法，则 `mOnBackPressedCallback` 的 `isEnabled` 属性会赋值为 true，进而起到拦截 activity 返回逻辑的作用。拦截后执行 `popBackStackImmediate` 方法
>
> 而 popBackStack系列方法会调用 popBackStackState 构造 `records` 和 `isRecordPop` 列表，`isRecordPop` 的内部元素的值均为true 后续流程和提交事务是一样的，根据 `isRecordPop` 值的不同选择执行 `executePopOps` 或 `executeOps` 方法



## 单返回栈和多返回栈的实现

[Ian Lake](https://medium.com/@ianhlake) 在 [Fragments: Past, Present, and Future (Android Dev Summit '19)](https://www.youtube.com/watch?v=RS1IACnZLy4) 

有提到未来会提供多返回栈的 api



![](https://user-gold-cdn.xitu.io/2020/3/15/170deeafa780da77?w=1844&h=948&f=png&s=634511)



那么以现有的 api 如何实现多返回栈呢？



首先我要弄清楚怎样才会有多返回栈，根据上文我们知道 `FragmentManager` 内部持有`mBackStack` list，这对应着一个返回栈，**如果想要实现多返回栈，则需要多个 FragmentManager**，而多 `FragmentManager` 则对应多个 fragment



因此我们可以创建多个宿主 frament 作为导航 fragment 这样就可以用不同的宿主 fragment 的 独立的`FragmentManager` 分别管理各自的返回栈，如果这样说比较抽象，可以参考下图



![](https://user-gold-cdn.xitu.io/2020/3/16/170defa88bc0e5f9?w=488&h=750&f=gif&s=4633102)



图中有四个返回栈，其中最外部有一个宿主 fragment ，内部有四个负责导航的 fragment 管理其内部的返回栈，外部的宿主负责协调各个返回栈为空后如何切换至其他返回栈


单返回栈就很容易了，我们只需在同一个 `FragmentManager` 上添加返回栈即可


![](https://gitee.com/flywith24/Album/raw/master/img/20200312175950)


详情参照 [demo](https://github.com/Flywith24/Flywith24-Fragment-Demo)



---

## 关于我

---

我是 [Fly_with24](https://flywith24.gitee.io/)

- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)

- [简书](https://www.jianshu.com/u/3d5ad6043d66)

- [Github](https://github.com/Flywith24)