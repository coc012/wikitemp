

# JAVA之旅（十）——异常的概述，Try-Catch，异常声明Throws，多异常处理，自定义异常，Throw和Throws的区别

* * *

> 不知不觉，JAVA之旅这个系列已经更新到第十篇了，感觉如梦如幻，时间过得很快，转眼一个月就这样过去了，我们不多说，继续我们的JAVA之旅

## 一.异常的概述

> 异常算是程序中一个比较重要的环节了，我们首先来看一下异常的体系，我们举一个小例子，定义一个除法方法

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        Demo d = new Demo();
        System.out.println(d.div(6, 2));
    }
}

class Demo {
    /**
     * 定义一个除法
     * 
     * @param a
     * @param b
     * @return
     */
    int div(int a, int b) {

        return a / b;
    }
}
~~~

> 这段程序很好理解吧，就是除嘛，传6和2进去，的出来的结果肯定是3呀，但是，我现在传一个4和0，那输出的结果又会是什么呢？

![这里写图片描述](http://img.blog.csdn.net/20160529191622355)

> OK，异常出现了

*   异常：就是程序在运行时出现的不正常情况

> 异常的由来

*   问题也是现实生活中一个具体的事物，也可以通过JAVA的类的形式进行描述，并封装成对象，其实就是JAVA对不正常情况进行描述后的对象体现

*   对于问题的划分，分为两种，一种是严重问题，一种是非严重问题，对于严重的问题JAVA通过ERROR类描述，非严重的，用Exception类来进行描述

*   对于ERROR，一般不编写针对性的代码进行描述

*   对于Exception可以通过正对性的处理方式进行处理

> 无论ERROR还是Exception，都具备一些共性的内容，比如：不正常情况的信息，引发原因等

*   Throwable 

    *   Error
    *   Exception

![这里写图片描述](http://img.blog.csdn.net/20160529195703063)

> Error的错误很多，你基本上很多都可以根据名字追到是什么错误

![这里写图片描述](http://img.blog.csdn.net/20160529200501441)

> 但是我们今天不讲Error，我们看的是异常

![这里写图片描述](http://img.blog.csdn.net/20160529200542769)

## 二.Try-Catch

> Try-Catch就是抛出异常，也就是异常的处理

~~~
    try {
            //需要被检测的代码
        } catch (Exception e) {
            // 处理异常的代码（处理方式）
        }
~~~

> 既然知道了处理方式，那我们就可以针对上面的异常进行处理了

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        Demo d = new Demo();
        try {
            System.out.println(d.div(4, 2));
        } catch (Exception e) {
            System.out.println("异常");
        }

    }
}

class Demo {
    /**
     * 定义一个除法
     * 
     * @param a
     * @param b
     * @return
     */
    int div(int a, int b) {

        return a / b;
    }
}
~~~

> 对捕获的异常对象进项常见的处理方法

*   getMessage() 错误信息
*   toString() 转换成string的异常信息
*   printStackTrace 打印内存中的跟踪信息

## 三.异常声明Throws

> 我们不确定这段代码有没有问题，那我们就得去标识，怎么标识？Throws

~~~
/**
     * 定义一个除法
     * 
     * @param a
     * @param b
     * @return
     */
    int div(int a, int b) throws Exception{

        return a / b;
    }
~~~

> 在功能上通过throws的关键字来声明了该功能有可能会出现问题，所以我们使用的时候就会有提示；

![这里写图片描述](http://img.blog.csdn.net/20160529204551757)

> 你不处理我就不让你用，提高了安全性

## 三.多异常处理

> 对多异常的处理方式是怎么样的呢？

*   1.声明异常时，建议声明更为具体的异常，这样处理的可以更加具体
*   2对方声明几个异常，就对应有几个catch块，如果多个catch块中的异常出现继承关系，父类异常catch放在最下面，不要定义多余的catch块
*   3.建议在进行catch处理时，catch钟一定要定义具体处理方式，不要简单的定义一句显示格式

> 标准格式

~~~
    try {

        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (NullPointerException e) {

        }
~~~

> 也就是多catch

## 四.自定义异常

> 我们知道，异常分很多种，我们也可以自定义异常，也就是自己定义一些规则，因为项目中会出现一些特有的异常，而这些问题并未被JAV封装成异常，针对这些问题，我们可以按照JAVA对问题封装的思想，将特有的问题进行自定义的异常封装
> 
> 如何去自定义异常？
> 
> 需求，在本程序中，对于出书是-1？也视为是错误的，是无法进行运算的，那么就需要对这个问题进行自定义的描述
> 
> 当在函数内部出现throw抛出异常对象，那么必须要给对应的处理动作
> 
> 要么在函数上声明让调用者处理
> 
> throw关键字自定义异常，一般情况下，函数内出现异常，却没有需要声明，发现打印的就黑锅只有异常的名称，却没有信息，因为自定义的异常并未定义的信息
> 
> 如何定义异常信息

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        Demo d = new Demo();
        try {
            d.dev(4, -1);
        } catch (FushuException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

/**
 * 负数异常
 * 
 * @author LGL
 *
 */
class FushuException extends Exception {

    private String msg;

    public FushuException(String msg) {
        this.msg = msg;
    }

    @Override
    public String getMessage() {
        // TODO Auto-generated method stub
        return msg;
    }
}

class Demo {
    int dev(int a, int b) throws FushuException {
        if (b < 0) {
            // 手动通过throw关键字抛出自定义异常对象
            throw new FushuException("出现了除数是负数的异常");
        }
        return a / b;
    }
}
~~~

> 这段代码挺好玩的

![这里写图片描述](http://img.blog.csdn.net/20160530204449841)

> 但是其实我们有一点是不知道的，这个其实父类已经完成了，所以子类只要构造时，将构造信息传递给父类就行了，用super,那么就可以直接通过getMessage()方法来获取自定义的异常信息了

~~~
/**
 * 负数异常
 * 
 * @author LGL
 *
 */
class FushuException extends Exception {

    private String msg;

    public FushuException(String msg) {

        super(msg);
    }

}

~~~

## 五.Throw和Throws的区别

> 我们来一个小插曲，就是异常的两个类的区别
> 
> Throw和Throws的区别

*   1.Throws使用在函数上，Throw使用在函数内
*   2.Throws后面跟异常类，可以跟多个，用逗号区别，Throw后面跟的是异常对象

> OK，我们本篇幅就先到这里，异常的内容还是有很多的，不出意外我们下篇还是讲异常，大家感兴趣的话，可以加群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。