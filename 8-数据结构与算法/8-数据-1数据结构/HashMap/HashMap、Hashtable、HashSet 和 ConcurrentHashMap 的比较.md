原文链接：
存档链接：

---



HashMap 和 Hashtable 的比较是 Java 面试中的常见问题，用来考验程序员是否能够正确使用集合类以及是否可以随机应变使用多种思路解决问题。HashMap 的工作原理、ArrayList 与 Vector 的比较以及这个问题是有关 Java 集合框架的最经典的问题。Hashtable 是个过时的集合类，存在于 Java API 中很久了。在 Java 4 中被重写了，实现了 Map 接口，所以自此以后也成了 Java 集合框架中的一部分。Hashtable 和 HashMap 在 Java 面试中相当容易被问到，甚至成为了集合框架面试题中最常被考的问题，所以在参加任何 Java 面试之前，都不要忘了准备这一题。

这篇文章中，我们不仅将会看到 HashMap 和 Hashtable 的区别，还将看到它们之间的相似之处。

### HashMap 和 Hashtable 的区别

HashMap 和 Hashtable 都实现了 Map 接口，但决定用哪一个之前先要弄清楚它们之间的分别。主要的区别有：线程安全性，同步 (synchronization)，以及速度。

1.  HashMap 几乎可以等价于 Hashtable，除了 HashMap 是非 synchronized 的，并可以接受 null(HashMap 可以接受为 null 的键值 (key) 和值 (value)，而 Hashtable 则不行)。
2.  HashMap 是非 synchronized，而 Hashtable 是 synchronized，这意味着 Hashtable 是线程安全的，多个线程可以共享一个 Hashtable；而如果没有正确的同步的话，多个线程是不能共享 HashMap 的。Java 5 提供了 ConcurrentHashMap，它是 HashTable 的替代，比 HashTable 的扩展性更好。
3.  另一个区别是 HashMap 的迭代器 (Iterator) 是 fail-fast 迭代器，而 Hashtable 的 enumerator 迭代器不是 fail-fast 的。所以当有其它线程改变了 HashMap 的结构（增加或者移除元素），将会抛出ConcurrentModificationException，但迭代器本身的 remove() 方法移除元素则不会抛出ConcurrentModificationException 异常。但这并不是一个一定发生的行为，要看 JVM。这条同样也是Enumeration 和 Iterato r的区别。
4.  由于 Hashtable 是线程安全的也是 synchronized，所以在单线程环境下它比 HashMap 要慢。如果你不需要同步，只需要单一线程，那么使用 HashMap 性能要好过 Hashtable。
5.  HashMap 不能保证随着时间的推移 Map 中的元素次序是不变的。

### 要注意的一些重要术语：

1) sychronized 意味着在一次仅有一个线程能够更改 Hashtable。就是说任何线程要更新 Hashtable 时要首先获得同步锁，其它线程要等到同步锁被释放之后才能再次获得同步锁更新 Hashtable。

2) Fail-safe 和 iterator 迭代器相关。如果某个集合对象创建了 Iterator 或者 ListIterator，然后其它的线程试图“结构上”更改集合对象，将会抛出 ConcurrentModificationException 异常。但其它线程可以通过 set() 方法更改集合对象是允许的，因为这并没有从“结构上”更改集合。但是假如已经从结构上进行了更改，再调用 set() 方法，将会抛出 IllegalArgumentException 异常。

3) 结构上的更改指的是删除或者插入一个元素，这样会影响到 map 的结构。

### 我们能否让 HashMap 同步？

HashMap 可以通过下面的语句进行同步：
Map m = Collections.synchronizeMap(hashMap);

### 结论

Hashtable 和 HashMap 有几个主要的不同：线程安全以及速度。仅在你需要完全的线程安全的时候使用Hashtable，而如果你使用 Java 5 或以上的话，请使用 ConcurrentHashMap 吧。

转载自：[HashMap和Hashtable的区别](https://link.juejin.im?target=http%3A%2F%2Fwww.importnew.com%2F7010.html)

* * *

关于 HashMap 线程不安全这一点，《Java并发编程的艺术》一书中是这样说的：

> HashMap 在并发执行 put 操作时会引起死循环，导致 CPU 利用率接近 100%。因为多线程会导致 HashMap 的 Node 链表形成环形数据结构，一旦形成环形数据结构，Node 的 next 节点永远不为空，就会在获取 Node 时产生死循环。

原因：

*   [疫苗：JAVA HASHMAP的死循环 —— 酷壳](https://link.juejin.im?target=http%3A%2F%2Fcoolshell.cn%2Farticles%2F9606.html)
*   [HashMap在java并发中如何发生死循环](https://link.juejin.im?target=http%3A%2F%2Ffirezhfox.iteye.com%2Fblog%2F2241043)
*   [How does a HashMap work in JAVA](https://link.juejin.im?target=http%3A%2F%2Fcoding-geek.com%2Fhow-does-a-hashmap-work-in-java%2F)

* * *

下面的是自己有道云笔记中记录的：

**HashMap ， HashTable 和 HashSet 区别**

1.  关于 HashMap 的一些说法：

a) HashMap 实际上是一个“链表散列”的数据结构，即数组和链表的结合体。HashMap 的底层结构是一个数组，数组中的每一项是一条链表。

b) HashMap 的实例有俩个参数影响其性能： “初始容量” 和 装填因子。

c) HashMap 实现不同步，线程不安全。 HashTable 线程安全

d) HashMap 中的 key-value 都是存储在 Entry 中的。

e) HashMap 可以存 null 键和 null 值，不保证元素的顺序恒久不变，它的底层使用的是数组和链表，通过hashCode() 方法和 equals 方法保证键的唯一性

f) 解决冲突主要有三种方法：定址法，拉链法，再散列法。HashMap 是采用拉链法解决哈希冲突的。

注： 链表法是将相同 hash 值的对象组成一个链表放在 hash 值对应的槽位；

用开放定址法解决冲突的做法是：当冲突发生时，使用某种探查(亦称探测)技术在散列表中形成一个探查(测)序列。 沿此序列逐个单元地查找，直到找到给定 的关键字，或者碰到一个开放的地址(即该地址单元为空)为止（若要插入，在探查到开放的地址，则可将待插入的新结点存人该地址单元）。

拉链法解决冲突的做法是： 将所有关键字为同义词的结点链接在同一个单链表中 。若选定的散列表长度为m，则可将散列表定义为一个由m个头指针组成的指针数 组T[0..m-1]。凡是散列地址为i的结点，均插入到以T[i]为头指针的单链表中。T中各分量的初值均应为空指针。在拉链法中，装填因子α可以大于1，但一般均取α≤1。拉链法适合未规定元素的大小。

1.  Hashtable 和 HashMap 的区别：

a) 继承不同。

public class Hashtable extends Dictionary implements Map

public class HashMap extends AbstractMap implements Map

b) Hashtable 中的方法是同步的，而 HashMap 中的方法在缺省情况下是非同步的。在多线程并发的环境下，可以直接使用 Hashtable，但是要使用 HashMap 的话就要自己增加同步处理了。

c) Hashtable 中， key 和 value 都不允许出现 null 值。 在 HashMap 中， null 可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为 null 。当 get() 方法返回 null 值时，即可以表示 HashMap 中没有该键，也可以表示该键所对应的值为 null 。因此，在 HashMap 中不能由 get() 方法来判断 HashMap 中是否存在某个键， 而应该用 containsKey() 方法来判断。

d) 两个遍历方式的内部实现上不同。Hashtable、HashMap 都使用了Iterator。而由于历史原因，Hashtable还使用了 Enumeration 的方式 。

e) 哈希值的使用不同，HashTable 直接使用对象的 hashCode。而 HashMap 重新计算 hash 值。

f) Hashtable 和 HashMap 它们两个内部实现方式的数组的初始大小和扩容的方式。HashTable 中 hash 数组默认大小是11，增加的方式是 old*2+1。HashMap 中 hash 数组的默认大小是 16，而且一定是2的指数。

注： HashSet 子类依靠 hashCode() 和 equal() 方法来区分重复元素。

HashSet 内部使用 Map 保存数据，即将 HashSet 的数据作为 Map 的 key 值保存，这也是 HashSet 中元素不能重复的原因。而 Map 中保存 key 值的,会去判断当前 Map 中是否含有该 Key 对象，内部是先通过 key 的hashCode, 确定有相同的 hashCode 之后，再通过 equals 方法判断是否相同。

* * *

《HashMap 的工作原理》

HashMap的工作原理是近年来常见的Java面试题。几乎每个Java程序员都知道HashMap，都知道哪里要用HashMap，知道 Hashtable和HashMap之间的区别，那么为何这道面试题如此特殊呢？是因为这道题考察的深度很深。这题经常出现在高级或中高级面试中。投资银行更喜欢问这个问题，甚至会要求你实现HashMap来考察你的编程能力。ConcurrentHashMap和其它同步集合的引入让这道题变得更加复杂。让我们开始探索的旅程吧！

### 先来些简单的问题

**“你用过HashMap吗？” “什么是HashMap？你为什么用到它？”**

几乎每个人都会回答“是的”，然后回答HashMap的一些特性，譬如HashMap可以接受null键值和值，而Hashtable则不能；HashMap是非synchronized;HashMap很快；以及HashMap储存的是键值对等等。这显示出你已经用过HashMap，而且对它相当的熟悉。但是面试官来个急转直下，从此刻开始问出一些刁钻的问题，关于HashMap的更多基础的细节。面试官可能会问出下面的问题：

**“你知道HashMap的工作原理吗？” “你知道HashMap的get()方法的工作原理吗？”**

你也许会回答“我没有详查标准的Java API，你可以看看Java源代码或者Open JDK。”“我可以用Google找到答案。”

但一些面试者可能可以给出答案，“HashMap是基于hashing的原理，我们使用put(key, value)存储对象到HashMap中，使用get(key)从HashMap中获取对象。当我们给put()方法传递键和值时，我们先对键调用hashCode()方法，返回的hashCode用于找到bucket位置来储存Entry对象。”这里关键点在于指出，HashMap是在bucket中储存键对象和值对象，作为Map.Entry。这一点有助于理解获取对象的逻辑。如果你没有意识到这一点，或者错误的认为仅仅只在bucket中存储值的话，你将不会回答如何从HashMap中获取对象的逻辑。这个答案相当的正确，也显示出面试者确实知道hashing以及HashMap的工作原理。但是这仅仅是故事的开始，当面试官加入一些Java程序员每天要碰到的实际场景的时候，错误的答案频现。下个问题可能是关于HashMap中的碰撞探测(collision detection)以及碰撞的解决方法：

**“当两个对象的hashcode相同会发生什么？”**

从这里开始，真正的困惑开始了，一些面试者会回答因为hashcode相同，所以两个对象是相等的，HashMap将会抛出异常，或者不会存储它们。然后面试官可能会提醒他们有equals()和hashCode()两个方法，并告诉他们两个对象就算hashcode相同，但是它们可能并不相等。一些面试者可能就此放弃，而另外一些还能继续挺进，他们回答“因为hashcode相同，所以它们的bucket位置相同，‘碰撞’会发生。因为HashMap使用链表存储对象，这个Entry(包含有键值对的Map.Entry对象)会存储在链表中。”这个答案非常的合理，虽然有很多种处理碰撞的方法，这种方法是最简单的，也正是HashMap的处理方法。但故事还没有完结，面试官会继续问：

**“如果两个键的hashcode相同，你如何获取值对象？”**

面试者会回答：当我们调用get()方法，HashMap会使用键对象的hashcode找到bucket位置，然后获取值对象。面试官提醒他如果有两个值对象储存在同一个bucket，他给出答案:将会遍历链表直到找到值对象。面试官会问因为你并没有值对象去比较，你是如何确定确定找到值对象的？除非面试者直到HashMap在链表中存储的是键值对，否则他们不可能回答出这一题。

其中一些记得这个重要知识点的面试者会说，找到bucket位置之后，会调用keys.equals()方法去找到链表中正确的节点，最终找到要找的值对象。完美的答案！

许多情况下，面试者会在这个环节中出错，因为他们混淆了hashCode()和equals()方法。因为在此之前hashCode()屡屡出现，而equals()方法仅仅在获取值对象的时候才出现。一些优秀的开发者会指出使用不可变的、声明作final的对象，并且采用合适的equals()和hashCode()方法的话，将会减少碰撞的发生，提高效率。不可变性使得能够缓存不同键的hashcode，这将提高整个获取对象的速度，使用String，Interger这样的wrapper类作为键是非常好的选择。

如果你认为到这里已经完结了，那么听到下面这个问题的时候，你会大吃一惊。

**“如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？”**

除非你真正知道HashMap的工作原理，否则你将回答不出这道题。默认的负载因子大小为0.75，也就是说，当一个map填满了75%的bucket时候，和其它集合类(如ArrayList等)一样，将会创建原来HashMap大小的两倍的bucket数组，来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫作rehashing，因为它调用hash方法找到新的bucket位置。

如果你能够回答这道问题，下面的问题来了：

**“你了解重新调整HashMap大小存在什么问题吗？”**

你可能回答不上来，这时面试官会提醒你当多线程的情况下，可能产生条件竞争(race condition)。

当重新调整HashMap大小的时候，确实存在条件竞争，因为如果两个线程都发现HashMap需要重新调整大小了，它们会同时试着调整大小。在调整大小的过程中，存储在链表中的元素的次序会反过来，因为移动到新的bucket位置的时候，HashMap并不会将元素放在链表的尾部，而是放在头部，这是为了避免尾部遍历(tail traversing)。如果条件竞争发生了，那么就死循环了。这个时候，你可以质问面试官，为什么这么奇怪，要在多线程的环境下使用HashMap呢？：）

热心的读者贡献了更多的关于HashMap的问题：

1.  **为什么String, Interger这样的wrapper类适合作为键？**

    String, Interger这样的wrapper类作为HashMap的键是再适合不过了，而且String最为常用。因为String是不可变的，也是final的，而且已经重写了equals()和hashCode()方法了。其他的wrapper类也有这个特点。不可变性是必要的，因为为了要计算hashCode()，就要防止键值改变，如果键值在放入时和获取时返回不同的hashcode的话，那么就不能从HashMap中找到你想要的对象。不可变性还有其他的优点如线程安全。如果你可以仅仅通过将某个field声明成final就能保证hashCode是不变的，那么请这么做吧。因为获取对象的时候要用到equals()和hashCode()方法，那么键对象正确的重写这两个方法是非常重要的。如果两个不相等的对象返回不同的hashcode的话，那么碰撞的几率就会小些，这样就能提高HashMap的性能。

2.  **我们可以使用自定义的对象作为键吗？**

    这是前一个问题的延伸。当然你可能使用任何对象作为键，只要它遵守了equals()和hashCode()方法的定义规则，并且当对象插入到Map中之后将不会再改变了。如果这个自定义对象时不可变的，那么它已经满足了作为键的条件，因为当它创建之后就已经不能改变了。

3.  **我们可以使用CocurrentHashMap来代替Hashtable吗？**

    这是另外一个很热门的面试题，因为ConcurrentHashMap越来越多人用了。我们知道Hashtable是synchronized的，但是ConcurrentHashMap同步性能更好，因为它仅仅根据同步级别对map的一部分进行上锁。ConcurrentHashMap当然可以代替HashTable，但是HashTable提供更强的线程安全性。看看 [这篇博客](https://link.juejin.im?target=http%3A%2F%2Fjavarevisited.blogspot.sg%2F2011%2F04%2Fdifference-between-concurrenthashmap.html) 查看Hashtable和ConcurrentHashMap的区别。

我个人很喜欢这个问题，因为这个问题的深度和广度，也不直接的涉及到不同的概念。让我们再来看看这些问题设计哪些知识点：

*   hashing的概念
*   HashMap中解决碰撞的方法
*   equals()和hashCode()的应用，以及它们在HashMap中的重要性
*   不可变对象的好处
*   HashMap多线程的条件竞争
*   重新调整HashMap的大小

### 总结

#### HashMap的工作原理

HashMap基于hashing原理，我们通过put()和get()方法储存和获取对象。当我们将键值对传递给put()方法时，它调用键对象的hashCode()方法来计算hashcode，让后找到bucket位置来储存值对象。当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。HashMap使用链表来解决碰撞问题，当发生碰撞了，对象将会储存在链表的下一个节点中。 HashMap在每个链表节点中储存键值对对象。

当两个不同的键对象的hashcode相同时会发生什么？ 它们会储存在同一个bucket位置的链表中。键对象的equals()方法用来找到键值对。

因为HashMap的好处非常多，我曾经在电子商务的应用中使用HashMap作为缓存。因为金融领域非常多的运用Java，也出于性能的考虑，我们会经常用到HashMap和ConcurrentHashMap。你可以查看更多的关于HashMap的文章:

*   [HashMap和Hashtable的区别](https://link.juejin.im?target=http%3A%2F%2Fwww.importnew.com%2F7010.html)
*   [HashMap和HashSet的区别](https://link.juejin.im?target=http%3A%2F%2Fwww.importnew.com%2F6931.html)

转载自：[HashMap的工作原理](https://link.juejin.im?target=http%3A%2F%2Fwww.importnew.com%2F7099.html)

* * *

其他的 HashMap 学习资料：

*   [jdk7中HashMap知识点整理](https://link.juejin.im?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000003617333)
*   [HashMap源码分析（四）put-jdk8-红黑树的引入](https://link.juejin.im?target=http%3A%2F%2Fblog.csdn.net%2Fq291611265%2Farticle%2Fdetails%2F46797557)
*   [JDK7与JDK8中HashMap的实现](https://link.juejin.im?target=https%3A%2F%2Fmy.oschina.net%2Fhosee%2Fblog%2F618953)
*   [JDK1.8HashMap原理和源码分析(java面试收藏)](https://link.juejin.im?target=https%3A%2F%2Fwenku.baidu.com%2Fview%2F6e1035943968011ca30091cd.html)
*   [谈谈ConcurrentHashMap1.7和1.8的不同实现](https://link.juejin.im?target=http%3A%2F%2Fwww.jianshu.com%2Fp%2Fe694f1e868ec)
*   [jdk1.8的HashMap和ConcurrentHashMap](https://link.juejin.im?target=https%3A%2F%2Fmy.oschina.net%2Fpingpangkuangmo%2Fblog%2F817973)
*   [ConcurrentHashMap源码分析（JDK8版本）](https://link.juejin.im?target=http%3A%2F%2Fblog.csdn.net%2Fu010723709%2Farticle%2Fdetails%2F48007881)

* * *

### 最后

谢谢阅读，如果可以的话欢迎大家转发和点赞。如需转载注明[原地址](https://link.juejin.im?target=www.54tianzhisheng.cn%2F2017%2F06%2F10%2FHashMap-Hashtable%2F)就行。

群 528776268 欢迎各位大牛进群一起讨论。

![](https://user-gold-cdn.xitu.io/2017/6/12/d193c8e4e67eeb6c1c6b5256fdf4923d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

作者：zhisheng
链接：https://juejin.im/post/593e5364ac502e006c0c7690
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。