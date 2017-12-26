原文链接

---

# HandlerThread 简介：

我们知道Thread线程是一次性消费品，当Thread线程执行完一个耗时的任务之后，线程就会被自动销毁了。如果此时我又有一

个耗时任务需要执行，我们不得不重新创建线程去执行该耗时任务。然而，这样就存在一个性能问题：多次创建和销毁线程是很耗

系统资源的。为了解这种问题，我们可以自己构建一个循环线程Looper Thread，当有耗时任务投放到该循环线程中时，线程执行耗

时任务，执行完之后循环线程处于等待状态，直到下一个新的耗时任务被投放进来。这样一来就避免了多次创建Thread线程导致的

性能问题了。也许你可以自己去构建一个循环线程，但我可以告诉你一个好消息，Aandroid SDK中其实已经有一个循环线程的框架

了。此时你只需要掌握其怎么使用的就ok啦！当然就是我们今天的主角HandlerThread啦！接下来请HandlerThread上场，鼓掌～～

HandlerThread的父类是Thread，因此HandlerThread其实是一个线程，只不过其内部帮你实现了一个Looper的循环而已。那么我们

先来了解一下Handler是怎么使用的吧！

【转载请注明出处：[Android HandlerThread 源码分析](http://blog.csdn.net/feiduclear_up/article/details/46840523) CSDN 废墟的树】

# HandlerThread使用步骤：

## 1.创建实例对象

~~~
HandlerThread handlerThread = new HandlerThread("handlerThread");
~~~

以上参数可以任意字符串，参数的作用主要是标记当前线程的名字。

## 2.启动HandlerThread线程

~~~
handlerThread.start();
~~~

到此，我们就构建完一个循环线程了。那么你可能会怀疑，那我怎么将一个耗时的异步任务投放到HandlerThread线程中去执行呢？当然是有办法的，接下来看第三部。

## 3.构建循环消息处理机制

~~~
Handler subHandler = new Handler(handlerThread.getLooper(), new Handler.Callback() {
            @Override
            public boolean handleMessage(Message msg) {
                //实现自己的消息处理
                return true;
            }
        });
~~~

第三步创建一个Handler对象，将上面HandlerThread中的looper对象最为Handler的参数，然后重写Handler的Callback接口类中的

handlerMessage方法来处理耗时任务。

**总结：**以上三步顺序不能乱，必须严格按照步骤来。到此，我们就可以调用subHandler以发送消息的形式发送耗时任务到线程

HandlerThread中去执行。言外之意就是subHandler中Callback接口类中的handlerMessage方法其实是在工作线程中执行的。

# HandlerThread实例：

~~~
package com.example.handlerthread;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.os.HandlerThread;
import android.os.Message;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.TextView;

public class MainActivity extends Activity {

    private Handler mSubHandler;
    private TextView textView;
    private Button button;

    private Handler.Callback mSubCallback = new Handler.Callback() {
        //该接口的实现就是处理异步耗时任务的，因此该方法执行在子线程中
        @Override
        public boolean handleMessage(Message msg) {

            switch (msg.what) {
            case 0:
                Message msg1 = new Message();
                msg1.what = 0;
                msg1.obj = java.lang.System.currentTimeMillis();
                mUIHandler.sendMessage(msg1);
                break;

            default:
                break;
            }

            return false;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        textView = (TextView) findViewById(R.id.textView);
        button = (Button) findViewById(R.id.button);

        HandlerThread workHandle = new HandlerThread("workHandleThread");
        workHandle.start();
        mSubHandler = new Handler(workHandle.getLooper(), mSubCallback);

        button.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                //投放异步耗时任务到HandlerThread中
                mSubHandler.sendEmptyMessage(0);
            }
        });

    }
}
~~~

# HandlerThread源码分析

## HandlerThread构造方法

~~~
/**
 * Handy class for starting a new thread that has a looper. The looper can then be 
 * used to create handler classes. Note that start() must still be called.
 */
public class HandlerThread extends Thread {
    //线程优先级
    int mPriority;
    //当前线程id
    int mTid = -1;
    //当前线程持有的Looper对象
    Looper mLooper;

    //构造方法
    public HandlerThread(String name) {
        //调用父类默认的方法创建线程
        super(name);
        mPriority = Process.THREAD_PRIORITY_DEFAULT;
    }
   //带优先级参数的构造方法
    public HandlerThread(String name, int priority) {
        super(name);
        mPriority = priority;
    }

...............

}

~~~

分析：该类开头就给出了一个描述：该类用于创建一个带Looper循环的线程，Looper对象用于创建Handler对象，值得注意的是在创建Handler

对象之前需要调用start()方法启动线程。这里可能有些人会有疑问？为啥需要先调用start（）方法之后才能创建Handler呢？后面我们会解答。

上面的代码注释已经很清楚了，HandlerThread类有两个构造方法，不同之处就是设置当前线程的优先级参数。你可以根据自己的情况来设置优先

级，也可以使用默认优先级。

## HandlerThrad的run方法

~~~
public class HandlerThread extends Thread {
  /**
     * Call back method that can be explicitly overridden if needed to execute some
     * setup before Looper loops.
     */
    protected void onLooperPrepared() {
    }

    @Override
    public void run() {
        //获得当前线程的id
        mTid = Process.myTid();
        //准备循环条件
        Looper.prepare();
        //持有锁机制来获得当前线程的Looper对象
        synchronized (this) {
            mLooper = Looper.myLooper();
            //发出通知，当前线程已经创建mLooper对象成功，这里主要是通知getLooper方法中的wait
            notifyAll();
        }
        //设置当前线程的优先级
        Process.setThreadPriority(mPriority);
        //该方法实现体是空的，子类可以实现该方法，作用就是在线程循环之前做一些准备工作，当然子类也可以不实现。
        onLooperPrepared();
        //启动loop
        Looper.loop();
        mTid = -1;
    }
}
~~~

分析：以上代码中的注释已经写得很清楚了，以上run方法主要作用就是调用了Looper.prepare和Looper.loop构建了一个循环线程。值得一提的

是，run方法中在启动loop循环之前调用了onLooperPrepared方法，该方法的实现是一个空的，用户可以在子类中实现该方法。该方法的作用是

在线程loop之前做一些初始化工作，当然你也可以不实现该方法，具体看需求。由此也可以看出，Google工程师在编写代码时也考虑到代码的可扩展性。牛B!

# HandlerThread的其他方法

## getLooper获得当前线程的Looper对象

~~~
 /**
     * This method returns the Looper associated with this thread. If this thread not been started
     * or for any reason is isAlive() returns false, this method will return null. If this thread 
     * has been started, this method will block until the looper has been initialized.  
     * @return The looper.
     */
    public Looper getLooper() {
        //如果线程不是存活的，则直接返回null
        if (!isAlive()) {
            return null;
        }

        // If the thread has been started, wait until the looper has been created.
        //如果线程已经启动，但是Looper还未创建的话，就等待，知道Looper创建成功
        synchronized (this) {
            while (isAlive() && mLooper == null) {
                try {
                    wait();
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }

~~~

分析：其实方法开头的英文注释已经解释的很清楚了：该方法主要作用是获得当前HandlerThread线程中的mLooper对象。

首先判断当前线程是否存活，如果不是存活的，这直接返回null。其次如果当前线程存活的，在判断线程的成员变量mLooper是否为null，如果为

null，说明当前线程已经创建成功，但是还没来得及创建Looper对象，因此，这里会调用wait方法去等待，当run方法中的notifyAll方法调用之后

通知当前线程的wait方法等待结束，跳出循环，获得mLooper对象的值。

**总结：**在获得mLooper对象的时候存在一个同步的问题，只有当线程创建成功并且Looper对象也创建成功之后才能获得mLooper的值。这里等待方法wait和run方法中的notifyAll方法共同完成同步问题。

## quit结束当前线程的循环

~~~
 /**
     * Quits the handler thread's looper.
     * <p>
     * Causes the handler thread's looper to terminate without processing any
     * more messages in the message queue.
     * </p><p>
     * Any attempt to post messages to the queue after the looper is asked to quit will fail.
     * For example, the {@link Handler#sendMessage(Message)} method will return false.
     * </p><p class="note">
     * Using this method may be unsafe because some messages may not be delivered
     * before the looper terminates.  Consider using {@link #quitSafely} instead to ensure
     * that all pending work is completed in an orderly manner.
     * </p>
     *
     * @return True if the looper looper has been asked to quit or false if the
     * thread had not yet started running.
     *
     * @see #quitSafely
     */
    public boolean quit() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quit();
            return true;
        }
        return false;
    }
//安全退出循环
 public boolean quitSafely() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quitSafely();
            return true;
        }
        return false;
    }
~~~

分析：以上有两种让当前线程退出循环的方法，一种是安全的，一中是不安全的。至于两者有什么区别? quitSafely方法效率比quit方法标率低一点，但是安全。具体选择哪种就要看具体项目了。

# 总结：

1.HandlerThread适用于构建循环线程。

2.在创建Handler作为HandlerThread线程消息执行者的时候必须调用start方法之后，因为创建Handler需要的Looper参数是从HandlerThread类中获得，而Looper对象的赋值又是在HandlerThread的run方法中创建。

3.关于HandlerThread和Service的结合使用请参考另一篇博客：[Android IntentService 源码分析](http://blog.csdn.net/feiduclear_up/article/details/46964457)