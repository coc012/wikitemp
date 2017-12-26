

# JAVA之旅（二十三）——System，RunTime，Date，Calendar，Math的数学运算

* * *

> map实在是太难写了，整理得我都晕都转向了，以后看来需要开一个专题来讲这个了，现在我们来时来学习一些新的小东西吧

## 一.System

> 翻译过来系统的意思，系统类，里面的方法都是静态的，可以直接调用

![这里写图片描述](http://img.blog.csdn.net/20160701212259022)

> 我们来演示一下吧，先从获取系统信息开始：

~~~
package com.lgl.hellojava;

import java.util.Properties;

public class HelloJJAVA {
    public static void main(String[] args) {
        /**
         * 描述系统的一些信息 获取系统的一些信息 ： 
         * Properties = getProperties 
         * out：标准输出，默认是控制台
         * in:标准输入，默认控制台
         */

        Properties properties = System.getProperties();
        /**
         * 因为Properties是HashTab的子类，也就是map集合的一个子类对象 
         * 那么可以用map的方法取出集合中的元素,该集合存储中都是字符串，
         * 没有泛型定义
         */

        for (Object obj : properties.keySet()) {
            String value = (String) properties.get(obj);
            System.out.println(obj + ":" + value);
        }

    }
}

~~~

> 获取到的信息太多了，就不列出来了，可以看到

![这里写图片描述](http://img.blog.csdn.net/20160701212405915)

> 他把什么都打印出来了，如果你想自定义一些信息，你可以这样做

~~~
        /**
         * 如何在系统中自定义一些特有信息
         */
        System.setProperty("mykey", "myvalue");

~~~

> 这样就可以用上面的方法获取信息，我们也可以获取单个属性信息，比如获取系统名称

~~~
package com.lgl.hellojava;

public class HelloJJAVA {
    public static void main(String[] args) {

        /**
         * 获取指定属性信息
         */
        String property = System.getProperty("os.name");
        System.out.println(property);

    }
}

~~~

> OK,就能获取到系统名字了

![这里写图片描述](http://img.blog.csdn.net/20160701213552698)

## 二.RunTime

> 该类是单例设计模式，不提供构造函数，也就是不能new对象，，发现该类还有非静态方法， 那他肯定会提供一个方法获取本类对象，而且该方法是静态并且返回值是本类对象，RunTime翻译过来就是执行的意思，是很强大的，就如cmd一样可以运行linux命令，我们来演示一下，比如我们执行打开gitbash

~~~
package com.lgl.hellojava;

import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {

        Runtime r = Runtime.getRuntime();
        try {
            r.exec("C:\\Program Files\\Git\\git-bash");
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

    }
}

~~~

> 这样，一运行就打开了。杀掉的话返回一个Process,执行destroy就可以了

## 三.Date

> 日期的描述类，这个比较实用，也是比较简单的，比如

~~~
package com.lgl.hellojava;

import java.util.Date;

public class HelloJJAVA {
    public static void main(String[] args) {
        Date date = new Date();
        System.out.println(date);
    }
}

~~~

> 我们就可以打印出时间了

![这里写图片描述](http://img.blog.csdn.net/20160701220607873)

> 日期，月份，号，时间，年，不过有点费劲，我们可以按照格式来输出，我们要看这里

![这里写图片描述](http://img.blog.csdn.net/20160701220759061)

> 我们要使用的就是DateFormat,我们要用的就是他的子类SimpleDateFormat

~~~
package com.lgl.hellojava;

import java.text.SimpleDateFormat;
import java.util.Date;

public class HelloJJAVA {
    public static void main(String[] args) {
        Date date = new Date();
        //将模式封装
        SimpleDateFormat format = new SimpleDateFormat("yyyy年MM月dd日 hh:mm:ss");
        //格式化Date对象
        String time = format.format(date);
        System.out.println(time);
    }
}

~~~

> 这样，我们就可以用格式来表示了

![这里写图片描述](http://img.blog.csdn.net/20160702094620596)

## 四.Calendar

> 如果我想单独获取一个时间呢？比如年，比如月，这个时候就可以用Calendar了

~~~
package com.lgl.hellojava;

import java.util.Calendar;

public class HelloJJAVA {
    public static void main(String[] args) {

        Calendar calendar = Calendar.getInstance();
        String[] mons = { "一月", "二月", "三月", "四月", "五月", "六月", "七月", "八月", "九月",
                "十月", "十一月", "十二月" };
        int index = calendar.get(Calendar.MONTH);

        //查询当前日期
        sop(calendar.get(Calendar.YEAR) + "年");
        sop((calendar.get(Calendar.MONTH) + 1) + "月");
        sop(mons[index]);
        sop(calendar.get(Calendar.DAY_OF_MONTH) + "日");
        sop("星期：" + calendar.get(Calendar.DAY_OF_WEEK));

    }

    public static void sop(Object obj) {
        System.out.println(obj);
    }
}

~~~

> OK，这样就可以获取到了

![这里写图片描述](http://img.blog.csdn.net/20160702101847082)

> 他比较灵活，还是有许多的小技巧的，这个我们自行去探索

## 五.Math

> 数学类，在特定领域用的是非常多的，里面度是静态的，也就是工具类，我们来认识一下他

### 1.ceil

> 返回大于指定数据的最小整数

~~~
//返回大于指定数据的最小整数
double ceil = Math.ceil(12.34);
~~~

> 这里输出就是13.0了

### 2.floor

> 返回小于指定数据的最小整数

~~~
//返回小于指定数据的最小整数
double ceil1 = Math.floor(12.34);
sop(ceil1);
~~~

> 返回就是12.0了

### 3.round

> 四舍五入

~~~
//四舍五入
long ceil2 = Math.round(12.34);
sop(ceil2);
~~~

> 很明显，输出12

### 4.pow

> 幂的运算

~~~
// 2的3次方
double ceil3 = Math.pow(2, 3);
sop(ceil3);
~~~

> 得到8

### 5.random

> 随机数

~~~
/ 随机数
int ceil4 = (int) (Math.random()*10);
sop(ceil4);
~~~

> 这个一定要学会，很好用，也很常用
> 
> 还可以这样写

~~~
Random r = new Random();
sop(r.nextInt(10));
~~~

> 这些大部分是工具类的使用，当然，这些也是比较使实用的类，大家一定要掌握，本篇有点短，因为我下一篇想开一个比较特别的知识类，因为JAVA学习也挺久的了，这个知识点大家一定要掌握，就是I/O，文件流的操作，是非常的重点的，为了知识的终结和归纳，我决定在开新文章来写，这篇博文到这里就结束了，

## 如果感兴趣，可以加群：555974449，我们一起学技术！

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。