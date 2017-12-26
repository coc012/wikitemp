原文链接：
存档链接：

---

# 引言

Service服务是Android四大组件之一，在Android中有着举足重轻的作用。Service服务是工作的UI线程中，当你的应用需要下载一个文件或者播放音乐等长期处于后台工作而有没有UI界面的时候，你肯定要用到Service+Thread来实现。因此你需要自己在Service服务里面实现一个Thread工作线程来下载文件或者播放音乐。然而你每次都需要自己去写一个Service+Thread来处理长期处于后台而没有UI界面的任务，这样显得很麻烦，没必要每次都去构建一个Service+Thread框架处理长期处于后台的任务。Google工程师给我们构建了一个方便开发者使用的这么一个框架---IntentService。

# IntentService简介

IntentService是一个基础类，用于处理Intent类型的异步任务请求。当客户端调用android.content.Context#startService(Intent)发送请求时，Service服务被启动，且在其内部构建一个工作线程来处理Intent请求。当工作线程执行结束，Service服务会自动停止。IntentService是一个抽象类，用户必须实现一个子类去继承它，且必须实现IntentService里面的抽象方法onHandleIntent来处理异步任务请求。

# IntentServic示例

## Client代码

~~~
public class ClientActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    //客户端同时发送两个任务到IntentService服务端执行
    public void send(View view) {
        Intent intent = new Intent(this, DownLoadService.class);
        intent.putExtra("key", 1);
        intent.putExtra("value", "the first task1");
        startService(intent);

        Intent intent1 = new Intent(this, DownLoadService.class);
        intent1.putExtra("key", 2);
        intent1.putExtra("value", "the second task2");
        startService(intent1);
    }
}
~~~

模拟两个异步任务同时请求，通过Intent实例携带数据启动Service服务。

## Service客户端

~~~
public class DownLoadService extends IntentService {

    public static final String TAG = "DownLoadService";
    //重写默认的构造方法
    public DownLoadService() {
        super("DownLoadService");
    }

    //在后台线程执行
    @Override
    protected void onHandleIntent(Intent intent) {
        int key = intent.getIntExtra("key", 0);
        String value = intent.getStringExtra("value");
        switch (key) {
            case 1:
                //模拟耗时任务1
                try {
                    Thread.sleep(3 * 1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                break;
            case 2:
                //模拟耗时任务1
                try {
                    Thread.sleep(3 * 1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                break;
            default:
                break;
        }

        Log.e(TAG, "\nthe current time is: " + System.currentTimeMillis()/1000
                + "\nthe Thread id is " + Thread.currentThread().getId()
                + "\nthe current task is " + value);
    }
}
~~~

DownLoadService子类继承IntentService类，然后实现onHandleIntent抽象方法进行处理Intent请求的异步任务。在服务端DownLoadService类中，我们并没有创建Thread线程去执行异步耗时任务请求。所有的异步耗时任务都是在onHandleIntent抽象方法中实现了。言外之意是IntentService类内部已经帮开发者搭建好了一个异步任务处理器，用户只需实现其中的onHandleIntent抽象方法去处理异步任务即可，从而让开发者更加简单方便的使用IntentService处理后台异步任务请求。那么IntentService内部是怎么搭建异步任务处理器的呢？我们不妨查看源码来窥探个究竟。

# IntentService源码分析

## IntentService构造方法

~~~
 /**
     * Creates an IntentService.  Invoked by your subclass's constructor.
     *
     * @param name Used to name the worker thread, important only for debugging.
     */
    public IntentService(String name) {
        super();
        mName = name;
    }
~~~

分析：该构造方法需在子类中调用，用于创建一个IntentService对象。参数name用于定义工作线程的名称，仅仅用于调式作用。我们知道Service服务的生命周期是从onCreate方法开始的。那么就来看看IntentService#onCreate方法吧。

## IntentService#onCreate方法

~~~
 public void onCreate() {
        // TODO: It would be nice to have an option to hold a partial wakelock
        // during processing, and to have a static startService(Context, Intent)
        // method that would launch the service & hand off a wakelock.

        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();

        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }
~~~

分析：该方法首先利用HandlerThread类创建了一个循环的工作线程thread，然后将工作线程中的Looper对象作为参数来创建ServiceHandler消息执行者。由另一篇博客[Android HandlerThread 源码分析](http://blog.csdn.net/feiduclear_up/article/details/46840523)可知，HandlerThread+Handler构建成了一个带有消息循环机制的异步任务处理机制。因此开发者就可以将异步任务封装成消息的形式发送到工作线程中去执行了。Service服务生命周期第二步执行IntentService#onStartCommand方法。

## IntentService#onStartCommand方法

~~~
    /**
     * You should not override this method for your IntentService. Instead,
     * override {@link #onHandleIntent}, which the system calls when the IntentService
     * receives a start request.
     * @see android.app.Service#onStartCommand
     */
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }
~~~

分析：在IntentService子类中你无需重写该方法。然后你需要重写onHandlerIntent方法，系统会在IntentService接受一个请求开始调用该方法。我们看到在该方法中仅仅是调用了onStart方法而已，跟踪代码：

~~~
 @Override
    public void onStart(Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }

~~~

分析：该方法中通过mServiceHandler获得一个消息对象msg，然后将startId作为该消息的消息码，将异步任务请求intent作为消息内容封装成一个消息msg发送到mServiceHandler消息执行者中去处理，那么我们来看看mServiceHandler的实现吧！

~~~
private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }
~~~

分析：实现也比较简单，ServiceHandler是IntentService的内部类，在重写消息处理方法handlerMessage里面调用了onHandlerIntent抽象方法去处理异步任务intent的请求，当异步任务请求结束之后，调用stopSelf方法自动结束IntentService服务。看过博客[Android HandlerThread 源码分析](http://blog.csdn.net/feiduclear_up/article/details/46840523)的人都应该知道，此处handleMessage方法是在工作线程中调用的，因此我们子类重写的onHandlerIntent也是在工作线程中实现的。我们来看看onHandlerIntent方法：

~~~
/**
     * This method is invoked on the worker thread with a request to process.
     * Only one Intent is processed at a time, but the processing happens on a
     * worker thread that runs independently from other application logic.
     * So, if this code takes a long time, it will hold up other requests to
     * the same IntentService, but it will not hold up anything else.
     * When all requests have been handled, the IntentService stops itself,
     * so you should not call {@link #stopSelf}.
     *
     * @param intent The value passed to {@link
     *               android.content.Context#startService(Intent)}.
     */
    protected abstract void onHandleIntent(Intent intent);
~~~

分析：该方法用于处理intent异步任务请求，在工作线程中调用该方法。每一个时刻只能处理一个intent请求，当同时又多个intent请求时，也就是客户端同时多次调用Content#startService方法启动同一个服务时，其他的intent请求会暂时被挂起，直到前面的intent异步任务请求处理完成才会处理下一个intent请求。直到所有的intent请求结束之后，IntentService服务会调用stopSelf停止当前服务。也就是当intent异步任务处理结束之后，对应的IntentService服务会自动销毁，进而调用IntentService#onDestroy方法：

~~~
@Override
    public void onDestroy() {
        mServiceLooper.quit();
    }
~~~

该方法中调用HandlerThread工作线程中Looper对象的quit方法让当前工作线程HandlerThread退出当前Looper循环，进而结束线程。进而结束当前IntentService服务。到此，整个IntentService服务结束，现在可以用一张流程图来描述整个过程如下：

![这里写图片描述](http://img.blog.csdn.net/20150721153122660)

# IntentService总结

1.  子类需继承IntentService并且实现里面的onHandlerIntent抽象方法来处理intent类型的任务请求。
2.  子类需要重写默认的构造方法，且在构造方法中调用父类带参数的构造方法。
3.  IntentService类内部利用HandlerThread+Handler构建了一个带有消息循环处理机制的后台工作线程，客户端只需调用Content#startService(Intent)将Intent任务请求放入后台工作队列中，且客户端无需关注服务是否结束，非常适合一次性的后台任务。比如浏览器下载文件，退出当前浏览器之后，下载任务依然存在后台，直到下载文件结束，服务自动销毁。
4.  只要当前IntentService服务没有被销毁，客户端就可以同时投放多个Intent异步任务请求，IntentService服务端这边是顺序执行当前后台工作队列中的Intent请求的，也就是每一时刻只能执行一个Intent请求，直到该Intent处理结束才处理下一个Intent。因为IntentService类内部利用HandlerThread+Handler构建的是一个单线程来处理异步任务。

【转载请注明出处：[http://www.cnblogs.com/feidu/p/8074268.html](http://www.cnblogs.com/feidu/p/8074268.html) 废墟的树】
扫码关注微信公众号“Android知识传播”，不定时传播常用Android基础知识。

![ Android知识传播](http://img.blog.csdn.net/20150526092737989?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmVpZHVjbGVhcl91cA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)