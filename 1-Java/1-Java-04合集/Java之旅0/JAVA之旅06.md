

# JAVA之旅（六）——单例设计模式，继承extends，聚集关系，子父类变量关系，super，覆盖

* * *

> java也越来越深入了，大家加油吧！咱们一步步来

## 一.单例设计模式

*   什么是设计模式？

    > JAVA当中有23种设计模式，解决某一问题最有效的方法

*   单例设计模式

    > 解决一个类在内存中只存在一个对象

> 想要保证对象唯一该怎么做》

*   1.为了避免其他程序过多建立该类对象，先禁止其他程序建立该类对象
*   2.还为了让其他程序访问到该类对象，只好在本类中自定义一个对象
*   3.为了方便其他程序对自定义对象的访问，可以对外提供一些访问方式

> 这三步怎么怎么实现呢？

*   1.将构造函数私有化
*   2.在类中创建一个本类对象
*   3.提供一个方法可以获取到该对象

> 然后我们的代码就出来了

~~~
/**
 * 单例模式
 * 
 * @author LGL
 *
 */
public class Single {

    // 将构造函数私有化
    private Single() {

    }

    // 在类中创建一个本类对象
    private static Single s = new Single();

    // 提供一个方法可以获取到该对象
    public static Single getInstance() {

        return s;
    }
}

~~~

> 当然，我们还有一种写法——懒汉式

~~~
/**
 * 单例模式
 * 懒汉式
 * @author LGL
 *
 */
public class Single {

    private static Single s = null;

    public Single() {
        // TODO Auto-generated constructor stub
    }

    public static Single getInstance() {
        if (s == null)
            s = new Single();

        return s;

    }
}

~~~

> 也有这么一种写法

~~~
/**
 * 单例模式 
 * 饿汉式
 * 
 * @author LGL
 *
 */
public class Single {

    private static Single s = new Single();

    public Single() {
        // TODO Auto-generated constructor stub
    }

    public static Single getInstance() {
        return s;

    }
}

~~~

> 单例模式的写法多种多样，只要掌握这个思想就可以了
> 
> 饿汉式的特点：single类一进内存就已经创建好了对象 
> 懒汉式的特点：对象是方法被调用时才初始化，也叫作对象的延迟加载
> 
> Single类进内存还没有存在，只有调用了getInstance()方法时才建立对象，这就叫做延迟加载
> 
> 我们一般用饿汉式，因为他安全，简单

## 二.继承

> 面向对象的思想中的一个思想，继承，我们可以这样去理解，我们可以定义一个学生类，他有名字和年龄，还有一个学习的方法

~~~
/**
 * 学生
 * @author LGL
 *
 */
class Student {

    String name;

    int age;

    void study() {
        System.out.println("学习");
    }
}
~~~

> 然后我们可以再定义一个工人类

~~~

/**
 * 工人
 * @author LGL
 *
 */
class Worker {

    String name;

    int age;

    void work() {
        System.out.println("工作");
    }
}
~~~

> 他同样有名字和年龄还有一个工作的方法
> 
> 这个时候我们可以发现，他们都存在相同的关系，他们都是人，人都具备年龄和姓名，所以我们可以将学生和工人的共性提取出来描述，只要让学生和工人与单独描述的类有关系就行，也就是继承

~~~
/**
 * 人
 * 
 * @author LGL
 *
 */
class Person {
    String name;

    int age;
}
~~~

> 这个Person就叫做父类，超类，基类
> 
> 然后我们的工人和学生就可以这样去写

~~~

/**
 * 学生
 * 
 * @author LGL
 *
 */
class Student extends Person {

    void study() {
        System.out.println("学习");
    }
}

/**
 * 工人
 * 
 * @author LGL
 *
 */
class Worker extends Person {

    void work() {
        System.out.println("工作");
    }
}
~~~

> 他们是Person的子类，继承是源于生活，所以工人不能继承学生，没有伦理性，千万不要为了获取其他类的功能，简化代码而继承，必须是类与类之间有所属关系才可以继承，is a 的关系 
> 继承的优点

*   1.提高代码的复用性
*   2.让类与类之间产生关系，有了这层关系才有了多态的特性

> JAVA语言中，java只支持单继承，不支持多继承

*   为什么不能多继承？ 

    > 因为多继承容易带来安全隐患，当多个父类中定义了相同的功能，当功能内容不同时，子类对象不确定运行哪一个，但是JAVA保留了这种机制，并用另一种体现形式来表示，多实现。

JAVA支持多层继承，也就是一个继承体系

如何使用一个继承体系的功能？

*   想要使用体系，先查阅体系中父类的描述，因为父类中定义的是该体系中的共性内容，通过了解共性功能就可以知道该体系的基本功能，那么，这么体系已经可以基本使用了。

*   那么集体调用时，要创建最子类的对象，为什么？一是因为有可能父类不能创建对象，二是创建子类对象可能使用更多的功能，包括基本的也包括特有的

> 简单的说一句：查阅父类功能，创建子类对象使用功能

## 三.聚集关系

> 这个概念可能很多人不是很熟悉，确实我们实际开发当中很少提到，我们对象和对象之间，不光有继承关系，还有组合关系，

*   聚合

    > 球员是球队中的一个，球队中有球员，这就是聚合关系

*   组合

    > 手是人身体的一部分，腿是身体的一部分，这就是组合关系，你说这和聚合没什么不同，那你就错了，要知道，球队少了个球员没事，人少胳膊少腿就有事哦，所以组合关系更加紧密一点

> 都是吧对象撮合在一起，像我们继承是秉承一个is a的关系，而聚集是has a的关系，谁里面有谁
> 
> 这个关系我们就不多做赘述了
> 
> 我们关系讲完了，现在应该可以用代码去表示表示了，这里我就用当继承关系时，子类和父类之间的变量关系来举例

## 四.子父类变量关系

> 我们来探索一下子父类出现后，类成员的特点

*   1.  变量
*   2.函数
*   3.构造函数

> 我们一步步来分析

### 1.变量

> 我们撸一串代码

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 子父类出现后，类成员的特点
         */

        Zi z = new Zi();
        System.out.println("num1 = " + z.num1 + "\n" + "num2 = " + z.num2);
    }
}

/**
 * 父类
 * 
 * @author LGL
 *
 */
class Fu {

    int num1 = 4;
}

/**
 * 子类
 * 
 * @author LGL
 *
 */
class Zi extends Fu {

    int num2 = 5;

}
~~~

> 这样，是不是很容易知道输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160523211826081)

> 但是我们那会这么简单，我们还有特殊情况

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 子父类出现后，类成员的特点
         */

        Zi z = new Zi();
        System.out.println("num1 = " + z.num + "\n" + "num2 = " + z.num);
    }
}

/**
 * 父类
 * 
 * @author LGL
 *
 */
class Fu {

    int num = 4;
}

/**
 * 子类
 * 
 * @author LGL
 *
 */
class Zi extends Fu {

    int num = 5;

}
~~~

> 当子类和父类都有相同的变量的时候，现在该打印什么呢？

![这里写图片描述](http://img.blog.csdn.net/20160523212055161)

> 没错，打印了子类的值，为什么？因为输出的num前面其实默认带了一个this,如果我们要打印父类的值，需要加super，这是一个新的关键字了

~~~
void show(){
        System.out.println(super.num);
    }
~~~

> 结果显而易见了

![这里写图片描述](http://img.blog.csdn.net/20160523212344978)

> 所以变量的特点：如果子类中出现非私有的同名成员变量时，子类要访问本类中的成员变量，用this,子类访问父类中的同名变量，用super,this和super的使用几乎一致，this代表的是本类对象的引用，super代表父类对象的引用
> 
> 不过我们也不会这么蛋疼在父类创建又在子类创建，我们来看第二个

### 2.函数

> 也就是方法，我们正常的应该是这样写的

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 子父类出现后，类成员的特点
         */

        Zi z = new Zi();
        z.show1();
        z.show2();
    }
}

/**
 * 父类
 * 
 * @author LGL
 *
 */
class Fu {
    void show1() {
        System.out.println("fu");
    }

}

/**
 * 子类
 * 
 * @author LGL
 *
 */
class Zi extends Fu {

    void show2() {
        System.out.println("zi");
    }
}
~~~

> 这是很正常的实现

![这里写图片描述](http://img.blog.csdn.net/20160523214703453)

> 我们当然要说一种特殊情况，子父类方法同名我们该怎么办？我们运行一下

![这里写图片描述](http://img.blog.csdn.net/20160523214831735)

> 这是子运行了，这种情况叫做覆盖（重写），这个特性的特点

*   当子类出现和父类一模一样的函数时，当子类对象被调用了该函数，会运行子类函数的内容，如同父类的函数被覆盖一样，这种情况就是覆盖了

> 当子类继承了父类，沿袭了父类的功能到子类中，到子类中，但是子类虽具备该功能，但是功能的内容却和父类不一致，这时没有必要定义新功能，而是使用覆盖特性，保留父类的功能定义并重写功能内容

*   覆盖

    > 1.子类覆盖父类，必须保证子类权限大于等于父类权限才可以覆盖，否则编译失败 
    > 2.静态只能覆盖静态（一般没人用）

*   记住

    > 重载：只看同名函数的参数列表 
    > 重写：字符类方法要一模一样 ，包括返回值类型

### 3.构造函数

> 最后一个特点了

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 子父类出现后，类成员的特点
         */
        Zi z = new Zi();

    }
}

/**
 * 父类
 * 
 * @author LGL
 *
 */
class Fu {
    public Fu() {
        System.out.println("fu");
    }
}

/**
 * 子类
 * 
 * @author LGL
 *
 */
class Zi extends Fu {
    public Zi() {
        System.out.println("zi");
    }
}
~~~

> 我们现在运行一下

![这里写图片描述](http://img.blog.csdn.net/20160523220715613)

> 你会疑问，我明明实例化的是子类，怎么父类也实例化了，因为子类构造函数的第一行默认有一个方法

~~~
super()；
~~~

> 在对子类对象进行初始化时，父类的构造函数也会执行，那就是因为子类的构造函数默认第一行有一条隐式的语句super()；他会访问父类中空参数的构造函数，而且子类中所有的构造函数默认第一行都是super()

*   为什么子类一定要访问父类中的构造函数？

    > 因为父类中的数据子类可以直接获取，所有子类建立对象的时候，需要先查看父类如何对这些数据进行初始化的，所有子类对这些对象初始化时，要先访问父类中的构造函数，如果要访问父类中指定的构造函数，可以手动定义super(xx,xx)的方式。

*   注意

    > super语句一定定义在子类构造函数的第一行

*   结论：

    > 子类的所有的构造函数，默认都会访问父类中空参数的构造函数，因为子类每一个构造函数内的第一行都有一句隐式的super，当父类中没有空参数的构造函数时，子类必须手动通过super或者this语句的形式指定要访问父类中的构造函数

> 当然，子类的构造函数也可以指定this来访问本类中的构造函数，子类中至少会有一个构造函数会访问父类中的构造函数，这个结论就是子类的实例化过程，逐级向上查找
> 
> 父类也向上查找，他找的是JAVA中所有类的老爹——Object
> 
> 行，我们本篇幅就闲到这里吧，概念多了，反而难以消化，我们要一点点的来

## 如果有兴趣可以加群：555974449，这可能是迄今为止最和谐的IT群了

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。