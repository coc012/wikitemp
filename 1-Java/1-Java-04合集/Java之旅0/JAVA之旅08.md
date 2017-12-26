

# JAVA之旅（八）——多态的体现，前提，好处，应用，转型，instanceof，多态中成员变量的特点，多态的案例

* * *

> 学习是不能停止的

## 一.多态

> 我们今天又要学习一个新的概念了，就是多态，它是面向对象的第三个特征，何谓多态？

*   定义

> 某一类事物的多种存在方式

*   比如

> 动物中的猫狗，人类中的男人，女人
> 
> 我们可以把多态理解为事物存在的多种体现形态
> 
> 当我们new一个猫类的时候，和new 一个动物，其实是一样的，多种形态变现
> 
> 所以我们可以分这几部分分析

*   1.  多态的体现
*   1.  多态的前提
*   3.多态的好处
*   4.多态的应用

> 我们定义一个需求，描述动物，正常的逻辑应该是这样描述的

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 动物： 猫，狗
         */
        Cat c = new Cat();
        c.eat();
        dog d = new dog();
        d.eat();
    }
}

/**
 * 动物
 * 
 * @author LGL
 *
 */
abstract class Animal {

    // 吃什么不确定，抽象
    abstract void eat();
}

/**
 * 猫
 * 
 * @author LGL
 *
 */
class Cat extends Animal {

    @Override
    void eat() {
        System.out.println("猫吃鱼");
    }

}

/**
 * 狗类
 * 
 * @author LGL
 *
 */
class dog extends Animal {

    @Override
    void eat() {
        System.out.println("狗吃骨头");
    }

}

~~~

> 这个体系我们展现出来一个为题，我们为了使用猫吃东西和狗吃东西，得new两个对象，要是多来几只小动物，我不还得new死，所以我们要想一个解决办法，他们有一个共性，就是都是动物，我们可以这样转换

~~~
Animal a = new Cat();
a.eat();
~~~

> 因为也是动物类型，我们输出

![这里写图片描述](http://img.blog.csdn.net/20160528165349402)

> 这就是多态在程序中的表现

*   父类的引用指向了自己的子类对象，这就是多态的代码体现形式，人 = new 男人，换句话说，父类的引用也可以接收子类的对象，所以我们可以这样定义一个方法

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 动物： 猫，狗
         */
        AnimalEat(new Cat());
        AnimalEat(new dog());
    }

    public static void AnimalEat(Animal a) {
        a.eat();
    }
}
~~~

> 这样就方便了，这样也就体现了多态的好处：

*   多态的出现大大的提升了程序的扩展性

> 但是有前提的

*   必须类与类之间有关系，要么继承，要么实现
*   通常，还有一个前提就是存在覆盖

> 不过，有利有弊，还是会存在弊端的

*   提高了扩展性，但是只能使用父类的引用访问父类的成员，这是局限性，但是我们侧重扩展性

> 我们再返回前面说多态的转型，我们看这段代码

~~~
//类型提升
Animal a = new Cat();  
a.eat();
~~~

> 我们也叫作向上转型，
> 
> 如果想要调属性，该如何操作（向下转型）？

*   强制将父类的引用转为子类类型

~~~
        Animal a = new Cat();
        a.eat();

        Cat c = (Cat)a;
        c.sleep();
~~~

![这里写图片描述](http://img.blog.csdn.net/20160528171943356)

> 也就是说，转型是强制将父类的引用，转为子类类型，向下转型。千万不要将父类对象转成子类对象，我们能转换的是父类引用指向子类对象的子类，多态自始至终都是子类对象在做着变化
> 
> 那么你会了强转之后，你就说，我可以这样做

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 动物： 猫，狗
         */

        AnimalEat(new Cat());
        AnimalEat(new dog());

    }

    public static void AnimalEat(Animal a) {
        a.eat();

        Cat c = (Cat) a;
        c.sleep();
    }
}
~~~

> 这样是不是可以？我们看结果

![这里写图片描述](http://img.blog.csdn.net/20160528173936098)

> 这里报错了，提示的是狗类型不行转换成猫类型，的确，不能随便乱转。我们价格判断，怎么判断呢？条件语句该怎么写呢？这里我们又有一个关键字了**instanceof**

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 动物： 猫，狗
         */

        AnimalEat(new Cat());
        AnimalEat(new dog());

    }

    public static void AnimalEat(Animal a) {
        a.eat();

        //如果a的类型是Cat就执行
        if(a instanceof Cat){
            Cat c = (Cat) a;
            c.sleep();
        }

    }
}
~~~

> 这样我们加了判断之后，我们就可以知道

![这里写图片描述](http://img.blog.csdn.net/20160528174201942)

> 既然多态说了这么多，我们来看看多态的应用吧，还是以一个需求开始去分析

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        /**
         * 需求：幼儿园有两个班 大班： 学习，睡觉 小班： 学习，睡觉 可以将两类事物进行抽取
         */
        SmallClass s = new SmallClass();
        s.study();
        s.sleep();

        BigClass b = new BigClass();
        b.study();
    }

}

/**
 * 学生类
 * 
 * @author LGL
 *
 */
abstract class Student {
    // 学习的内容不一样，抽象
    public abstract void study();

    // 睡觉
    public void sleep() {
        System.out.println("躺着睡");
    }
}

/**
 * 大班
 * 
 * @author LGL
 *
 */
class BigClass extends Student {

    @Override
    public void study() {
        System.out.println("学习大班知识");
    }

}

/**
 * 小班
 * 
 * @author LGL
 *
 */
class SmallClass extends Student {

    @Override
    public void study() {
        System.out.println("学习小班知识");
    }

    @Override
    public void sleep() {
        System.out.println("卧着睡");
    }

}
~~~

> 这个例子输出

![这里写图片描述](http://img.blog.csdn.net/20160528175801355)

> 你拿到一想，是不是根据上面的方法直接复用父类对象的引用？这里我们可以拿到一个单独的类去复用封装

~~~
/**
 * 封装工具类
 * 
 * @author LGL
 *
 */
class DoStudent {
    public void dosome(Student s) {
        s.study();
        s.sleep();
    }
}
~~~

> 这样我们使用

~~~
        DoStudent dos = new DoStudent();
        dos.dosome(new BigClass());
        dos.dosome(new SmallClass());
~~~

> 得到的结果

![这里写图片描述](http://img.blog.csdn.net/20160528180609327)

> 我们再来看下多态的代码特点，我们举个例子

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        zi z = new zi();
        z.method1();
        z.method2();
        z.method3();
    }
}

class Fu {
    void method1() {
        System.out.println("fu method1");
    }

    void method2() {
        System.out.println("fu method2");
    }
}

class zi extends Fu {
    void method1() {
        System.out.println("zi method1");
    }

    void method3() {
        System.out.println("zi method3");
    }

}
~~~

> 你能告诉我打印的结果吗？

![这里写图片描述](http://img.blog.csdn.net/20160528181406080)

> 我们现在用多态的思想去做

![这里写图片描述](http://img.blog.csdn.net/20160528181615487)

> 你会知道，3是引用不了的，我现在把报错的的地方注释掉，然后你能告诉我运行的结果吗

![这里写图片描述](http://img.blog.csdn.net/20160528181803092)

> 我们可以总结出特点（在多态中成员函数的特点）

*   在编译时期。参阅引用型变量所属的类是否有调用的方法，如果由，编译通过。如果没有编译失败
*   在运行时期，参阅对象所属的类中是否有调用的方法
*   简单总结就是成员函数在多态调用时，编译看左边，运行看右边

> 我们再在子类和父类中都定义一个int值分别是5和8
> 
> 我们这么输出

~~~
        Fu f = new zi();
        System.out.println(f.num);

        zi z = new zi();
        System.out.println(z.num);
~~~

> 输出多少呢？

![这里写图片描述](http://img.blog.csdn.net/20160528182536071)

> 这里就总结出

*   在多态中，成员变量的特点:无论编译和运行，都参考左边（引用型变量所属）
*   在多态中，静态成员变量的特点：无论编译和运行，都参考左边

> 我们把学到的应用在案例上

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        /**
         * 需求：电脑运行实例，电脑运行基于主板
         */
        MainBoard b = new MainBoard();
        b.run();
    }
}

/**
 * 主板
 * 
 * @author LGL
 *
 */
class MainBoard {

    public void run() {
        System.out.println("主板运行了");
    }

}
~~~

> 我们程序这样写， 无疑看出来很多弊端，我想上网，看电影，他却没有这功能，我们要怎么去做，我们重新设计程序，再增加

~~~
/**
 * 网卡
 * 
 * @author LGL
 *
 */
class NetCard {
    public void open() {
        System.out.println("打开网络");
    }

    public void close() {
        System.out.println("关闭网络");
    }
}
~~~

> 但是这样，还是主板的耦合性是在是太强了，不适合扩展，所以，这个程序一定不是一个好的程序我，我们重新设计，用一个标准的接口

~~~
import javax.print.attribute.standard.MediaName;

//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        /**
         * 需求：电脑运行实例，电脑运行基于主板
         */
        MainBoard m = new MainBoard();
        m.run();
        // 没有设备，有设备的话之类传进去
        m.userPCI(null);

    }
}

/**
 * 扩展接口
 * 
 * @author LGL
 *
 */
interface PCI {
    public void open();

    public void close();
}

/**
 * 主板
 * 
 * @author LGL
 *
 */
class MainBoard {

    public void run() {
        System.out.println("主板运行了");
    }

    public void userPCI(PCI p) {
        if (p != null) {
            p.open();
            p.close();
        } else {
            System.out.println("没有设备");
        }

    }

}
~~~

> 我们现在不管增加听音乐还是上网的功能，只要实现PCI的接口，就可以实现，我们现在增加一个上网功能，该怎么做？

~~~

//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        /**
         * 需求：电脑运行实例，电脑运行基于主板
         */
        MainBoard m = new MainBoard();
        m.run();
        // 没有设备
        m.userPCI(null);

        // 有设备
        m.userPCI(new NetCard());

    }
}

/**
 * 扩展接口
 * 
 * @author LGL
 *
 */
interface PCI {
    public void open();

    public void close();
}

/**
 * 主板
 * 
 * @author LGL
 *
 */
class MainBoard {

    public void run() {
        System.out.println("主板运行了");
    }

    public void userPCI(PCI p) {
        if (p != null) {
            p.open();
            p.close();
        } else {
            System.out.println("没有设备");
        }

    }

}

/**
 * 网卡
 * 
 * @author LGL
 *
 */
class NetCard implements PCI {
    public void open() {
        System.out.println("打开网络");
    }

    public void close() {
        System.out.println("关闭网络");
    }
}
~~~

> 这样我们运行

![这里写图片描述](http://img.blog.csdn.net/20160528192108605)

> 现在的主板是不是扩展性特别强，这就是多态的扩展性
> 
> OK,我们本节的篇幅就先到这里，如果感兴趣的话，可以加群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。