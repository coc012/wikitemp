

# JAVA之旅（十五）——多线程的生产者和消费者，停止线程,守护线程,线程的优先级，setPriority设置优先级，yield临时停止

* * *

> 我们接着多线程讲

## 一.生产者和消费者

> 什么是生产者和消费者？我们解释过来应该是生产一个，消费一个，的意思，具体我们通过例子来说

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {
        /**
         * 生产者和消费者
         */
        Resrource res = new Resrource();

        Produce pro = new Produce(res);
        Consumer con = new Consumer(res);

        Thread t1 = new Thread(pro);
        Thread t2 = new Thread(con);

        t1.start();
        t2.start();

    }
}

// 资源
class Resrource {
    private String name;
    private int count = 1;
    private boolean flag = false;

    // 生产
    public synchronized void set(String name) {

        if (flag) {
            try {
                wait();
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }

            // 每次设置添加编号
            this.name = name + "-" + count++;
            System.out.println(Thread.currentThread().getName() + "--生产者--"
                    + this.name);
            flag = true;
            this.notify();
        }
    }

    // 消费
    public synchronized void out() {

        if (!flag)
            try {
                wait();
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        System.out.println(Thread.currentThread().getName() + "--消费者--"
                + this.name);
        flag = false;
        this.notify();
    }

}

// 生产
class Produce implements Runnable {

    private Resrource res;

    public Produce(Resrource res) {
        this.res = res;
    }

    @Override
    public void run() {
        while (true) {
            System.out.println("Android");
        }
    }

}

// 消费
class Consumer implements Runnable {
    private Resrource res;

    public Consumer(Resrource res) {
        this.res = res;
    }

    @Override
    public void run() {
        while (true) {
            res.out();
        }
    }

}
~~~

> 当我们生产一个，消费一个，就具有多线程的特性，如果出现其他现象，那就说明你的线程存在安全隐患了

## 二.停止线程

> 怎么让线程停？你会想到stop方法

![这里写图片描述](http://img.blog.csdn.net/20160607214403658)

> 既然已过时，我们就的去想其他办法了，跟其原理，是什么？run方法结束就是线程停止，那怎么让run方法结束？

*   只要控制循环，就可以让run方法结束，也就是线程的结束

> 我们写个实例

~~~
package com.lgl.hellojava;

import org.omg.CORBA.FloatHolder;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {
        /**
         * 线程停止
         */
        stopThread s = new stopThread();

        Thread t1 = new Thread(s);
        Thread t2 = new Thread(s);

        t1.start();
        t2.start();

        int num = 0;

        while (true) {

            if (num++ == 60) {
                s.changeFlag();
                break;
            } else {
                System.out.println(Thread.currentThread().getName()
                        + "Main run");
            }
        }
    }
}

class stopThread implements Runnable {

    private boolean flag = true;

    @Override
    public void run() {
        while (flag) {
            System.out.println(Thread.currentThread().getName() + "Thread run");
        }
    }

    public void changeFlag() {
        flag = false;
    }

}

~~~

> 逻辑十分简单，只要达到要求，就停止，但是还有一种特殊情况，当线程处于冻结状态，就不会读取到标记，那线程就不会结束，我们看

~~~
package com.lgl.hellojava;

import org.omg.CORBA.FloatHolder;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {
        /**
         * 线程停止
         */
        stopThread s = new stopThread();

        Thread t1 = new Thread(s);
        Thread t2 = new Thread(s);

        t1.start();
        t2.start();

        int num = 0;

        while (true) {

            if (num++ == 60) {
                s.changeFlag();
                break;
            } else {
                System.out.println(Thread.currentThread().getName()
                        + "Main run");
            }
        }
    }
}

class stopThread implements Runnable {

    private boolean flag = true;

    @Override
    public synchronized void run() {
        while (flag) {

            try {
                wait();
            } catch (InterruptedException e) {
                System.out.println(Thread.currentThread().getName()
                        + "InterruptedException run");
            }
            System.out.println(Thread.currentThread().getName() + "Thread run");
        }
    }

    public void changeFlag() {
        flag = false;
    }

}

~~~

> 这样就循环了。而在我们多线程中，提供了一个中断的方法Interupted

## 三.守护线程

> 守护线程其实也是Interupted中的东西，我们来看

![这里写图片描述](http://img.blog.csdn.net/20160607222859345)

> 你只要在启动线程前调用就可以了，就标记成了守护线程，就是一个依赖关系，你在我在，你不在我也不在；

## 四.Join方法

> 这个也是一个方法，意思是等待线程终止

![这里写图片描述](http://img.blog.csdn.net/20160608204155299)

> 我们倒是可以写个小例子

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {
        /**
         * Join
         */
        Demo d = new Demo();
        Thread t1 = new Thread(d);
        Thread t2 = new Thread(d);

        t1.start();

        try {
            // t1要申请加入到运行中来
            t1.join();
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        t2.start();

        for (int i = 0; i < 100; i++) {
            System.out.println("miam" + i);
        }
        System.out.println("main over");
    }
}

class Demo implements Runnable {

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + "=---" + i);
        }
    }

}

~~~

> 我们可以满足条件下 ，临时加入一个线程
> 
> 当A线程执行到了B线程的join方法时，A线程就会等待，等B线程都执行完，A才会执行，A可以用来临时加入线程执行。

## 五.线程的优先级

> 线程有优先级，默认的优先级都是5，这个是可以改变的，t1.setPriority(优先级);

![这里写图片描述](http://img.blog.csdn.net/20160608212535646)

> 我们可以拿上面的例子来做个比较

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {
        /**
         * Join
         */
        Demo d = new Demo();
        Thread t1 = new Thread(d);
        Thread t2 = new Thread(d);

        t1.start();
        //权限虽然高，只是频率高而已
        t1.setPriority(Thread.MAX_PRIORITY);
        t2.start();

        for (int i = 0; i < 100; i++) {
            System.out.println("miam" + i);
        }
        System.out.println("main over");
    }
}

class Demo implements Runnable {

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + "=---" + i);
        }
    }

}

~~~

> 我们这里还有一个小方法yield，临时停止的意思，我们可以看例子

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {
        /**
         * Join
         */
        Demo d = new Demo();
        Thread t1 = new Thread(d);
        Thread t2 = new Thread(d);

        t1.start();
        t2.start();

        for (int i = 0; i < 100; i++) {
            // System.out.println("miam" + i);
        }
        System.out.println("main over");
    }
}

class Demo implements Runnable {

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + "=---" + i);
            Thread.yield();
        }
    }

}

~~~

> 我们可以看到

![这里写图片描述](http://img.blog.csdn.net/20160608213244961)

> 主线程并没有运行，那就对了，因为暂停了
> 
> 我们到这里，本篇就结束了，同时线程所讲的知识也讲完了，线程博大精深，很值得我们学习，我所讲的，仍然只是一些皮毛罢了，希望大家多用心研究一下

### 我的群555974449也可以欢迎各位来讨论

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。