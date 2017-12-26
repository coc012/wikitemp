

# JAVA之旅（十四）——静态同步函数的锁是class对象，多线程的单例设计模式，死锁，线程中的通讯以及通讯所带来的安全隐患，等待唤醒机制

* * *

> JAVA之旅，一路有你，加油！

## 一.静态同步函数的锁是class对象

> 我们在上节验证了同步函数的锁是this,但是对于静态同步函数，你又知道多少呢？
> 
> 我们做一个这样的小实验，我们给show方法加上static关键字去修饰

~~~
private static synchronized void show() {
        if (tick > 0) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread() + "show:" + tick--);
        }
    }
~~~

> 然后我们来打印一下

![这里写图片描述](http://img.blog.csdn.net/20160605104517114)

> 发现他打印出0票了，说明他还是存在this隐患，同时也说明了一点就是他使用的锁不是this，那既然不是this，那是什么呢？

*   因为静态方法中，不可以定义this，我们可以分析，静态进内存中，内存中没有本类对象，但是一定有该类的字节码文件对象类名.class,我们可以这样同步

~~~
synchronized (MyThread.class)
~~~

> 你就会发现，线程是安全的了
> 
> 静态同步的方法，使用的锁是该字节码的对象 类名.class

## 二.多线成中的单例设计模式

> 还记得我们讲的单例设计模式吗？我们今年温习一下这两个实现单例模式的方法

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {

        /**
         * 单例设计模式
         */

    }
}

/**
 * 饿汉式
 * 
 * @author LGL
 *
 */
class Single1 {
    private static final Single1 s = new Single1();

    private Single1() {

    }

    public static Single1 getInstance() {
        return s;
    }
}

/**
 * 懒汉式
 * @author LGL
 *
 */
class Single {
    private static Single s = null;

    private Single() {

    }

    public static Single getInstance() {
        if (s == null) {
            s = new Single();
        }
        return s;
    }

}
~~~

> 我们着重点来看懒汉式，你会发现这个s是共享数据，所以我们所以延迟访问的话，一定会出现安全隐患的，但是我们使用synchronized来修饰的话，多线程启动每次都要判断有没有锁，势必会麻烦的，所以我们可以这样写

~~~
public static Single getInstance() {
        if (s == null) {
            synchronized (Single.class) {
                if (s == null) {
                    s = new Single();
                }
            }
        }
        return s;
    }
~~~

> 这样其实是比较麻烦的，我们用饿汉式比较多，懒汉式作用是延时加载，多线成访问就会有安全问题

## 三.多线程的死锁

> 我们同步当中会产生一个问题，那就是死锁

*   同步中嵌套同步

> 是怎么个意思？我们来实现一下这段代码

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {

        /**
         * 需求：简单的卖票程序，多个线程同时卖票
         */
        MyThread myThread = new MyThread();
        Thread t1 = new Thread(myThread);
        Thread t2 = new Thread(myThread);

        t1.start();
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        myThread.flag = false;
        t2.start();
    }

}

/**
 * 卖票程序
 * 
 * @author LGL
 *
 */
class MyThread implements Runnable {

    // 票数
    private int tick = 100;

    Object j = new Object();

    boolean flag = true;

    @Override
    public void run() {

        if (flag) {
            while (true) {
                synchronized (j) {
                    show();
                }
            }
        } else {
            while (true) {
                show();
            }
        }

    }

    private synchronized void show() {
        synchronized (j) {
            if (tick > 0) {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "show:" + tick--);
            }
        }
    }
}
~~~

> 这段代码里面，this锁中又object锁，object中又this锁，就会导致死锁，不信？我们运行一下

![这里写图片描述](http://img.blog.csdn.net/20160605191725280)

> 你会看到他会停止不动了，这就是死锁，而在我们开发中，我们应该尽量避免死锁的发生。

## 四.线程中的通讯

> 线程中的通讯，是比较重要的，我们看一下这张例图

![这里写图片描述](http://img.blog.csdn.net/20160605192129097)

> 存什么，取什么
> 
> 线程中通讯，其实就是多个线程在操作同一个资源，但是操作的动作不同。我们来具体看看例子

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {
        /**
         * 线程间通讯
         */
    }

}

// 资源
class Res {
    String name;
    String sex;
}

// 输入
class Input implements Runnable {

    @Override
    public void run() {

    }

}

// 输出
class Output implements Runnable {

    @Override
    public void run() {

    }

}
~~~

> 我们定义这些个类，对吧，一个资源，两个操作，紧接着，我们应该怎么去操作他？

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {
        /**
         * 线程间通讯
         */

        Res s = new Res();
        Input in = new Input(s);
        Output out = new Output(s);

        Thread t1 = new Thread(in);
        Thread t2 = new Thread(out);

        t1.start();
        t2.start();

    }

}

// 资源
class Res {
    String name;
    String sex;
}

// 输入
class Input implements Runnable {

    private Res s;

    public Input(Res s) {
        this.s = s;
    }

    @Override
    public void run() {

        int x = 0;

        while (true) {

            if (x == 0) {
                s.name = "lgl";
                s.sex = "男";
            } else if (x == 1) {
                s.name = "zhangsan";
                s.sex = "女";
            }
            // 交替
            x = (x + 1) % 2;
        }
    }

}

// 输出
class Output implements Runnable {

    private Res s;

    public Output(Res s) {
        this.s = s;
    }

    @Override
    public void run() {

        while (true) {
            System.out.println(s.name + "..." + s.sex);
        }
    }

}
~~~

> 这样去操作，你看下输出，这里出现了一个有意思的现象

![这里写图片描述](http://img.blog.csdn.net/20160606203343552)

> 你回发现他输出的竟然有女，这就是存在了安全隐患，但是也进一步的证实了，线程间的通讯

## 五.线程通讯带来的安全隐患

> 我们线程通讯，会有安全隐患，那已经怎么去解决呢？我们是不是一来就想到了同步synchronized？其实这样做没用的， 因为你传的锁是不一样的，你要想让锁唯一，就类名.class

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {
        /**
         * 线程间通讯
         */

        Res s = new Res();
        Input in = new Input(s);
        Output out = new Output(s);

        Thread t1 = new Thread(in);
        Thread t2 = new Thread(out);

        t1.start();
        t2.start();

    }

}

// 资源
class Res {
    String name;
    String sex;
}

// 输入
class Input implements Runnable {

    private Res s;

    Object o = new Object();

    public Input(Res s) {
        this.s = s;
    }

    @Override
    public void run() {

        int x = 0;

        while (true) {
            synchronized (Input.class) {
                if (x == 0) {
                    s.name = "lgl";
                    s.sex = "男";
                } else if (x == 1) {
                    s.name = "zhangsan";
                    s.sex = "女";
                }
                // 交替
                x = (x + 1) % 2;
            }
        }
    }

}

// 输出
class Output implements Runnable {

    private Res s;

    public Output(Res s) {
        this.s = s;
    }

    @Override
    public void run() {

        while (true) {
            synchronized (Input.class) {
                System.out.println(s.name + "..." + s.sex);
            }
        }
    }

}
~~~

> 这样，就解决了问题了

## 六.多线程等待唤醒机制

> 我们不需要多线成高速消耗CPU，而是在适当的时候唤醒他，所以我们需要定义一个布尔值

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {
        /**
         * 线程间通讯
         */

        Res s = new Res();
        Input in = new Input(s);
        Output out = new Output(s);

        Thread t1 = new Thread(in);
        Thread t2 = new Thread(out);

        t1.start();
        t2.start();

    }

}

// 资源
class Res {
    String name;
    String sex;
    boolean flag = false;
}

// 输入
class Input implements Runnable {

    private Res s;

    Object o = new Object();

    public Input(Res s) {
        this.s = s;
    }

    @Override
    public void run() {

        int x = 0;

        while (true) {
            synchronized (Input.class) {

                if (s.flag) {
                    try {
                        // 等待线程都存放在线程池
                        wait();
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                }

                if (x == 0) {
                    s.name = "lgl";
                    s.sex = "男";
                } else if (x == 1) {
                    s.name = "zhangsan";
                    s.sex = "女";
                }
                // 交替
                x = (x + 1) % 2;
                s.flag = true;
                // 通知
                notify();
            }
        }
    }

}

// 输出
class Output implements Runnable {

    private Res s;

    public Output(Res s) {
        this.s = s;
    }

    @Override
    public void run() {

        while (true) {
            synchronized (Input.class) {
                if (!s.flag) {
                    try {
                        wait();
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                } else {
                    System.out.println(s.name + "..." + s.sex);
                    s.flag = false;
                    notify();
                }

            }
        }
    }

}
~~~

> 都使用在同步中，因为要对待件监视器（锁）的线程操作，所以要使用在线程中，因为只有同步才具有锁
> 
> 为什么这些操作线程的方法要定义在Object类中呢？因为这些方法在操作同步线程中，都必须要标识它们所操作线程持有的锁，只有同一个锁上的被等待线程可以被同一个锁上notify，不可以对不同锁中的线程进行唤醒，也就是说，等待和唤醒必须是同一把锁！而锁可以是任意对象，所以可以被任意对象调用的方法定义在Object类中。
> 
> 我们今天介绍就先到这里，线程的概念比较多，我们要写好几篇！！！

#### 如果有兴趣，可以加群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。