

# JAVA之旅（二十二）——Map概述,子类对象特点，共性方法，keySet，entrySet，Map小练习

* * *

> 继续坚持下去吧，各位骚年们！
> 
> 事实上，我们的数据结构，只剩下这个Map的知识点了，平时开发中，也是能看到他的，所以还是非常值得去学习的一个知识点的，我们直接开车了

## 一.Map概述

![这里写图片描述](http://img.blog.csdn.net/20160626144031517)

> 泛型 键值对，映射关系
> 
> 基本特点

*   该集合存储键值对，是一对一对往里存，而且要保证键的唯一性 

    *   1.添加 

        *   put(key ,values)
        *   putAll()
    *   2.删除 

        *   clear()
        *   remove(object key)
    *   3.判断 

        *   containsValue(Object value)
        *   containsKey(Object key)
        *   isEmpty()
    *   4.获取 

        *   get(Object key)
        *   size()
        *   values()
        *   entrySet()
        *   keySet()

> 我们的学习步骤也是这样来的,

## 二.子类对象特点

> Map有三个子类

*   Hashtable 

    *   底层是哈希表数据结构，不可以存入null值或者键，该集合是线程同步的

![这里写图片描述](http://img.blog.csdn.net/20160626151802694)

*   HashMap 

    *   底层是哈希表数据结构，允许使用null的键值对，线程是不同步的。效率高

![这里写图片描述](http://img.blog.csdn.net/20160626152117471)

*   TreeMap 

    *   底层是二叉树数据结构，线程不同步，可以用于给map集合中的键进行排序

> Map和Set很像，其实Set底层就是使用了Map集合

## 三.共性方法

> 我们看一下他们的共同点

~~~
package com.lgl.hellojava;

import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

public class HelloJJAVA {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<String, String>();

        // 添加元素
        map.put("001", "zhangsan");
        map.put("002", "lisi");
        map.put("003", "wangwu");

        System.out.println("原数据："+map);

        // 判断是否存在002的key
        System.out.println(map.containsKey("002"));
        //刪除
        System.out.println(map.remove("002"));
        System.out.println("删除后："+map);

        //获取
        System.out.println("获取："+map.get("001"));

        //可以通过get方法的返回值来判断一个键是否存在
        map.put(null, "haha");
        System.out.println("null:"+map);

        //获取map集合中所有的值
        Collection<String> values = map.values();
        System.out.println("map的值："+values);
    }
}
~~~

> 这里可以看到输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160626174139547)

> 但是这里要注意的是，添加元素，如果添加的时候，相同的键，那么后面的，会被后添加的覆盖原有的键对应的值，并put方法会返回被覆盖的值

## 四.keySet

> 想取出他的值，他并没有迭代器，那我们的思路可以转变一下拿到他的所有的键再去get不就可以拿到键值对了，我们来看一下

*   keySet 

    *   将map中所有的值存入到Set集合中，因为Set具备迭代器，所有可以迭代方法取出的所有的键，根据get方法，获取每一个键对应的值

~~~
package com.lgl.hellojava;

import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;

public class HelloJJAVA {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<String, String>();

        map.put("001", "zhangsan");
        map.put("002", "lisi");
        map.put("003", "wangwu");

        // 先获取map集合中的所有键的Set集合
        Set<String> keySet = map.keySet();

        // 有了Set集合就可以获取迭代器
        Iterator<String> iterator = keySet.iterator();

        while (iterator.hasNext()) {
            String string = iterator.next();
            // 有了键可以通过map集合的get方法获取其对应的值
            String value = map.get(string);
            System.out.println("key:" + string + "values:" + value);

        }

    }
}
~~~

> 这种方法还是比较好理解的，对吧,但是这样比较麻烦，我们来看另一种

## 五.entrySet

~~~
package com.lgl.hellojava;

import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.Map.Entry;
import java.util.Set;

public class HelloJJAVA {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<String, String>();

        map.put("001", "zhangsan");
        map.put("002", "lisi");
        map.put("003", "wangwu");

        // 将map集合中的映射关系取出，存入到Set集合中
        Set<Entry<String, String>> entrySet = map.entrySet();

        Iterator<Entry<String, String>> iterator = entrySet.iterator();

        while (iterator.hasNext()) {
            Map.Entry<String, String> entry = iterator.next();
            System.out.println(entry.getKey() + ":" + entry.getValue());
        }
    }
}
~~~

> 定义泛型虽然比较麻烦，但是取出来还是比较简单的，原理是什么？其实我们可以写一段伪代码来说明的

~~~
package com.lgl.hello;

public class HashMap implements Map {

    class Hahs implements Map.Entry {

        @Override
        public Object getKey() {
            // TODO Auto-generated method stub
            return null;
        }

        @Override
        public Object getValue() {
            // TODO Auto-generated method stub
            return null;
        }

    }
}

interface Map {
    public static interface Entry {
        public abstract Object getKey();

        public abstract Object getValue();
    }
}
~~~

> 父子接口，直接访问，内部规则

## 六.Map小练习

> 我们可以通过一个小练习来学习一下使用规则，而需求，我直接写在注释上

~~~
package com.lgl.hellojava;

import java.util.HashMap;
import java.util.Iterator;
import java.util.Map.Entry;
import java.util.Set;

public class HelloJJAVA {
    public static void main(String[] args) {
        /**
         * 每个学生都有对应的归属地 学生Student,地址String 学生属性：姓名和年龄
         * 注意：姓名和年龄相同的视为同一个学生，保证学生的唯一性
         * 
         * 1.描述学生 2.定义Map容器，将学生作为键，地址作为值存入 3.获取Map容器中的元素
         */
        HashMap<Student, String> hm = new HashMap<Student, String>();

        hm.put(new Student("zhangsan", 15), "beijing");
        hm.put(new Student("lisi", 16), "shanghai");
        hm.put(new Student("wangwu", 17), "guangzhou");
        hm.put(new Student("liliu", 10), "shenzhen");

        // 第一种取出方式keySet
        Set<Student> keySet = hm.keySet();
        Iterator<Student> iterator = keySet.iterator();
        while (iterator.hasNext()) {
            Student student = iterator.next();
            String addr = hm.get(student);
            System.out.println(student + ":" + addr);
        }

        //第二种取出方式 entrySet
        Set<Entry<Student, String>> entrySet = hm.entrySet();
        Iterator<Entry<Student, String>> iterator2 = entrySet.iterator();
        while (iterator2.hasNext()) {
            Entry<Student, String> next = iterator2.next();
            System.out.println(next.getKey()+":"+next.getValue());
        }
    }
}

/**
 * 描述学生
 * 
 * @author LGL
 *
 */
class Student implements Comparable<Student> {

    private String name;
    private int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int hashCode() {
        // TODO Auto-generated method stub
        return name.hashCode() + age * 34;
    }

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Student))
            throw new RuntimeException("类型不匹配");
        Student s = (Student) obj;
        return this.name.equals(s.name) && this.age == s.age;
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

    @Override
    public int compareTo(Student s) {
        int num = new Integer(this.age).compareTo(new Integer(s.age));
        if (num == 0)
            return this.name.compareTo(s.name);
        return num;
    }
}
~~~

> OK,例子就是用两种取出的方式罢了，相信你自己也一定能做好的，好的，我们本节课到这里也就结束了，下节再见

## 有兴趣的话，加下群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。