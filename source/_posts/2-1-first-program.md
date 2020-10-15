---
title: 2.1开始第一个爬虫程序
date: 2017.06.18 10:05
updated: 2017.06.18 10:05
categories: 
- python
tags: 
- python
- 爬虫
image: https://gitee.com/flywith24/Album/raw/master/img/20201015085629.png
---

### 1. 安装IDE以及hello world
 > 一个优秀的IDE可以极大地提高工作效率，在这里我选择使用JetBrains公司的PyCharm。是不是有些眼熟？没错，IDEA 和Android Studio就是他们做的，JetBrains出品，必属精品。

<!-- more -->

[点击进入下载链接](https://www.jetbrains.com/pycharm/)

点击download选择相应平台及版本，我这里选择的是社区版。

![下载界面](http://upload-images.jianshu.io/upload_images/3155702-752e691985633d3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

安装过程，切换主题，调整字体之类的都跟跟IDEA类似，不需赘述。

那么开始我们的hello world程序吧。
新建一个Python file ，然后写入
```
print ("hello world")
```
第一次运行在工作区右击 run即可


![右击运行](http://upload-images.jianshu.io/upload_images/3155702-4ffc1a9c05699113.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
之后就会在Toolbar上显示run按钮了

![运行截图](http://upload-images.jianshu.io/upload_images/3155702-a540d17991f6282d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
很棒有木有

### 2.1 先爬他一个网页下来
敲入如下代码
```
import urllib2

response = urllib2.urlopen("http://www.baidu.com")
print response.read()
```
点击运行，我们可以得到结果


![](http://upload-images.jianshu.io/upload_images/3155702-165b12f5711bcd47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个网页的源码被我们爬下来了，是不是很简单！

### 2.2 分析代码
下面我们来分析下这段代码
```
import urllib2
```
urllib2库是学习Python爬虫最基本的模块，利用这个模块我们可以得到网页的内容，并对内容用正则表达式提取分析，得到我们想要的结果

```
response = urllib2.urlopen("http://www.baidu.com")
```
首先我们调用的是urllib2库里面的urlopen方法，传入一个URL，这个网址是百度首页，协议是HTTP协议，urlopen一般接受三个参数，它的参数如下：

![urlopen参数](http://upload-images.jianshu.io/upload_images/3155702-f03abc16265de8dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第一个参数url即为URL，第二个参数data是访问URL时要传送的数据，第三个timeout是设置超时时间。

第二三个参数是可以不传送的，data默认为空None，timeout默认为 socket._GLOBAL_DEFAULT_TIMEOUT

第一个参数URL是必须要传送的，在这个例子里面我们传送了百度的URL，执行urlopen方法之后，返回一个response对象，返回信息便保存在这里面。

```
print response.read()
```
response对象有一个read方法，可以返回获取到的网页内容

### 2.3 构造Request
上面的urlopen参数可以传入一个request请求,它其实就是一个Request类的实例，构造时需要传入Url,Data等等的内容。我们可以将上面的代码改写为
```
import urllib2

request = urllib2.Request("http://www.baidu.com")
response = urllib2.urlopen(request)
print response.read()
```
运行结果是完全一样的，只不过中间多了一个request对象，推荐大家这么写，因为在构建请求时还需要加入好多内容，通过构建一个request，服务器响应请求得到应答，这样显得逻辑上清晰明确。