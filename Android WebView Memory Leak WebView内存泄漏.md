

在这次开发过程中，需要用到webview展示一些界面，但是加载的页面如果有很多图片就会发现内存占用暴涨，并且在退出该界面后，即使在包含该webview的Activity的destroy()方法中，使用webview.destroy();webview=null;对内存占回收用还是没有任何效果。有人说，一旦在你的xml布局中引用了webview甚至没有使用过，都会阻碍重新进入Application之后对内存的gc。包括使用MapView有时一会引发OOM，几经周折在网上看到各种解决办法，在这里跟大家分享一下。但是到目前为止还没有找到根本的解决办法，网上也有说是sdk的bug。但是不管怎么样，我们还是需要使用的。

网友评论都说下面传递Application 上下文的方式极力不推荐，存在很多问题，那么希望再看到该文的同行慎重。在这里谢谢各位的评论。特别是@[Qiujuer](https://my.oschina.net/qiujuer)， @[levi-daivide](https://my.oschina.net/u/1015336) 。再一个就是好久没关注Webkit以及WebView了。希望大家申辩着看。再次感谢各位。

要使用WebView不造成内存泄漏，首先应该做的就是不能在xml中定义webview节点，而是在需要的时候动态生成。即：可以在使用WebView的地方放置一个LinearLayout类似ViewGroup的节点，然后在要使用WebView的时候，动态生成即：

~~~
WebView      mWebView = new WebView(getApplicationgContext()); 
LinearLayout mll      = findViewById(R.id.xxx); 
mll.addView(mWebView);
~~~

， 然后一定要在onDestroy()方法中显式的调用

~~~
protected void onDestroy() {       super.onDestroy(); mWebView.removeAllViews(); mWebView.destroy() }
~~~

;注意： new  WebView(getApplicationgContext()) ;必须传入ApplicationContext如果传入Activity的Context的话，对内存的引用会一直被保持着。有人用这个方法解决了当Activity被消除后依然保持引用的问题。但是你会发现，如果你需要在WebView中打开链接或者你打开的页面带有flash，获得你的WebView想弹出一个dialog，都会导致从ApplicationContext到ActivityContext的强制类型转换错误，从而导致你应用崩溃。这是因为在加载flash的时候，系统会首先把你的WebView作为父控件，然后在该控件上绘制flash，他想找一个Activity的Context来绘制他，但是你传入的是ApplicationContext。后果，你可以晓得了哈。

于是大牛们就Activity销毁后还保持引用这个问题，提供了另一种解决办法：既然你不能给我删除引用，那么我就自己来吧。于是下面的这种方法诞生了：

（作者说这个方法是依赖android.webkit implementation有可能在最近的版本中失败）

~~~
public void setConfigCallback(WindowManager windowManager) {
    try {
        Field field = WebView.class.getDeclaredField("mWebViewCore");
        field = field.getType().getDeclaredField("mBrowserFrame");
        field = field.getType().getDeclaredField("sConfigCallback");
        field.setAccessible(true);
        Object configCallback = field.get(null);

        if (null == configCallback) {
            return;
        }

        field = field.getType().getDeclaredField("mWindowManager");
        field.setAccessible(true);
        field.set(configCallback, windowManager);
    } catch(Exception e) {
    }
}
~~~

然后在Activity中调用上面的方法：

~~~
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setConfigCallback((WindowManager)getApplicationContext().getSystemService(Context.WINDOW_SERVICE));
}

public void onDestroy() {
    setConfigCallback(null);
    super.onDestroy();
}
~~~

该反射方法在我的实验中（2.3.6）确实有些用处，在应用内存占用到70M左右的时候会明显释放到50M或者60M然后的释放就有些缓慢，其实就是看不出来了。之前在没使用该方法的时候可能达到120M。

但是！！！我们的应用要求占用内存更低啊，这肿么拌？凉拌么？No。在各种纠结之后，终于找到了终极解决办法！！！该办法适用于我们的需求，在退出WebView的界面之后，迅速回收内存。要问这个方法是什么，不要9999，不要8999，只要你仔细看好下面一句话：那就是为加载WebView的界面开启新进程，在该页面退出之后关闭这个进程。

这一点说了之后，你懂了吧？
但是在这个其中，杀死自己进程的时候又遇到了问题，网上介绍的各种方法都不好使，
killBackgroundProcesses(getPackageName());各种不好用，最后使用System.exit(0);直接退出虚拟机（Android为每一个进程创建一个虚拟机的）。这个肯定不用纠结了，一旦退出，内存里面释放。听涛哥说QQ也是这么做。

最后英雄要问出处，附上大牛解说引起该问题的出处

这个泄漏出现在external/webkit/Source/WebKit/android/WebCoreSupport/UrlInterceptResponse.cpp.中。具体我自己真心没有深入研究。大家有兴趣的话，可以看看哈。

~~~
--- a/Source/WebKit/android/WebCoreSupport/UrlInterceptResponse.cpp
+++ b/Source/WebKit/android/WebCoreSupport/UrlInterceptResponse.cpp
@@ -63,10 +63,10 @@ public:
         JNIEnv* env = JSC::Bindings::getJNIEnv();
         // Initialize our read buffer to the capacity of out.
         if (!m_buffer) {
-            m_buffer = env->NewByteArray(out->capacity());
-            m_buffer = (jbyteArray) env->NewGlobalRef(m_buffer);
+            ScopedLocalRef<jbyteArray> buffer_local(env, env->NewByteArray(out->capacity()));
+            m_buffer = static_cast<jbyteArray>(env->NewGlobalRef(buffer_local.get()));
         }
         int size = (int) env->CallIntMethod(m_inputStream, m_read, m_buffer);
         if (checkException(env) || size < 0)
             return;
         // Copy from m_buffer to out.
~~~

而且从这里[https://github.com/android/platform_external_webkit/commit/1e3e46a731730c02d916ea805ec4b20191509282](https://github.com/android/platform_external_webkit/commit/1e3e46a731730c02d916ea805ec4b20191509282)这个bug的解决状态。

还有一个问题要说的，也是在WebView使用的时候出现的问题：WebView中包含一个ZoomButtonsController，当使用web.getSettings().setBuiltInZoomControls(true);启用该设置后，用户一旦触摸屏幕，就会出现缩放控制图标。这个图标过上几秒会自动消失，但在3.0系统以上上，如果图标自动消失前退出当前Activity的话，就会发生ZoomButton找不到依附的Window而造成程序崩溃，解决办法很简单就是在Activity的ondestory方法中调用web.setVisibility(View.GONE);方法，手动将其隐藏，就不会崩溃了。在3.0一下系统上不会出现该崩溃问题，真是各种崩溃，防不胜防啊！

最后还有内存泄漏的一些个建议：

In summary, to avoid context-related memory leaks, remember the following:

*   Do not keep long-lived references to a context-activity (a reference to an activity should have the same life cycle as the activity itself)
*   Try using the context-application instead of a context-activity
*   Avoid non-static inner classes in an activity if you don’t control their life cycle, use a static inner class and make a weak reference to the activity inside

And remember that a garbage collector is not an insurance against memory leaks. Last but not least, we try to make such leaks harder to make happen whenever we can.

