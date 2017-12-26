

# JAVA之旅（二）——if,switch,for，while,do while,语句嵌套，流程控制break , continue ，函数，重载的示例总结

* * *

> JAVA的思想真的很重要，所以要专心的学——献给刚入门的小程序员们

## 一.语句

> 一般语句也就三个类型

*   判断语句 if
*   选择语句 switch
*   循环语句 for
*   当然，还有其他的

> 我们这里一个一个来讲

### 1.if

> if,如果，就是判断，if(条件){}

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        int a = 5;
        if (a > 10) {
            System.out.println("我比10大");
        } else {
            System.out.println("我比10小");
        }
    }
}
~~~

> 输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160508151837822)

> 这里我们来写一个经典的在题目

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        // 根据1-7来判断使星期几

        int day = 3;

        // 判断
        if (day == 1) {
            System.out.println("今天星期一");
        } else if (day == 2) {
            System.out.println("今天星期二");
        } else if (day == 3) {
            System.out.println("今天星期三");
        } else if (day == 4) {
            System.out.println("今天星期四");
        } else if (day == 5) {
            System.out.println("今天星期五");
        } else if (day == 6) {
            System.out.println("今天星期六");
        } else if (day == 7) {
            System.out.println("今天星期七");
        } else {
            System.out.println("不是1-7范围内的数字");
        }

    }
}
~~~

> 应该很容易看懂吧，根据day的值来判断是星期几，如果不是1-7的话就提示不在范围内 
> 所以输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160508165447027)

> 当然，如果你要判断季节什么的，也可以用逻辑运算来判断，这里就不讲了；

### 2.switch

> 这个就是选择了，结构是

~~~
switch (表达式) {
        case 取值1:

            break;
        //最终执行
        default:
            break;
        }
~~~

> 我们具体来看看怎么执行的吧

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        int a = 3;

        switch (a) {
        case 1:
            System.out.println("1");
            break;
        case 2:
            System.out.println("2");
            break;
        case 3:
            System.out.println("3");
            break;
        case 4:
            System.out.println("4");
            break;
        case 5:
            System.out.println("5");
            break;
        // 最终执行
        default:
            System.out.println("都不是");
            break;
        }

    }
}
~~~

> 这里可以看到，根据a的值来选择要执行的代码块，所以这里输出的是3，如果你把a改成6，那就会输出都不是了
> 
> 我们这里也来做一个挺经典的题目

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        /**
         * 根据用户指定的月份，打印相应的季节
         */

        int month = 7;

        switch (month) {
        case 1:
        case 2:
        case 3:
            System.out.println("春");
            break;
        case 4:
        case 5:
        case 6:
            System.out.println("夏");
            break;
        case 7:
        case 8:
        case 9:
            System.out.println("秋");
            break;
        case 10:
        case 11:
        case 12:
            System.out.println("冬");
            break;

        default:
            System.out.println("输入不再范围内");
            break;
        }

    }
}
~~~

> 这里，输出的是秋
> 
> if和switch很像，那什么时候使用呢？如果判断的具体数值不多，而是符合byte short int char 类型，使用switch，其他情况，使用if，虽然都可以，但是switch效率据说稍微高一点点…

### 3.while

> 循环语句的一种，循环有三种

*   while
*   do while
*   for

> 先来看看while

~~~
while (条件) {
            // 输出

        }
~~~

> 代码来说明

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        int a = 5;
        // 循环5次
        while (a < 10) {
            a++;
            System.out.println("a = " + a);
        }

    }
}
~~~

> 这个代码是while，他会一直循环，当我们循环第一次的时候，a他自增就是6了，他继续循环，。一直到他<10，这样就循环了五次，我们看看输出结果

![这里写图片描述](http://img.blog.csdn.net/20160508202359054)

### 4\. do while

> do while要结合while语句这样更容易说明一些事情

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        int a = 1;
        do {
            System.out.println("a = " + a);
            a++;
        } while (a < 10);
    }
}
~~~

> 这里运行的结果

![这里写图片描述](http://img.blog.csdn.net/20160508202730012)

> 我们可以得到的区别就是while会先判断条件再去执行语句，而后者是先去执行再去判断是否要循环

### 5.for

> 这个是个大学问，我们看一下语法格式

~~~

        /**
         * 条件表达式 循环条件表达式  循环后的操作表达式
         */
        for (int i = 0; i < str.length; i++) {
            //执行语句
        }
~~~

> 我们再具体的看

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        /**
         * 条件表达式 循环条件表达式 循环后的操作表达式
         */
        for (int i = 0; i < 10; i++) {
            // 执行语句
            System.out.println("i = " + i);
        }
    }
}
~~~

> 让i去自增十次

![这里写图片描述](http://img.blog.csdn.net/20160508203214540)

> 这个过程其实while也是可以写的

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        int i = 0;
        while (i < 10) {
            System.out.println("i = " + i);
            i++;
        }
    }
}
~~~

> 运行的结果都是一样的，那这两个有什么区别呢？

*   作用域不同

> 我们还是以小练习为主吧 
> 首先我们看第一个

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        /**
         * 获取1-10的和并且打印
         */
        // 用于存储不断变化的和
        int sum = 0;
        // 记录累加的值
        for (int i = 1; i < 11; i++) {
            sum += i;
        }
        System.out.println("和为："+sum);
    }
}
~~~

> 这里打印值

![这里写图片描述](http://img.blog.csdn.net/20160511202331116)

> 好，是不是很简单，我们继续来看下一个

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 打印1-100之间7倍数的个数
         */

        int temp = 0;
        for (int i = 1; i <= 100; i++) {
            if (i % 7 == 0) {
                temp++;
            }
        }
        System.out.println("个数为：" + temp);
    }
}
~~~

> 这个是不是也很简单，最重要的是思路

### 6.语句嵌套

> 就是语句中还有语句，上面那个例子就是，不过我们这里主讲双层for循环，又叫循环嵌套

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 4; j++) {
                System.out.println("Hello");
            }
        }
    }
}
~~~

> 这种格式的，所以我们可以利用这种特点打印一个长方形

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        for (int i = 0; i < 4; i++) {
            for (int j = 0; j < 4; j++) {
                System.out.print("*");
            }
            //换行
            System.out.println();
        }
    }
}
~~~

> 看结果

![这里写图片描述](http://img.blog.csdn.net/20160511203826841)

> 这样，我们再来打印一个直角三角形

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        for (int i = 0; i < 10; i++) {
            for (int j = 0; j < i; j++) {
                System.out.print("*");
            }
            // 换行
            System.out.println();
        }
    }
}
~~~

> 得到的结果

![这里写图片描述](http://img.blog.csdn.net/20160511204139276)

> 那我们换种思路去写一个倒的

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        for (int i = 0; i < 10; i++) {
            for (int j = i; j < 10; j++) {
                System.out.print("*");
            }
            // 换行
            System.out.println();
        }
    }
}
~~~

![这里写图片描述](http://img.blog.csdn.net/20160511204323546)

> 小练习是好玩，也是大学时候的经典，我们继续

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        for (int i = 0; i < 6; i++) {
            for (int j = 1; j < i; j++) {
                System.out.print(" "+j);
            }
            System.out.println();
        }
    }
}
~~~

> 打印

![这里写图片描述](http://img.blog.csdn.net/20160511204814048)

> 这些都不算难的，我们来一个九九乘法表，还记得大学里面学这个也费了不少功夫

~~~
import java.time.Year;

//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 九九乘法表
         */
        for (int i = 1; i < 10; i++) {
            for (int j = 1; j < i + 1; j++) {

                System.out.print(j + "*" + i + " = " + j * i + " ");
            }
            System.out.println(" ");
        }
    }
}
~~~

> 打印下结果

![这里写图片描述](http://img.blog.csdn.net/20160511205504300)

### 7.流程控制语句

> 这个是什么意思呢

*   break : 跳出 选择和循环结构
*   continue: 继续 循环结构

> 根本在于，这两个只要不是应用在应用范围内，就是无效的了，这两个语句下面都不能有语句，因为不会执行，continue是结束本次循环继续下次循环，作用范围，我们来案例

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        for (int i = 0; i < 3; i++) {
            System.out.println("i : " + i);
            // 跳出循环
            break;
        }
    }
}
~~~

> 运行的结果

![这里写图片描述](http://img.blog.csdn.net/20160511210417978)

> 当我们循环第一次的时候，break已执行就跳出循环体了，也就不再循环了
> 
> 那我们再来看看continue

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        for (int i = 0; i < 10; i++) {

            if (i % 2 == 1) {
                // 继续循环
                continue;
            }
            System.out.println("i : " + i);
        }
    }
}
~~~

> 打印的结果

![这里写图片描述](http://img.blog.csdn.net/20160511210916353)

> 当符合i % 2 == 1的时候就继续循环，不再执行下面的语句了
> 
> 好的，我们语句就暂时到这里，我们用一个等腰三角形的练习来结束

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 等腰三角形
         */
        for (int i = 0; i < 5; i++) {
            for (int j = i + 1; j < 5; j++) {
                System.out.print(" ");
            }
            for (int z = 0; z <= i; z++) {
                System.out.print("* ");
            }
            System.out.println();
        }
    }
}
~~~

> OK，结束

![这里写图片描述](http://img.blog.csdn.net/20160511211724433)

## 二.函数

> 函数是什么？
> 
> 函数就是定义在类中具有特定功能的一段独立小程序，函数也称方法
> 
> 格式
> 
> 修饰符 返回类型 函数名 （参数1…参数2）{ 执行语句 return 返回值} 
> 毕竟文笔不好，还是直接用代码表达

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 求任何数+5的和
         */
        System.out.println(getNum(5));
    }

    private static int getNum(int a) {
        return a + 5;
    }
}
~~~

> 这样，我们输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160511213305220)

> 由此可以看出函数的特点

*   便于对该功能进行复用
*   函数只有被调用的时候才会执行
*   函数的出现提高了代码的复用性
*   对于函数没有具体返回值的情况，返回值类型可以用void，那么return就可以不用写了

> 要注意的是

*   函数中只能调用函数，不可以在函数内部定义函数
*   定义函数时，函数的结果返回给调用者，交由调用者处理

> 我们还是通过实际案例来吧

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        System.out.println(getNum());
    }

    private static int getNum() {
        return 6 + 5;
    }
}
~~~

> 这样就知道打印了6+5的值

## 三.重载

> 函数的重载，我们在构造方法中是见过的，我们来说一下特点

*   概念

    > 在同一个类中，允许存在一个以上的同名函数，只要他们的参数或者参数类型不同即可

*   特点

    > 与返回值类型无关，只看参数列表

*   好处 
    方便阅读，优化程序设计

代码演示

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

    }

    private static int getNum(int a) {
        return a;
    }

    private static int getNum(int a, int b) {
        return a + b;
    }

    private static int getNum(int a, int b, int c) {
        return a + b + c;
    }

}
~~~

> 这就是重载 
> 什么时候用重载？ 
> 当定义的功能相同、但参与的未知运算不同 ，那么，就定义一个函数名称以表示功能，方便阅读，而通过参数列表的不同来区分同名函数
> 
> OK，这个重载的示例就不写了，我们本篇就先到这里吧，下节我们讲数组之类的数据处理，嘻嘻！

## 我的群，通往Android的神奇之旅 ：555974449，欢迎大家进来交流技术！

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。