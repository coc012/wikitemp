

# JAVA之旅（十六）——String类，String常用方法，获取，判断，转换，替换，切割，子串，大小写转换，去除空格，比较

* * *

> 过节耽误了几天，我们继续JAVA之旅

## 一.String概述

> String时我们很常用的数据类型，他也是一个类的使用

![这里写图片描述](http://img.blog.csdn.net/20160611123924843)

> 我们来看

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {
        /**
         * String
         */
        /**
         * s1是一个类类型变量，“abc”是一个对象 字符串最大的特点是，一旦被初始化，就不可以被改变
         */
        String s1 = "abc";
        s1 = "kk";
        System.out.println(s1);
    }
}

~~~

> 为什么说初始化之后不可以改变，我们又重新赋值，输出多少？肯定是kk，那不是变了吗？

*   这里注意，他是s1变了，但是这个abc这个对象还是abc

> 这个要搞清楚，s1开始指向的abc后来指向kk而已
> 
> 我们再来对比一下

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {

        String s1 = "abc";
        String s2 = new String("abc");

        System.out.println(s1 == s2);
        System.out.println(s1.equals(s2));
    }
}

~~~

> 输出的结果？

![这里写图片描述](http://img.blog.csdn.net/20160611125443802)

> 我们可以发现，==是不正确的，因为他是比较地址，而equals，则是比较值
> 
> 为什么？

*   String类复写了object类中的equals方法，定义了自己独特的内容，该方法用于判断字符串是否相同

> 那s1和s2有什么区别？

*   s1代表一个对象
*   s2代表两个对象（new 和 abc）

## 二.String常用方法

> 我们知道了String的基本概述了，那我们就可以开始来学习他的一些常用的方法了，我们还是以例子为准

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {

        String s1 = "abc";
        String s2 = new String("abc");

        String s3 = "abc";

        System.out.println(s1 == s2);
        System.out.println(s1 == s3);
    }
}

~~~

> 这里大家知道输出的是什么嘛

![这里写图片描述](http://img.blog.csdn.net/20160611130610728)

> s1 = s3 为true是因为当内存中存在了对象就不会再创建了
> 
> String是用于描述字符串事物，那么它就提供了多个方法的对字符串进行操作

常见的操作有哪些？我们来分析一下

*   1.获取 

    *   字符串中包含的字符数，也就是字符串的长度，也就是int length()获取长度
    *   根据位置获取位置上的某个字符，也就是char charAt(int index)
    *   根据字符获取该字符在字符串的位置 int indexOf(int ch),返回的是ch在字符串中第一次出现的位置
    *   int indexOf(int ch , int fromIndex):从fromIndex指定位置开始，获取ch在字符串中出现的位置
    *   根据字符串获取该字符在字符串的位置 int indexOf(String str),返回的是ch在字符串中第一次出现的位置
    *   int indexOf(String str , int fromIndex):从fromIndex指定位置开始，获取ch在字符串中出现的位置
*   2.判断

    *   字符串是否包含某一个子串

        > boolean contains(str):判断字符串是否存在 
        > 特殊之处：indexOf(str)可以索要str第一次出现的位置，返回-1的话，表示str不再字符串中存在，索要，也可以用于对指定判断是否包含，if(str.indexOf(“aa”) != -1)

    *   字符串中是否有内容

        > Boolean isEmpty():原理就是判断长度是否为0

    *   字符串是否是以指定的内容开头

        > boolean startWith(String str)

    *   字符串是否是以指定的内容结尾

        > boolean startWith(String str)

*   3.转换

    *   将字符数组转换成字符串

        > 构造函数String(char []) 
        > 构造函数（char [] , offset ，count）将字符数组中的一部分转成字符串 
        > 静态方法static String copyValueOf(char [] ) 
        > 静态方法static String copyValueOf(char [],int offset,int count )

    *   将字符串转换成字符数组

        > char [] toCharArray()

    *   讲字节数组转成字符串

        > 构造函数String(byte[]) 
        > 构造函数（byte[] , offset ，count）将字节数组中的一部分转成字符串

    *   将字符串转成字节数组

        > byte [] getBytes()

    *   将基本数据类型转换成字符串

        > String valueOf(xxx);

*   4.替换

    *   String replace(oldchar,newchar);
*   5.切割

    *   String [] split(regex);
*   6.子串

    > 获取字符串中的一部分 
    > String subString(begin) 
    > String subString(begin,end)

*   7.大小写转换，去除空格，比较

    *   将字符串转换成大小写

        > String toUuperCase() 
        > String toLowerCase();

    *   将字符串两端的多个空格去掉

        > String trim();

    *   对两个字符串进行自然顺序的比较

        > int compareTo(String)

> 我们可以对获取做一个小演示

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {

        method_get();

    }

    /**
     * String操作演示
     */
    public static void method_get() {
        String str = "abcdef";

        //长度
        sop(str.length());
        //根据索引获取字符
        //当访问到字符串中不存在角标的时候会发生错误：StringIndexOutOfBoundsException角标越界
        sop(str.charAt(3));
        //根据字符获取索引
        //没有角标不会报错，返回-1
        sop(str.indexOf('d'));

        //反向索引一个字符出现的位置
        sop(str.lastIndexOf('c'));
    }

    // 输出语句
    public static void sop(Object obj) {
        System.out.println(obj);
    }
}

~~~

> 输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160611230407101)

> 我们再来看看判断的小例子

~~~
    /**
     * 判断
     */
    public static void method_is() {
        String str = "LiuGuiLin";
        // 判断是以Liu开头
        sop(str.startsWith("Liu"));
        // 判断是以Lin结尾
        sop(str.endsWith("Lin"));
        // 判断是否存在Gui
        sop(str.contains("Gui"));

    }
~~~

> 我们的输出

![这里写图片描述](http://img.blog.csdn.net/20160611232034477)

> 字符串和字节数组在转换过程中是可以指定编码表，我们可以看一下转换的小例子

~~~

    /**
     * 转换
     */
    private static void method_trans() {
        // 字符数组
        char[] arr = { 'a', 'b', 'c', 'd', 'e', 'f', 'g' };
        // 转换成字符串
        String str = new String(arr);
        sop("str = :" + str);

        // 截取
        String str1 = new String(arr, 1, 3);
        sop("str1 = :" + str1);

        String str3 = "ddvdvdv";
        char[] arr3 = str3.toCharArray();
        for (int i = 0; i < arr3.length; i++) {
            sop("arr3 = :" + arr3[i]);
        }
    }
~~~

> 我们再来看下替换的方法

~~~
/**
     * 替换
     */
    public static void method_replace() {
        String s = "Hello JAVA";

        // 替换
        String s1 = s.replace('J', 'A');
        //如果要替换的字符不存在，返回的还是原串
        //当然，也可以替换字符串，这里就不演示了
        sop(s1);
    }
~~~

> 输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160612211317461)

> 当然，也是可以替换字符串的，这里就不演示了
> 
> 我们再来看切割的小例子

~~~
/**
     * 切割
     */
    public static void method_split() {
        String string = "zhangsan,lisi,wangwu";
        // 切割
        String[] arr = string.split(",");
        for (int i = 0; i < arr.length; i++) {
            sop("arr = :" + arr[i]);
        }
    }
~~~

> 这里我们按照逗号区分

![这里写图片描述](http://img.blog.csdn.net/20160612214743380)

> 我们再来看下子串

~~~
    /**
     * 子串
     */
    public static void method_sub() {
        String ss = "ferfefqwdqXXFV";
        sop(ss.substring(2));
        sop(ss.substring(2, 5));

    }
~~~

> 这个直接截图。很简单

![这里写图片描述](http://img.blog.csdn.net/20160612222811505)

> 好了我们再来演示最后几个方法的功能来结束本篇博客

~~~
/**
     * 最后几个
     */
    public static void method_7() {

        String st = "    Hello Java And Android   ";

        // 转换大写
        sop(st.toUpperCase());
        // 转换小写
        sop(st.toLowerCase());
        //去掉空格
        sop(st.trim());

        //比较
        String st1 = "acc";
        String st2 = "aaa";
        //一个相同
        sop(st1.compareTo(st2));

    }
~~~

> OK，这个也没什么可难的，输出

![这里写图片描述](http://img.blog.csdn.net/20160612224637200)

> 好的，本篇博客就先到这里了

## 我的群：555974449，欢迎加入

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。