

# JAVA之旅（十三）——线程的安全性，synchronized关键字，多线程同步代码块，同步函数，同步函数的锁是this

* * *

> 我们继续上个篇幅接着讲线程的知识点

## 一.线程的安全性

> 当我们开启四个窗口（线程）把票陆陆续续的卖完了之后，我们要反思一下，这里面有没有安全隐患呢？在实际情况中，这种事情我们是必须要去考虑安全问题的，那我们模拟一下错误

~~~
package com.lgl.hellojava;

import javax.security.auth.callback.TextInputCallback;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {

        /**
         * 需求：简单的卖票程序，多个线程同时卖票
         */
        MyThread myThread = new MyThread();
        Thread t1 = new Thread(myThread);
        Thread t2 = new Thread(myThread);
        Thread t3 = new Thread(myThread);
        Thread t4 = new Thread(myThread);

        t1.start();
        t2.start();
        t3.start();
        t4.start();
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
    private  int tick = 100;

    @Override
    public void run() {
        while (true) {
            if (tick > 0) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                System.out.println("卖票:" + tick--);
            }
        }
    }
}

~~~

> 我们输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160604101606637)

> 这里出现了0票，如果你继续跟踪的话，你会发现，还会出现-1，-2之类的票，这就是安全隐患，那原因是什么呢？

*   当多条语句在操作同一个线程共享数据时，一个线程对多条语句只执行了一个部分，还没有执行完，另外一个线程参与了执行，导致共享数据的错误

> 解决办法：对多条操作共享数据的语句，只能让一个线程都执行完再执行过程中其他线程不可以参与运行
> 
> JAVA对多线程的安全问题提供了专业的解决办法，就是同步代码块

~~~

    synchronized(对象){
        //需要同步的代码
    }

~~~

> 那我们怎么用呢？

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
        Thread t3 = new Thread(myThread);
        Thread t4 = new Thread(myThread);

        t1.start();
        t2.start();
        t3.start();
        t4.start();
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
    private  int tick = 100;

    Object oj = new Object();
    @Override
    public void run() {
        while (true) {
            synchronized(oj){
                if (tick > 0) {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                    System.out.println("卖票:" + tick--);
                }
            }

        }
    }
}

~~~

> 这样，就输出

![这里写图片描述](http://img.blog.csdn.net/20160604102742924)

## 二.多线程同步代码块

> 我们为什么可以这样去同步线程？
> 
> 对象如同锁，持有锁的线程可以在同步中执行，没有执行锁的线程即使获取了CPU的执行权，也进不去，因为没有获取锁，我们可以这样理解

*   四个线程，哪一个进去就开始执行，其他的拿不到执行权，所以即使拿到了执行权，也进不去，这个同步能解决线程的安全问题

> 但是，同步是有前提的

*   1.必须要有两个或者两个以上的线程，不然你同步也没必要呀
*   2.必须是多个线程使用同一锁

> 必须保证同步中只能有一个线程在运行
> 
> 但是他也有一个弊端：那就是多个线程都需要判断锁，较为消耗资源

## 三.多线成同步函数

> 我们可以写一段小程序，来验证这个线程同步的问题，也就是说我们看看下面这段程序是否有安全问题，有的话，如何解决？

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {

        /**
         * 需求：银行里有一个金库 有两个人要存钱300
         */
        MyThread myThread = new MyThread();
        Thread t1 = new Thread(myThread);
        Thread t2 = new Thread(myThread);

        t1.start();
        t2.start();

    }

}

/**
 * 存钱程序，一次100
 * @author LGL
 *
 */
class MyThread implements Runnable {

    private Bank b = new Bank();

    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            b.add(100);
        }
    }

}

/**
 * 银行
 * @author LGL
 *
 */
class Bank {
    private int sum;

    public void add(int n) {
        sum = sum + n;
        System.out.println("sum:" + sum);
    }
}

~~~

> 当你执行的时候你会发现

![这里写图片描述](http://img.blog.csdn.net/20160604175804400)

> 这里是没错的，存了600块钱，但是，这个程序是有安全隐患的
> 
> 如何找到问题？

*   1.明确哪些代码是多线成运行代码
*   2.明确共享数据
*   3.明确多线成运行代码中哪些语句是操作共享数据的

> 那我们怎么找到安全隐患呢？我们去银行的类里面做些认为操作

~~~
/**
 * 银行
 * @author LGL
 *
 */
class Bank {
    private int sum;

    public void add(int n) {
        sum = sum + n;
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        System.out.println("sum:" + sum);
    }
}
~~~

> 让他sleep一下你就会发现

![这里写图片描述](http://img.blog.csdn.net/20160604180316485)

> 这样的话，我们就可以使用我们的同步代码了

~~~
/**
 * 银行
 * 
 * @author LGL
 *
 */
class Bank {
    private int sum;

    Object j = new Object();

    public void add(int n) {
        synchronized (j) {
            sum = sum + n;
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
            System.out.println("sum:" + sum);
        }
    }
}
~~~

> 这样代码就可以同步了

![这里写图片描述](http://img.blog.csdn.net/20160604180509216)

> 哪些代码该同步，哪些不该同步，你一定要搞清楚，根据上面的3个条件
> 
> 大家有没有注意到，函数式具有封装代码的特定，而我们所操作的同步代码块也是有封装代码的特性，拿这样的话我们就可以换一种形式去操作，那就是写成函数的修饰符

~~~

/**
 * 银行
 * 
 * @author LGL
 *
 */
class Bank {
    private int sum;

    public synchronized void add(int n) {
        sum = sum + n;
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        System.out.println("sum:" + sum);
    }
}

~~~

> 这样也是OK的

## 四.同步函数的锁是this

> 既然我们学习了另一种同步函数的写法，那我们就可以把刚才的买票小例子进一步封装一下了

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
        Thread t3 = new Thread(myThread);
        Thread t4 = new Thread(myThread);

        t1.start();
        t2.start();
        t3.start();
        t4.start();
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

    @Override
    public synchronized void run() {
        while (true) {
            if (tick > 0) {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread()+"卖票:" + tick--);
            }
        }
    }
}

~~~

> 但是这样做，你却会发现一个很严重的问题，那就是

![这里写图片描述](http://img.blog.csdn.net/20160604181319734)

> 永远只有0线程在执行卖票
> 
> 那是因为我们并没有搞清楚需要同步哪一个代码段，我们应该执行的只是里面的那两段代码，而不是整个死循环，所以我们得封装个函数进行线程同步

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
        Thread t3 = new Thread(myThread);
        Thread t4 = new Thread(myThread);

        t1.start();
        t2.start();
        t3.start();
        t4.start();
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

    @Override
    public void run() {
        while (true) {
            show();
        }
    }

    private synchronized void show() {
        if (tick > 0) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread() + "卖票:" + tick--);
        }
    }
}

~~~

> 这样输出解决了

![这里写图片描述](http://img.blog.csdn.net/20160604181751286)

> 问题是被解决了，但是随之问题也就来了

*   同步函数用的是哪一个锁呢？

> 函数需要被对象调用，那么函数都有一个所属对象的引用，就是this，所以同步函数所引用的锁是this,我们来验证一下，我们把程序改动一下
> 
> 使用两个线程来卖票，一个线程在同步代码块中，一个线程在同步函数中，都在执行卖票动作

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
        // Thread t3 = new Thread(myThread);
        // Thread t4 = new Thread(myThread);

        t1.start();
        myThread.flag = false;
        t2.start();
        // t3.start();
        // t4.start();
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
                    if (tick > 0) {
                        try {
                            Thread.sleep(10);
                        } catch (InterruptedException e) {
                            // TODO Auto-generated catch block
                            e.printStackTrace();
                        }
                        System.out.println(Thread.currentThread() + "code:"
                                + tick--);
                    }
                }
            }
        } else {
            while (true) {
                show();
            }
        }

    }

    private synchronized void show() {
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

~~~

> 当我们运行的时候就发现

![这里写图片描述](http://img.blog.csdn.net/20160604182642189)

> 他只在show中进行，那是为什么呢？因为主线程开启的时候瞬间执行，我们要修改一下，让线程1开启的时候，主线程睡个10毫秒试试

~~~
    t1.start();

        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

        myThread.flag = false;
        t2.start();
~~~

> 这样输出的结果貌似是交替进行

![这里写图片描述](http://img.blog.csdn.net/20160604183045569)

> 但是所知而来的，是0票，这说明这个线程不安全，我们明明加了同步啊，怎么还是不安全呢？因为他用的不是同一个锁，一个用Object，一个是用this的锁，我们再改动一下，我们把Object更好为this，这样输出

![这里写图片描述](http://img.blog.csdn.net/20160604183312384)

> 现在就安全，也正确了
> 
> 好的，我们本篇幅就先到这里了，我们下篇也继续讲线程

#### 如果有兴趣，可以加入群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。