

# JAVA之旅（九）——Object类，equals，toString，getClass，内部类访问规则，静态内部类,内部类原则，匿名内部类

* * *

> 天天被一些琐事骚扰，学习还得继续

## 一.Object类

> Object是什么呢？我们可以翻阅JAVA API文档看他的介绍

![这里写图片描述](http://img.blog.csdn.net/20160529091530145)

> 上面介绍说，object是类层次结构的根类，也就是超类

*   Object：是所有对象的直接后者间继承关系，传说中的老祖宗，你父亲还有父亲，你父亲的父亲还有父亲是爷爷，这是继承关系，但是你的祖宗却只有一位，该类中定义的肯定是所有对象都具备的功能

~~~
class Demo { // extends Object

}

~~~

### 1.equals

> 我们可以用equals去比较两个对象是否相同

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        // 比较
        Demo1 demo1 = new Demo1();
        Demo2 demo2 = new Demo2();
        System.out.println(demo1.equals(demo2));
    }
}

class Demo1 {

}

class Demo2 {

}
~~~

> 结果肯定返回的是false

![这里写图片描述](http://img.blog.csdn.net/20160529092855306)

> 这里我们要注意的是，他比较的是**内存地址**
> 
> 假设我们需要定义一个比较内容

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        // 比较
        Demo demo1 = new Demo(5);
        Demo demo2 = new Demo(8);
        System.out.println(demo1.Comper(demo2));
    }
}

class Demo {

    private int num;

    public Demo(int num) {
        this.num = num;
    }

    public boolean Comper(Demo d) {
        return this.num == d.num;
    }
}

~~~

> 他返回的也是false，但是我们有必要这样去做吗？

*   object类中已经提供了对对象是否相同的比较方法，如果自定义类中也有相同的功能，没有必要重新定义，只要沿袭父类的功能，简历自己的特有内容即可，这就是覆盖，所以我们已经复写

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        // 比较
        Demo demo1 = new Demo(5);
        Demo demo2 = new Demo(5);
        System.out.println(demo1.equals(demo2));
    }
}

class Demo {

    private int num;

    public Demo(int num) {
        this.num = num;
    }

    @Override
    public boolean equals(Object obj) {
        // TODO Auto-generated method stub
        return this.num == ((Demo)obj).num;
    }
}

~~~

> 这里运用了多态的向下转型

![这里写图片描述](http://img.blog.csdn.net/20160529094448172)

### 2.toString

> 转换成字符串

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        // 比较
        Demo demo = new Demo(5);
        System.out.println(demo.toString());
    }
}

class Demo {

    private int num;

    public Demo(int num) {
        this.num = num;
    }
}

~~~

> 转换的结果十什么呢？

![这里写图片描述](http://img.blog.csdn.net/20160529095413113)

> 这个是什么值呢？

*   类名@哈希值

> 什么是哈希值？我们可以用toHexString来转换

~~~
System.out.println(demo.toString());
System.out.println(Integer.toHexString(demo.hashCode()));
~~~

![这里写图片描述](http://img.blog.csdn.net/20160529095753208)

### 3.getClass

> 这个就不用多说，返回当前运行的Class，所以

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        // 比较
        Demo demo = new Demo(5);
        /*
         * System.out.println(demo.toString());
         * System.out.println(Integer.toHexString(demo.hashCode()));
         */

        System.out.println(demo.getClass());
    }
}

class Demo {

    private int num;

    public Demo(int num) {
        this.num = num;
    }
}

~~~

> 我们这里就直接返回了Demo

![这里写图片描述](http://img.blog.csdn.net/20160529103145172)

> 方法还有很多，比如getName

~~~
System.out.println(demo.getClass().getName());
~~~

> 得到的就是Demo这个名称了

## 二.内部类

> 这是一个小知识点，我们先看一下概念

*   将一个类定义在另一个类里面，对立面那个类就称为内部类（内置类，嵌套类）

*   访问特点

    *   内部类可以直接访问外部类的成员，包括私有成员
    *   而外部类要访问内部类中的成员就必须建立内部类的对象

> 我们来写一个内部类

~~~
class Outer {
    int x = 3;

    void show() {
        System.out.println("x = " + x);
        new Inner().fini();
    }

    /**
     * 内部类
     * 
     * @author LGL
     *
     */
    class Inner {
        void fini() {
            System.out.println("内部类"+x);
        }
    }
}

~~~

> **内部类的访问规则上面已经体现了**

*   内部类可以直接访问外部类的成员，包括私有成员
*   而外部类要访问内部类中的成员就必须建立内部类的对象

> 那我们可以不可以直接访问内部类中的成员呢？

~~~
Outer.Inner inner = new Outer().new Inner();
inner.fini();
~~~

> 这样就可以访问了，不过我们一般都不这样做，因为大多数的情况我们会将内部类私有
> 
> 那你有没有想过？为什么匿名内部类可以访问外部的成员？我们可以做一个这样的小测试，在内部类里面定义一个x分别是成员变量和局部变量

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        Outer.Inner inner = new Outer().new Inner();
        inner.fini();
    }
}

class Outer {
    int x = 3;

    void show() {
        System.out.println("x = " + x);
        new Inner().fini();
    }

    /**
     * 内部类
     * 
     * @author LGL
     *
     */
    class Inner {
        int x = 5;

        void fini() {
            int x = 6;
            System.out.println("内部类" + x);
        }
    }
}

~~~

> 我们现在输出的这个x你知道是多少吗？结果显而易见，是6

![这里写图片描述](http://img.blog.csdn.net/20160529110853579)

> 那我现在想打印这个5怎么打？用this就行了

![这里写图片描述](http://img.blog.csdn.net/20160529110918051)

> 那我们想打印这个3呢？this是内部类的，那我们需要外面的this，就用Outer.this.x，输出的就是3了

![这里写图片描述](http://img.blog.csdn.net/20160529111000427)

> 之所以可以直接访问外部类中的成员是因为内部类中持有了一个外部类的引用，该引用写法是：外部类名.this

## 三.静态内部类

> 当内部类在成员位置上，就可以被成员修饰符所修饰，比如：

*   private,将内部类在外部类中进行封装
*   static,内部类就具备了static的特性，当内部类被static修饰后，只能直接访问外部类中的static的成员，出现了访问局限，但是静态内部类出现的不是很多，毕竟有访问局限

![这里写图片描述](http://img.blog.csdn.net/20160529112407632)

> 在外部类中，我们是如何访问静态内部类呢？

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        new Outer.Inner().fini();
    }
}

class Outer {
    private static int x = 3;

    /**
     * 内部类
     * 
     * @author LGL
     *
     */
    static class Inner {

        void fini() {
            System.out.println("内部类" + x);
        }
    }
}

~~~

> 这样就可以访问了

## 四.内部类原则

> 我们来分析下内部类是怎么来的，为什么这样用

*   当描述事物时，事物的内部还有事物，该事物用内部类描述，因为内部事物在使用外部事物

> 内部类就是能直接访问外部类中的具体事物，一般都用于程序设计上

## 五.匿名内部类

> 一般内部类不会被公有实现，我们内部类可以定义在任意位置，也可以这样做

~~~
class Outer {

    int x = 3;

    void fini() {

        class fini {

            void show() {

                System.out.println("内部类");
            }
        }

    }
}
~~~

> 这段程序，内部类会运行吗？答案是不会，因为没有对象，我们就给他new一个对象呗

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        new Outer().fini();
    }
}

class Outer {

    int x = 3;

    void fini() {

        class fini {

            void show() {

                System.out.println("内部类");
            }
        }
        new fini().show();
    }
}

~~~

> 这样就可以访问了，内部类定义在局部

*   1.不可以被成员修饰符修饰
*   2.可以直接访问外部类中的成员，因为还持有类中的引用，但是不可以访问他所在的局部变量，只能访问被final修饰的局部变量

> 而我们的匿名内部类，是什么概念？我们顾名思义，匿名内部类，是没有名字的

*   1.匿名内部类其实就是内部类的简写格式
*   2.定义匿名内部类的前提，就是内部类必须继承一个类或者实现接口

> 正常的逻辑代码

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        new Outer().function();
    }
}

class Outer {

    int x = 3;

    class Inner extends AdsDemo {
        @Override
        void show() {
            System.out.println("method:" + x);
        }
    }

    public void function() {
        new Inner().show();
    }
}

abstract class AdsDemo {

    abstract void show();
}

~~~

> 而我们现在要使用匿名内部类，就简化了代码，具体怎么做？

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        new Outer().function();
    }
}

class Outer {

    int x = 3;

    // class Inner extends AdsDemo {
    // @Override
    // void show() {
    // System.out.println("method:" + x);
    // }
    // }

    public void function() {
        // new Inner().show();
        new AdsDemo() {
            @Override
            void show() {
                System.out.println("x:" + x);
            }
        };

    }
}

abstract class AdsDemo {

    abstract void show();
}

~~~

> 这个就是匿名内部类

*   匿名内部类的格式：new 父类或者接口（）{定义子类的内容}
*   其实匿名内部类就是一个匿名子类对象。而且这个对象有点胖，你也可以把他理解为带内容的对象
*   匿名内部类中定义的方法最好不超过三个

> OK,本篇幅就到这里，我们的JAVA之旅这个课程不知不觉已经讲了这么多了，从当初的不确定，想尝试一下写，现在已经积累到第九篇了，所以总结出来，我们想做的事情还是得去尝试一下，万一实现了呢？

## 有志同道合的人，欢迎加群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。