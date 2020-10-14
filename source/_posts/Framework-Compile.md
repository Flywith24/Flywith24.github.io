---
title: 【折腾Framework】源码编译与烧写
date: 2020-08-18 14:11:46
tags: [ROM,framework]
categories: 
- ROM
keywords: 编译，烧写，源码，Android
image: https://gitee.com/flywith24/Album/raw/master/img/20201014220906.png
top: 100
---

基于 Android 10 的源码编译与烧写。

<!-- more-->


## 前言

环境搭建等内容网上资料很多，这里不再赘述。

此处以 Pixel 3a & Android 10 为例介绍如何编译 ROM 包并烧录

手上没有真机的小伙伴可以选择制作模拟器，本文最后提供了基于 Android 10 编译的自定义 AVD 下载链接

## 编译环境

[官方文档](https://source.android.com/setup/build/initializing)

## 下载源码

这里推荐使用 [清华镜像](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)

![](https://gitee.com/flywith24/Album/raw/master/img/20200628155443.png)

下载 [每月更新的初始化包](https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar) 并解压

## 下载驱动（可选）

如果要刷到真机上，需要下载相应机型的驱动，进入 [该链接](https://developers.google.com/android/drivers)，选择相应的机型对应的 Android 版本号和驱动

![](https://gitee.com/flywith24/Album/raw/master/img/20200628160058.png)

我这里选择的是 Android 10.0.0（QQ3A.200605.002.A1）

将两个驱动文件下载并解压，并执行

```
./extract-qcom-sargo.sh
./extract-google_devices-sargo.sh
```

点击 Enter 并输入 I ACCEPT 同意 License



根据 Build 在 [该链接](https://source.android.com/setup/start/build-numbers) 中找到相匹配的分支，本例中对应 `android-10.0.0_r39`

![QQ3A.200605.002.A1 对应的分支](https://gitee.com/flywith24/Album/raw/master/img/20200628160259.png)



## 选择分支

执行完上述操作如果直接编译的话实际上是编译的 master 分支，我们还需要切换到想要编译的分支

``` 
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-10.0.0_r39 --depth=1
repo sync -j8
repo start android-10.0.0_r39 --all
```



## 编译与烧写

在源码目录执行

`source build/envsetup.sh`

`lunch` 进入菜单

选择相应的版本

![](https://gitee.com/flywith24/Album/raw/master/img/20200628162935.png)

我这里选择的是 19

## 编译

接着便可以进行编译了，可以使用 `make -j` 命令，其中 j 代表的是编译 job 的数量

执行（我这里指定了生成 log 文件）

`make -j32 2>&1 | tee build_20200628_1635.log`

编译成功后会出现一个 out 目录，ROM 镜像文件就在此处

例如我的路径为：`~/aosp/out/target/product/sargo`

## 烧写

设备进入 bootloader 并执行：`fastboot flashall -w`，其中 -w 代表清空数据

烧写成功，开机！



## 制作自定义 AVD（Android Virtual Devices） 系统镜像

很多小伙伴没有相应的真机，不过可以制作出自定义的 AVD 作为模拟器使用。

`lunch` 进入菜单时选择相应的模拟器，例如选择上图的 24，64 位的通用设备

想要制作 AVD 系统镜像需要制作附加 `sdk` 和 `sdk_repo` 软件包

执行

```shell
$ make -j32 sdk sdk_repo
```

该操作可能会出现异常，例如

![](https://gitee.com/flywith24/Album/raw/master/img/20200818135050.png)

这是由于没有编译这些工具导致的，解决办法是依次编译这些工具

依次输入如下命令，后面工具视情况而定

```shell
$ make libaapt2_jni

$ make dmtracedump

$ make etc1tool

$ make deployagent

$ make aapt

$ make split-select

$ make bcc_compat

$ make apksigner

$ make dx

$ make layoutlib-legacy
```

编译好相关工具我们再次执行 `make -j32 sdk sdk_repo`



编译成功后会在 `out/host/linux-x86/sdk/aosp_x86_64` 目录下生成 `sdk-repo-linux-system-images-eng.[username].zip` 文件

按照官方文档中使用镜像的方式我没有成功

![](https://gitee.com/flywith24/Album/raw/master/img/20200818135851.png)



这里我使用了一个取巧的方式

我们在 Android Studio 创建 AVD 时可选的镜像一般有三种，这里还是以 Android 10 为例

![](https://gitee.com/flywith24/Album/raw/master/img/20200818140305.png)

Google Play，Google APIs，和默认的

它们会下载到 SDK/system-images/android-29 中

Google APIs 版本对应的目录就是 google_apis

我们可以将我们编译出的 AVD 镜像 copy 到其中的一个目录

例如，我将自定义的 AVD 镜像放置在了这里：

![](https://gitee.com/flywith24/Album/raw/master/img/20200818140734.png)

我们在创建模拟器时选择自定义的 AVD 镜像即可创建出自己编译 ROM  的模拟器

![](https://gitee.com/flywith24/Album/raw/master/img/20200818141020.png)



这里提供了我编译出来的自定义 AVD，不方便自己编译的小伙伴可以在此处下载

链接：https://pan.baidu.com/s/1LIcuycoU4Ou42VsSBM_MuQ 
提取码：CAVD