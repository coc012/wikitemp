

# JAVA之旅（十九）——ListIterator列表迭代器，List的三个子类对象，Vector的枚举，LinkedList,ArrayList和LinkedList的小练习

* * *

> 关于数据结构，所讲的知识太多了，我们只能慢慢的来分析了

## 一.ListIterator列表迭代器

> ListIterator列表迭代器是个什么鬼？我们通过一个小例子来认识他

~~~
package com.lgl.hellojava;

import java.util.ArrayList;
import java.util.Iterator;

import javax.print.attribute.standard.MediaSize.Other;

//公共的   类   类名
public class HelloJJAVA {
    public static void main(String[] args) {
        listiterator();
    }

    /**
     * 演示列表迭代器
     */
    public static void listiterator() {
        /**
         * 需求：对集合中的元素取出，在取的过程中进行操作
         */
        ArrayList al = new ArrayList<>();
        al.add("hello 01");
        al.add("hello 02");
        al.add("hello 03");
        al.add("hello 04");

        // 在迭代过程中，准备添加或者删除元素
        Iterator it = al.iterator();
        while (it.hasNext()) {
            Object obj = it.next();

            if (obj.equals("hello 02")) {
                // 将这个元素删除
                al.add("hello 05");
                sop(obj);
            }
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

> 我们用老方法的思路去做我们的需求，是这样的，你运行后悔发现

![这里写图片描述](http://img.blog.csdn.net/20160621220228450)

> 他打印了02缺报异常了，我们于是要去查看API

![这里写图片描述](http://img.blog.csdn.net/20160621220408546)

> 他不清楚怎么去操作，这也是这个迭代器的缺陷

*   list集合特有的迭代器叫做ListIterator，是Iterator的子接口，在迭代时，不可以通过集合对象的方法操作集合中的元素，因为会发现并发修改异常，所以，在迭代器时，只能用迭代器的方式操作元素，可是Iterator方法有限，如果想要其他的操作如添加，修改等，就需要使用其子接口：ListIterator，该接口只能通过list集合的ListIterator方法获取

> 所以我们可以这样去修改

~~~
package com.lgl.hellojava;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.ListIterator;

//公共的   类   类名
public class HelloJJAVA {
    public static void main(String[] args) {
        listiterator();
    }

    /**
     * 演示列表迭代器
     */
    public static void listiterator() {
        /**
         * 需求：对集合中的元素取出，在取的过程中进行操作
         */
        ArrayList al = new ArrayList<>();
        al.add("hello 01");
        al.add("hello 02");
        al.add("hello 03");
        al.add("hello 04");

        sop(al);

        ListIterator li = al.listIterator();
        while (li.hasNext()) {
            Object object = li.next();
            if (object.equals("hello 03")) {
                li.add("lgl");
            }

        }
        sop(al);

        // 在迭代过程中，准备添加或者删除元素
        // Iterator it = al.iterator();
        // while (it.hasNext()) {
        // Object obj = it.next();
        //
        // if (obj.equals("hello 02")) {
        // // 将这个元素删除
        // // al.add("hello 05");
        // it.remove();
        // sop(obj);
        // }
        // sop(al);
        // }
    }

    /**
     * 输出
     */
    public static void sop(Object obj) {
        System.out.println(obj);
    }

}

~~~

> 不仅可以增加，还可以修改，删除等操作

![这里写图片描述](http://img.blog.csdn.net/20160621222847376)

## 二.List的三个子类对象

> List有三个子类对象

*   ArrayList
*   LinkedList
*   Vector

> 为什么会有三个？因为底层的数据结构不一样，那具体是怎么样的呢？

*   ArrayList 

    *   底层的数据结构使用的是数组结构，特点：查询速度很快，但是增删稍慢，元素不多的话，你体会不到的。线程不同步。
*   LinkedList 

    *   底层使用的链表数据结构，特点：增删的速度很快，查询的速度慢。
*   Vector 

    *   底层是数组数据结构，线程同步。做什么都慢，被ArrayList替代了

> ArrayList和Vector都是数组结构，他们有什么具体的区别呢？

*   ArrayList构造一个初始容量为10的空列表，当长度超过10之后增加50%,而后者，长度也为10，超过的话，是100%延长至20，比较浪费空间

## 三.Vector的枚举

> 上面说了这么多，虽然Vector不用，但是我们还是要熟悉一下，面试的时候也是会问到的，所以我们写个小例子

~~~
package com.lgl.hellojava;

import java.util.Enumeration;
import java.util.Vector;

//公共的   类   类名
public class HelloJJAVA {
    public static void main(String[] args) {
        Vector v = new Vector();

        v.add("hello 01");
        v.add("hello 02");
        v.add("hello 03");
        v.add("hello 04");

        // 返回枚举
        Enumeration elements = v.elements();
        while (elements.hasMoreElements()) {
            sop(elements.nextElement());
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

> 这样就输出了 
> 枚举就是Vector特有的取出方式，我们发现枚举和迭代器很像，其实枚举和迭代时一样的，因为枚举的名称以及方法的名称都过长，所以被迭代器所取代了。枚举就郁郁而终了

## 四.LinkedList

> 我们继续来说list的子类对象LinkedList，事实上区别都不是很大，所以我们只说一些特有的特点
> 
> LinkedList特有方法：

*   addFirst()
*   addLast();
*   getFirst();
*   getLast();
*   removeFirst();
*   removeLast();

> 我们来看

~~~
package com.lgl.hellojava;

import java.util.LinkedList;

//公共的   类   类名
public class HelloJJAVA {
    public static void main(String[] args) {

        LinkedList list = new LinkedList<>();

        list.addFirst("hello 0");
        list.addFirst("hello 1");
        list.addFirst("hello 2");
        list.addFirst("hello 3");

        sop(list);
    }

    /**
     * 输出
     */
    public static void sop(Object obj) {
        System.out.println(obj);
    }

}

~~~

> 输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160622220756761)

> 我们一直添加头部，就是倒序了，这个逻辑应该都知道吧！要是addLast()，那就是顺序了，我们要想知道他们的头部和尾部的数值，也就直接get就是了

~~~
    sop(list.getFirst());
    sop(list.getLast());
~~~

> 获取元素，但是元素被删除，如果集合中没有元素，会出异常，在JDK1.6版本后出现了替代方法

*   offerFirst();
*   offerLast();

*   peekFirst();

*   peekLast();

*   pollFist();

*   pollLast(); 

> 之类的

## 五.小练习

> 我们写个小练习吧，我们先看LinkedList

~~~
package com.lgl.hellojava;

import java.util.LinkedList;

//公共的   类   类名
public class HelloJJAVA {
    public static void main(String[] args) {
        /**
         * 使用LinkedList模拟一个堆栈或者队列数据模式 堆栈：现进后出 队列：先进先出
         */
        DuiLie dl = new DuiLie();
        dl.MyAdd("hello 01");
        dl.MyAdd("hello 02");
        dl.MyAdd("hello 03");
        dl.MyAdd("hello 04");

        while (!dl.isNull()) {
            sop(dl.myGet());
        }

    }

    /**
     * 输出
     */
    public static void sop(Object obj) {
        System.out.println(obj);

    }

}

/**
 * 队列
 * 
 * @author LGL
 *
 */
class DuiLie {

    private LinkedList link;

    public DuiLie() {
        link = new LinkedList<>();
    }

    public void MyAdd(Object obj) {
        link.addFirst(obj);
    }

    public Object myGet() {
        return link.removeLast();
    }

    public boolean isNull() {
        return link.isEmpty();
    }

}

~~~

> 自己写的一个链表，去使用它，这样输出的结果就是

![这里写图片描述](http://img.blog.csdn.net/20160622224908384)

> OK，那我们继续，写一个ArrayList的小例子

~~~
package com.lgl.hellojava;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.LinkedList;

//公共的   类   类名
public class HelloJJAVA {
    public static void main(String[] args) {
        /**
         * 去除ArrayList的重复元素
         */

        ArrayList list = new ArrayList<>();

        list.add("Hello 01");
        list.add("Hello 02");
        list.add("Hello 03");
        list.add("Hello 02");
        list.add("Hello 05");

        sop(list);
        list = Method(list);
        sop(list);

    }

    public static ArrayList Method(ArrayList list){
        //临时容器
        ArrayList newList = new ArrayList<>();
        Iterator iterator = list.iterator();
        while (iterator.hasNext()) {
            Object object = (Object) iterator.next();

            if(!newList.contains(object)){
                newList.add(object);
            }
        }
        return newList;
    }

    /**
     * 输出
     */
    public static void sop(Object obj) {
        System.out.println(obj);

    }

}

~~~

> 输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160622230216952)

> 好的，我们本篇幅到这里，就写完了，但是我们的数据结构只是讲了一些皮毛而已，我们接下来的几天，将会一一为大家讲解

## 我的群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。