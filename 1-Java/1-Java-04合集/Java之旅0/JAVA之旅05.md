

# JAVA之旅（五）——this，static，关键字，main函数，封装工具类，生成javadoc说明书，静态代码块

* * *

> 周末收获颇多，继续学习

## 一.this关键字

> 用于区分局部变量和成员变量同名的情况
> 
> this的特点 
> this就代表本类对象
> 
> 这在我们的set方法里面是有的

~~~
    public void setName(String name) {
        this.name = name;
    }
~~~

> this代表他所在的函数对属对象的引用
> 
> 现在我们这里有这么一个需求

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 需求：给人定义一个用于比较年龄相同的功能，也就是是否是同龄人
         */
        Person p1 = new Person(20);
        Person p2 = new Person(25);
        boolean b = p1.compare(p2);
        System.out.println(b);

    }

}

class Person {

    private int age;

    // 一初始化就有年龄
    public Person(int age) {
        this.age = age;

    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public boolean compare(Person p) {

        return this.age == p.age;
    }

}

~~~

> 得到的结果肯定是false啦

![这里写图片描述](http://img.blog.csdn.net/20160522100445043)

> 可以知道，当定义类中的功能时，该函数内部要用到该函数的对象时，这时用this来表示，但凡本类功能内部使用到了本类对象，都用this表示
> 
> 我们再来看个小知识点，我们看一个需求

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        // 构造函数间只能用this语句
    }

}

class Person {

    private int age;
    private String name;

    public Person(int age) {
        this.age = age;
    }

    public Person(int age, String name) {
        this(age); // 代表p的age
        this.name = name;
    }

}

~~~

> this()函数的引用，这里我们要注意，this语句只能放在构造函数的第一行对对象进行初始化

## 二.static关键字

> 静态，我们来看一个小例子

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        Person p = new Person();
        p.age = 18;
        p.show();
    }

}

class Person {

    int age;
    String name = "lgl";
    static String country = "cn";

    public void show() {
        System.out
                .println("你的名字:" + name + "今年:" + age + "岁" + "国籍：" + country);
    }

}

~~~

> static是一个修饰符，是一个修饰成员变量，成员函数的关键字，被静态修饰之后，他就不在内存中了，被单独提取出来，每个人都能访问，静态修饰内容被对象所共享
> 
> 当成员被静态修饰后，就多了一种调用方式，除了可以被对象调用外，还可以直接被类名调用：类名.静态成员

~~~
System.err.println(Person.country);
~~~

> 一样可以

![这里写图片描述](http://img.blog.csdn.net/20160522102822159)

> 这样，我们可以总结一下static的特点

*   1.随着类的加载而加载

    > 所谓随着类的加载而加载，Person这个类你一使用的时候，静态就已经存在了

*   2.优先于对象存在

    > 明确一点，静态时先存在的，参照1

*   3.被所有对象所共享

*   4.可以直接被类名所调用

> 实例变量和类变量的区别：

*   存放位置 

    *   类变量随着类的加载而存在于方法区中，随着类的消失而消失
    *   实例变量随着对象的建立而存在于堆内存中
*   生命周期 

    *   类变量生命周期最长，随着类的消失而消失
    *   实例变量的生命周期随着对象的消失而消失

> 静态变量的使用注意事项

*   静态方法只能访问静态成员 

    *   非静态方法方法即可以访问静态也可以访问非静态
*   静态方法中不可以定义this，super关键字，因为静态优先于对象存在，所在静态方法中不可以出现他们
*   主函数是静态

> 静态有利有弊

*   利：对对象的共享数据进行单独控件的存储，节省空间，没必要每个对象都存储一遍，也可以直接被类名调用
*   弊：生命周期过长，访问出现局限性（静态虽好，只能访问静态）

> 这里要说一个小知识点：

### main函数

> 主函数大家应该都很熟悉了
> 
> 主函数是一个特殊的函数，可以被JVM调用，作为程序的入口

*   public:代表的该访问权限是最大的
*   static:代表主函数随着类的加载而存在
*   void：主函数没有具体的返回值
*   main:不是关键字，但是是一个特殊的单词，可以被jvm识别
*   函数的参数：参数类型是一个数组，该数组中的元素是字符串，字符串类型的数组

> 主函数的格式是固定的，jvm识别
> 
> OK，了解了主函数，我们回到静态，什么时候使用static?
> 
> 要从两方面下手，因为静态修饰的内容有成员变量和函数

*   什时候定义静态变量

    > 当对象中出现共享数据时，该数据被静态修饰，对象中的特有数据，定义成非静态，存在于堆内存中

*   什么时候定义静态函数

    > 当功能内部没有访问到非静态数据（对象的特有数据），那么该功能可以定义为静态

## 封装工具类

> 我们可以看我们是怎么求最大值的

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        int[] arr = { 0, 9, 0, 6, 2, 8 };
        int max = getMa(arr);
        System.out.println("最大值：" + max);

    }

    /**
     * 求一个数组的最大值
     * @param arr
     * @return
     */
    public static int getMa(int[] arr) {
        int max = 0;
        for (int i = 0; i < arr.length; i++) {
            if (arr[i] > arr[max]) {
                max = i;
            }
        }
        return arr[max];
    }
}

~~~

> 我们把这个方法提取出来，确实很方便，但是，要是其他的类也有数组需要求最大值呢？这个时候我们就可以封装成一个工具类了

*   每一个应用程序都有共性的部分，可以将这些功能抽取，独立封装，以便使用，所以我们可以这样写：

~~~
/**
 * 数组工具
 * 
 * @author LGL
 *
 */
public class ArrayTools {

    /**
     * 求一个数组的最大值
     * 
     * @param arr
     * @return
     */
    public static int getMax(int[] arr) {
        int max = 0;
        for (int i = 0; i < arr.length; i++) {
            if (arr[i] > arr[max]) {
                max = i;
            }
        }
        return arr[max];
    }

    /**
     * 求一个数组的最小值
     * 
     * @param arr
     * @return
     */
    public static int getMin(int[] arr) {
        int min = 0;
        for (int i = 0; i < arr.length; i++) {
            if (arr[i] < arr[min]) {
                min = i;
            }
        }
        return arr[min];
    }

}

~~~

> 把获取最大值和最小值的方法都封装起来，用static去修饰，这样

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        int[] arr = { 0, 9, 1, 6, 2, 8 };

        int max = ArrayTools.getMax(arr);
        int min = ArrayTools.getMin(arr);

        System.out.println("最大值：" + max);
        System.out.println("最小值：" + min);

    }

}

~~~

> 我们就可以直接去获取

![这里写图片描述](http://img.blog.csdn.net/20160522115756008)

> 虽然我们可以通过ArrayTools的对象使用这些方法，对数组进行操作，但是，会存在一些问题‘’

*   对象是用来封装数据的，可是ArrayTools 对象病没有封装特有的数据
*   操作一个数组的每个方法都没有用到ArrayTools 对象中的特有数据

> 这个时候我们可以考虑，让程序更加的严谨，不需要对象，可以将ArrayTools 的方法都定义成静态的，直接通过类名调用即可

### 生成javadoc说明书

> 当我们写好一个工具类的时候，是可用广为流传的，但是，人家也不知道呢写了啥呀，所以，我们写个说明书是必须的，首先回到我们的工具类

~~~
/**
 * 数组工具
 * 
 * @author LGL
 *
 */
public class ArrayTools {

    /**
     * 求一个数组的最大值
     * 
     * @param arr
     *            接收到数组
     * @return
     */
    public static int getMax(int[] arr) {
        int max = 0;
        for (int i = 0; i < arr.length; i++) {
            /**
             * 两数比较
             */
            if (arr[i] > arr[max]) {
                max = i;
            }
        }
        /**
         * 返回最大值
         */
        return arr[max];
    }

    /**
     * 求一个数组的最小值
     * 
     * @param arr
     *            接收到数组
     * @return
     */
    public static int getMin(int[] arr) {
        int min = 0;
        for (int i = 0; i < arr.length; i++) {
            /**
             * 两数比较
             */
            if (arr[i] < arr[min]) {
                min = i;
            }
        }
        /**
         * 返回最小值
         */
        return arr[min];
    }

}

~~~

> 你可以看到，我们添加了很多的注释，现在我们可以去生成了，我们在jdk安装目录的bin文件下看到一个文件叫做javadoc,我们就需要他，我们可以在java类目录下

~~~
// myhelp:文件夹    -author：作者，可以不写
javadoc -d myhelp -author ArrayTools.java
~~~

![这里写图片描述](http://img.blog.csdn.net/20160522122555724)

> 生成之后我们打开index.html

![这里写图片描述](http://img.blog.csdn.net/20160522122616099)

### 静态代码块

> 我们讲一个小知识点静态代码快，我们先看一下格式

~~~
class StaticDemo {
    static {
        //静态代码快
    }
}
~~~

> 静态代码块的特点：随着类的加载而执行，只执行一次

~~~
//公共的   类   类名
public class HelloJJAVA {
    static {
        // 静态代码快
        System.err.println("b");
    }

    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        new StaticDemo();
        System.out.println("over");
    }

    static {
        // 静态代码快
        System.err.println("c");
    }

}

class StaticDemo {
    static {
        // 静态代码快
        System.err.println("a");
    }
}
~~~

> 我们可以猜猜看，执行的顺序是什么？静态方法是从上往下执行优先于mian方法的，所以是b,然后走main方法输出a,over

## 好的，本篇幅就先到这里，有兴趣的可以加群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。