---
title: 2.2从教务系统查询成绩并计算绩点——山东建筑大学为例
date: 2017.06.18 23:57
updated: 2017.06.18 23:57
categories: 
- python
tags: 
- python
- 爬虫
---

>前两天面试时被问到绩点是多少，但学校教务系统不提供绩点查询的功能，那么能不能写一个爬虫程序并计算出绩点呢？答案是肯定的！

 [感谢该博客提供的思路](http://blog.csdn.net/pleasecallmewhy/article/details/9305229)

<!-- more -->

### 1. 准备
HttpFox插件，是一款http协议分析插件，分析页面请求和响应的时间、内容、以及浏览器用到的COOKIE等。是火狐浏览器的插件。谷歌浏览器和Safari都有自带的分析工具，可是感觉太复杂，没有这款好用。不过火狐浏览器对学校的教务系统兼容性不是很好，我还下载了IE tab插件。
![插件的安装](http://upload-images.jianshu.io/upload_images/3155702-79a8f42d306c605c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/3155702-fdddcc6520d47f18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以非常直观的查看相应的信息。
点击start是开始检测，点击stop暂停检测，点击clear清除内容。
### 2. 探究过程
下面就去山东建筑大学官网登录到数字校园综合信息门户，看一看在登录的时候，到底发送了那些信息。
先来到登录页面，把httpfox打开，clear之后，点击start开启检测：

![开启检测](http://upload-images.jianshu.io/upload_images/3155702-0a6c714e2c2ff3a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
输入完账号密码，确保httpfox处于开启状态，然后点击登录。
这个时候可以看到，httpfox检测到了好多信息：
![捕捉到的信息](http://upload-images.jianshu.io/upload_images/3155702-d757be497cda18ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那么我们来分析一下这些数据

看起来红框里的两条数据比较有意思，先看看这个post

![post数据](http://upload-images.jianshu.io/upload_images/3155702-f72b7d8e3375dafc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

PostData中我们看到了比较熟悉的词，username和password，学过java web的我们很清楚这段数据的含义，点击登录后将这用户名和你们提交到服务器比对。
![重定向到这里](http://upload-images.jianshu.io/upload_images/3155702-2a025613cb9cd840.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到这里使用get的方式在链接上以?的方式显示的加上了参数，跳转到信息门户。
我们的post的数据就发送到了这个地址
```
http://urpe.sdjzu.edu.cn/loginPortalUrlForIndexLogin.portal
```
需要的post数据是用户名密码，也就是说我们需要输入这两种数据来模拟登录过程。

![进入到教务系统](http://upload-images.jianshu.io/upload_images/3155702-40ea67af08eb6ab1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![点击成绩查询](http://upload-images.jianshu.io/upload_images/3155702-c30fa7de450e5621.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
进入教务系统后点击成绩查询，我们看到请求的地址为
```
http://jwfw1.sdjzu.edu.cn/ssfw/jwnavmenu.do?menuItemWid=1E057E24ABAB4CAFE0540010E0235690
```
我们整理一下整个过程的思路。

1. POST学号和密码--->然后返回cookie的值
2. 发送cookie给服务器--->返回页面信息。
3. 获取到成绩页面的数据，用正则表达式将成绩和学分单独取出并计算加权平均数。

ok，理顺思路后剩下的就只有编码问题了。

### 3. 实验
我们先来实验下是否能够获得查询成绩界面的源码

我们先准备一个POST的数据，再准备一个cookie的接收，然后写出源码如下：
```
# coding=utf-8
import urllib
import urllib2
import cookielib

cookie = cookielib.CookieJar()
opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cookie))

# 需要的POST数据
postdata = urllib.urlencode({
    'userName': '20140216064',
    'password': '*********'
})
# 自定义一个请求
req1 = urllib2.Request(
    url='http://urpe.sdjzu.edu.cn/loginPortalUrlForIndexLogin.portal',
    data=postdata
)

req2 = urllib2.Request(
    url='http://jwfw1.sdjzu.edu.cn/ssfw/jwnavmenu.do?menuItemWid=1E057E24ABAB4CAFE0540010E0235690'
)

# 访问登录链接
opener.open(req1)
result = opener.open(req2)
# 打印返回的内容
print result.read()
```

![运行结果](http://upload-images.jianshu.io/upload_images/3155702-c2476106a0e29a70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

很棒呦，看来跟我们预期的一样。

### 4. 整理数据
> 获得了成绩查询界面的源码后我们需要将数据进行整理，获得我们想要的数据，课程名称，学分，成绩。

将网页源码贴到Sublime Text中，方便我们查看源码


![分析源码](http://upload-images.jianshu.io/upload_images/3155702-480626d6803e99fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![分析源码](http://upload-images.jianshu.io/upload_images/3155702-6a978cc4d4c27ba1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通过查看源码我们看到从这个<div>开始到3640行都是关于成绩的代码，而成绩是存放在这个table标签下。
![需要提取的信息](http://upload-images.jianshu.io/upload_images/3155702-b1573f6441632f38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
红色框中分别为课程名，学分，以及成绩。这些是我们需要抽取出来的数据。
看到这里竟然有意外收获！注意黄色框被注释的部分，看来学校的教务系统有计算绩点的功能的，不知由于何种原因不用呢？

这里是这段程序中最困难的部分，我也踩了很多坑。我参照的博客是使用正则表达式来抽取想要的信息的，但我对正则表达式掌握的并不好，弄了好久也没写出合适的表达式，于是我果断放弃了使用正则表达式，采用BeautifulSoup来进行信息的筛选。
[BeautifulSoup用法参考](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html)

```
 # 将内容从页面源码中提取出来
    def deal_data(self, myPage):

        soup = BeautifulSoup(myPage)

        # 从title属性为有效成绩的标签中获取所有class属性为t_con的TAG(tr标签)
        trs = soup.find(attrs={"title": "有效成绩"}).findAll(attrs={"class": "t_con"})

        # 从tr标签中的td标签中获取需要的信息。下标为3，7，8的分别为课程名，学分，成绩
        for tr in trs:
            for index, td in enumerate(tr.findAll('td')):  # enumerate能在for循环中使用下标

                if index == 3:
                    print td.text
                elif index == 7:
                    self.weights.append(td.text.encode('utf8'))
                    print td.text
                elif index == 8:
                    self.points.append(td.text.encode('utf8'))
                    print td.text

            print
```
整个逻辑我简单说一下，我觉得还可以改进。
首先查找title属性为”有效成绩“的标签，通过上文的截图我们可以知道这是那个div标签，之后在该div标签中定位class为t_con的tr标签。你也许会问为什么不直接定位到tr标签，因为后面的网页代码中还存在class 为t_con的tr标签，但不是我们需要的成绩。然后在每个tr标签下抽取下标为3，7，8的标签，这里我是把它存到数组里了。
接下来就清晰了，先打印成绩信息，然后计算绩点。


![运行截图](http://upload-images.jianshu.io/upload_images/3155702-f6d0aff398d0089a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![运行截图](http://upload-images.jianshu.io/upload_images/3155702-3d48f5287d16407e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
学渣一个，绩点低请忽略。

源码
```
# encoding=utf8
import urllib
import urllib2
import cookielib
import re
import string
from BeautifulSoup import BeautifulSoup
import sys

reload(sys)
sys.setdefaultencoding('utf8')


class SDJZU_Crawler:
    # 声明相关的属性
    def __init__(self):

        self.loginUrl = 'http://urpe.sdjzu.edu.cn/loginPortalUrlForIndexLogin.portal'  # 登录的url
        self.resultUrl = 'http://jwfw1.sdjzu.edu.cn/ssfw/jwnavmenu.do?menuItemWid=1E057E24ABAB4CAFE0540010E0235690'  # 查询成绩的url
        self.cookieJar = cookielib.CookieJar()  # 初始化一个CookieJar来处理Cookie的信息
        self.postdata = urllib.urlencode({'userName': '', 'password': ''})  # 登录需要POST的数据
        self.weights = []  # 存储权重，也就是学分
        self.points = []  # 存储分数，也就是成绩
        self.opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(self.cookieJar))

    def sdjzu_init(self):
        username = raw_input('请输入学号:')  # 这里不要用input，二者区别请自行查询
        password = raw_input('请输入密码:')
        self.postdata = urllib.urlencode({'userName': username, 'password': password})  # 将用户名密码加入到POST中
        # 初始化链接并且获取cookie
        myRequest = urllib2.Request(url=self.loginUrl, data=self.postdata)  # 自定义一个请求
        result = self.opener.open(myRequest)  # 访问登录页面，获取到必须的cookie的值
        result = self.opener.open(self.resultUrl)  # 访问成绩页面，获得成绩的数据
        self.deal_data(result.read())
        self.calculate_gpa()

    # 将内容从页面源码中提取出来
    def deal_data(self, myPage):

        soup = BeautifulSoup(myPage)

        # 从title属性为有效成绩的标签中获取所有class属性为t_con的TAG(tr标签)
        trs = soup.find(attrs={"title": "有效成绩"}).findAll(attrs={"class": "t_con"})

        # 从tr标签中的td标签中获取需要的信息。下标为3，7，8的分别为课程名，学分，成绩
        for tr in trs:
            for index, td in enumerate(tr.findAll('td')):  # enumerate能在for循环中使用下标

                if index == 3:
                    print td.text
                elif index == 7:
                    self.weights.append(td.text.encode('utf8'))
                    print td.text
                elif index == 8:
                    self.points.append(td.text.encode('utf8'))
                    print td.text

            print

    # 计算绩点，如果成绩还没出来，就不算该成绩，
    def calculate_gpa(self):
        point = 0.0  # 成绩
        weight = 0.0  # 学分
        for i in range(len(self.points)):
            if self.points[i].isdigit() and (self.weights[i] != 0):
                point += string.atof(self.points[i]) * string.atof(self.weights[i])  # 成绩*学分累加求和
                weight += string.atof(self.weights[i])  # 学分累加求和

        print "绩点为："
        print point / weight  # 输出绩点 值成绩*学分累加求和 / 学分累加求和


# 调用
mySpider = SDJZU_Crawler()
mySpider.sdjzu_init()
```
[我的github地址](https://github.com/Flywith24)