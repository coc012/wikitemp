

# JAVA之旅（三）——数组，堆栈内存结构，静态初始化，遍历，最值，选择/冒泡排序，二维数组，面向对象思想

* * *

> 我们继续JAVA之旅

## 一.数组

### 1.概念

> 数组就是同一种类型数据的集合，就是一个容器

*   数组的好处：可以自动给数组中的元素从0开始编号，方便操作这些元素

> 数组的格式

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 格式：元素类型 [] 数组名 = new 元素类型[元素个数] 
         * 定义一个可以存储3个整数的容器
         */
        int[] x = new int[3];
    }

}
~~~

### 2.内存结构

> JAVA程序在运行时，需要在内存中分配空间，为了提高效率，有对空间进行不同区域的划分，因为每一片区域都有特定的处理数据方式和内存内存管理方式

*   堆内存：用于存储局部变量，当数据使用完，所占空间会自动释放

*   栈内存

    *   数组和对象，通过new建立的实例都放在堆内存中
    *   每一个实体都有内存地址值
    *   实体中的变量都有默认的初始值
    *   实体不再被使用，会在不确定的时间被垃圾回收器回收
*   方法区

    *   本地方法，寄存器

> 数组还有另外一种格式

~~~
int [] arr = new int[]{3,6,8,74,99,12};
int [] arr1 = {3,6,8,74,99,12};
~~~

> 我们把他叫做静态初始化

### 3.遍历

> 数组的操作，我们先看一个最简单的，直接循环

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 数组操作
         */
        int [] arr = {3,6,8,74,99,12};
        for (int i = 0; i < arr.length; i++) {
            System.out.println("遍历"+arr[i]);
        }
    }

}
~~~

> 得到的结果

![这里写图片描述](http://img.blog.csdn.net/20160514115018671)

> 我们再来写个小例子

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 数组操作
         */
        int[] arr = { 3, 6, 8, 74, 99, 12 };
        printArray(arr);

    }

    /**
     * 定义一个方法打印数组中的元素用逗号隔开
     */

    public static void printArray(int[] array) {
        for (int i = 0; i < array.length; i++) {
            System.out.print(array[i] + ",");
        }
    }

}
~~~

> 我们的运行结果

![这里写图片描述](http://img.blog.csdn.net/20160514125924958)

> 现在我们有一个需求就是不要最后面的逗号，我们该怎么去做？其实就是判断

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 数组操作
         */
        int[] arr = { 3, 6, 8, 74, 99, 12 };
        printArray(arr);

    }

    /**
     * 定义一个方法打印数组中的元素用逗号隔开
     */

    public static void printArray(int[] array) {
        for (int i = 0; i < array.length; i++) {
            // 最后一个
            if ((array.length - 1) - i == 0) {
                System.out.print(array[i]);
            } else {
                System.out.print(array[i] + ",");
            }

        }
    }

}
~~~

> 只要我的长度-1 再减去你的循环数是0，说明是最后一个，那就不加逗号，输出的结果

### 4.获取最值

> 获取最值就是最大值，最小值之类的，我们一个个来获取

#### -1.最大/最小值

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {
        /**
         * 获取数组中的最大值和最小值
         */

        int[] arr = { 3, 6, 8, 74, 99, 12 };

        // 最大值
        int max = arr[0];
        // 最小值
        int min = arr[0];

        /**
         * 思路，定义一个变量对数组的元素进行比较，大于/小于 就记录
         */
        for (int i = 1; i < arr.length; i++) {
            if (arr[i] > max) {
                max = arr[i];
            }
        }
        System.out.println("最大值：" + max);

        for (int i = 0; i < arr.length; i++) {
            if (arr[i] < min) {
                min = arr[i];
            }
        }
        System.out.println("最小值：" + min);
    }
}
~~~

> 输出的结果‘

![这里写图片描述](http://img.blog.csdn.net/20160514132053285)

### 五.排序

> 这个就比较有意思了

#### -1.选择排序

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        /**
         * 选择排序
         */
        int[] arr = { 3, 6, 8, 74, 99, 12 };

        for (int i = 0; i < arr.length; i++) {
            for (int j = 0; j < arr.length; j++) {
                if (arr[i] > arr[j]) {
                    int temp = arr[i];
                    arr[i] = arr[j];
                    arr[j] = temp;
                }
            }
        }

        // 打印
        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
    }
}
~~~

> 拿着每一个元素都比较一遍，然后从大到小排列

![这里写图片描述](http://img.blog.csdn.net/20160514133326739)

> 我们再来一个更好用的

#### -2.冒泡排序

> 这个比较逻辑就更加效率了，相邻的两个元素进行比较，换位置

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        /**
         * 冒泡排序
         */
        int[] arr = { 3, 6, 8, 74, 99, 12 };

        for (int i = 0; i < arr.length - 1; i++) {
            // 每一次比较的元素-1，避免角标越界
            for (int j = 0; j < arr.length - i - 1; j++) {
                if (arr[j] < arr[j + 1]) {
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                }
            }
        }
        // 输出
        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i] + " ");
        }

    }
}
~~~

> 这样我们也可以输出

![这里写图片描述](http://img.blog.csdn.net/20160514133940310)

## 二.二维数组

> 说完了数组，我们又来了一个二维数组，数组中的数组，其实就是一个装数组类型的数组，这样说就比较清晰了
> 
> 格式

*   int [] [] arr = new int [3] [2] ;
*   int [] [] arr = new int [ ] [2] ;

> 这是个小知识点，我们简单说一下

~~~
//公共的   类   类名
public class HelloJJAVA {
    // 公共的 静态 无返回值 main方法 数组
    public static void main(String[] str) {

        /**
         * 二维数组的和
         */

        int[][] arr = { { 4, 6, 8 }, { 99, 22, 88 }, { 74, 36, 1 } };

        int num = 0;

        for (int i = 0; i < arr.length; i++) {
            for (int j = 0; j < arr[i].length; j++) {
                num = num + arr[i][j];
            }
        }
        System.out.println("num:" + num);
    }
}
~~~

> 输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160514170016045)

> 我们来个小练习
> 
> (选择题)int [] x,y [] ; //x是一维 y是二维

*   a.x[0] = y; //error
*   b.y[0]= x; //yes
*   c.y[0][0] = x; //error
*   d.x[0][0] = y; // error
*   e.y[0][0] = x[0]; //yes
*   f.x=y //error

## 三.面向对象

> 这篇博文其实到这里本该结束的，为了埋个伏笔，我再加点面向对象的思想，我们先理解一下面向对象的思想

*   面向对象是相对于面向过程而已
*   面向对象和面向过程都是一种思想
*   面向过程 

    *   强调的是功能行为
*   面向对象 

    *   讲功能封装进对象，强调具备了功能的对象
*   面向对象是基于面向过程的

> 这么说可能有点笼统，我们举个例子来说明
> 
> 还记得那个故事吗，把大象放进冰箱里，这里分几步？

*   第一步：打开冰箱门
*   第二步：把大象放进去
*   第三步：关闭冰箱门

> 这个行为艺术，叫做过程，这个行为过程，我很强调过程，不管是大象还是小象，无所谓，打开，放进去，关上，这就是面向过程
> 
> 而面向对象 
> 只需要我们封装这个冰箱，那么那就有打开，存储，和关闭的功能，那么我们作用的就是冰箱这个对象了，我们直接面向他
> 
> 这样说，是不是有点空洞？我下篇博客会继续深入，生动的把这个思想给描绘出来，尽情期待！

## 有兴趣的可以加群555974449！

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。