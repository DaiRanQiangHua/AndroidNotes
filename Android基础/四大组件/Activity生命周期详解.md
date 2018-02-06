### Activity是什么？

Activity是用户和应用程序交互的界面，用户可以在Activity上进行点击、滚动、触摸等操作。一般来说，一个应用是由多个Activity组成，首次进入的Activity称为主Activity。至于如何判断一个Activity是不是主Activity。本篇文章我们先不讨论。后面会讲到。



### Activity的活动状态

当我查阅关于Activity的官方文档的时候，我发现，官方文档中谈到Activity有三种状态，运行中、暂停、停止。我觉得少了一个销毁。所以这边我简单介绍一下Activity的四种活动状态。

- 运行中(running)  

  此acitvity位于前台，并且用户可以在activity中执行触摸、点击、滚动等操作。

- 暂停(paused)

  activity可见，但并不可以操作，比如，当一个弹窗弹出来的时候。

- 停止(stopped)

  activity不可见，一般来说当用户按了home键之后。如果系统内存不够的时候，并且其他应用需要内存时，系统会回收已经停止的activity。

- 销毁(destroy)

  activity不可见，一般处于这种状态的activity会被系统回收掉。

  ​

### Activity的生命周期

首先我们先看下图解是怎么描述activity的生命周期的。

![](https://developer.android.google.cn/images/activity_lifecycle.png)

在正常情况下，Acitivity的生命周期会先后经历如下的生命周期：

1）onCreate：表示Activity正在被创建，一般来说，我们会在这个方法中设置布局以及一些数据的初始化。

2）onStart：表示Activity正在被启动，这个Activity已经可见，但并不是前台，这种情况下，我们无法操作这个Activity。

3）onResume：表示Activity已经可见并且位于前台了，此时，我们可以对当前的Activity进行操作。

4）onPasue：表示Activity暂停了，一般来说，执行到了这一步，后续会调用onStop方法。所以，在这边我们尽可能不要去执行耗时操作，因为这会影响新的Activity的显示。

5）onStop：表示Activity停止了，这边我们可以对一些数据进行回收。同样不能太耗时。

6）onDestroy：表示Activity即将销毁，这是Activity最后一个生命周期，同样，我们可以对一些数据进行回收。

7）onRestart：表示Activity正在被重启，这个方法一般只有在重启Activity的时候才被调用，比如，打开一个新的Activity然后在回退到当前的Activity。此方法便会被调用。



我们可以通过上述介绍以及图片，可以分析出一般Activity会有如下几种情况：

启动一个Activity：onCreate()-->onStart()-->onResume()。

按Home键之后或者重新打开一个Activity：onPause()--> onStop()（注：如果新的Activity是透明主题，当前Acitvity不会走onStop方法，例如dialog弹出的时候，当然，dialog关闭当前Activity会走onResume()）。

重新启动一个Activity：onRestart()-->onStart()-->onResume()。

销毁一个Activity：onPasue()-->onStop()-->onDestroy()。



### 异常情况下Activity的生命周期

首先，我们需要什么情况算异常情况。最好的例子莫过于横竖屏切换，这种情况Activity会被销毁并且重建，一般来说，系统会调用onSaveInstanceState 方法进行数据的存储。然后通过调用onRestoreInstanceState进行数据的恢复。如果我们想知道当前activity是否被重建了。我们可以在onCreate中判断bundle是否为null或者重写onRestoreInstanceState方法。一般推荐使用后者，因为只有在重建的情况下，此方法才会被调用。

如果我们希望当系统配置参数发生改变时，当前Activity不会被重建应该怎么做呢？自然是有的，我们可以在Mainfest文件中为Activity配置configChanges属性。比如，我们想当屏幕切换的时候当前Activity不被重启。我们可以这样：

```
android:configChanges="orientation|screenSize"
```

configChanges的选项有很多，这里我们常用的只有locale、orentation、keyboardHidden以及screenSize。（需要注意的是：screenSize这个属性比较特殊，minSdkVersion 和targetSdkVersion 低于13时，此选项不会导致activity重启。否则会导致activity重启。）

此时，我们可以通过重写此方法来进行一些特殊处理：

```
    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
    }
```

###Handler消息机制

```
Message：消息；其中包含了消息ID，消息对象以及处理的数据等，由MessageQueue统一列队，终由Handler处理

Handler：处理者；负责Message发送消息及处理。Handler通过与Looper进行沟通，从而使用Handler时，需要实现handlerMessage(Message msg)方法来对特定的Message进行处理，例如更新UI等（主线程中才行）

MessageQueue：消息队列；用来存放Handler发送过来的消息，并按照FIFO（先入先出队列）规则执行。当然，存放Message并非实际意义的保存，而是将Message以链表的方式串联起来的，等Looper的抽取。

Looper：消息泵，不断从MessageQueue中抽取Message执行。因此，一个线程中的MessageQueue需要一个Looper进行管理。Looper是当前线程创建的时候产生的（UI Thread即主线程是系统帮忙创建的Looper，而如果在子线程中，需要手动在创建线程后立即创建Looper[调用Looper.prepare()方法]）。也就是说，会在当前线程上绑定一个Looper对象。

Thread：线程；负责调度消息循环，即消息循环的执行场所。

 

具体流程：

0、准备数据和对象：

①、如果在主线程中处理message（即创建handler对象），那么如上所述，系统的Looper已经准备好了（当然，MessageQueue也初始化了），且其轮询方法loop已经开启。【系统的Handler准备好了，是用于处理系统的消息】。【Tips：如果是子线程中创建handler，就需要显式的调用Looper的方法prepare()和loop()，初始化Looper和开启轮询器】

②、通过Message.obtain()准备消息数据（实际是从消息池中取出的消息）

③、创建Handler对象，在其构造函数中，获取到Looper对象、MessageQueue对象（从Looper中获取的），并将handler作为message的标签设置到msg.target上

1、发送消息：sendMessage()：通过Handler将消息发送给消息队列

2、给Message贴上handler的标签：在发送消息的时候，为handler发送的message贴上当前handler的标签

3、开启HandlerThread线程，执行run方法。

4、在HandlerThread类的run方法中开启轮询器进行轮询：调用Looper.loop()方法进行轮询消息队列的消息

【Tips：这两步需要再斟酌，个人认为这个类是自己手动创建的一个线程类，Looper的开启在上面已经详细说明了，这里是说自己手动创建线程（HandlerThread）的时候，才会在这个线程中进行Looper的轮询的】

5、在消息队列MessageQueue中enqueueMessage(Message msg, long when)方法里，对消息进行入列，即依据传入的时间进行消息入列（排队）

6、轮询消息：与此同时，Looper在不断的轮询消息队列

7、在Looper.loop()方法中，获取到MessageQueue对象后，从中取出消息（Message msg = queue.next()）

8、分发消息：从消息队列中取出消息后，调用msg.target.dispatchMessage(msg);进行分发消息

9、将处理好的消息分发给指定的handler处理，即调用了handler的dispatchMessage(msg)方法进行分发消息。

10、在创建handler时，复写的handleMessage方法中进行消息的处理

11、回收消息：在消息使用完毕后，在Looper.loop()方法中调用msg.recycle()，将消息进行回收，即将消息的所有字段恢复为初始状态
```
### Thanks

《Android开发艺术探索》

Android官方API文档



