

# JAVA之旅（七）——final关键字 , 抽象类abstract，模板方法模式，接口interface，implements，特点，扩展

* * *

> OK，我们继续学习JAVA，美滋滋的

## 一.final

> 我们来聊聊final这个关键字

*   final可以修饰类，方法和变量
*   final修饰的类不可以被继承
*   final修饰的方法不可以被覆盖
*   final修饰的变量是一个常量，只能被赋值一次
*   内部类只能访问被final修饰的局部变量

> final，故名思意，就是最终的意思，由以上的五种特性，不过final的出现，也是有弊端的，他破坏了封装性，对继承有了挑战，为了避免被继承，被子类复写功能，还有，当你描述事物时，一些数据的出现值是固定的，那么，这时为了增强阅读行，都给这个值起个名字方便阅读，而这值不需要改变，就会用到final去修饰，作为常量，常量的书写规范是所有字母都大写，如果由多个单词组成，单词间通过下划线链接！而且内部类定义在类中的局部位置只能访问该局部被final修饰的局部变量
> 
> final大家只要记住他的特性就行

## 二.抽象类

> 这个抽象类应该是属于继承的下半部分的，我们看一个标注的代码

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

    }
}

/**
 * 学生
 * 
 * @author LGL
 *
 */
class student extends Base {

    void speak() {
        System.out.println("学习");
    }
}

/**
 * 工人
 * 
 * @author LGL
 *
 */
class worker extends Base {

    void speak() {
        System.out.println("工作");
    }
}

/**
 * 基类
 * 
 * @author LGL
 *
 */
class Base {

    void speak() {
        System.out.println("Hello");
    }
}

~~~

> 这里，学生和工人都是要说话，所以我们可以抽取，但是这里，他们说话的内容却是不同的，当多个类出现相同功能，但是功能主体不同，这个时候就可以进行向上抽取，这时只抽取功能主体；这个时候就得用到我们的抽象类了abstract；所以我们的基类是这样的

~~~
/**
 * 基类
 * 
 * @author LGL
 *
 */
abstract class Base {

    abstract void speak();
}

~~~

> 什么叫抽象？

*   看不懂

> 特点

*   1.抽象方法一定定义在抽象类中
*   2.抽象方法和抽象类都必须被abstract关键字修饰
*   3.抽象类不可以用new创建对象，因为调用抽象方法没意义
*   4.抽象类中的方法要被使用必须由子类复写其所有的抽象方法后建立子类对象调用 

    *   如果子类只覆盖了部分抽象方法，那么该子类还是一个抽象类

> 其实抽象类和一般的类没有什么太大的不同，只是要注意该怎么描述事物就怎么描述事物，只不过该事物中出现了一些看不懂的东西，这些不确定的部分也是该事物的功能，需要明确出来，但是无法定义主体，通过抽象方法来表示！

*   抽象类比其他类多了抽象函数，就是在类中可以定义抽象方法
*   抽象类不可以实例化

> 特殊：抽象类中可以不顶用抽象方法，看上去很没有意义，但是这样做可以做到不让该类建立对象，不是很多见
> 
> 抽象方法文字部分说了这， 多，我们做一个小练习

*   题目：假如我们在开发一个系统时需要对员工进行建模，员工包含三个属性 姓名，工号和工资，经理也是员工，除了含有员工的属性外，另外还 有一个奖金属性，请使用继承的思路设计出员工类和经理类，要求类 种提供必要的方法进行属性访问！

> 我们实现之前可以简单的分析一下，我们的员工类应该有三个属性，name,id,pay，而经历类，理论上是继承了员工类并且有资金的奖金属性，行，这样的话我们可以用代码去测试一下

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        /**
         * 假如我们在开发一个系统时需要对员工进行建模，员工包含三个属性 姓名，工号和工资，经理也是员工，除了含有员工的属性外，另外还
         * 有一个奖金属性，请使用继承的思路设计出员工类和经理类，要求类 种提供必要的方法进行属性访问！
         */
    }
}

/**
 * 员工类
 * 
 * @author LGL
 *
 */
abstract class Employee {
    // 姓名
    private String name;
    // 工号
    private String id;
    // 工资
    private double pay;

    // 这个员工一生成，这三个属性必须有
    public Employee(String name, String id, double pay) {
        this.name = name;
        this.id = id;
        this.pay = pay;
    }

    // 员工做什么是不确定的
    public abstract void work();

}

/**
 * 经理类
 * 
 * @author LGL
 *
 */
class Manager extends Employee {

    // 奖金
    private int bonus;

    public Manager(String name, String id, double pay, int bonus) {

        super(name, id, pay);
        this.bonus = bonus;
    }

    // 复写
    @Override
    public void work() {
        System.out.println("管理");
    }

}

~~~

> 这代码很清晰的就表现了抽象的关系

## 三.模板方法模式

> 这是一个小案例，获取一段程序的运行时间，这个需求应该很简单吧，计时，

*   原理：获取程序开始和结束的时间并相减

> 获取时间的方法：System.currentTimeMillis()
> 
> 代码逻辑是这样写的

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        GetTime gt = new GetTime();
        gt.getTime();
    }
}

/**
 * 时间类
 * 
 * @author LGL
 *
 */
class GetTime {
    // 获取时间
    public void getTime() {
        long start = System.currentTimeMillis();
        // 耗时
        for (int i = 0; i < 10000; i++) {
            System.out.print("" + i);
        }
        long end = System.currentTimeMillis();
        System.out.println("耗时：" + (end - start));
    }
}
~~~

> 我们就可以得到你代码运行的毫秒数了

![这里写图片描述](http://img.blog.csdn.net/20160527201030818)

> 但是我们发现，其实这个耗时的部分是不确定的，对吧，那既然这样，我们复写的话，就有点多余了，我们可以使用使用模板方法模式，也就是抽成一个方法公用

~~~
import org.ietf.jgss.Oid;

//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        GetTime gt = new GetTime();
        gt.getTime();
    }
}

/**
 * 时间类
 * 
 * @author LGL
 *
 */
class GetTime {
    // 获取时间
    public void getTime() {
        long start = System.currentTimeMillis();

        runCode();

        long end = System.currentTimeMillis();
        System.out.println("耗时：" + (end - start));
    }

    /**
     * 耗时方法
     */
    private void runCode() {
        // 耗时
        for (int i = 0; i < 10000; i++) {
            System.out.print("" + i);
        }
    }
}
~~~

> 这个时候，要是其他类想使用的话，就只要复写一个方法就行了，Test想使用的话，就可以直接继承复写了

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        // GetTime gt = new GetTime();
        Test t = new Test();
        t.getTime();
    }
}

/**
 * 时间类
 * 
 * @author LGL
 *
 */
class GetTime {
    // 获取时间
    public void getTime() {
        long start = System.currentTimeMillis();

        runCode();

        long end = System.currentTimeMillis();
        System.out.println("耗时：" + (end - start));
    }

    /**
     * 耗时方法
     */
    public void runCode() {
        // 耗时
        for (int i = 0; i < 10000; i++) {
            System.out.print("" + i);
        }
    }
}

class Test extends GetTime {
    @Override
    public void runCode() {
        // 耗时
        for (int i = 0; i < 50000; i++) {
            System.out.print("" + i);
        }
    }
}
~~~

> 这样，输出的内容就是我们随便改的了

![这里写图片描述](http://img.blog.csdn.net/20160527202228451)

> 我们还可以用抽象去做，你就一定要去做这件事儿

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        // GetTime gt = new GetTime();
        Test t = new Test();
        t.getTime();
    }
}

/**
 * 时间类
 * 
 * @author LGL
 *
 */
abstract class GetTime {
    // 获取时间
    public void getTime() {
        long start = System.currentTimeMillis();

        runCode();

        long end = System.currentTimeMillis();
        System.out.println("耗时：" + (end - start));
    }

    /**
     * 耗时方法
     */
    public abstract void runCode();
}

class Test extends GetTime {
    @Override
    public void runCode() {
        // 耗时
        for (int i = 0; i < 50000; i++) {
            System.out.print("" + i);
        }
    }
}
~~~

> 我把这个延时的操作留给子类，你爱咋地就咋滴，这样是不是更方便？当然，我们可以给getTime加一个final修饰，这样就让程序更加的健壮；
> 
> 当代码完成优化之后，就可以解决这类问题了，我们把这种方式叫做：模板方法设计模式

*   什么是模板方法？ 

> 在定义功能的时候，功能的一部分是不确定的，而确定的部分在使用不确定的部分的时候，那么这时就将不确定的 部分暴露出去让子类去完成，这就是吗，吗，模板方法模式了

## 四.接口

> 接口的关键字是interface，接口中的成员修饰符是固定的

*   成员常量：public static final
*   成员函数：public abstract

> 接口的出现将“多继承”通过另一种形势体现，即“多实现”
> 
> 上面的都是概念。我还是通俗易懂的来说吧，接口，初期理解，你可以认为是一个特殊的抽象类，当抽象类中的方法都是抽象的，那么该类可以通过接口的形式表示，interface，class用于定义类，定义接口
> 
> 接口定义时，格式特点在于 
> - 1.接口中常见的定义一个是常量，一个是抽象方法 
> - 2.接口中的成员都有固定修饰符 
> - 常量：public static final 
> - 方法：public abstract
> 
> 具体格式：

~~~
/**
 * 人的接口
 * 
 * @author LGL
 *
 */
interface Person {

    public static final int AGE = 20;

    /**
     * 说话
     */
    public abstract void speak();
}

~~~

> 这里注意，接口中的成员都是public，我们要想使用这个接口，需要用到另一个关键字了**implements**
> 
> 为什么我们类不能继承类呢？因为接口，是不可以创建对象的，因为有抽象方法，需要被子类去实现，子类对接口中的抽象方法全都覆盖过后子类才可以实例化，否则子类是一个抽象类

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        Student s = new Student();
        System.out.println(s.AGE);
        System.out.println(Student.AGE);
        System.out.println(Person.AGE);
    }
}

class Student implements Person {

    @Override
    public void speak() {

    }

}

/**
 * 人的接口
 * 
 * @author LGL
 *
 */
interface Person {

    public static final int AGE = 20;

    /**
     * 说话
     */
    public abstract void speak();
}

~~~

> 这样执行的后，我们就等得到数据了

![这里写图片描述](http://img.blog.csdn.net/20160527210631848)

> 不过接口不仅仅是这么简单，接口是可以被类多实现了，什么叫做多实现？就是一个类单继承，但是可以实现多个接口，对继承不支持的形式，java支持多实现

~~~
class Student implements Person, Person2 {

    @Override
    public void speak() {

    }

    @Override
    public void work() {

    }

}

/**
 * 人的接口
 * 
 * @author LGL
 *
 */
interface Person {

    public static final int AGE = 20;

    /**
     * 说话
     */
    public abstract void speak();
}

interface Person2 {
    public abstract void work();
}
~~~

> 可以看到实现了；两个接口，但是他为什么没有继承的弊端呢？因为他没有方法主体，子类爱怎么着就怎么着了
> 
> 接口间的关系时继承

### 接口的特点

> 这个接口的重点比较实在，所以单独提取出来讲一下，首先我们来连接一下接口的特点

*   接口谁对外暴露的规则
*   接口是程序的功能扩展
*   接口可以用力多实现
*   类和接口之间是实现关系，而且类可以继承一个类的同时实现多个接口
*   接口与接口之间可以有继承关系

> 就好比笔记本，我们扩展一样，这就是接口的概念，说不如做，我们写个例子来展现：

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

    }
}

/**
 * 学生类
 * 
 * @author LGL
 *
 */
abstract class Student {
    // 学习不确定
    abstract void study();

    // 都要睡觉
    void Sleep() {
        System.out.println("sleep");
    }
    //都抽烟
    abstract void smoke();
}

/**
 * 我
 * @author LGL
 *
 */
class lgl extends Student{

    @Override
    void study() {
        System.out.println("lgl学习");
    }

    @Override
    void smoke() {
        System.out.println("lgl 抽烟");
    }

}

~~~

> 我这里定义了一个人，他有抽烟，睡觉，学习的方法，学习的方法不确定，所以要抽象，睡觉都要，抽烟，有的人抽，有的不抽，品牌也不一样，所以也得弄成这样，现在我去继承这个人，机会有学习和睡觉的方法，但是强制性的抽烟了，这就是这个例子的问题，那我们要怎么改善？还得使用接口了，我们可以把抽烟的方法抽写成接口，有需要接实现这个接口

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        lgl l = new lgl();
        l.smoke();
        l.study();
    }
}

/**
 * 学生类
 * 
 * @author LGL
 *
 */
abstract class Student {
    // 学习不确定
    abstract void study();

    // 都要睡觉
    void Sleep() {
        System.out.println("sleep");

    }
}

/**
 * 我
 * 
 * @author LGL
 *
 */
class lgl extends Student implements smoke{

    @Override
    void study() {
        System.out.println("lgl学习");
    }

    @Override
    public void smoke() {
        System.out.println("lgl 抽烟");
    }

}
/**
 * 抽烟的接口
 * @author LGL
 *
 */
interface smoke {
    void smoke();
}

~~~

> 这样就可以输出

![这里写图片描述](http://img.blog.csdn.net/20160527215130659)

> 好的，我们本篇幅就先到这里，如果有不明白的地方，还是要多温习几遍哦，想学好android，java功底是必不可少的呢！

### 我创建一个很有意思的群：555974449，要是感兴趣的可以进来一起交流一下哦！

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。