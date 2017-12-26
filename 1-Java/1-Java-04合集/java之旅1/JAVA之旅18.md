

# JAVA之旅（十八）——基本数据类型的对象包装类，集合框架，数据结构，Collection，ArrayList,迭代器Iterator，List的使用

* * *

> JAVA把完事万物都定义为对象，而我们想使用数据类型也是可以引用的

## 一.基本数据类型的对象包装类

> 左为基本数据类型，又为引用数据类型

*   byte Byte
*   int Integer
*   long Long
*   boolean Booleab
*   float Float
*   double Double
*   char Character

> 我们拿Integer来举例子

~~~
//整数类型的最大值/最小值
sop("最大值："+Integer.MAX_VALUE);
sop("最小值："+Integer.MIN_VALUE);
~~~

> 输出

![这里写图片描述](http://img.blog.csdn.net/20160618161134665)

> 基本数据类型对象包装类的最常见作用

*   就是用于基本数据类型和字符串数据类型之间的转换

    *   基本数据类型转成字符串

        > 基本数据类型+“” 
        > Integer.toString（34）

    *   字符串转成基本数据类型

        ~~~
        // 将一个字符串转为整数
        int num = Integer.parseInt("123");
        sop("值：" + (num + 5));
        ~~~

        > 输出的结果

        ![这里写图片描述](http://img.blog.csdn.net/20160618162213434)

> 像其他的使用方法都是类似的，但是有特殊的，那就是boolean，一个是true一个是“true”，再比如你传的是abc转int类型，这就奇葩了，他会报数据格式异常的
> 
> 当然，还有各种进制的转换，说白了就是几个方法，大家可以研究一下，这里就不多做赘述了
> 
> 我们实际来一个小例子来突出他们的特性

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {

        /**
         * 基本数据类型的对象包装类
         */
        Integer x = new Integer("123");
        Integer y = new Integer(123);
        // 问
        sop("x == y :" + (x == y));
        sop("x.equals(y) : " + (x.equals(y)));

    }

    /**
     * 输出
     */
    public static void sop(Object obj) {
        System.out.println(obj);
    }

}

~~~

> 这样得到的结果呢？

![这里写图片描述](http://img.blog.csdn.net/20160618163033627)

> 这里就好理解了
> 
> JDK1.5版本以后出现的新特性

*   自动装箱

~~~
Integer x = new Integer(4);
Integer x = 4;  //自动装箱

x = x + 2; //进行了自动拆箱，变成了int类型，和2进行加法运算，再将和进行装箱，赋给x
~~~

> 再来一个有意思的例子

~~~

        Integer x = 128;
        Integer y = 128;

        Integer i = 127;
        Integer j = 127;

        sop("x == y :" + (x == y));
        sop("i == j : " + (i == j));
~~~

> 这里输出多少？

![这里写图片描述](http://img.blog.csdn.net/20160618174148203)

> 为什么会这样？

*   因为i和j是同一个Integer对象，在byte范围内，对于新特性，如果该数值已经存在，则不会再开辟新的空间

## 二.集合框架

> 讲完杂七杂八的数据类型，我们接着讲数据类型存储，首先我们聊聊集合

*   为什么出现集合类

    *   面向对象语言对事物的体现都是以对象的形式，所以为了方便对多个对象的操作，就对对象进行了存储，集合就是存储对象最常用的一种方式
*   数组和集合类同时容器有何不同？

    *   数组虽然也可以存储对象，但是长度是固定的，集合长度是可变的，数组中可以存储数据类型，集合只能存储对象
*   集合的特点

    *   集合只用于存储对象，集合长度是可变的，集合可以存储不同类型的对象

> 集合框架是不断的向上抽取的

![这里写图片描述](http://img.blog.csdn.net/20160618181913093)

> 为什么会出现这么多的容器呢？

*   因为每一个容器对数据的存储方式都有不同，这个存储方式我们称之为：**数据结构**

> 我们会依次的学习这个数据结构

## 三.Collection

![这里写图片描述](http://img.blog.csdn.net/20160618182443274)

> 根接口，我们来学习他们的共性方法

~~~
package com.lgl.hellojava;

import java.util.ArrayList;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {
        /**
         * Collection
         */
        // 创建一个集合容器，使用Collection接口的子类Arraylist
        ArrayList list = new ArrayList();
        // 添加元素
        list.add("Hello 0"); 
        list.add("Hello 1");
        list.add("Hello 2");
        list.add("Hello 3");
        list.add("Hello 4");

        // 获取集合的长度
        sop("长度:" + list.size());
    }

    /**
     * 输出
     */
    public static void sop(Object obj) {
        System.out.println(obj);
    }

}

~~~

> 写法是这样写的，这里有疑问，为什么add参数是Object?

*   1.add方法的参数类型是Object，已便于接收任意类型的对象
*   2.集合中存储的都是对象的引用和地址，

> 所以我们还可以

~~~
        // 获取集合的长度
        sop("长度:" + list.size());

        //打印集合
        sop(list);

        //删除元素
        list.remove("Hello 1");
        sop(list);

        //判断
        sop("Hello 3是否存在："+list.contains("Hello 3"));

        //清空
        list.clear();

        //是否为空
        sop(list.isEmpty());

        sop(list);

~~~

> 得出结论

![这里写图片描述](http://img.blog.csdn.net/20160618202852290)

> 我们再来讲一个交集

~~~
public static void method_2() {
        ArrayList list = new ArrayList();
        // 添加元素
        list.add("Hello 0");
        list.add("Hello 1");
        list.add("Hello 2");
        list.add("Hello 3");
        list.add("Hello 4");

        ArrayList list1 = new ArrayList();
        // 添加元素
        list1.add("java 0");
        list1.add("Hello 1");
        list1.add("java 2");
        list1.add("Hello 3");
        list1.add("java 4");

        // 取交集 list只会保留和list1中相同的元素
        list.retainAll(list1);
        sop(list);
    }
~~~

> list只会保留和list1中相同的元素

![这里写图片描述](http://img.blog.csdn.net/20160618204259153)

## 四.迭代器Iterator

> 我们再来说下迭代器，也就是怎么取出数据操作

~~~
/**
     * 迭代器
     */
    public static void method_get() {
        ArrayList list = new ArrayList();
        // 添加元素
        list.add("Hello 0");
        list.add("Hello 1");
        list.add("Hello 2");
        list.add("Hello 3");
        list.add("Hello 4");
        //获取迭代器，用于取出集合中的元素
        Iterator iterator = list.iterator();
        while (iterator.hasNext()) {
            sop(iterator.next());
        }
    }
~~~

> 这样就能全部打印出来了

![这里写图片描述](http://img.blog.csdn.net/20160618215347157)

> 那我们理解什么是迭代器？

*   其实就是集合取出元素的方式 

> 就把取出方式定义在集合的内部，这样取出的方式就直接访问集合内容的集合，那么取出方式就会定义成内部类，而每个容器的数据结构都不同，所以取出的动作细节也不同，但是都有共性内容，判断，取出，那么可以将写共性抽取，那么这些内部类都符合一个规则，该规则就是iterator,如何获取集合的取出对象呢？通过一个对外提供的方法interator(); 
> 大体的方向我们掌握了，这样我们就应该细分下去讲了，我们先说下List

## 六.List

> Collection下有两个子接口，list和set，他们两个的区别就是

*   list:元素是有序的，元素可以重复，因为该集合体系有索引
*   set:元素是无序，元素不可以重复，不能索引

> 我们只要说的就是list，共性的就不讲了，我们、他独有的
> 
> List特有方法：凡是可以操纵交表的方法都是该体系的特有方法，也就是

*   增 

    *   add(index,element)
    *   addAll(index Collection);
*   删 

    *   remove(index)
*   改 

    *   set(index)
*   查 

    *   get(index)
    *   subList(from,to)
    *   listIterator()

> 我们挨个说一遍就好了，这个本来就是老套路了，我们就算总结一下前面的知识

~~~
package com.lgl.hellojava;

import java.util.ArrayList;
import java.util.Iterator;

//公共的   类   类名
public class HelloJJAVA {

    public static void main(String[] args) {
        /**
         * List
         */
        ArrayList al = new ArrayList();
        // 添加元素
        al.add("java 0");
        al.add("java 1");
        al.add("java 2");

        sop("原集合 : " + al);

        // 在指定位置添加元素
        al.add(1, "java 3");
        sop("添加后 ： " + al);

        // 删除指定位置的元素
        al.remove(2);
        sop("删除后： " + al);

        // 修改元素
        al.set(2, "java 4");
        sop("修改后： " + al);

        // 通过角标获取元素
        sop("查找 ： " + al.get(1));

        sop("所有元素");
        // 获取所有元素
        for (int i = 0; i < al.size(); i++) {
            sop("元素" + i + " = " + al.get(i));
        }

        sop("迭代器");
        // 通过迭代器
        Iterator it = al.iterator();
        while (it.hasNext()) {
            sop("next:" + it.next());
        }

    }

    /**
     * 输出
     */
    public static void sop(Object obj) {
        System.out.println(obj);
    }

}

~~~

> 这里就涵盖了很多的list的知识点，不断向上抽取的一个过程了，我们输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160618223727920)

> 好的，那这样的话，我们本节课也就到这里，OK了，感谢你看了这么久，累了就喝杯水吧！

## 我的群：555974449，欢迎你的到来！

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。