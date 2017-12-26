

# JAVA之旅（十二）——Thread，run和start的特点，线程运行状态，获取线程对象和名称，多线程实例演示，使用Runnable接口

* * *

> 开始挑战一些难度了，线程和I/O方面的操作了，继续坚持

## 一.Thread

> 如何在自定义的代码中，自定义一个线程呢？

![这里写图片描述](http://img.blog.csdn.net/20160603195312386)

> 我们查看API文档，我们要启动一个线程，先实现一个子类，

~~~
package com.lgl.hellojava;

public class MyThread extends Thread {

    @Override
    public void run() {
        System.out.println("我是一个线程");
    }
}

~~~

> 我们要调用它的话，只需要start就可以

~~~
MyThread thread = new MyThread();
thread.start();
~~~

> 这样就会执行我们的run方法

![这里写图片描述](http://img.blog.csdn.net/20160603200131274)

> 我们来理一下思路,线程有两种创建方式，我们先将第一种：

*   1.继承Thread 类
*   2.复写Thread类的run方法
*   3.调用线程的start方法 

    *   该方法有两个作用，启动线程和调用run方法

> 我们可以把代码改一下

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {

        MyThread thread = new MyThread();
        thread.start();
        for (int i = 0; i < 1140; i++) {
            System.out.println("Hello JAVA");
        }
    }

}

class MyThread extends Thread {

    @Override
    public void run() {
        for (int i = 0; i < 1140; i++) {
            System.out.println("我是一个线程");
        }
    }
}

~~~

> 这样输出多少？

![这里写图片描述](http://img.blog.csdn.net/20160603200904954)

> 你注意一下，他们交叉输出了，这种，就是多线成，两个线程同时在跑,都在获取cpu的使用权，cpu执行到谁，谁就执行，明确一点，在某一个时刻，只能有一个程序在运行（多核除外），cpu在做着快速的切换以达到看上去是同时运行的效果
> 
> 我们可以形象的把多线成的运行，形容为在互相抢夺CPU的资源，那么这就是多线成的一个特性叫随机性，谁抢到谁执行，但是执行多长，cpu说了算

## 二.run和start的特点

> 为什么要覆盖run方法
> 
> Thread类用于描述线程，该类就定义了一个功能，用于存储线程要运行的代码，该存储功能就是run方法
> 
> 也就是说Thread类汇总的run方法是用于存储线程执行的代码
> 
> 目的：将自定义的代码存储在run方法中让线程运行

## 三.线程运行状态

> 线程运行会有几种状态，我们要了解他的状态，才能了解他运行的原理，我们先来看一张图

![这里写图片描述](http://img.blog.csdn.net/20160603213356335)

> 我们一步步来分析，首先线程的第一种状态是创建，你new一个线程就是被创建了，紧接着，就是运行的状态，他们的过程，就是start，当然，线程还有一种为冻结，处于某一种状态，就交冻结，他们通过sleep来交替。最后就是线程结束了，通过stop，当然，还有其他一些状态，比如阻塞，这是临时状态，这是具备运行资格，但是没有执行权

## 四.获取线程对象和名称

> 线程都是有名称的，通过格式Thread-编号来区分，我们可以这样来验证

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {

        MyThread thread = new MyThread();
        thread.start();

    }

}

class MyThread extends Thread {

    @Override
    public void run() {

        System.out.println("线程名称："+this.getName());
    }
}

~~~

> 它输出的结果就是

![这里写图片描述](http://img.blog.csdn.net/20160603220039059)

> 他是从0开始，当然，他既然有getrName，那肯定有setName方法，其实他初始化的时候就有方法，父类已经给我们提供好了

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {

        MyThread thread = new MyThread("hello");
        thread.start();

    }

}

class MyThread extends Thread {

    public MyThread(String name) {
        super(name);
    }

    @Override
    public void run() {

        System.out.println("线程名称："+this.getName());
    }
}

~~~

> 那么我们输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160603220303095)

> 它还可以通过Thread.currentThread()来获取对象名称，它等同于this.getName();

## 五.多线程实例演示

> 我们来一个简单的实例来结束本篇blog，那就是卖票了，很多窗口都能卖票，这就是同时运行

~~~
package com.lgl.hellojava;

import javax.security.auth.callback.TextInputCallback;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {

        /**
         * 需求：简单的卖票程序，多个线程同时卖票
         */
        MyThread my1 = new MyThread();
        MyThread my2 = new MyThread();
        MyThread my3 = new MyThread();
        MyThread my4 = new MyThread();

        my1.start();
        my2.start();
        my3.start();
        my4.start();
    }

}

/**
 * 卖票程序
 * 
 * @author LGL
 *
 */
class MyThread extends Thread {

    // 票数
    private int tick = 100;

    @Override
    public void run() {
        while (true) {
            if (tick > 0) {
                System.out.println(currentThread().getName()+"卖票:" + tick--);
            }
        }
    }
}

~~~

> 我们这样就实现了票卖了，但是这里出了一个问题，四个线程，他一共卖了400张票，那可不行，火车就一百张票，这是不符合规则的，我们需要怎么改？让四个对象共享一个票数，那我们就需要静态了

~~~
// 票数
private static int tick = 100;
~~~

> 但是我们一般不定义静态，他的生命周期有点长，我们换一种角度考虑，其实这就关乎到创建方法了，我们在之前就讲个，线程创建有两种方法。

## 六.Runnable接口

> 我们需要使用第二种方法，所以是这样写的，实现Runnable的接口

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
                System.out.println("卖票:" + tick--);
            }
        }
    }
}

~~~

> 我们得到的输出结果就正确了

![这里写图片描述](http://img.blog.csdn.net/20160603223324961)

> 创建线程的第二种方式，实现Runnable接口

*   1.定义类实现Runnable接口
*   2.覆盖Runnable接口的run方法 

    *   将线程要运行的代码存放在该run方法中
*   3.通过Thread类建立线程对象
*   4.将Runnable接口的子类对象作为实际参数传递给Thread类的构造函数 

    *   为什么要将Runnable接口的子类对象传递给Thread的构造函数，因为，自定义的run方法所属的对象是Runnable接口的子类对象，所以要让线程去指定对象的run方法，就必须明确该run方法所属的对象
*   5.调用Thread类的start方法开启线程并调用Runnable接口的run方法

> 这两种方式有什么区别呢？

*   实现方式好处，避免了单继承的局限性，在定义线程时，建议使用实现方式
*   线程代码存放的位置不一样

### 小伙伴们有没有对线程了解的更深刻一点呢？不明白没关系，我们下篇还是接着讲线程，如果有兴趣可以加入群：555974449，欢迎一起交流

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。