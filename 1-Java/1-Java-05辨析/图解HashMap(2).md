



---

# 图解HashMap(二)

### 概述

上篇分析了HashMap的设计思想以及Java7和Java8源码上的实现，当然还有一些"坑"还没填完，比如大家都知道HashMap是线程不安全的数据结构，多线程情况下HashMap会引起死循环引用，它是怎么产生的？Java8引入了红黑树，那是怎么提高效率的？本篇先填第一个坑，还是以图解的形式加深理解。

### Java7分析

通过上一篇的整体学习，可以知道当存入的键值对超过HashMap的阀值时，HashMap会扩容，即创建一个新的数组，并将原数组里的键值对"转移"到新的数组中。在“转移”的时候，会根据新的数组长度和要转移的键值对key值重新计算在新数组中的位置。重温下Java7中负责"转移"功能的代码

~~~
void transfer(Entry[] newTable, boolean rehash) {
    //获取新数组的长度
    int newCapacity = newTable.length;
    //遍历旧数组中的键值对
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            //计算在新表中的索引，并到新数组中
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}

~~~

为了加深理解，画个图如下

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eea95cacabc?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

这里假设扩容前后5号坑石头、盖伦、蒙多的hash值与新旧数组长度取模运算后还是5。上篇文章也总结了，Java7扩容转移前后链表顺序会倒置。当只有单线程操作hashMap时，一切都是那么美好，但是如果多线程同时操作一个hashMap，问题就来了，下面看下多线程操作一个hashMap在Java7源码下是怎样引起死循环引用。

前戏是这样的：有两个线程分别叫Thread1和Thread2，它们都有操作同一个hashMap的权利，假设hashMap中的键值对是12个，石头和盖伦扩容前后的hash值与新旧数组长度取模运算后还是5。扩容前的模拟堆内存情况如图

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eea91aa7e92?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Thread1得到执行权(Thread2被挂起)，Thread1往hashMap里put第13个键值对的时候判断超过阀值，执行扩容操作，Thread1创建了一个新数组，还没来得及转移旧键值对的时候，系统时间片反手切到Thread2(Thread1被挂起)，整个过程用图表示

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eea92264bc2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

可以看到Thread1只是创建了个新数组，还没来得及转移就被挂起了，新数组没有内容，此时在Thread1的视角认的是e是石头，next是盖伦；此时的模拟内存图情况

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eea9468e39a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

再看下Thread2的操作，同样Thread2往hashMap里put第13个键值对的时候判断超过阀值，执行扩容操作，Thread2先创建一个新数组，不同的是，Thread2运气好，在时间片轮换前转移工作也走完了。第一次遍历

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eea929d1dd7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第二次遍历

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eea9397915c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

此时模拟的内存情况

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eead7b0c8b2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

可以看到此时对于盖伦来说，他的next是石头；对于石头来说，它的next为null，隐患就此埋下。接下来时间片又切到Thread1(停了半天终于轮到我出场了)，先看下Thread1的处境

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeadbbc896b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

结合代码分析如下

第一步：

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeade859242?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第二步：

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeae4dd036d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第三步：

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeaec054b6f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第四步：

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeadda9a26c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第五步：

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeb048c6f07?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第六步：

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeb0b1ae0ca?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第七步：

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeb0c31970f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第八步：

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeb1db16a31?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第九步：

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeb11f5f472?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第10步：

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeb31ae99e2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

到这终于看到盖伦和石头"互指"，水乳交融。

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeb37a1680a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

那这会带来什么后果呢？后续操作新数组的5号坑会进入死循环(注意，操作其他坑并不会有问题)，例如Java7 put操作

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeb3f375769?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Java7 get操作会执行getEntry，同样会引起死循环。

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeb398f2304?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

到此，Java7多线程操作HashMap可能形成死循环的原因剖析完成。

### Java8分析

通过上一篇的学习可知，Java7转移前后位置颠倒，而Java8转移键值对前后位置不变。同样的前戏，看下代码

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeb3e07d3c9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

此时模拟堆内存情况

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eea9468e39a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Thread1的情况

![](https://user-gold-cdn.xitu.io/2017/12/4/16021eeb4bfa7149?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

这时候Thread2获得执行权，扩容并完成转移工作，通过上篇的学习可知，Java8在转移前会创建两条链表，即扩容后位置不加原数组长度的lo链和要加原数组长度的hi链，这里假设石头和盖伦扩容前后都在5号坑，即这是一条lo链(其实就算不在同一个坑也不影响，原因就是Java8扩容前后链顺序不变)。Thread2遍历第一次![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82a4ca56ccf?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第二次

![](https://user-gold-cdn.xitu.io/2017/12/4/16021efcb47fd57f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

可以看到Thread2全程是没有去修改石头和盖伦的引用关系，石头.next是盖伦，盖伦.next是null。那么Thread1得到执行权后其实只是重复了Thread2的工作。

### 总结

通过源码分析，Java7在多线程操作hashmap时可能引起死循环，原因是扩容转移后前后链表顺序倒置，在转移过程中修改了原来链表中节点的引用关系；Java8在同样的前提下并不会引起死循环，原因是扩容转移后前后链表顺序不变，保持之前节点的引用关系。那是不是意味着Java8就可以把HashMap用在多线程中呢？个人感觉即使不会出现死循环，但是通过源码看到put/get方法都没有加同步锁，多线程情况最容易出现的就是：无法保证上一秒put的值，下一秒get的时候还是原值，建议使用ConcurrentHashMap。

### 感谢

[讲HashMap多线程死循环最详细的外国小哥](https://link.juejin.im/?target=http%3A%2F%2Fjavabypatel.blogspot.ca%2F2016%2F01%2Finfinite-loop-in-hashmap.html)

