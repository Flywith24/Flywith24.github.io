---
title: Mac下Android反编译初探
date: 2017-07-06 16:46:14
updated: 2017-07-06 16:46:14
categories: 
- app
tags: 
- 反编译
---

> 工作第四天，被要求学习逆向开发方面的知识，于是先将自己之前写的未经混淆的apk反编译，记录之。

[感谢该博文提供的思路](http://blog.csdn.net/wj_november/article/details/51527286)

<!-- more -->

### 1. 工具准备

![工具](http://upload-images.jianshu.io/upload_images/3155702-569189364ef992cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

需要的三件套，[下载请戳](http://download.csdn.net/detail/wj_november/9657372)

1. AndroidCrackTool 用于反编译apk文件
   与直接解压apk不同，用该工具获得的文件资源可以直接打开阅读，而直接解压得到的是字节码。
2. dex2jar 用于将.dex文件转为jar文件
    传统的Java程序经过编译，生成Java字节码保存在class文件中，Java虚拟机通过解码class文件中的内容来运行程序。而Dalvik虚拟机运行的是Davik字节码，所有的Davik字节码由Java字节码转换而来，并被打包到一个DEX（Dalvik Executable）可执行文件中，Dalvik虚拟机通过解释DEX文件来执行这些字节码。
3. jd-gui 用于阅读源码

### 2. 开始工作
使用AndroidCrackTool反编译apk，设置好目录点击执行按钮，出现end字样即成功。
![反编译apk](http://upload-images.jianshu.io/upload_images/3155702-9dde0fe67fd84a07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在这里可以查看一些资源文件

![查看资源文件](http://upload-images.jianshu.io/upload_images/3155702-33d7aeb6f6b20770.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
将apk以普通解压的方式解压出来，找到其中的classes.dex文件，
![classes.dex位置](http://upload-images.jianshu.io/upload_images/3155702-46a0928132602efb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
将其复制到dex2jar目录，使用终端进入dex2jar目录并执行如下命令。
```
sh dex2jar.sh classes.dex  
```
可以看到在dex2jar目录下生成了classes_dex2jar.jar的文件。

![终端下操作](http://upload-images.jianshu.io/upload_images/3155702-a489b4dd84c84ecd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![生成的classes_dex2jar.jar](http://upload-images.jianshu.io/upload_images/3155702-6c746a8c946c3275.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
使用  jd-gui打开classes_dex2jar.jar即可看到源码，可以看到我的apk并没有混淆，所以名字都是正常的命名，经过混淆的名字大都是些字母。

![阅读源码](http://upload-images.jianshu.io/upload_images/3155702-b7afdfda627a9e6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)