

# JAVA之旅（四）——面向对象思想，成员/局部变量，匿名对象，封装 , private，构造方法，构造代码块

* * *

> 加油吧，节奏得快点了

### 1.概述

> 上篇幅也是讲了这点，这篇幅就着重的讲一下思想和案例
> 
> 就拿买电脑来说吧，首先，你不懂电脑，你去电脑城买电脑，和大象装冰箱里一样，是什么步骤？咨询 砍价 ，谈妥了就那电脑走人，对吧，这就是面向过程的思想，而面向对象是：你有一个哥们，他懂电脑，什么都会，你只要带他去，就行，你这个哥们就是对象，在JAVA中，我们就是操作一个对象去完成各种各样的操作的，这就是面向对象的思想

### 2.成员变量

> 面向对象有三大特征

*   封装
*   继承

> 那具体是什么意思呢？ 
> 我们开发的话，就是就是在找对象，没有对象的话，就new 一个对象，对象和类，对象和对象都是有关系的，我们需要去维护
> 
> 类就是生活中对事物的描述 
> 对象就是这类事物实实在在存在的个体
> 
> 需求：描述汽车（颜色，轮胎个数），描述事物就是在描述事物的属性和行为 
> 我们可以直接定义一个class

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        Car car = new Car();
        // 输出颜色
        System.out.println(car.color + "的小汽车");
        // 输出轮胎个数
        System.out.println(car.num + "个轮子");
        // 输出行为
        car.run();
    }

}

/**
 * 汽车类
 * 
 * @author LGL
 *
 */
class Car {

    // 颜色
    String color = "红色";

    // 轮胎个数
    int num = 4;

    // 行为
    void run() {
        System.out.println("我是" + color + "的小汽车，我有" + num + "个轮子");
    }

}
~~~

> 这样大家看的懂吗，我们猴子姐new一个Car就可以

![这里写图片描述](http://img.blog.csdn.net/20160515113618819)

> 其实定义类，就是描述事物。就是在定义属性和行为，属性和行为共同成为类中的成员（**成员变量**）

### 3.局部变量

> 其实局部变量我们一直在写，他和成员变量的区别在于作用的范围不一样
> 
> 我们以上述的例子

*   成员变量：作用在全局中
*   局部变量：作用在Car类里

> 在内存中的存储和位置

*   成员变量在堆内存中，因为对象的存在才在内存中存在
*   局部变量在栈内存中

### 4.匿名对象

> 这是一个小知识点，匿名换句话其实就是没有名字的意思

*   匿名对象是对象的简化版
*   匿名对象两种使用情况 

    *   当对对象方法仅进行一次调用的时候
    *   匿名对象可以作为实际参数进行传递

> 我们用简单的例子

~~~
//正常的写法
Car c = new Car();
c.num = 5;

//匿名对象
new Car().num = 5;
~~~

> 可以看到我不起名字直接去更改num的值了，这就是匿名对象
> 
> 第二种使用方式实际参数去传递，其实在上面我用到了

~~~
// 输出颜色
System.out.println(new Car().color + "的小汽车");
~~~

> 这样就OK了

### 5.封装

> OK，终于说道我们的核心思想了Encapsulation
> 
> 封装的含义：是指隐藏的对象的属性和实现细节，仅对外提供访问方式
> 
> 好处

*   将变化隔开
*   便于使用
*   提高复用性
*   提高安全性

> 封装原则
> 
> 将不需要对外提供的内容都隐藏起来； 
> 把属性都隐藏，提供对外访问方式
> 
> 我们写例子

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        showString("我是封装");
    }

    /**
     * 封装
     * 
     * @param str
     */
    public static void showString(String str) {
        System.out.println(str);
    }
}
~~~

> 这个就是最简单的封装了，你给我个字符串我就打印，过程你不必知道，函数本身就是一个最小的封装体

### 6.private

> 私有的，怎么使用？

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        Person p = new Person();
        p.age = 20;
        p.speak();
    }
}

class Person {
    int age;

    // 说话方法
    void speak() {
        System.out.println("我今年" + age + "岁");
    }
}
~~~

> 我们现在的代码是这样写的，输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160515134850023)

> 这里我们直接访问了arg，这里就存在了一个安全隐患，这里也就是用到private修饰符去修饰arg了

![这里写图片描述](http://img.blog.csdn.net/20160515135149280)

> 在这里，就看到一个错误提示了，因为我们用private修饰了之后，你就不能拿到了
> 
> private:私有，权限修饰符：用于修饰类中的成员（成员变量，成员函数） 
> 注意的是，私有只在本类中有效
> 
> 那我们怎么去访问呢？你既然私有了，就需要对外提供一个方法

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        Person p = new Person();
        // p.age = 20;
        p.setAges(20);
        p.speak();
    }
}

class Person {

    // 私有
    private int age;

    /**
     * 对外提供方法
     * 
     * @param a
     */
    public void setAges(int a) {
        age = a;
    }

    // 说话方法
    void speak() {
        System.out.println("我今年" + age + "岁");
    }
}
~~~

> 我们这样做，也是可以的

![这里写图片描述](http://img.blog.csdn.net/20160515135736190)

> 但是我们一般也不会这样做，我们有规范

~~~
public int getAge() {
        return age;
    }

public void setAge(int age) {
        this.age = age;
    }

~~~

> 所以我们的完整代码应该是这样写

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        Person p = new Person();
        // p.age = 20;
        // p.setAges(20);
        p.setAge(20);
        p.speak();
    }
}

class Person {

    // 私有
    private int age;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    // 说话方法
    void speak() {
        System.out.println("我今年" + age + "岁");
    }
}
~~~

> 输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160515140049396)

> 但是你要切记，注意，私有仅仅是封装的一种表现形式；
> 
> 我们之所以对外提供访问方式就是为了方便我们加入逻辑判断语句，对访问的数据进行操作，提高代码的健壮性

### 7.构造方法

> 特点

*   函数名和类名相同
*   不用定义返回值类型
*   不可以写return语句

> 作用

*   给对象进行初始化

> 注意

*   默认构造函数的特点
*   多个构造函数是以重载的形式存在的

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        Person p = new Person();
    }
}

class Person {

    //构造方法
    public Person() {
        System.out.println("我是构造方法");
    }

}
~~~

> 我们只要new了，就执行了构造方法

![这里写图片描述](http://img.blog.csdn.net/20160515151422393)

> 对象一建立就会调用与之对应的构造函数 
> 构造函数的作用：可以用于对对象的初始化 
> 构造函数的小细节，当一个类中没有定义构造函数时，系统默认给该类加入一个空参数构造方法 
> 当该类定义了构造方法，那就默认的没有了，构造方法用了重载

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        Person p = new Person();
        Person p1 = new Person("我是小米");
        Person p2 = new Person("我是小王", 20);
    }
}

class Person {

    // 构造方法
    public Person() {
        System.out.println("我是构造方法");
    }

    // 构造方法
    public Person(String str) {
        System.out.println(str);
    }

    // 构造方法
    public Person(String str, int age) {
        System.out.println("我是构造方法" + age);
    }

}
~~~

> 就是这样，我们输出

![这里写图片描述](http://img.blog.csdn.net/20160515152832555)

### 8.构造代码块

> 这里提个小知识点来完结本篇幅 
> 我们看一段代码

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        Person p = new Person();
    }
}

class Person {

    {
        System.out.println("我是构造方法");
    }

}
~~~

> 想知道他的运行结果是什么吗

![这里写图片描述](http://img.blog.csdn.net/20160515154454799)

> 咦，为什么方法都没有名字，就运行了，这个{}就是构造方法吗？ 
> 如果你代用多个构造方法的话你会发现他掉欧勇多次，这个现象，我们可以这样解释
> 
> 构造代码块：

*   作用就是给对象初始化
*   而且优先于构造方法

> 和构造方法的区别：

*   构造代码块是给所有对象进行统一初始化
*   而构造函数是给对应的对象初始化

> 构造方法中定义的是不同对象共性的初始化内容（抽取）

#### OK，我们下篇继续，JAVA的思想概念性的东西需要吃透才行，所有博客就慢慢的更新

#### 热爱Android可以加群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。