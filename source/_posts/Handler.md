---
title: Handler深入分析
date: 2017-07-04 07:46:14
updated: 2017-07-04 07:46:14
categories: 
- app
tags: 
- handler
- 异步
image: 
---

> 在android中我们可以有很多方式去实现异步，比如AsyncTask，Rxjava。不过它们底层都是使用的Handler，所以我们来研究一下Handelr的实现。

<!-- more -->

### 1. TreadLocal的使用
下面我们来写一个小demo，创建两个子线程，在两个子线程中分别为字符串result2，result3赋值，在主线程中调用两个子线程，并且为字符串result1赋值，最后打印输出结果。

![主线程](http://upload-images.jianshu.io/upload_images/3155702-2e352e80d8f708cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![子线程1](http://upload-images.jianshu.io/upload_images/3155702-3593c9860a191e71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![子线程2](http://upload-images.jianshu.io/upload_images/3155702-106032139bef8248.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后我们看一下打印结果


![测试结果](http://upload-images.jianshu.io/upload_images/3155702-0fded8b582305652.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

很明显，这不是我们想要的结果。看来线程之间相互影响了，那么有没有办法实现上述的功能呢？

当然有，我们可以使用 TreadLocal
> 我们可以把TreadLocal看做成一个容器，调用其中的set和get方法，可以设值和取值。下面我们看看是如何实现的。

首先创建一个ThreadLocal对象，并设置泛型为String
![ThreadLocal](http://upload-images.jianshu.io/upload_images/3155702-b274968c0d87042d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里不同的是首先将要打印的字串放入ThreadLocal中，然后从ThreadLocal中取出。
![主线程](http://upload-images.jianshu.io/upload_images/3155702-b8a37fcc57787a3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

子线程的操作也是类似的。
![子线程](http://upload-images.jianshu.io/upload_images/3155702-9b8b4a3c8f495526.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面我们看一下打印结果

![运行结果](http://upload-images.jianshu.io/upload_images/3155702-71885237f2c74d29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样就完成了上述功能。那么这里说的ThreadLocal与Handler有什么关系呢？别急，往下看。

### 2. 在子线程中创建Handelr
我们在子线程中创建一个Handler对象，然后运行程序。
![子线程中创建Handler对象](http://upload-images.jianshu.io/upload_images/3155702-4673d3b294d1ff08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到运行时出现了异常
>  java.lang.RuntimeException: Can't create handler inside thread that has not called Looper.prepare()

![异常](http://upload-images.jianshu.io/upload_images/3155702-42a0e89fbd6cfe2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看字面意思是不能在调用Looper.prepare()之前在线程中创建handler。
那么我们在创建handler之前去调用Looper.prepare()。
![调用Looper.prepare()](http://upload-images.jianshu.io/upload_images/3155702-036e11a4312a1dd7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
之后就能成功运行了。

那么我们来研究一下为什么会这样。
鼠标放在Handler()上，win按住control+鼠标左键，Mac按住command+鼠标左键。进入Handler的构造器。
![Handler构造器](http://upload-images.jianshu.io/upload_images/3155702-ddcf57bb316f4d6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击this

![更进一步](http://upload-images.jianshu.io/upload_images/3155702-322cff8f6407b3f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们找到了源头，如果looper为空则抛出这个异常。

在这里从looper里取出mQueue赋值给mQueue

然后我们看一下这个Looper.prepare方法，
![prepare](http://upload-images.jianshu.io/upload_images/3155702-8092676c88450ad6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上方的注释说得很清楚，在真正开始这个loop之前，该方法为你提供了创建引用这个looper的handelr的机会。在调用完该方法后，应该确保调用了loop()方法，并且使用quit()方法去结束它。

我们还看到如果多次调用prepare方法会抛出Only one Looper may be created per thread异常。
在这里我们看到了熟悉的身影，ThreadLocal。在这里使用ThreadLocal来存looper。

看到这我们不禁要问，在主线程我们并没有调用prepare方法啊，没错，在主线程使用的是prepareMainLooper
![prepareMainLooper](http://upload-images.jianshu.io/upload_images/3155702-28263341bcc4e038.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到这个main looper已经被android environment创建了，所以不需要自己调用该方法。

下面我们来看一下在子线程中创建Handler的标准写法。
```java
class ThreadLooper extends Thread {
        public Handler mHandler;

        @Override
        public void run() {
            super.run();
            Looper.prepare();
            mHandler = new Handler() {
                @Override
                public void handleMessage(Message msg) {
                }
            };
            Looper.loop();
        }
    }
```
### 3. Message的发送和处理过程
Handler里提供了几个消息入队的方法
> post()
> postAtTime()
> postDelayed()
> postAtFrontOfQueue()
> sendMessageAtTime(Message msg , long uptimeMillis)

其中post()，postAtTime()，postDelayed()都会直接或间接调用sendMessageAtTime方法。

![post](http://upload-images.jianshu.io/upload_images/3155702-8f15bc3b507a098f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![postAtTime](http://upload-images.jianshu.io/upload_images/3155702-1b8a08501a2a9251.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![postDelayed](http://upload-images.jianshu.io/upload_images/3155702-c5ef6085b80726ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![sendMessageDelayed](http://upload-images.jianshu.io/upload_images/3155702-dbe0fccf4cc81722.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
下面我们看一下sendMessageAtTime方法

![sendMessageAtTime](http://upload-images.jianshu.io/upload_images/3155702-678aac95b1c8ad85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

有两个参数，msg 和uptimeMillis ，如果消息队列为空，则打印警告，同时返回false。反之则调用enqueueMessage方法。

下面看一下enqueueMessage

![enqueueMessage](http://upload-images.jianshu.io/upload_images/3155702-ba4cd0593a2e8df7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里有两个比较重要的操作。
msg.target赋值为this，有两层含义，一是message的来源是当前handler，二是当前的handler来处理消息。
将消息加入到消息队列中，既然是队列就有顺序，那么根据什么来判断顺序呢？就是根据uptimeMillis,这个时间，时间短就在前面，长就在后面。

细心的你可能发现刚刚我提到post()，postAtTime()，postDelayed()都会直接或间接调用sendMessageAtTime方法，那postAtFrontOfQueue()呢？
从字面上看该方法是将消息置于消息队列的最前边。是不是这样呢？我们看一下源码。

![postAtFrontOfQueue](http://upload-images.jianshu.io/upload_images/3155702-76f2c74c7c0a3e51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里将入队的时间参数直接写死为0，那么肯定就是消息队列的最前边啦。

我们再来分析下入队之后的过程，上文提到调用Looper.prepare()方法后应调用Looper.loop()方法开始消息的轮询。那么我们看看loop方法做了些什么。
```java
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            final long traceTag = me.mTraceTag;
            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            try {
                msg.target.dispatchMessage(msg);
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }
```

![looper](http://upload-images.jianshu.io/upload_images/3155702-5797c552356958b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
首先从ThreadLocal中取出looper并判断是否为空，之后将looper中的消息队列赋值，再然后进入一个死循环，循环内去不断寻找消息队列的下一项，没有消息发生阻塞。

找到 msg.target.dispatchMessage(msg);这一行，
之前我们提到target就是handler对象，这里handler把消息派发出去，接下来就进入消息的处理了。

进入到msg.target.dispatchMessage方法，
![dispatchMessage](http://upload-images.jianshu.io/upload_images/3155702-3d37b2fa8bb03c78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里分三步
1.判断msg的回调是否为空
 如果不为空则直接该回调自己处理，反之判断自己的回调

![message的回调](http://upload-images.jianshu.io/upload_images/3155702-5cabf5368375e861.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
把Runnable 封装成msg的callback

2.判断自己的回调是否为空
3.调用handleMessage方法

![handleMessage](http://upload-images.jianshu.io/upload_images/3155702-ad49dc188525989f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里由子类重写来处理message
### 4. Handler机制的总结
> Thread 负责业务逻辑
> Handler 负责发送消息和处理消息
> MessageQueue 负责保存消息
> Looper 负责轮询消息队列