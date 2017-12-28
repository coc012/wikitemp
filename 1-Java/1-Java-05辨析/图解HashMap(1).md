



---


# 图解HashMap(一)

### 概述

HashMap是日常开发中经常会用到的一种数据结构，在介绍HashMap的时候会涉及到很多术语，比如时间复杂度O、散列(也叫哈希)、散列算法等，这些在大学课程里都有教过，但是由于某种不可抗力又还给老师了，在深入学习HashMap之前先了解HashMap设计的思路以及以及一些重要概念，在后续分析源码的时候就能够有比较清晰的认识。

### HashMap是什么

在回答这个问题之前先看个例子：小明打算从A市搬家到B市，拿了两个箱子把自己的物品打包就出发了。![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82999598954?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

到了B市之后，他想拿手机给家里报个平安，这时候问题来了，东西多了他忘记手机放在哪个箱子了？小明开始打开1号箱子找手机，没找到；再打开2号箱子找，找到手机。当只有2个箱子的时候，东西又不多的情况下，他可能花个2分钟就找到手机了，假如有20个箱子，每个箱子的东西又多又杂，那么花的时间就多了。小明总结了下查找耗时的原因，发现是因为这些东西放的没有规律，如果他把每个箱子分个类别，比如定一个箱子专门放手机、电脑等电子设备，有专门放衣服的箱子等等，那么他找东西花的时间就可以大大缩短了。

其实HashMap也是用到这种思路，HashMap作为一种数据结构，像数组和链表一样用于常规的增删改查，在存数据的时候(put)并不是随便乱放，而是会先做一次类似“分类”的操作再存储，一旦“分类”存储之后，下次取(get)的时候就可以大大缩短查找的时间。我们知道数组在执行查、改的效率很高，而增、删(不是尾部)的效率低，链表相反，HashMap则是把这两者结合起来，看下HashMap的数据结构![](https://user-gold-cdn.xitu.io/2017/12/3/1601c8299c395016?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

从上面的结构可以看出，通常情况下HashMap是以数组和链表的组合构成(Java8中将链表长度超过8的链表转化成红黑树)。结合上面找手机的例子，我们简单分析下HashMap存取操作的心路历程。put存一个键值对的时候(比如存上图盖伦)，先根据键值"分类"，"分类"一顿操作后告诉我们，盖伦应该属于14号坑，直接定位到14号坑。接下来有几种情况：

*   14号坑没人，nice，直接存值；
*   14号有人，也叫盖伦，替换原来的攻击值；
*   14号有人，叫老王！插队到老王前面去(单链表的头插入方式，同一位置上新元素总会被放在链表的头部位置)

get取的时候也需要传键值，根据传的键值来确定要找的是哪个"类别"，比如找火男，"分类"一顿操作够告诉我们火男属于2号坑，于是我们直接定位到2号坑开始找，亚索不是…找到火男。

#### 小结

HashMap是由数组和链表组合构成的数据结构，Java8中链表长度超过8时会把长度超过8的链表转化成红黑树；存取时都会根据键值计算出"类别"(hashCode)，再根据"类别"定位到数组中的位置并执行操作。

### HashCode是什么

还是举个栗子：一个工厂有500号人，下图用两种方案来存储厂里员工的信件。![](https://user-gold-cdn.xitu.io/2017/12/3/1601c8299c1702f1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

左右各有27个信箱，左边保安大哥存信的时候不做处理，想放哪个信箱就放哪个信箱，当员工去找信的时候，只好挨个信箱找，再挨个比对信箱里信封的名字，万一哥们脸黑，要找的放在最后一个信箱的最底下，悲剧…所以这种情况的时间复杂度为O(N)；右边采用HashCode的方式将27个信箱分类，分类的规则是名字首字母(第一个箱子放不写名字的哥们)，保安大哥将符合对应姓名的信件放在对应的信箱里，这样员工就不用挨个找了，只需要比对一个信箱里的信件即可，大大提高了效率，这种情况的时间复杂度趋于一个常数O(1)。

例子中右图其实就是hashCode的一个实现，每个员工都有自己的hashCode，比如李四的hashCode是L，王五的hashCode是W(这取决于你的hash算法怎么写)，然后我们根据确定的hashCode值把信箱分类，hashCode匹配则存在对应信箱。在Java的Object中可以调用hashCode()方法获取对象hashCode，返回一个int值。那么会出现两个对象的hashCode一样吗?答案是会的，就像上上个例子中盖伦和老王的hashCode就一样，这种情况网上有人称之为"hash碰撞"，出现这种所谓"碰撞"的处理上面已经介绍了解决思路，具体源码后续介绍。

#### 小结

hashCode是一个对象的标识，Java中对象的hashCode是一个int类型值。通过hashCode来指定数组的索引可以快速定位到要找的对象在数组中的位置，之后再遍历链表找到对应值，理想情况下时间复杂度为O(1)，并且不同对象可以拥有相同的hashCode。

### HashMap的时间复杂度

通过上面信箱找信的例子来讨论下HashMap的时间复杂度，在使用hashCode之后可以直接定位到一个箱子，时间的耗费主要是在遍历链表上，理想的情况下(hash算法写得很完美)，链表只有一个节点，就是我们要的![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82998d97a79?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

那么此时的时间复杂度为O(1)，那不理想的情况下(hash算法写得很糟糕)，比如上面信箱的例子，假设hash算法计算每个员工都返回同样的hashCode

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82ad01a62e7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

所有的信都放在一个箱子里，此时要找信就要依次遍历C信箱里的信，时间复杂度不再是O(1)，而是O(N)，因此HashMap的时间复杂度取决于算法的实现上，当然HashMap内部的机制并不像信箱这么简单，在HashMap内部会涉及到扩容、Java8中会将长度超过8的链表转化成红黑树，这些都在后续介绍。

#### 小结

HashMap的时间复杂度取决于hash算法，优秀的hash算法可以让时间复杂度趋于常数O(1)，糟糕的hash算法可以让时间复杂度趋于O(N)。

### 负载因子是什么

我们知道HashMap中数组长度是16(什么？你说不知道，看下源码你就知道)，假设我们用的是最优秀的hash算法，即保证我每次往HashMap里存键值对的时候，都不会重复，当hashmap里有16个键值对的时候，要找到指定的某一个，只需要1次；

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c8299ad8de92?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

之后继续往里面存值，必然会发生所谓的"hash碰撞"形成链表，当hashmap里有32个键值对时，找到指定的某一个最坏情况要2次；当hashmap里有128个键值对时，找到指定的某一个最坏情况要8次

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c829d46f2722?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

随着hashmap里的键值对越来越多，在数组数量不变的情况下，查找的效率会越来越低。那怎么解决这个问题呢？只要增加数组的数量就行了，键值对超过16，相应的就要把数组的数量增加(HashMap内部是原来的数组长度乘以2)，这就是网上所谓的扩容，就算你有128个键值对，我们准备了128个坑，还是能保证"一个萝卜一个坑"。

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c829d7ecd7a1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 其实扩容并没有那么风光，就像ArrayList一样，扩容是件很麻烦的事情，要创建一个新的数组，然后把原来数组里的键值对"放"到新的数组里，这里的"放"不像ArrayList那样用原来的index，而是根据新表的长度重新计算hashCode，来保证在新表的位置，老麻烦了，所以同一个键值对在旧数组里的索引和新数组中的索引通常是不一致的(火男："我以前是3号，怎么现在成了127号，给我个完美的解释！"新表："大清亡了，现在你得听我的")。另外，我们也可以看出这是典型的以空间换时间的操作。

说了这么多，那负载因子是个什么东西？负载因子其实就是规定什么时候扩容。上面我们说默认hashmap数组大小为16，存的键值对数量超过16则进行扩容，好像没什么毛病。然而HashMap中并不是等数组满了(达到16)才扩容，它会存在一个阀值(threshold)，只要hashmap里的键值对大于等于这个阀值，那么就要进行扩容。阀值的计算公式：

> 阀值 = 当前数组长度✖负载因子

hashmap中默认负载因子为0.75，默认情况下第一次扩容判断阀值是16 ✖ 0.75 = 12；所以第一次存键值对的时候，在存到第13个键值对时就需要扩容了；或者另外一种理解思路：假设当前存到第12个键值对：12 / 16 = 0.75，13 / 16 = 0.8125(大于0.75需要扩容) 。肯定会有人有疑问，我要这铁棒有何用？不，我要这负载因子有何用?直接规定超过数组长度再扩容不就行了，还省得每次扩容之后还要重新计算新的阀值，Google说取0.75是一个比较好的权衡，当然我们可以自己修改，HashMap初识化时可以指定数组大小和负载因子，你完全可以改成1。

~~~
public HashMap(int initialCapacity, float loadFactor)

~~~

我的理解是这负载因子就像人的饭量，有的人吃要7分饱，有的人要10分饱，稳妥起见默认让我们7.5分饱。

#### 小结

在数组大小不变的情况下，存放键值对越多，查找的时间效率会降低，扩容可以解决该问题，而负载因子决定了什么时候扩容，负载因子是已存键值对的数量和总的数组长度的比值。默认情况下负载因子为0.75，我们可在初始化HashMap的时候自己修改。

### hash与Rehash

hash和rehash的概念其实上面已经分析过了，每次扩容后，转移旧表键值对到新表之前都要重新rehash，计算键值对在新表的索引。如下图火男这个键值对被存进hashmap到后面扩容，会经过hash和rehash的过程

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c829dcc61525?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第一次hash可以理解成'"分类"'，方便后续取、改等操作可以快速定位到具体的"坑"。那么为什么要进行rehash，按照之前元素在数组中的索引直接赋值，例如火男之前3号坑，现在跑到30号坑。

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82a01e3f06d?imageslim)

个人理解是，在未扩容前，可以看到如13号链的长度是3，为了保证我们每次查找的时间复杂度O趋于O(1)，理想的情况是"一个萝卜一个坑"，那么现在"坑"多了，原来"3个萝卜一个坑"的情况现在就能有效的避免了。

### 源码分析

#### Java7源码分析

先看下Java7里的HashMap实现，有了上面的分析，现在在源码中找具体的实现。

~~~
//HashMap里的数组
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
//Entry对象，存key、value、hash值以及下一个节点
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    int hash;
}
//默认数组大小，二进制1左移4位为16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4；
//负载因子默认值
static final float DEFAULT_LOAD_FACTOR = 0.75f; 
//当前存的键值对数量
transient int size; 
//阀值 = 数组大小 * 负载因子
int threshold;
//负载因子变量
final float loadFactor;

//默认new HashMap数组大小16，负载因子0.75
public HashMap() {
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
}

public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
//可以指定数组大小和负载因子
public HashMap(int initialCapacity, float loadFactor) {
    //省略一些逻辑判断
    this.loadFactor = loadFactor;
    threshold = initialCapacity;
    //空方法
    init();
}

~~~

以上就是HashMap的一些先决条件，接着看平时put操作的代码实现，put的时候会遇到3种情况上面已分析过，看下Java7代码：

~~~
public V put(K key, V value) {
        //数组为空时创建数组
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        //key为空单独对待
        if (key == null)
            return putForNullKey(value);
        //①根据key计算hash值
        int hash = hash(key);
        //②根据hash值和当前数组的长度计算在数组中的索引
        int i = indexFor(hash, table.length);
        //遍历整条链表
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            //③情况1.hash值和key值都相同的情况，替换之前的值
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                //返回被替换的值
                return oldValue;
            }
        }

        modCount++;
        //③情况2.坑位没人,直接存值或发生hash碰撞都走这
        addEntry(hash, key, value, i);
        return null;
    }

~~~

先看上面key为空的情况(上面画图的时候总要在第一格留个空key的键值对)，执行 putForNullKey() 方法单独处理，会把该键值对放在index0，所以HashMap中是允许key为空的情况。再看下主流程：

步骤①.根据键值算出hash值 — > hash(key)

步骤②.根据hash值和当前数组的长度计算在数组中的索引 — > indexFor(hash, table.length)

~~~
    static int indexFor(int h, int length) {
        //hash值和数组长度-1按位与操作，听着费劲？其实相当于h%length;取余数(取模运算)
        //如：h = 17，length = 16;那么算出就是1
        //&运算的效率比%要高
        return h & (length-1);
    }

~~~

步骤③情况1.hash值和key值都相同，替换原来的值，并将被替换的值返回。

步骤③情况2.坑位没人或发生hash碰撞 — > addEntry(hash, key, value, i)

~~~
    void addEntry(int hash, K key, V value, int bucketIndex) {
        //当前hashmap中的键值对数量超过阀值
        if ((size >= threshold) && (null != table[bucketIndex])) {
            //扩容为原来的2倍
            resize(2 * table.length);
            hash = (null != key) ? hash(key) : 0;
            //计算在新表中的索引
            bucketIndex = indexFor(hash, table.length);
        }
        //创建节点
        createEntry(hash, key, value, bucketIndex);
    }

~~~

如果put的时候超过阀值，会调用 resize() 方法将数组大小扩大为原来的2倍，并且根据新表的长度计算在新表中的索引(如之前17%16 =1，现在17%32=17)，看下resize方法

~~~
    void resize(int newCapacity) { //传入新的容量
        //获取旧数组的引用
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        //极端情况，当前键值对数量已经达到最大
        if (oldCapacity == MAXIMUM_CAPACITY) {
            //修改阀值为最大直接返回
            threshold = Integer.MAX_VALUE;
            return;
        }
        //步骤①根据容量创建新的数组
        Entry[] newTable = new Entry[newCapacity];
        //步骤②将键值对转移到新的数组中
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        //步骤③将新数组的引用赋给table
        table = newTable;
        //步骤④修改阀值
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }

~~~

上面的重点是步骤②，看下它具体的转移操作

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

这段for循环的遍历会使得转移前后键值对的顺序颠倒(Java7和Java8的区别)，画个图就清楚了，假设石头的key值为5，盖伦的key值为37,这样扩容前后两者还是在5号坑。第一次：

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82a0bb93627?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第二次

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82a1940bdfb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

最后再看下创建节点的方法

~~~
    void createEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        size++;
    }

~~~

创建节点时，如果找到的这个坑里面没有存值，那么直接把值存进去就行了，然后size++；如果是碰撞的情况，

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82a2e50b9bb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

前面说的以单链表头插入的方式就是这样(盖伦：”老王已被我一脚踢开！“)，总结一下Java7 put流程图

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82a1be1474e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

相比put，get操作就没这么多套路，只需要根据key值计算hash值，和数组长度取模，然后就可以找到在数组中的位置(key为空同样单独操作)，接着就是遍历链表，源码很少就不分析了。

#### Java8源码分析

基本思路是一样的

~~~
//定义长度超过8的链表转化成红黑树
static final int TREEIFY_THRESHOLD = 8;
//换了个马甲还是认识你！！！
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
}

~~~

看下Java8 put的源码

~~~
public V put(K key, V value) {
    //根据key计算hash值
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //步骤1.数组为空或数组长度为0，则扩容(咦，看到不一样咯)
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //步骤2.根据hash值和数组长度计算在数组中的位置
        //如果"坑"里没人，直接创建Node并存值
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            //步骤3."坑"里有人，且hash值和key值都相等，先获取引用，后面会用来替换值
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //步骤4.该链是红黑树
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //步骤5.该链是链表
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        //步骤5.1注意这个地方跟Java7不一样，是插在链表尾部！！！
                        p.next = newNode(hash, key, value, null);
                        //链表长度超过8，转化成红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //步骤5.2链表中已存在且hash值和key值都相等，先获取引用，后面用来替换值
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    //统一替换原来的值
                    e.value = value;
                afterNodeAccess(e);
                //返回原来的值
                return oldValue;
            }
        }
        ++modCount;
        //步骤6.键值对数量超过阀值，扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

~~~

通过上面注释分析，对比和Java7的区别，Java8一视同仁，管你key为不为空的统一处理，多了一步链表长度的判断以及转红黑树的操作，并且比较重要的一点，新增Node是插在尾部而不是头部！！！。当然上面的主角还是扩容resize操作

~~~
final Node<K,V>[] resize() {
    //旧数组的引用
    Node<K,V>[] oldTab = table;
    //旧数组长度
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    //旧数组阀值
    int oldThr = threshold;
    //新数组长度、新阀值
    int newCap, newThr = 0;
    if (oldCap > 0) {
        //极端情况，旧数组爆满了
        if (oldCap >= MAXIMUM_CAPACITY) {
            //阀值改成最大，放弃治疗直接返回旧数组
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //扩容咯，这里采用左移运算左移1位，也就是旧数组*2
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            //同样新阀值也是旧阀值*2
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    //初始化在这里
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    //更新阀值
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        //创建新数组
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                //遍历旧数组，把原来的引用取消，方便垃圾回收
                oldTab[j] = null;
                //这个链只有一个节点，根据新数组长度计算在新表中的位置
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                //红黑树的处理
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                //链表长度大于1，小于8的情况，下面高能，单独拿出来分析
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}

~~~

可以看到，Java8把初始化数组和扩容全写在resize方法里了，但是思路还是一样的，扩容后要转移，转移要重新计算在新表中的位置，上面代码最后一块高能可能不太好理解，刚开始看的我一脸懵逼，看了一张[美团博客](https://link.juejin.im/?target=https%3A%2F%2Ftech.meituan.com%2Fjava-hashmap.html)的分析图才豁然开朗，在分析前先捋清楚思路

> 下面我们讲解下JDK1.8做了哪些优化。经过观测可以发现，我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。看下图可以明白这句话的意思，n为table的长度，图（a）表示扩容前的key1(5)和key2(21)两种key确定索引位置的示例，图（b）表示扩容后key1和key2两种key确定索引位置的示例，其中hash1是key1对应的哈希与高位运算结果。

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82a352d460e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 图a中key1(5)和key(21)计算出来的都是5，元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82a34f36480?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 图b中计算后key1(5)的位置还是5，而key2(21)已经变成了21，因此，我们在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”。

有了上面的分析再回来看下源码

~~~
else { // preserve order
    //定义两条链
    //原来的hash值新增的bit为0的链，头部和尾部
    Node<K,V> loHead = null, loTail = null;
    //原来的hash值新增的bit为1的链，头部和尾部
    Node<K,V> hiHead = null, hiTail = null;
    Node<K,V> next;
    //循环遍历出链条链
    do {
        next = e.next;
        if ((e.hash & oldCap) == 0) {
            if (loTail == null)
                loHead = e;
            else
                loTail.next = e;
            loTail = e;
        }
        else {
            if (hiTail == null)
                hiHead = e;
            else
                hiTail.next = e;
            hiTail = e;
        }
    } while ((e = next) != null);
    //扩容前后位置不变的链
    if (loTail != null) {
        loTail.next = null;
        newTab[j] = loHead;
    }
    //扩容后位置加上原数组长度的链
    if (hiTail != null) {
        hiTail.next = null;
        newTab[j + oldCap] = hiHead;
    }
}

~~~

为了更清晰明了，还是举个栗子，下面的表定义了键和它们的hash值(数组长度为16时，它们都在5号坑)

| Key | Hash |
| --- | --- |
| 石头 | 5 |
| 盖伦 | 5 |
| 蒙多 | 5 |
| 妖姬 | 21 |
| 狐狸 | 21 |
| 日女 | 21 |

假设一个hash算法刚好算出来的的存储是这样的，在存第13个元素时要扩容

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82a412462aa?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

那么流程应该是这样的(只关注5号坑键值对的情况)，第一次：

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82a4ca56ccf?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第二次：

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c8bf5c623e7c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

省略中间几次，第六次

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82a6547a175?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

两条链找出来后，最后转移一波，大功告成。

~~~
    //扩容前后位置不变的链
    if (loTail != null) {
        loTail.next = null;
        newTab[j] = loHead;
    }
    //扩容后位置加上原数组长度的链
    if (hiTail != null) {
        hiTail.next = null;
        newTab[j + oldCap] = hiHead;
    }

~~~

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82a6a7cdd97?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

总结下Java8 put流程图

![](https://user-gold-cdn.xitu.io/2017/12/3/1601c82a6d0623be?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 对比

1.发生hash冲突时，Java7会在链表头部插入，Java8会在链表尾部插入

2.扩容后转移数据，Java7转移前后链表顺序会倒置，Java8还是保持原来的顺序

3.关于性能对比可以参考[美团技术博客](https://link.juejin.im/?target=https%3A%2F%2Ftech.meituan.com%2Fjava-hashmap.html)，引入红黑树的Java8大程度得优化了HashMap的性能

### 感谢

[讲的很详细的外国小哥](https://link.juejin.im/?target=http%3A%2F%2Fjavabypatel.blogspot.ca%2F)

[美团技术博客](https://link.juejin.im/?target=https%3A%2F%2Ftech.meituan.com%2Fjava-hashmap.html)



