---
title: 0.从零开始，Python的安装
date: 2017.06.05 21:09
updated: 2017.06.05 21:09
categories: 
- python
tags: 
- python
- 爬虫
image: https://gitee.com/flywith24/Album/raw/master/img/20201015085038.png
---

> 这学期计算机网络课程有一个课程设计，要求使用Python写一个小程序。之前也没接触过Python，从优达学城里看到一个关于Python的课程，在此记录。

<!-- more -->

### Windows安装
下载地址：https://www.python.org/downloads/

---

![安装步骤1](http://upload-images.jianshu.io/upload_images/3155702-837d626cf0404cc4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

确保安装了 pip 并且 Python 添加到了你的 PATH。

![](http://upload-images.jianshu.io/upload_images/3155702-1ce43800911a180c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![安装步骤2](http://upload-images.jianshu.io/upload_images/3155702-9bccbfe4b2d0e766.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

要检查安装是否成功，打开 IDLE（Python 安装的一款程序，使你能够轻松地编辑和运行 Python 代码）。

a) Windows 7（及更早版本）：依次点击“开始”菜单>“所有程序”>“Python 2.7”，最后选择 IDLE (Python GUI)。

b) Windows 8/10：搜索 IDLE。目前，你可以从屏幕右侧向左滑动或用鼠标点击屏幕的右下角进行搜索。

---
### MAC安装
要在 Mac 机器上安装 Python，你可以采用两种方法：在命令行中使用 Homebrew，或在官网上找到普通的 Python 安装程序。

方法 1：程序包安装程序
安装地址：https://www.python.org/downloads/release/python-2713/
检查是否安装成功

a) IDLE 应该位于您的应用程序文件夹中。
![](http://upload-images.jianshu.io/upload_images/3155702-746f86c26858feec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
b)通过按下 ⌘+空格键，打开 Spotlight ，并输入“idle”来查找 IDLE

![](http://upload-images.jianshu.io/upload_images/3155702-327506f6845e4d77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
以下是它在我们的计算机上运行的屏幕截图！


![](http://upload-images.jianshu.io/upload_images/3155702-f5b789535b05b7ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

方法 2：Homebrew
要通过 Homebrew 安装 Python，只需执行以下两步：

打开终端，并输入命令：
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)”。
```
安装过程中，系统会多次发出提示 。
安装完 Homebrew 后，你可以通过在命令行中输入 brew help，验证一切是否正常。现在输入 brew install python 获取 Python 2 的最新版本。这样就可以了！

通过在命令行里输入 python 即可验证 Python 是否安装正确。系统应该欢迎你使用 Python Shell。