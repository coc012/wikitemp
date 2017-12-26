

# JAVA之旅（十七）——StringBuffer的概述，存储，删除，获取，修改，反转，将缓存区的数据存储到数组中，StringBuilder

* * *

> 讲完String，我们来聊聊他的小兄弟

## 一.StringBuffer概述

> 关于StringBuffer这个对象，Buffer是什么意思？缓冲区的意思，String一旦初始化时不可以被改变的，而StringBuffer是可以的，这就是区别，特点：

*   StringBuffer是一个容器
*   可以字节操作多个数据类型
*   最终会通过toString方法变成字符串

![这里写图片描述](http://img.blog.csdn.net/20160615211938069)

*   存储

    > StringBuffer append():将指定的数据作为参数添加到已有数据的结尾处

*   删除

    > StringBuffer delete(start , end)删除缓冲区的数据，包含start,不包含end 
    > StringBuffer deleteCharAt(index)删除指定位置的字符

*   获取

    > char charAt(int index) 
    > int indexOf(String str) 
    > int lasrIndexOf(String str) 
    > String subString(int start,int end)

*   修改

    > StringBuffer replace(start,end,string) 
    > void setChatAt(int dex,char ch)

*   反转

    > String reverse()

*   将缓存区的数据存储到数组中

    > void getChars(int srcBegin,int srcEnd ,char[] dst,int dstBegin)

> 有着这样的特性，那我们逐步来讲一下

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {

        /**
         * StringBuffer
         */
        StringBuffer sb = new StringBuffer();
        StringBuffer append = sb.append(78);
        sop(sb == append);
        sop(sb.toString());
        sop(append.toString());

    }

    /**
     * 输出
     */
    public static void sop(Object obj) {
        System.out.println(obj);
        }

    }
~~~

> 这算是比较常见的吧，我们没必要这么麻烦，我们可以简化

~~~
sb.append("abc").append(36);
sop(sb.toString());
~~~

> 我们可以直接输出字符串

![这里写图片描述](http://img.blog.csdn.net/20160615212354602)

> 这个连续的方法叫做方法调用链
> 
> 因为StringBuffer的特性，我们可以在里面插入数据，我现在想在a后面插入字符串，怎么实现呢？

~~~
sb.append("abc").append(36);
sb.insert(1, "lgl");
sop(sb.toString());
~~~

> 没错。insert，他的两个参数，一个是下标，一个是数据，这样，我们就插入成功了

![这里写图片描述](http://img.blog.csdn.net/20160615214639501)

> 我们再来聊一下删除

~~~

    /**
     * 删除
     */
    public static void method_delete() {
        StringBuffer sb = new StringBuffer("abcdefg");

        sop(sb.toString());
        // 删除bc
        // sop(sb.delete(1, 3).toString());
        // 删除d
        sop(sb.deleteCharAt(3));
        // 清空缓冲区
        sop("all:" + sb.delete(0, sb.length()));

    }
~~~

> 其实这些都是比较简单的

![这里写图片描述](http://img.blog.csdn.net/20160615223727630)

> OK,按照顺序我们现在讲获取了,其实我们在将String的时候就已经讲过了，这里就不多说了。我们说修改，修改是比较经典的，修改数据我们这样写

~~~

    /**
     * 修改
     */
    public static void method_update() {
        StringBuffer sb = new StringBuffer("abcdefg");
        // 替换一部分
        sop(sb.replace(1, 4, "java"));
        // 替换一个
        sb.setCharAt(sb.length() - 1, 'k');
        sop(sb.toString());
    }
~~~

> 结果

![这里写图片描述](http://img.blog.csdn.net/20160615225002510)

> OK，修改成功，将缓冲区存储到数组中

~~~
/**
     * 将缓存区的数据存储到数组中
     */
    public static void method_getchar() {
        StringBuffer sb = new StringBuffer("abcdefg");
        char[] chs = new char[4];
        /**
         * 从1开始，4结束，存在chs里，从头1开始存
         */
        sb.getChars(1, 4, chs, 1);

        for (int i = 0; i < chs.length; i++) {
            sop("char[" + i + "] = " + chs[i] + ";");
        }
    }

~~~

> 输出的结果，嘿嘿

![这里写图片描述](http://img.blog.csdn.net/20160618154550305)

## 二.StringBuilder

> 这个在JDK1.5之后才有

*   StringBuffer：线程同步
*   StringBuilder：线程不同步

> 开发中不建议使用StringBuilder
> 
> 我们看一下他的API说明：

![这里写图片描述](http://img.blog.csdn.net/20160618155730441)

> 用法差不多，就不多讲了，本篇闲到这里

## 有兴趣加群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。