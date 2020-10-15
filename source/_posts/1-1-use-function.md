---
title: 1.1使用函数
date: 2017.06.05 21:58
updated: 2017.06.05 21:58
categories: 
- python
tags: 
- python
- 爬虫
image: https://gitee.com/flywith24/Album/raw/master/img/20201015085429.png
---

> 我们编程时很容易疲劳，所以让我们来设计一个可以在一段时间后提醒你休息的小程序。比如每隔两个小时打开http://lines.frvr.com 此网站来玩一会儿小游戏。

<!-- more -->

让我们来分析下需要哪些步骤
我们首先要让程序等待两个小时，在需要休息的时候打开浏览器并转到这个小游戏的网站。也许我们一天要休息多次，所以我们需要一个循环来让其实现多次。

```
1. 等待两小时
2. 打开浏览器
重复
```
现在，让我们开始吧~

首先让我们google一下如何用Python来打开浏览器
![查询Pyhon如何打开浏览器](http://upload-images.jianshu.io/upload_images/3155702-ec11c33f9934a657.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
webbrowser.open("http://lines.frvr.com") 
```
可以看到上述代码可以使用默认浏览器打开指定网页。

让我们试试吧~
![保存](http://upload-images.jianshu.io/upload_images/3155702-ebeadc44e9ed109a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
保存并执行

![运行截图](http://upload-images.jianshu.io/upload_images/3155702-07994ea86762efd2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

出现了错误，不过学过java的你肯定能看懂是什么原因。

![修正](http://upload-images.jianshu.io/upload_images/3155702-5f2951beae5bec8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
导入webbrowser模块就可以正常运行了，不要被这个网站的小游戏吸引走哦，我们还没有结束。

下面我们看看Python如何能让程序等待2小时，为了方便测试，我们把等待时间设置为3秒

![Python让程序等待](http://upload-images.jianshu.io/upload_images/3155702-0690b4988c6f6ae1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到使用time.sleep()方法可以是程序等待一段时间执行，参数以秒为单位
所以我们在程序中添加以下代码
```
time.sleep(3)
```
当然也要导入相应模块。
![](http://upload-images.jianshu.io/upload_images/3155702-3d2bb6b60a4ff0e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

很简单有没有？

接下来我们来让其循环3次

输入以下代码
```
import webbrowser
import time
total_breaks = 3
break_count = 0

print("This program started on" + time.ctime())
while(break_count < total_breaks):
    time.sleep(3)
    webbrowser.open("http://lines.frvr.com")
    break_count = break_count + 1
```
代码很简单，首先我们定义了总的休息次数为3，我们又定义了已休息次数初始值为0。接下来是一个while循环，当已休息次数小于总休息次数时执行循环体。最后将已休息次数加1。

值得注意的是while循环并没有花括号。

学习 Python 与其他语言最大的区别就是，Python 的代码块不使用大括号 {} 来控制类，函数以及其他逻辑判断。python 最具特色的就是用缩进来写模块。
缩进的空白数量是可变的，但是所有代码块语句必须包含相同的缩进空白数量，这个必须严格执行。如下所示：
```
if True:
    print "True"
else:
  print "False"
```
以下代码将会执行错误：
```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
# 文件名：test.py
 if True:
    print "Answer"
    print "True"
else:
    print "Answer"
    # 没有严格缩进，在执行时会报错
  print "False"
```
执行以上代码，会出现如下错误提醒：
```
$ python test.py  
  File "test.py", line 5
    if True:
    ^
IndentationError: unexpected indent
```
 IndentationError: unexpected indent 错误是 python 编译器是在告诉你"Hi，老兄，你的文件里格式不对了，可能是tab和空格没对齐的问题"，所有 python 对格式要求非常严格。
如果是 IndentationError: unindent does not match any outer indentation level错误表明，你使用的缩进方式不一致，有的是 tab 键缩进，有的是空格缩进，改为一致即可。
因此，在 Python 的代码块中必须使用相同数目的行首缩进空格数。
建议你在每个缩进层次使用 单个制表符 或 两个空格 或 四个空格 , 切记不能混用。