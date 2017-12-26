版权信息
原文链接：

---
# 前言

无论是从事Javaee开发或者是Android开发，JDK的基础知识都尤为重要。我们在代码里经常使用ArrayList、HashMap等，但却很少思考为什么是使用它，使用的时候需要注意什么。甚至有可能去面试的时候，人家一问HashMap的实现原理，但却只知道put和get，非常尴尬。

所以为了开发更高质量的程序，写出更优秀的代码，还是需要好好研究一下JDK的一些关键源码。本文主要对JDK进行一些重要的的知识的梳理及整理，便于学习及复习。

# 基础知识

## 基础数据类型

变量就是申请内存来存储值。也就是说，当创建变量的时候，需要在内存中申请空间。 内存管理系统根据变量的类型为变量分配存储空间，分配的空间只能用来储存该类型数据

| 类型 | 位 | 默认值 |
| --- | --- | --- |
| byte | 8(1字节) | 0 |
| short | 16(2字节) | 0 |
| int | 32(4字节) | 0 |
| long | 64(8字节) | 0L |
| float | 32(4字节) | 0.0f |
| double | 64(8字节) | 0.0d |
| boolean | 1 | false |
| char | 16 位 Unicode 字符 | “” |

## equal hashcode ==的区别

| == | 内存地址比较 |
| --- | --- |
| equal | Object默认内存地址比较，一般需要复写 |
| hashcode | 主要用于集合的散列表，Object默认为内存地址，一般不用设置，除非作用于散列集合。 |

hashCode 方法的常规协定，该协定声明相等对象必须具有相等的哈希码。当equals方法被重写时，通常有必要重写 hashCode 方法。但hashCode相等，不一定equals（）

## String、StringBuffer与StringBuilder的区别。

Java 平台提供了两种类型的字符串：String和StringBuffer / StringBuilder，它们可以储存和操作字符串。其中String是只读字符串，也就意味着String引用的字符串内容是不能被改变的。而StringBuffer和StringBulder类表示的字符串对象可以直接进行修改。StringBuilder是JDK1.5引入的，它和StringBuffer的方法完全相同，区别在于它是单线程环境下使用的，因为它的所有方面都没有被synchronized修饰，因此它的效率也比StringBuffer略高。

# Java的四种引用，强弱软虚，用到的场景。

JDK1.2之前只有强引用,其他几种引用都是在JDK1.2之后引入的.

强引用（Strong Reference） 最常用的引用类型，如Object obj = new Object(); 。只要强引用存在则GC时则必定不被回收。

软引用（Soft Reference） 用于描述还有用但非必须的对象，当堆将发生OOM（Out Of Memory）时则会回收软引用所指向的内存空间，若回收后依然空间不足才会抛出OOM。一般用于实现内存敏感的高速缓存。 当真正对象被标记finalizable以及的finalize()方法调用之后并且内存已经清理, 那么如果SoftReference object还存在就被加入到它的 ReferenceQueue.只有前面几步完成后,Soft Reference和Weak Reference的get方法才会返回null

弱引用（Weak Reference） 发生GC时必定回收弱引用指向的内存空间。 和软引用加入队列的时机相同

虚引用（Phantom Reference) 又称为幽灵引用或幻影引用，虚引用既不会影响对象的生命周期，也无法通过虚引用来获取对象实例，仅用于在发生GC时接收一个系统通知。 当一个对象的finalize方法已经被调用了之后，这个对象的幽灵引用会被加入到队列中。通过检查该队列里面的内容就知道一个对象是不是已经准备要被回收了. 虚引用和软引用和弱引用都不同,它会在内存没有清理的时候被加入引用队列.虚引用的建立必须要传入引用队列,其他可以没有

# Java集合框架

![Java 集合类结构图](https://user-gold-cdn.xitu.io/2017/9/12/061c920e42003dea0ab96eb21f9c5b20?imageView2/0/w/1280/h/960)
*Java 集合类结构图*

Collection是List、Set等集合高度抽象出来的接口，它包含了这些集合的基本操作，它主要又分为两大部分：List和Set。

List接口通常表示一个列表（数组、队列、链表、栈等），其中的元素可以重复，常用实现类为ArrayList和LinkedList，另外还有不常用的Vector。另外，LinkedList还是实现了Queue接口，因此也可以作为队列使用。

Set接口通常表示一个集合，其中的元素不允许重复（通过hashcode和equals函数保证），常用实现类有HashSet和TreeSet，HashSet是通过Map中的HashMap实现的，而TreeSet是通过Map中的TreeMap实现的。另外，TreeSet还实现了SortedSet接口，因此是有序的集合（集合中的元素要实现Comparable接口，并覆写Compartor函数才行）。 我们看到，抽象类AbstractCollection、AbstractList和AbstractSet分别实现了Collection、List和Set接口，这就是在Java集合框架中用的很多的适配器设计模式，用这些抽象类去实现接口，在抽象类中实现接口中的若干或全部方法，这样下面的一些类只需直接继承该抽象类，并实现自己需要的方法即可，而不用实现接口中的全部抽象方法。

Map是一个映射接口，其中的每个元素都是一个key-value键值对，同样抽象类AbstractMap通过适配器模式实现了Map接口中的大部分函数，TreeMap、HashMap、WeakHashMap等实现类都通过继承AbstractMap来实现，另外，不常用的HashTable直接实现了Map接口，它和Vector都是JDK1.0就引入的集合类。

Iterator是遍历集合的迭代器（不能遍历Map，只用来遍历Collection），Collection的实现类都实现了iterator()函数，它返回一个Iterator对象，用来遍历集合，ListIterator则专门用来遍历List。而Enumeration则是JDK1.0时引入的，作用与Iterator相同，但它的功能比Iterator要少，它只能再Hashtable、Vector和Stack中使用。

Arrays和Collections是用来操作数组、集合的两个工具类，例如在ArrayList和Vector中大量调用了Arrays.Copyof()方法，而Collections中有很多静态方法可以返回各集合类的synchronized版本，即线程安全的版本，当然了，如果要用线程安全的结合类，首选Concurrent并发包下的对应的集合类。

## Collection List Set Map 区别

| 接口 | 是否有序 | 允许元素重复 |
| --- | --- | --- |
| collection | 否 | 是 |
| List | 是 | 是 |
| AbstractSet | 否 | 否 |
| HashSet | 否 | 否 |
| TreeSet | 是（用二叉树排序） | 否 |
| AbstractMap | 否 | 使用 key-value 来映射和存储数据， Key 必须惟一， value 可以重复 |
| HashMap | 否 | 使用 key-value 来映射和存储数据， Key 必须惟一， value 可以重复 |
| TreeMap | 是（用二叉树排序） | 使用 key-value 来映射和存储数据， Key 必须惟一， value |

## 常用集合分析

| 集合 | 主要算法 | 源码分析 |
| --- | --- | --- |
| ArrayList | 基于数组的List，封装了动态增长的Object[] 数组 | [huangjunbin.com/2016/11/14/…](https://link.juejin.im/?target=http%3A%2F%2Fhuangjunbin.com%2F2016%2F11%2F14%2F%25E7%25BA%25BF%25E6%2580%25A7%25E5%25AD%2598%25E5%2582%25A8%25E7%25BB%2593%25E6%259E%2584-ArrayList%25E3%2580%2581Vector%2F) |
| Stack | 是Vector 的子类，栈 的结构（后进先出） | [huangjunbin.com/2016/11/14/…](https://link.juejin.im/?target=http%3A%2F%2Fhuangjunbin.com%2F2016%2F11%2F14%2F%25E7%25BA%25BF%25E6%2580%25A7%25E5%25AD%2598%25E5%2582%25A8%25E7%25BB%2593%25E6%259E%2584-Stack%2F) |
| LinkedList | 实现List，Deque；实现List，可以进行队列操作，可以通过索引来随机访问集合元素；实现Deque，也可当作双端队列，也可当作栈来使用 | [huangjunbin.com/2016/11/14/…](https://link.juejin.im/?target=http%3A%2F%2Fhuangjunbin.com%2F2016%2F11%2F14%2F%25E7%25BA%25BF%25E6%2580%25A7%25E5%25AD%2598%25E5%2582%25A8%25E7%25BB%2593%25E6%259E%2584-LinkedList%2F) |
| HashMap | 基于哈希表的 Map 接口的实现, 使用顺序存储及链式存储的结构 | [huangjunbin.com/2016/11/24/…](https://link.juejin.im/?target=http%3A%2F%2Fhuangjunbin.com%2F2016%2F11%2F24%2F%25E9%25A1%25BA%25E5%25BA%258F%25E5%25AD%2598%25E5%2582%25A8%25E4%25B8%258E%25E9%2593%25BE%25E5%25BC%258F%25E5%25AD%2598%25E5%2582%25A8%25E7%259A%2584%25E9%259B%2586%25E5%2590%2588-HashMap%25E3%2580%2581HashTable%2F) |
| LinkedHashMap | LinkedHashMap是HashMap的子类，与HashMap有着同样的存储结构，但它加入了一个双向链表的头结点，将所有put到LinkedHashmap的节点一一串成了一个双向循环链表，因此它保留了节点插入的顺序，可以使节点的输出顺序与输入顺序相同 | [github.com/francistao/…](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Ffrancistao%2FLearningNotes%2Fblob%2Fmaster%2FPart2%2FJavaSE%2FLinkedHashMap%25E6%25BA%2590%25E7%25A0%2581%25E5%2589%2596%25E6%259E%2590.md) |
| TreeMap | TreeMap的实现是红黑树算法的实现，支持排序 | [blog.csdn.net/chenssy/art…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fchenssy%2Farticle%2Fdetails%2F26668941) |

## 并发

Lists

*   ArrayList——基于泛型数组
*   LinkedList——不推荐使用
*   Vector——已废弃（deprecated）

CopyOnWriteArrayList——几乎不更新，常用来遍历

Queues / deques

*   ArrayDeque——基于泛型数组
*   Stack——已废弃（deprecated）
*   PriorityQueue——读取操作的内容已排序

*   ArrayBlockingQueue——带边界的阻塞式队列
*   ConcurrentLinkedDeque / ConcurrentLinkedQueue——无边界的链表队列（CAS）
*   DelayQueue——元素带有延迟的队列
*   LinkedBlockingDeque / LinkedBlockingQueue——链表队列（带锁），可设定是否带边界
*   LinkedTransferQueue——可将元素`transfer`进行w/o存储
*   PriorityBlockingQueue——并发PriorityQueue
*   SynchronousQueue——使用Queue接口进行Exchanger

Maps

*   HashMap——通用Map
*   EnumMap——键使用enum
*   Hashtable——已废弃（deprecated）
*   IdentityHashMap——键使用==进行比较
*   LinkedHashMap——保持插入顺序
*   TreeMap——键已排序
*   WeakHashMap——适用于缓存（cache）

*   ConcurrentHashMap——通用并发Map
*   ConcurrentSkipListMap——已排序的并发Map

Sets

*   HashSet——通用set
*   EnumSet——enum Set
*   BitSet——比特或密集的整数Set
*   LinkedHashSet——保持插入顺序
*   TreeSet——排序Set

*   ConcurrentSkipListSet——排序并发Set
*   CopyOnWriteArraySet——几乎不更新，通常只做遍历

## 总结

Set的选择

1.  HashSet的性能总是比TreeSet好(特别是最常用的添加、查询元素等操作)，因为TreeSet需要额外的红黑树算法来维护集合元素的次序。只有当需要一个保持排序的Set时，才应该使用TreeSet，否则都应该使用HashSet
2.  对于普通的插入、删除操作，LinkedHashSet比HashSet要略慢一点，这是由维护链表所带来的开销造成的。不过，因为有了链表的存在，遍历LinkedHashSet会更快
3.  EnumSet是所有Set实现类中性能最好的，但它只能保存同一个枚举类的枚举值作为集合元素
4.  HashSet、TreeSet、EnumSet都是"线程不安全"的，通常可以通过Collections工具类的synchronizedSortedSet方法来"包装"该Set集合。
    SortedSet s = Collections.synchronizedSortedSet(new TreeSet(...));

List 选择

1.  java提供的List就是一个"线性表接口"，ArrayList(基于数组的线性表)、LinkedList(基于链的线性表)是线性表的两种典型实现
2.  Queue代表了队列，Deque代表了双端队列(既可以作为队列使用、也可以作为栈使用)
3.  因为数组以一块连续内存来保存所有的数组元素，所以数组在随机访问时性能最好。所以的内部以数组作为底层实现的集合在随机访问时性能最好。
4.  内部以链表作为底层实现的集合在执行插入、删除操作时有很好的性能
5.  进行迭代操作时，以链表作为底层实现的集合比以数组作为底层实现的集合性能好
6.  当要大量的插入，删除，应当选用LinkedList；当需要快速随机访问则选用ArrayList;

Map 的选择

1.  HashMap和Hashtable的效率大致相同，因为它们的实现机制几乎完全一样。但HashMap通常比Hashtable要快一点，因为Hashtable需要额外的线程同步控制
2.  TreeMap通常比HashMap、Hashtable要慢(尤其是在插入、删除key-value对时更慢)，因为TreeMap底层采用红黑树来管理key-value对
3.  使用TreeMap的一个好处就是： TreeMap中的key-value对总是处于有序状态，无须专门进行排序操作
4.  HahMap 是利用hashCode 进行查找，而TreeMap 是保持者某种有序状态
5.  所以，插入，删除，定位操作时，HashMap 是最好的选择；如果要按照自然排序或者自定义排序，那么就选择TreeMap

# 参考

[Java 学习之集合类（Collections）](https://link.juejin.im/?target=http%3A%2F%2Fwww.imooc.com%2Farticle%2F1080)

[Java集合总览](https://link.juejin.im/?target=http%3A%2F%2Fwww.importnew.com%2F13801.html)

[JavaSE(Java基础)](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Ffrancistao%2FLearningNotes%2Ftree%2Fmaster%2FPart2%2FJavaSE)