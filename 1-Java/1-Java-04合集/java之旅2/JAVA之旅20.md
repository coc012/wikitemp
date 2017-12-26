

# JAVA之旅（二十）—HashSet，自定义存储对象,TreeSet，二叉树，实现Comparator方式排序，TreeSet小练习

* * *

> 我们继续说一下集合框架

*   Set：元素是无序（存入和取出的顺序不一定一致），元素不可以重复

> Set集合的功能和Collection是一致的

![这里写图片描述](http://img.blog.csdn.net/20160624213603921)

> 我们重点关注的是子类对象

![这里写图片描述](http://img.blog.csdn.net/20160624213723820)

> 我们来聊聊

## 一.HashSet

> HashSet底层结构是哈希表
> 
> 什么是HashSet?

~~~
package com.lgl.hellojava;

//公共的   类   类名
public class HelloJJAVA {
    public static void main(String[] args) {
        Demo d1 = new Demo();
        Demo d2 = new Demo();

        sop(d1);
        sop(d2);
    }

    // 输出
    public static void sop(Object obj) {
        System.out.println(obj);

    }

}

class Demo {

}
~~~

> 我们这样输出的结果就是哈希值

![这里写图片描述](http://img.blog.csdn.net/20160624214048088)

> 当然，我们是来介绍HashSet的，我们演示一下

~~~
package com.lgl.hellojava;

import java.util.HashSet;
import java.util.Iterator;

//公共的   类   类名
public class HelloJJAVA {
    public static void main(String[] args) {
        HashSet h = new HashSet();
        h.add("hello 01");
        h.add("hello 02");
        h.add("hello 03");
        h.add("hello 04");

        // set取出只有一种办法，迭代器
        Iterator iterator = h.iterator();
        while (iterator.hasNext()) {
            sop(iterator.next());
        }

    }

    // 输出
    public static void sop(Object obj) {
        System.out.println(obj);

    }

}

~~~

> 是不是很类似，但是输出，你们仔细看了

![这里写图片描述](http://img.blog.csdn.net/20160624223205013)

> 输出是无序的，我们还有一个现象，就是直接输出

~~~
sop(h.add("lgl"));
sop(h.add("lgl"));
~~~

> 相同的

![这里写图片描述](http://img.blog.csdn.net/20160624223521454)

> 因为他不能重复

## 二.自定义存储对象

> 我们可以存数据，那肯定可以自定义存储数据咯？

~~~
package com.lgl.hellojava;

import java.util.HashSet;
import java.util.Iterator;

//公共的   类   类名
public class HelloJJAVA {
    public static void main(String[] args) {
        HashSet h = new HashSet();
        h.add(new Person("lgl1", 18));
        h.add(new Person("lgl2", 19));
        h.add(new Person("lgl3", 20));
        h.add(new Person("lgl4", 21));

        // set取出只有一种办法，迭代器
        Iterator iterator = h.iterator();
        while (iterator.hasNext()) {
            Person p = (Person) iterator.next();
            sop(p.getName() + ":" + p.getAge());
        }

    }

    // 输出
    public static void sop(Object obj) {
        System.out.println(obj);

    }

}

/**
 * 存储对象
 * 
 * @author LGL
 *
 */
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.setName(name);
        this.setAge(age);
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

~~~

> 这样就可以定下来了

*   HashSet是如何保证元素的唯一性呢？ 

    *   是通过元素的两个方法，hasCode和equals来完成的
    *   如果元素的hasCode相同。才会去判断equals是否为true
    *   如果元素的hasCode不同。不会调用equals

> 这里要注意一点的就是，对于判断元素是否存在的话，以及删除的操作，依赖的方法就是元素的hasCode和equals

## 三.TreeSet

> hashSet说完，我们再来看一下TreeSet，我们用小例子来说明

~~~
package com.lgl.hellojava;

import java.util.Iterator;
import java.util.TreeSet;

import org.omg.PortableInterceptor.Interceptor;

//公共的   类   类名
public class HelloJJAVA {
    public static void main(String[] args) {
        TreeSet s = new TreeSet();
        s.add("abc");
        s.add("acd");
        s.add("age");
        s.add("abf");

        Iterator iterator = s.iterator();

        while (iterator.hasNext()) {
            sop(iterator.next());
        }
    }

    // 输出
    public static void sop(Object obj) {
        System.out.println(obj);

    }
}

~~~

> 我们仔细看他的输出

![这里写图片描述](http://img.blog.csdn.net/20160625194505587)

> 他会排序，那我们就知道TreeSet的特性了

*   可以对Set集合中的元素进行排序

> 如果你用自定义对象去村粗的话，你会发现他可以存一个对象，但是不能存储多个对象，为什么？因为他会强制进行排序，如果是对象的话，他没法排序，是不行的
> 
> 对了我们没有讲TreeSet的数据结构呢，他的数据结构是二叉树，这是一个比较难的概念了

## 四.二叉树

> 二叉树其实通俗一点，就是树形图数据，比如

![这里写图片描述](http://img.blog.csdn.net/20160625200137314)

> 就是比较，一直分支，很大的节约了计算方式，我们比较，大的话，开一个分支，小的话，再开一个分支，就这样一直比较！
> 
> 那TreeSet保证元素唯一性的是compareTo方法return 0;

*   TreeSet排序的第一种方式，让元素自身具备比较性，元素需要实现Comparable 接口，覆盖compareTo方法，这种也称为元素的自然顺序！

## 五.实现Comparator方式排序

> 当元素不具备比较性时，或者具备的元素的比较性不是所需要的，这时就需要让集合自身具备比较性，那就是在集合一初始化时就有了比较方式.这么说有点绕啊，我们还是用代码来说明吧，原理都是二叉树

~~~
package com.lgl.hellojava;

import java.util.Comparator;
import java.util.Iterator;
import java.util.TreeSet;

//公共的   类   类名
public class HelloJJAVA {
    public static void main(String[] args) {
        /**
         * 当元素自身不具备比较性或者具备的比较性不是所需要的，这时需要让容器自生具备比较性，定义一个比较器，
         * 将比较器对象作为参数传递给TreeSet集合的构造函数
         */
        TreeSet s = new TreeSet(new MyCompare());
        s.add(new Student("lgl1", 22));
        s.add(new Student("lgl2", 26));
        s.add(new Student("lgl3", 10));
        s.add(new Student("lgl4", 19));

        Iterator iterator = s.iterator();
        while (iterator.hasNext()) {
            Student student = (Student) iterator.next();
            sop(student.getName() + ":" + student.getAge());
        }

    }

    // 输出
    public static void sop(Object obj) {
        System.out.println(obj);

    }
}

class Student {
    private String name;
    private int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // 比较
    public int compareTo(Object obj) {
        if (!(obj instanceof Student)) {
            throw new RuntimeException("不是学生对象");
        }
        Student s = (Student) obj;
        if (this.age > s.age) {
            return 1;
        } else if (this.age == s.age) {
            return this.name.compareTo(s.name);
        }
        return -1;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

}

// 定义比较器
class MyCompare implements Comparator {

    public int compare(Object o1, Object o2) {
        Student s1 = (Student) o1;
        Student s2 = (Student) o2;

        return s1.getName().compareTo(s2.getName());
    }

}

~~~

## 六.TreeSet小练习

> 我们到这里，就用一个小练习来结束吧，毕竟在后面就需要讲泛型了，我们的需求就是按照字符串長度排序

~~~
package com.lgl.hellojava;

import java.util.Comparator;
import java.util.Iterator;
import java.util.TreeSet;

//公共的   类   类名
public class HelloJJAVA {
    public static void main(String[] args) {
        /**
         * 按照字符串長度排序
         */
        TreeSet s = new TreeSet(new StringLengthComparator());
        s.add("ffffffff");
        s.add("fffff");
        s.add("ff");
        s.add("ffffff");

        Iterator iterator = s.iterator();
        while (iterator.hasNext()) {
            sop(iterator.next());
        }

    }

    // 输出
    public static void sop(Object obj) {
        System.out.println(obj);

    }
}

// 定义比较性
class StringLengthComparator implements Comparator {

    @Override
    public int compare(Object o1, Object o2) {

        String s1 = (String) o1;
        String s2 = (String) o2;

        if (s1.length() > s2.length())
            return 1;
        if (s1.length() == s2.length())
            return 0;
        return -1;

    }

}

~~~

> 这样就OK了，输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160625205043694)

> 这样就O了，好的，但是我们重复元素也会被干掉的，这时候我们就要处理了

~~~
    @Override
    public int compare(Object o1, Object o2) {

        String s1 = (String) o1;
        String s2 = (String) o2;

        int num = new Integer(s1.length()).compareTo(new Integer(s2.length()));
        if (num == 0) {
            return s1.compareTo(s2);
        }
        return -num;

    }
~~~

> 到这里，就基本上都搞定了，我们的博文到这里也结束了，如果有机会

## 可以加群讨论：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。