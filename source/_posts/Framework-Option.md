---
title: 记录Framework开发的常用指令
date: 2019-08-16 14:11:46
tags: [ROM,framework]
categories: 
- ROM
image: https://gitee.com/flywith24/Album/raw/master/img/20201014234249.png
top : true
---

> 本篇博客是记录一些学习 `framework` 开发时知识的整理。

<!-- more -->

###  源码全盘编译指令

1. `source build/envsetup.sh`
2. 输入 `lunch` 同时选择欲编译的源码
3. 编译指令：`make -j32 2>&1 | tee build_20190717_1724.log`
4. 编译后out目录可查看编译出的镜像

前两步执行完毕即可执行 `mmm` 等命令，使用 `make clean`可以在编译前clean源码，该操作会导致编译时间边长。

### ROM烧录（mtk）

1. 打开 `flash_tool.exe`
2. 选择源码的配置文件
3. 切换下载/全部格式化和下
4. 点击下载，同时将已关机的设备连接电脑
5. 等待红条，绿条，黄条走完烧录成功

### ROM写号
1. 打开 `SN Writer.exe`
2. 点击 `System Config` 配置欲写哪些号，点击AP_DB进行配置文件，点击 `Save`
3. 点击 `Start` 将关机的设备连入电脑

### 源码检出
```
repo-local init -u git://{ip}/shupai_android7.1/manifests.git && repo-local sync -j8 && repo-local start --all master
```
{ip} 使用源码地址替换

### 开机动画
`/system/media/`

### 重新打包 `System` 生成镜像
```
make -j8 snod 
```

###  对 `Settings` 部分的修改
1. 移除某项（removePreference方法）
2. 禁用某项 （配置enabled属性）
3. 隐藏 `Settings` 主界面选项：`TileUtils`

###  三大金刚键
1. `HOME` 和  `BACK`
`PhoneWindowManager` 类 `interceptKeyBeforeDispatching` 方法
修改 `KEYCODE_HOME`  `KEYCODE_BACK` case 中的逻辑 
2. `RECENT`
`src/com/android/systemui/recents/RecentsImpl` 类 `startRecentsActivity` 方法

### 预装应用
`device/mediatek/common device.mk`

### 关机 重启操作
```
base/services/core/java/com/android/server/policy/GloablActions PowerAction
```
