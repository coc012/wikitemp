

# JAVA之旅（二十四）——I/O流，字符流，FileWriter，IOException，文件续写，FileReader，小练习

* * *

> JAVA之旅林林总总也是写了二十多篇了，我们今天终于是接触到了I/O了。如果你初学，不懂IO流，你可以从前往后慢慢看，但是你工作了一段时间你会发现，流的使用场景以及技术点是非常的强硬的，我们势必要掌握这个知识点，如果你觉得翻阅API比较鼓噪，看视频得不到精髓，看书看不到要点，你就跟随我的JAVA之旅，一起去探索吧！

## 一.I/O概述

> I/O全名：Input Output，输入输出的意思

*   IO流用来处理设备之间的数据传输
*   JAVA对数据的操作都是通过流的方式
*   JAVA用于操作流的对象都在IO包里
*   流操作分两种：字节流，字符流
*   流按流向分为：输入流，输出流

> 对数据的操作，其实就是File文件，我们可以去网上偷张图片来描述我们本大系列的所有知识点

![这里写图片描述](http://img.blog.csdn.net/20160702105442157)

> **图片来自于网络**

*   字节流的抽象积累

    *   InputStream
    *   OutputStream
*   字符流的抽象基类

    *   Reader
    *   Writer

> 从图中可以看出，都是从这四个类中派生出来的子类，但是他们的后缀都是这四个

## 二.FileWriter

> 我们先从字符流开始，肯定是从子类对象下手，我们对文件操作开始吧！

*   需求：在硬盘上创建一个文件，并且写入数据

> 那我们怎么去做？他构造函数比较多的，我们看例子

~~~
package com.lgl.hellojava;

import java.io.FileWriter;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {
        /**
         * 需求：在硬盘上创建一个文件，并且写入数据
         */

        // 一被初始化就必须要有被操作的文件
        // 如果不指定位置，就创建在同目录下
        // 如果目录下存在同名文件，覆盖
        try {
            FileWriter fileWriter = new FileWriter("test.txt");
            // 写入数据到内存
            fileWriter.write("abcde");
            // 刷新该流的缓冲
            // fileWriter.flush();

            // 关闭流 关闭之前会刷新，和flush的区别在于flush刷新后流可以继续工作
            fileWriter.close();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

~~~

> 这样在我们的项目根目录下就可以看到生成的文件了

![这里写图片描述](http://img.blog.csdn.net/20160702123225142)

> 我用白话再说一遍吧，其实就是创建fileWriter ，他没有空构造函数，你创建一个文件，可以传文件名或者路径，然后wirter写数据，这样你是看不到的，你需要刷新，刷新是刷新缓冲区，你现在就可以看到了，抛异常，还有关闭，关闭之前会刷新的，但是这个流就没用了，根据自己的场景来分析

## 三.IOException、

> 我们来看看怎么处理IO的异常，IO异常大致有三个，一个是IO异常，一个是找不到文件异常，还有一个就是没有对象异常了，我们比较严谨的写法

~~~
package com.lgl.hellojava;

import java.io.FileWriter;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {

        FileWriter fileWriter = null;
        try {
            fileWriter = new FileWriter("demo.txt");
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } finally {
            try {
                if (fileWriter != null) {
                    fileWriter.close();
                }
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }
}

~~~

## 四.文件续写

> 我们知道，文件存在的话就会覆盖，但是我们不想这样，我们想在原有的数据中续写，这该去怎么做？

~~~
package com.lgl.hellojava;

import java.io.FileWriter;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {

        try {
            //参数2代表不覆盖已有的文件，支持续写
            FileWriter fileWriter = new FileWriter("demo.txt",true);
            fileWriter.write("你好");
            fileWriter.close();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

    }
}

~~~

> 构造传参的时候设置为true就可以续写文件了

## 五.FileReader

> 既然写已经会了，那我们就来读取了

~~~
package com.lgl.hellojava;

import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {
        try {
            // 创建一个文件读取流对象，和指定名称的文件关联，保证文件存在，
            // 如果不存在,异常
            FileReader fileReader = new FileReader("demo.txt");
            // 读取单个字符,自动往下读
            int cd = fileReader.read();
            System.out.println((char) cd);

            //全部打印
            int ch = 0;
            while ((ch = fileReader.read()) != -1) {
                System.out.println(ch);
            }

            fileReader.close();

        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

~~~

> 这样就可以按照字节读取了，我们也可以把读到的字符存储在数组中

~~~
package com.lgl.hellojava;

import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {

        try {
            FileReader fileReader = new FileReader("demo.txt");
            char[] buf = new char[3];

            int num = fileReader.read(buf);

            System.out.println("num:" + num + new String(buf));

        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

    }
}

~~~

> OK，读出来了

![这里写图片描述](http://img.blog.csdn.net/20160702143241921)

## 六.小练习

> 我们字符流的读取和一些小操作算是了解了一点了，我们用一个小练习来结束本篇幅吧

*   需求：读取一个.java的文件，打印出来

> 好的，其实这个是比较简单的，我们看代码

~~~
package com.lgl.hellojava;

import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {
        try {
            FileReader fileReader = new FileReader("Single.java");
            char[] cs = new char[1024];
            int num = 0;
            while ((num = fileReader.read(cs)) != -1) {
                System.out.println(new String(cs, 0, num));
            }
            fileReader.close();
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

    }
}

~~~

> 是不是比较简单，读取到之后就直接存在数组中，打印出来

![这里写图片描述](http://img.blog.csdn.net/20160702143816202)

> OK,到这里我们的IO入门算是了解了一点，不过这还不够，我们应该继续深入一下，我们下一篇继续跟进IO，敬请期待！

## 欢迎加群：555974449，我们一起探索！

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。