| 

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/20/133700g2b03cx9pzw3690f.jpg) 

未标题-1.jpg (12.96 KB, 下载次数: 77)

[下载附件](http://bugly.qq.com/bbs/forum.php?mod=attachment&aid=NTE4fDM3ZDZmYWU5fDE0ODA4NDk1MTB8MHw1MTY%3D&nothumb=yes)

2016-1-20 13:37 上传

Bugly 技术干货系列内容主要涉及移动开发方向，是由 Bugly 邀请腾讯内部各位技术大咖，通过日常工作经验的总结以及感悟撰写而成，内容均属原创，转载请标明出处。

一，内存泄漏的 Bug 猛增

最近接入了公司组件分析云，在 App 进行 mokey 测试的时候顺便检测内存泄漏问题。于是，就在前天的测试中，楼主一瞬间收到了4个这样的 Bug 单，瞬间心理无比纠结，真有千万只羊驼向我奔来。

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/20/133650nczepwzqexucedfk.png)

登录页面出现内存泄漏？？！！楼主的代码是如此的完美而无懈可击，这么可能出现这么多泄漏的问题？分析云测漏的工具有问题吧！？

插播什么是 Activity 泄漏：Android 中 Activity 代表一个页面，拥有一段生命周期，生命周期结束后，Activity 对象应当在之后某个合适的时机被 VM 回收内存。出现了泄漏就意味着 Activity 生命周期结束后，VM 发现 Activity 一直被持有，没有回收这些无用的内存。

按照以往的经验，大部分 Activity 泄漏的原因都是由于 Handler 内部类长时间挂在线程中导致的。而这块我们 App 已经考虑便处理了。究竟是哪泄漏了？

二，WebView导致内存泄漏众所周知

带着怀疑的心态并且为了证明清白，我一个个点进去看了，总共有三条不同的引用链。为了后续说明，这里取了个名字：

① AuthDialog 引用链

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/20/095356aqwcwwrrpccssbsv.png)

② BrowserFrame 引用链

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/20/095408ifzbwyaj99zlgggy.jpg)

③ IClipboradDataPaste 引用链

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/20/095426jpyhih6aihpu8z0m.png)

看来这次情况有点不同！由于Monkey测试的机型比较少，这里所有的Bug都来自一部三星GT-I9300@android+4.3手机。

为了快速解决问题，楼主询问了其他同事和StackOverflow，发现这其中有三个类CookieSyncManager, WebView, WebViewClassic已经被很多人提起过，它们会导致内存泄漏！初步有如下的结论如下：

1\. CookieSyncManager是个全局静态单例，操作系统内部使用了App的Activity作为Context构造了它的实例。我们应该在App启动的时候，抢先帮系统创建这个单例，而且要用applicationContext，让它不会引用到Activity。

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093432baf64tn61r44qax3.png)

2\. 使用WebView的页面（Activity），在生命周期结束页面退出（onDestory）的时候，需要主动调用WebView.onPause()以及WebView.destory()以便让系统释放WebView相关资源。

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093518k47xuf4vfn4wexkh.png)

3\. IClipboardDataPasteEventImpl是三星手机才有的类，这个东西还会让EditText也发生内存泄漏！

4\. WebView内存泄漏是众所周知的，建议另外启动一个进程专门运行WebView。不要9998，不要9999，我们要100%！WebView用完之后就把进程杀死，即使泄漏了也无碍。

按照以上的种种结论，我们都认定了这里面就是WebView引起的。

但是！我们的应用主进程LoginActivity根本没有用到WebView啊！！！

三，第三方jar包使用WebView这可如何是好

根据以上的AuthDialog引用链，楼主把目标锁定了某SDK：

翻了一阵子恶心的混淆后的代码，找到下面这么一段。SDK确实创建了WebView实例，并且用的是客户程序的Activity对象作为WebView的Context如下：

c 跟 j 都是SDK中继承于WebView的一个子类，k 是登录接口的输入参数Activity。这里创建了 c 对象之后向上塑形赋给了 j 。

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093538jbi9ok59qbbiqep9.png)

网上已经有很多例子表明，直接用Activity作为参数构建WebView就非常有可能导致Activity泄漏。

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093553vruq84x7cr3me7ec.png)

不过也看到了代码中，有调用了WebView的destory()方法释放资源。但是这里似乎无法保证dismiss()一定会被执行。

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093604eyyybvcjzvv3m333.png)

问题到这里发现比较麻烦了，openSDK对我们来说是第三方包，我们没法让第三方包不用WebView，或者让第三方包把WebView放在另外一个进程中运行啊！于是，在App上面做规避暂时不好实现。于是找了SDK的童鞋一起分析了。

最终，大家都有了一个初步的共识，在Android4.3以下的旧版本，使用Activity对象创建WebView，确实有可能导致内存泄漏。非常高兴能得到SDK童鞋的大力支持，一起分析，问题到这里有了初步的进展。

四，心结未解，翻看WebView源码了解根源

不过，问题到这里楼主心理还是有个很严重的疑惑没有解开（是什么疑惑呢？）。于是拿了Android4.3的源码又翻了一遍希望找寻这里头的根本原因，做了一点记录，针对WebView在Java层的结构画了一个不严谨的类图：源码来源：[http://androidxref.com/4.3_r2.1/](http://androidxref.com/4.3_r2.1/)

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093624icor63boi7v3a3y3.jpg)

大概情况是这样：WebView这套结构中，有一个工厂类WebViewFactory提供静态方法。

Android4.3(JellyBean)版本通过WebViewFactory工厂类创建了一个全局单例对象WebViewClassic$Factory，然后使用这个Factory创建了一整套实现的代码(XXXClassic)：WebViewClassic, CookieManagserClassic, WebViewDatabaseClassic。

WebViewClassic才是真正地实现WebView的各种API。WebViewClassic创建并维护了WebViewCore对象。

WebViewCore创建了一个子线程“WebViewCoreThread”，这里是一个全局的单例的而且一旦启动就不会停止的Thread！WebViewCore会在这个子线程中创建维护并调用BrowserFrame的方法。

BrowserFrame本身是一个属于“WebViewCoreThread”线程的Handler子类。BrowserFrame会被native(c++)层调用，然后将这些调用切换到”WebViewCoreThread”线程中去执行，比如刷新进度或者处理屏幕旋转事件等等。

BrowserFrame还会调用CookieSyncManager.createIntance()，这也是系统框架中唯一一处调用的地方！

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093642qsxxj8j09esyy80s.png)

看到这里之后，楼主发现以上所说的，提前帮系统调用

CookieSyncManager.createInstance(contenxt.getApplicationContext()) 可能是没有效果的，因为系统本来就是这么做的。手机厂商修改这里的可能性不大。

CookieSyncManager又是什么东西？同样的，它自己也创建一个子线程，线程名就叫“CookieSyncManager”，又是全局单例不会停！这个线程每过5分钟就会把缓存在内存中的Cookie进行持久化syncFromRamToFlash()。

这里我们比较关心为什么Activity会泄漏，所以关键看看哪些类对象中持有了Activity(Context)引用：WebViewClassic, WebViewCore, BrowserFrame。

这套结构中有很多静态单例，还有子线程，想想也挺恶心的。而且三个关键的类都持有Activity引用。不过我们发现，其实WebViewClassic, WebViewCore这两个个对象跟WebView对象的生命周期是一致的，Activity销毁于是WebView销毁了，WebView销毁了另外两个对象也跟着销毁。烟消云散。。。

留下两个孤独的子线程还在跑，还有全局静态的钉子户对象。

但是！BrowserFrame本身是Handler，假如它因为native层的调用往”WebViewCoreThread”挂了一个消息，那么便可以建立一条引用链：

Thread->MessageQueue->Message->Handler(BrowserFrame)->Activity

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093654dyyagtqgcaoq5dq2.png)

好了，楼主的疑惑是什么？

五，最后的疑惑

我们再来看看AuthDialog的引用链。

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093708dpo93o9o499jf89p.png)

换成MAT看会比较清晰：

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093725c62ugseqk2zbw4kz.png)

楼主发现，这里CookieSyncManager线程，居然直接引用了Message对象！这是什么鬼？一般情况下，HandlerThread持有一个MessageQueue对象，MessageQueue才持有Message队列。

*Java Local : A local variable. For example, input parameters, or locally created objects of methods that are still in the stack of a thread. Native stack.*

*Input or output parameters in native code, for example user-defined JNI code or JVM internal code. Many methods have native parts, and the objects that are handled as method parameters become garbage collection roots. For example, parameters used for file, network, I/O, or reflection operations.*

这里表明，CookieSyncManager线程中存在某个Message的局部变量，而由于线程一直没有结束，所以局部变量一直没有被释放。而这个Message.obj成员引用了AuthDialog$3对象。

这是一个内部类，楼主发现内部类混淆之后的命名规则就是：第几个出现就命名为几。

AuthDialog里面有很多内部类：

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093740hxntfmmqfqwcgd61.png)

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093752r1go8eigi11gg88f.png)

如上图，MAT中的引用链中的AuthDialog$3指的就是这里的OnDismissListener匿名内部类！接着我们来看看Dialog.setOnDismissListener里面做了什么勾搭：

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093803uaweooi2kww6woto.png)

纳尼！OnDismissListener居然被赋给了Message.obj成员！

于是，我们心中生成的一条引用链是这样的：

Thread(main) -> MessageQueue->Message -> obj(OnDismissListener) -> AuthDialog -> Activity

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093814ulghhj9jrn4dzvol.png)

可是不对啊，我们所能找到的引用链跟CookieSyncManager子线程一点关系都没有！

再对比一下：

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093825gemajljx1kyllklx.png)

子线程CookieSyncManager拿到了主线程的Message！！ Oh no !! 这是什么情况？？？这个Message被某处地方错误引用了？子线程通过JNI在native中拿到Java层的对象？

好吧，楼主承认研究了一个晚上没有任何进展。。。

有时候熬夜跟进问题真的很容易卡死在某个地方，脑子迟钝了短路了，效率降低了。。。今天还要黑着眼圈来上班。中午跟同事讨论了一下排除了一些不可能存在的情况，然后继续在StackOverFlow上面游荡（假如有其他不错的程序员社区类似StackOverFlow这种的，麻烦亲们分享给楼主，感激不尽！）

六，原来是它！--Dialog
*注：以下的分析感悟来自**Github上面的**一篇文章：*[*《**一个内存泄漏引发的血案**》*](https://github.com/bboyfeiyu/android-tech-frontier/blob/master/issue-25/%E4%B8%80%E4%B8%AA%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F%E5%BC%95%E5%8F%91%E7%9A%84%E8%A1%80%E6%A1%88-Square.md)

有兴趣的童鞋请阅读原文

这里简要说明一下，作者的结论是：在 Android Lollipop 之前使用 AlertDialog 可能会导致内存泄漏！

作者发现，局部变量的生命周期在Dalvik VM跟ART/JVM中有区别。在DVM中，假如线程死循环或者阻塞，那么线程栈帧中的局部变量假如没有被置为null，那么就不会被回收。

如下代码使用阻塞队列说明问题：

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093836kmslhbz5bmqsl6zv.png)

子线程中调用loop()死循环，不停地从阻塞队列中取出一个MyMessage对象并且将对象的引用赋值给局部变量message，一次while循环之后，虚拟机应当结束while花括号中的局部变量的生命周期，并且释放对应的堆内存中的MyMessage对象。可是，DVM没有这么做！！

在 VM 中，每一个栈帧都是本地变量的集合，而垃圾回收器是保守的：只要存在一个存活的引用，就不会回收它。在每次循环结束后，本地变量不再可访问，然而本地变量仍持有对 Message 的引用，interpreter/JIT 理论上应该在本地变量不可访问时将其引用置为 null，然而它们并没有这样做，引用仍然存活，而且不会被置为 null，使得它不会被回收！！

这种场景不就是Android Handler消息机制的处理方式么？！

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093850yomwwdnzxdeocioi.png)

Looper不停地从阻塞队列MessageQueue中取出下一条消息Message并将引用赋给本地变量msg。一旦一次循环结束了，msg没有被置为null，对应的Message对象没有被回收，于是就泄漏了。

不过，Message是自带回收机制的，而且任何线程共用，从上面源码可以看到，每个Message被Handler处理完之后都会recycle()，置空所有的成员变量，并且放到回收池中。

好了，被CookieSyncManager子线程的Looper轮过一次的Message对象也跟其他人一样，被回收并放在了回收池中。这个时候，刚好遇到了Dialog！！

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093902ki2iiha4wt2i02w4.png)

这家伙刚刚好通过obtainMessage()从回收池中拿到了这个Message（被CookieSyncManager线程的本地变量引用住了），而且Message.obj变量就是OnDismissListener。

拿到之后，Dialog居然据为己有！！作为一个成员宠爱着！

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093916ueeaaozoaf3bz1yq.png)

Dialog自从拥有了mDismissMessage对象之后就不会让它挂到消息队列中了，每次要用都是拷贝一份而已。Message.obtain(mDismissMessage)，所以这个Message再也不会回到回收池中，直到Dialog被销毁，mDismissMessage变量也被置为null了。

![](http://bugly.qq.com/bbs/data/attachment/forum/201601/25/093925lpfm0f07k0g70ek8.png)

但是，这个Message依然占据着堆内存，而且被一个“游离”着的子线程局部变量msg引用着！！于是有了这条引用链：
Thread(CookieSyncManager) -> Message -> AuthDialog$3(OnDismissListener) -> AuthDialog -> Activity

七，总结一些注意点

针对Android4.3及以下版本，或者使用DVM的Android版本

1\. 使用WebView的时候，需要注意确保调用destroy()

2\. 考虑是否使用applicationContext()来构建WebView实例

3\. 调用Dialog设置OnShowListener、OnDismissListener、OnCancelListener的时候，注意内部类是否泄漏Activity对象

4\. 尽量不要自己持有Message对象

* * *

如果你觉得内容意犹未尽，如果你想了解更多相关信息，请扫描以下二维码，关注我们的公众账号，每周为您推送当下最热门的技术干货，让您随时随地轻松阅读,还有精彩活动与你分享~

                                                     ![](http://bugly.qq.com/bbs/data/attachment/forum/201602/25/162328be084tmlhop0yl54.png)

[腾讯 Bugly](http://bugly.qq.com/)是一款专为移动开发者打造的质量监控工具，帮助开发者快速，便捷的定位线上应用崩溃的情况以及解决方案。智能合并功能帮助开发同学把每天上报的数千条 Crash 根据根因合并分类，每日日报会列出影响用户数最多的崩溃，精准定位功能帮助开发同学定位到出问题的代码行，实时上报可以在发布后快速的了解应用的质量情况，适配最新的 iOS, Android 官方操作系统，鹅厂的工程师都在使用，快来加入我们吧！

 |

[13.png](http://bugly.qq.com/bbs/forum.php?mod=attachment&aid=NTExfGY4YTU1NTRlfDE0ODA4NDk1MTB8MHw1MTY%3D&nothumb=yes) (39.5 KB, 下载次数: 57)

![13.png](http://bugly.qq.com/bbs/data/attachment/forum/201601/20/095540z3323mp3mtd4xxd9.png "13.png")

[14.png](http://bugly.qq.com/bbs/forum.php?mod=attachment&aid=NTEyfGUxNTFmNDk5fDE0ODA4NDk1MTB8MHw1MTY%3D&nothumb=yes) (25.67 KB, 下载次数: 62)

![14.png](http://bugly.qq.com/bbs/data/attachment/forum/201601/20/095549k1e3550ncnec311e.png "14.png")

[15.png](http://bugly.qq.com/bbs/forum.php?mod=attachment&aid=NTEzfDE2MTJlZjg3fDE0ODA4NDk1MTB8MHw1MTY%3D&nothumb=yes) (64.33 KB, 下载次数: 62)

![15.png](http://bugly.qq.com/bbs/data/attachment/forum/201601/20/095601k1tu33t3hdjxyohh.png "15.png")

[20.png](http://bugly.qq.com/bbs/forum.php?mod=attachment&aid=NTE0fGZkMjJkOGFkfDE0ODA4NDk1MTB8MHw1MTY%3D&nothumb=yes) (24.98 KB, 下载次数: 63)

![20.png](http://bugly.qq.com/bbs/data/attachment/forum/201601/20/095622ie1ua5q2eeq1q2b5.png "20.png")

