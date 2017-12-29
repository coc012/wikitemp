>文档当前状态：**beta0.8**
>大部分内容整理自：[Exploring Kotlin’s hidden costs](https://medium.com/@BladeCoder/exploring-kotlins-hidden-costs-part-1-fbb9935d9b62)
，更多见文末的 [参考引文](#参考引文)。

[TOC]

---


>通过对[《Kotlin 官方参考文档 中文》](https://www.kotlincn.net/docs/reference/) 的学习，已经能够在项目，使用kotlin进行开发了。kotlin提供的一些 “语法糖”特性，能很大程度上改善java 沉冗的语法，提高编码效率，但这些的背后是如何实现的、对代码运行性能又有怎么样的影响，所知并不多，下文会通过对部分kotli特性的进一步说明，来帮助我们在 更有效的使用kotlin。



需要说明的是，相较于java，文中提及到kotlin 语法糖会带来的一些额外成本：
* 基本类型的装箱、拆箱
* 分配短期对象
* 生成额外的方法

这些在大部分场景下，并不会对性能造成很大的影响，但涉及到一些高频调用的场景下，就需要我们慎重对待。

### **空值安全**

Kotlin 语言中最好的特性之一就是明确区分了可空与不可空类型。这可以使编译器在运行时通过禁止任何代码将 null 或者可空值分配给不可空变量来有效地阻止意想不到的 NullPointerException。

#### **不可空参数运行时检查**

>**Tips: 在生产环境下，可以在混淆规则中 优化kotlin的 空值检查**

让我们声明一个公共的接收一个不可空 String 做为参数的函数：
```
fun sayHello(who: String) {
    println("Hello $who")
}
```
现在看一下编译之后的等同的 Java 代码：
```
public static final void sayHello(@NotNull String who) {
   Intrinsics.checkParameterIsNotNull(who, "who");
   String var1 = "Hello " + who;
   System.out.println(var1);
}
```
注意，Kotlin 编译器是 Java 的好公民，它在参数上添加了一个 @NotNull 注解，因此当一个 null 值传过来的时候 Java 工具可以据此来显示一个警告。

但对于外部调用者，一个注解并不足以实现空值安全。这就是为什么编译器在函数的刚开始处还添加了一个可以检测参数并且如果参数为 null 就抛出 IllegalArgumentException 的静态方法调用。为了使不安全的调用代码更容易修复，这个函数会快速失败而不是在后期随机地抛出 NullPointerException。

在实践中，每一个公共的函数都会在每一个不可空引用参数上添加一个 Intrinsics.checkParameterIsNotNull() 静态调用。私有函数不会有这些检查，因为编译器会保证 Kotlin 类中的代码是空值安全的。

这些静态调用对性能的影响可以忽略不计，并且他们在调试或者测试一个 app 时确实很有用。话虽这么说，但你还是可能将他们视为一种正式版本中不必要的额外成本。在这种情况下，可以通过使用编译器选项中的 -Xno-param-assertions 或者添加以下的混淆规则来禁用运行时空值检查：
```
-assumenosideeffects class kotlin.jvm.internal.Intrinsics {
    static void checkParameterIsNotNull(java.lang.Object, java.lang.String);
}
```
注意，这条混淆规则只有在优化功能开启的时候有效。优化功能在默认的安卓混淆配置中是禁用的。

### **装箱、拆箱**

#### **可空的基本类型**
> **Tips: 为了更好的可读性和更佳的性能尽量使用不可空基础类型。**

可空类型都是引用类型。将基础类型变量声明为 可空的话，会阻止 Kotlin 使用 Java 中类似 int或者 float 那样的基础类型，相应的类似 Integer 或者 Float 那样的装箱引用类型会被使用，这就引起了额外的装箱或拆箱成本。

此外，当使用可空类型，Kotlin编译器会迫使你编写安全的代码：
```
fun add(a: Int, b: Int): Int {
    return a + b
}
fun add(a: Int?, b: Int?): Int {
    return (a ?: 0) + (b ?: 0)
}
```
为了更好的可读性和更佳的性能,尽量使用不可空基础类型。

#### **基础类型的 泛型使用**
>**Tips: 涉及基本类型相关的操作时，优先使用指定类型的方法，而不是泛型。**

Kotlin 中有三种数组类型：
* IntArray, FloatArray 还有其他的：基础类型数组。编译为 int[], float[] 和其他的类型。
* Array<T>：不可空对象引用类型化数组，这涉及到对基础类型的装箱。
* Array<T?>：可空对象引用类型化数组。很明显，这也涉及到基础类型的装箱。
如果你需要一个不可空的基础类型数组，最好用 IntArray 而不是 Array<Int> 来避免隐式的拆/装箱。



### **可变参数**
>Tips: 如果 频繁调用的代码中 含有 可变参数，考虑使用 一个真正的数组 替代它


Kotlin 允许声明具有数量可变的参数的函数，就像 Java 那样。声明语法有点不一样：

```
fun printDouble(vararg values: Int) {
    values.forEach { println(it * 2) }
}
```
就像 Java 中那样，vararg 参数实际上被编译为一个给定类型的 数组 参数。你可以用三种不同的方式来调用这些函数：

#### 1. 传入多个参数
```
printDouble(1, 2, 3)
```
Kotlin 编译器会将这行代码转化为创建并初始化一个新的数组，和 Java 编译器做的完全一样：
```
printDouble(new int[]{1, 2, 3});
```
因此有创建一个新数组的开销，但与 Java 相比这并不是什么新鲜事。

#### 2. 传入一个单独的数组

这就是不同之处。在 Java 中，你可以直接传入一个现有的数组引用作为可变参数。但是在 Kotlin 中你需要使用 分布操作符:
```
val values = intArrayOf(1, 2, 3)
printDouble(*values)
```
在 Java 中，数组引用被“原样”传入函数，而无需分配额外的数组内存。然而，分布操作符编译的方式不同，正如你在（等同的）Java 代码中看到的：
```
int[] values = new int[]{1, 2, 3};
printDouble(Arrays.copyOf(values, values.length));
```
每当调用这个函数时，现在的数组总会被复制。好处是代码更安全：允许函数在不影响调用者代码的情况下修改这个数组。但是会分配额外的内存。

注意，在 Kotlin 代码中调用一个有可变参数的 Java 方法会产生相同的效果。

#### 3. 传入混合的数组和参数

分布操作符主要的好处是，它还允许在同一个调用中数组参数和其他参数混合在一起进行传递。
```
val values = intArrayOf(1, 2, 3)
printDouble(0, *values, 42)
```
这是如何编译的呢？生成的代码十分有意思：
```
int[] values = new int[]{1, 2, 3};
IntSpreadBuilder var10000 = new IntSpreadBuilder(3);
var10000.add(0);
var10000.addSpread(values);
var10000.add(42);
printDouble(var10000.toArray());
```
除了创建新数组外，一个临时的 builder 对象被用来计算最终的数组大小并填充它。这就使得这个方法调用又增加了另一个小的成本。

> **在 Kotlin 中调用一个具有可变参数的函数时会增加创建一个新临时数组的成本，即使是使用已有数组的值。对方法被反复调用的性能关键性的代码来说，考虑添加一个以真正的数组而不是 可变数组 为参数的方法。**


### **Lambda 表达式**
#### **捕获、非捕获的Lambda**
>**Tips：使用lambda表达式时，在保证简洁的前提下，优先考虑写成非捕获**


什么是“捕获和非捕获的Lambda表达式”？（参考自[Java 8 的新特性和改进总览](http://www.oschina.net/translate/everything-about-java-8) ）​
当Lambda表达式访问一个定义在Lambda表达式体外的非静态变量或者对象时，这个Lambda表达式称为“捕获的”。比如，下面这个lambda表达式捕捉了变量x：
```
int x = 5; return y -> x + y;
```
为了保证这个lambda表达式声明是正确的，被它捕获的变量必须是“有效final”的。所以要么它们需要用final修饰符号标记，要么保证它们在赋值后不能被改变。Lambda表达式是否是捕获的和性能悄然相关。一个非不捕获的lambda通常比捕获的更高效，虽然这一点没有书面的规范说明（据我所知），而且也不能为了程序的正确性指望它做什么，**非捕获的lambda只需要计算一次. 然后每次使用到它都会返回一个唯一的实例。而捕获的lambda表达式每次使用时都需要重新计算一次**，而且从目前实现来看，它很像实例化一个匿名内部类的实例。,

当然，我们并不说 一定要写成 非捕获的，在某些场景，相比“刻意”非捕获，捕获式的性能 也不是那么难看。

在这个工程中，我们使用了大量的Lambda表达式来实现回调处理。然而在这些使用Lambda实现的回调中很多并没有捕获局部变量，而是需要引用当前类的变量或者调用当前类的方法。然而目前仍需要对象分配。下面就是我们提到的例子的代码：
```
public MessageProcessor() {} 

public int processMessages() {
    return queue.read(obj -> {
        if (obj instanceof NewClient) {
            this.processNewClient((NewClient) obj);
        } 
        ...
    });
}
```
有一个简单的办法解决这个问题，我们将Lambda表达式的代码提前到构造方法中，并将其赋值给一个成员属性。在调用点我们直接引用这个属性即可。下面就是修改后的代码：

```
private final Consumer<Msg> handler; 

public MessageProcessor() {
    handler = obj -> {
        if (obj instanceof NewClient) {
            this.processNewClient((NewClient) obj);
        }
        ...
    };
} 

public int processMessages() {
    return queue.read(handler);
}
```

然而上面的“优化后”后代码给却给整个工程带来了一个严重的问题：性能分析表明，这种修改产生很大的对象申请，其产生的内存申请在总应用的60%以上。
类似这种无关上下文的优化可能带来其他问题。

1.  纯粹为了优化的目的，使用了非惯用的代码写法，可读性会稍差一些。
2. 内存分配方面的问题，示例中为MessageProcessor增加了一个成员属性，使得MessageProcessor对象需要申请更大的内存空间。Lambda表达式的创建和捕获位于构造方式中，使得MessageProcessor的构造方法调用缓慢一些。


我们遇到这种情况，需要进行内存分析，结合合理的业务用例来进行优化。有些情况下，我们使用成员属性确保为经常调用的Lambda表达式只申请一个对象，这样的缓存策略大有裨益。任何性能调优的科学的方法都可以进行尝试。

上述的方法也是其他程序员对Lambda表达式进行优化应该使用的。书写整洁，简单，函数式的代码永远是第一步。任何优化，如上面的提前代码作为成员属性，都必须结合真实的具体问题进行处理。变量捕获并申请对象的Lambda表达式并非不好，就像我们我们写出new Foo()代码并非一无是处一样。

链接中有这方面更详细的说明—— [深入探索Java 8 Lambda表达式](http://www.infoq.com/cn/articles/Java-8-Lambdas-A-Peek-Under-the-Hood)


### **伴生对象**
Kotlin 类没有静态变量和方法。相应的，类中与实例无关的字段和方法可以通过**伴生对象**或**包级属性**来声明。

#### **kotlin中的 “static 全局变量(伴生对象实现)**
>**Tips: 始终用 const 关键字来声明基本类型和字符串常量**



在 Kotlin 中你通常在一个伴生对象中声明在类中使用的“静态”常量。
```
class MyClass {

    companion object {
        private val TAG = "TAG"
    }

    fun helloWorld() {
        println(TAG)
    }
}
```

这段代码看起来干净整洁，但是幕后发生的事情却十分不堪。

基于上述原因，访问一个在伴生对象中声明为私有的常量实际上会在这个伴生对象的实现类中生成一个额外的、合成的 getter 方法。
```
GETSTATIC be/myapplication/MyClass.Companion : Lbe/myapplication/MyClass$Companion;
INVOKESTATIC be/myapplication/MyClass$Companion.access$getTAG$p (Lbe/myapplication/MyClass$Companion;)Ljava/lang/String;
ASTORE 1
```
但是更糟的是，这个合成方法实际上并没有返回值；它调用了 Kotlin 生成的实例 getter 方法：
```
ALOAD 0
INVOKESPECIAL be/myapplication/MyClass$Companion.getTAG ()Ljava/lang/String;
ARETURN
```
当常量被声明为 public 而不是 private 时，getter 方法是公共的并且可以被直接调用，因此不需要上一步的方法。但是 Kotlin 仍然必须通过调用 getter 方法来访问常量。

所以我们真的解决了问题吗？并没有！事实证明，为了存储常量值，Kotlin 编译器实际上在主类级别上而不是伴生对象中生成了一个 private static final 字段。但是，因为在类中静态字段被声明为私有的，在伴生对象中需要有另外一个合成方法来访问它
```
INVOKESTATIC be/myapplication/MyClass.access$getTAG$cp ()Ljava/lang/String;
ARETURN
```
最终，那个合成方法读取实际值：
```
GETSTATIC be/myapplication/MyClass.TAG : Ljava/lang/String;
ARETURN
```
换句话说，当你从一个 Kotlin 类来访问一个伴生对象中的私有常量字段的时候，与 Java 直接读取一个静态字段不同，你的代码实际上会：

* 在伴生对象上调用一个静态方法，
* 然后在伴生对象上调用实例方法，
* 然后在类中调用静态方法，
* 读取静态字段然后返回它的值。

这是等同的 Java 代码：
```
public final class MyClass {
    private static final String TAG = "TAG";
    public static final Companion companion = new Companion();

    // synthetic
    public static final String access$getTAG$cp() {
        return TAG;
    }

    public static final class Companion {
        private final String getTAG() {
            return MyClass.access$getTAG$cp();
        }

        // synthetic
        public static final String access$getTAG$p(Companion c) {
            return c.getTAG();
        }
    }

    public final void helloWorld() {
        System.out.println(Companion.access$getTAG$p(companion));
    }
}
```
如何改善？

首先，通过 const 关键字声明值为编译时常量来完全避免任何的方法调用是有可能的。这将有效地在调用代码中直接内联这个值，但是只有基本类型和字符串才能如此使用。
```
class MyClass {

    companion object {
        private const val TAG = "TAG"
    }

    fun helloWorld() {
        println(TAG)
    }
}
```
第二，你可以在伴生对象的公共字段上使用 @JvmField 注解来告诉编译器不要生成任何的 getter 和 setter 方法，就像纯 Java 中的常量一样做为类的一个静态变量暴露出来。实际上，这个注解只是单独为了兼容 Java 而创建的，如果你的常量不需要从 Java 代码中访问的话，我是一点也不推荐你用一个晦涩的交互注解来弄乱你漂亮 Kotlin 代码的。此外，它只能用于公共字段。在 Android 的开发环境中，你可能只在实现 Parcelable 对象的时候才会使用这个注解：
```
class MyClass() : Parcelable {

    companion object {
        @JvmField
        val CREATOR = creator { MyClass(it) }
    }

    private constructor(parcel: Parcel) : this()

    override fun writeToParcel(dest: Parcel, flags: Int) {}

    override fun describeContents() = 0
}
```
最后，你也可以用 ProGuard 工具来优化字节码，希望通过这种方式来合并这些链式方法调用，但是绝对不保证这是有效的。

>* 与 Java 相比，在 Kotlin 中从伴生对象里读取一个 static 常量会增加 2 到 3 个额外的间接级别并且每一个常量都会生成 2 到 3个方法。
>* 始终用 const 关键字来声明基本类型和字符串常量从而避免这些（成本）
>* 对其他类型的常量来说，你不能使用const，因此如果你需要反复访问这个常量的话，你或许可以把它的值缓存在一个本地变量中。同时，最好在它们自己的对象而不是伴生对象中来存储公共的全局常量。

#### kotlin中的**static属性**：使用**伴生对象** 还是 **包级属性**？
这里并不打算给出选择 哪个更好的建议，你需要了解两者的幕后实现后，结合具体的代码进行选择。

这里我们使用相同的 例子来说明：
在java中 ，代码是这样：
```
class MyClass {
   private static final String TAG = "TAG";

   public void helloWorld(){
   		System.out.println(TAG );
  	}
}
```

再来看看其他写法
写法1：包级属性的方法：
```
private const  val TAG = "TAG"
class MyClass {
    fun helloWorld() {
        println(TAG)
    }
}
```
写法1：decompile后像这样
```
//额外的一个类
public final class MyClassKt {
   private static final String TAG = "TAG";
}
public final class MyClass {
   public final void helloWorld() {
      String var1 = "TAG";
      System.out.println(var1);
   }
}

```

写法2：写成包级属性，但不写成 const
```
private val TAG = "TAG"
class MyClass {
    fun helloWorld() {
        println(TAG)
    }
}
```
写法2：decompile后像这样
```
//额外的一个类
public final class MyClassKt {
   private static final String TAG = "TAG";
   // $FF: synthetic method
   @NotNull
   public static final String access$getTAG$p() {
      return TAG;
   }
}

public final class MyClass {
   public final void helloWorld() {
      String var1 = MyClassKt.access$getTAG$p();
      System.out.println(var1);
   }
}

```
写法3：将全局常量值 写成自己类的成员属性
```
class MyClass {
    //无法使用const
    private val TAG = "TAG"
    fun helloWorld() {
        println(TAG)
    }
}
```
写法3：decompile后像这样

```
public final class MyClass {
   private final String TAG = "TAG";

   public final void helloWorld() {
      String var1 = this.TAG;
      System.out.println(var1);
   }
}
```
了解多种写法的幕后实现后，根据实际情况选择不用的 写法

### **委托属性**
#### **泛型委托**
>Tips: **对应基本类型的委托，最好使用具体类型 来避免 装/拆箱**

委托方法也可以被声明成泛型的，这样一来不同类型的属性就可以复用同一个委托类了。
```
private var maxDelay: Long by SharedPreferencesDelegate<Long>()
```
然而，如果像上例那样对基本类型使用泛型委托的话，即便声明的基本类型非空，也会在每次读写属性的时候触发装箱和拆箱的操作。

对于非空基本类型的委托属性来说，最好使用给定类型的特定委托类而不是泛型委托来避免每次访问属性时增加装箱的额外开销。
#### **lazy的线程安全**
>**Tips: 使用lazy()时，考虑指定LazyThreadSafetyMode.NONE**

针对常见情形，Kotlin 提供了一些标准委托，如 Delegates.notNull()、 Delegates.observable() 和 lazy()。

lazy() 是一个在第一次读取时通过给定的 lambda 值来计算属性的初值，并返回只读属性的委托。
```
private val dateFormat: DateFormat by lazy {
    SimpleDateFormat("dd-MM-yyyy", Locale.getDefault())
}
```

这是一种简洁的延迟高消耗的初始化至其真正需要时的方式，在保留代码可读性的同时提升了性能。

需要注意的是，lazy() 并不是内联函数，传入的 lambda 参数也会被编译成一个额外的 Function 类，并且不会被内联到返回的委托对象中。

经常被忽略的一点是 lazy() 有可选的 mode 参数 来决定应该返回 3 种委托的哪一种：
```
public fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer)
public fun <T> lazy(mode: LazyThreadSafetyMode, initializer: () -> T): Lazy<T> =
        when (mode) {
            LazyThreadSafetyMode.SYNCHRONIZED -> SynchronizedLazyImpl(initializer)
            LazyThreadSafetyMode.PUBLICATION -> SafePublicationLazyImpl(initializer)
            LazyThreadSafetyMode.NONE -> UnsafeLazyImpl(initializer)
        }
```

默认模式 LazyThreadSafetyMode.SYNCHRONIZED 将提供相对耗费昂贵的 双重检查锁 来保证一旦属性可以从多线程读取时初始化块可以安全地执行。

如果你确信属性只会在单线程（如主线程）被访问，那么可以选择 LazyThreadSafetyMode.NONE 来代替，从而避免使用锁的额外开销。
```
val dateFormat: DateFormat by lazy(LazyThreadSafetyMode.NONE) {
    SimpleDateFormat("dd-MM-yyyy", Locale.getDefault())
}
```
>**使用 lazy() 委托来延迟初始化时的大量开销以及指定模式来避免不必要的锁。**


### **区间**

#### **基本类型区间—if 包含**
>**Tips: if中 使用区间表达式(基本类型)，尽量显式声明而不是使用 间接方法返回的区间；**


区间表达式的主要作用是使用 in 和 !in 操作符实现包含和不包含。
```
if (i in 1..10) {
    println(i)
}
```
该实现针对非空基本类型的区间（包括 Int、Long、Byte、Short、Float、Double 以及 Char 的值）实现了优化，所以上面的代码可以被优化成这样：
```
if(1 <= i && i <= 10) {
   System.out.println(i);
}
```
零额外支出并且没有额外对象开销。区间也可以被包含在 when 表达式中：
```
val message = when (statusCode) {
    in 200..299 -> "OK"
    in 300..399 -> "Find it somewhere else"
    else -> "Oops"
}
```
相比一系列的 if{...} else if{...} 代码块，这段代码在不降低效率的同时提高了代码的可读性。

然而，如果在声明和使用之间有至少一次间接调用的话，range 会有一些微小的额外开销。比如下面的代码：
```
private val myRange get() = 1..10

fun rangeTest(i: Int) {
    if (i in myRange) {
        println(i)
    }
}
```
在编译后会创建一个额外的 IntRange 对象：
```
private final IntRange getMyRange() {
   return new IntRange(1, 10);
}

public final void rangeTest(int i) {
   if(this.getMyRange().contains(i)) {
      System.out.println(i);
   }
}
```
将属性的 getter 声明为 inline 的方法也无法避免这个对象的创建。

#### **基本类型区间—for遍历**
>**Tips: for 中使用基本类型的区间的 优先考虑..或downTo()**

整型区间 （除了 Float 和 Double之外其他的基本类型）也是 级数：它们可以被迭代。这就可以将经典 Java 的 for 循环用一个更短的表达式替代。

```
for (i in 1..10) {
    println(i)
}
```
经过编译器优化后的代码实现了零额外开销：
```
int i = 1;
byte var3 = 10;
if(i <= var3) {
   while(true) {
      System.out.println(i);
      if(i == var3) {
         break;
      }
      ++i;
   }
}
```
如果要反向迭代，可以使用 downTo() 中缀方法来代替 ..：
```
for (i in 10 downTo 1) {
    println(i)
}
```
编译之后，这也实现了零额外开销：
```
int i = 10;
byte var3 = 1;
if(i >= var3) {
   while(true) {
      System.out.println(i);
      if(i == var3) {
         break;
      }
      --i;
   }
}
```
然而，其他迭代器参数并没有如此好的优化。

反向迭代还有一种结果相同的方式，使用 reversed() 方法结合区间：
```
for (i in (1..10).reversed()) {
    println(i)
}
```
编译后的代码并没有看起来那么少：
```
IntProgression var10000 = RangesKt.reversed((IntProgression)(new IntRange(1, 10)));
int i = var10000.getFirst();
int var3 = var10000.getLast();
int var4 = var10000.getStep();
if(var4 > 0) {
   if(i > var3) {
      return;
   }
} else if(i < var3) {
   return;
}

while(true) {
   System.out.println(i);
   if(i == var3) {
      return;
   }

   i += var4;
}
```
会创建一个临时的 IntRange 对象来代表区间，然后创建另一个 IntProgression 对象来反转前者的值。

事实上，任何结合不止一个方法来创建递进都会生成类似的至少创建两个微小递进对象的代码。

这个规则也适用于使用 step() 中缀方法来操作递进的步骤，即使只有一步：
```
for (i in 1..10 step 2) {
    println(i)
}
```
一个次要提示，当生成的代码读取 IntProgression 的 last 属性时会通过对边界和步长的小小计算来决定准确的最后值。在上面的代码中，最终值是 9。

最后，until() 中缀函数对于迭代也很有用，该函数（执行结果）不包含最大值。
```
for (i in 0 until size) {
    println(i)
}
```
遗憾的是，编译器并没有针对这个经典的包含区间围优化，迭代器依然会创建区间对象：
```
IntRange var10000 = RangesKt.until(0, size);
int i = var10000.getFirst();
int var1 = var10000.getLast();
if(i <= var1) {
   while(true) {
      System.out.println(i);
      if(i == var1) {
         break;
      }
      ++i;
   }
}
```
不知道后续版本的kotlin编译器会不会对此进行优化。当前可以通过这样写来优化代码：
```
for (i in 0..size - 1) {
    println(i)
}
```
#### **非基本类型 区间-if**
>**Tips：对实现了 Comparable 的非基本类型的区间进行频繁的if包含判断，考虑这个区间声明为常量来避免重复创建区间类**

除了基本类型外，区间也可以用于其他实现了 Comparable 的非基本类型。
如下：
```
if (name in "Alfred".."Alicia") {
    println(name)
}
```
在这种情况下，最终实现并不会优化，而且总是会创建一个 ClosedRange 对象，如下面编译后的代码所示：
```
if(RangesKt.rangeTo((Comparable)"Alfred", (Comparable)"Alicia")
   .contains((Comparable)name)) {
   System.out.println(name);
}
```

#### **迭代区间for 还是 forEach()**
>**Tips:迭代区间时，最好只使用 for 循环而不是区间上的 forEach() 方法。**

作为 for 循环的替代，使用区间内联的扩展方法 forEach() 来实现相似的效果可能更吸引人。
```
(1..10).forEach {
    println(it)
}
```
但如果仔细观察这里使用的 forEach() 方法签名的话，你就会注意到并没有优化区间，而只是优化了 Iterable，所以需要创建一个 iterator。下面是编译后代码的 Java 形式：
```
Iterable $receiver$iv = (Iterable)(new IntRange(1, 10));
Iterator var1 = $receiver$iv.iterator();

while(var1.hasNext()) {
   int element$iv = ((IntIterator)var1).nextInt();
   System.out.println(element$iv);
}
```
这段代码相比前者更为低效，原因是为了创建一个 IntRange 对象，还需要额外创建 IntIterator。但至少它还是生成了基本类型的值。

#### **迭代集合：indices**
>**Tips: 除了迭代 数组和标准集合时，可以使用indices，其他类迭代 考虑 使用常规区间**

Kotlin 标准库提供了内置的 indices 扩展属性来生成数组和 Collection 的区间。
```
val list = listOf("A", "B", "C")
for (i in list.indices) {
    println(list[i])
}
```
令人惊讶的是，对这个 indices 的迭代得到了编译器的优化：
```
List list = CollectionsKt.listOf(new String[]{"A", "B", "C"});
int i = 0;
int var2 = ((Collection)list).size() - 1;
if(i <= var2) {
   while(true) {
      Object var3 = list.get(i);
      System.out.println(var3);
      if(i == var2) {
         break;
      }
      ++i;
   }
}
```
从上面的代码中我们可以看到没有创建 IntRange 对象，列表的迭代是以最高效率的方式运行的，这适用于数组和实现了 Collection 的类。

看到这里 你可能会尝试在特定的类上使用自己的 indices 扩展属性，来获得相同的迭代器性能的。
```
inline val SparseArray<*>.indices: IntRange
    get() = 0..size() - 1

fun printValues(map: SparseArray<String>) {
    for (i in map.indices) {
        println(map.valueAt(i))
    }
}
```
但编译之后，我们可以发现这并没有那么高效率，因为编译器无法足够智能地避免区间对象的产生：
```
public static final void printValues(@NotNull SparseArray map) {
   Intrinsics.checkParameterIsNotNull(map, "map");
   IntRange var10002 = new IntRange(0, map.size() - 1);
   int i = var10002.getFirst();
   int var2 = var10002.getLast();
   if(i <= var2) {
      while(true) {
         Object $receiver$iv = map.valueAt(i);
         System.out.println($receiver$iv);
         if(i == var2) {
            break;
         }
         ++i;
      }
   }
}
```
所以，可以结合实际场景 进行编码，比如这里 推荐 常规写法 进行遍历：
(注：原文使用until,但基于上面的论述，使用.. 生成迭代区间更好)
```
fun printValues(map: SparseArray<String>) {
    for (i in 0 .. map.size()-1) {
        println(map.valueAt(i))
    }
}
```

### **高阶函数**
>**Tips: 如果一个 含有捕获lamada高阶函数 调用频繁，考虑内联它。**

Kotlin 支持将函数赋值给变量并将他们做为参数传给其他函数。接收其他函数做为参数的函数被称为高阶函数。Kotlin 函数可以通过在他的名字前面加 :: 前缀来引用，或者在代码中中直接声明为一个匿名函数，或者使用最简洁的 lambda 表达式语法 来描述一个函数。

Kotlin 是为 Java 6/7 JVM 和 Android 提供 lambda 支持的最好方法之一。
考虑下面的工具函数，在一个数据库事务中执行任意操作并返回受影响的行数：
```
fun transaction(db: Database, body: (Database) -> Int): Int {
    db.beginTransaction()
    try {
        val result = body(db)
        db.setTransactionSuccessful()
        return result
    } finally {
        db.endTransaction()
    }
}
```

我们可以通过传递一个 lambda 表达式做为最后的参数来调用这个函数：
```
val deletedRows = transaction(db) {
    it.delete("Customers", null, null)
}
```
他们是如何转化为字节码的呢？，lambdas 和匿名函数被编译成 Function 对象。


这是上面的的 lamdba 表达式编译之后的 Java 表现形式。
```
class MyClass$myMethod$1 implements Function1 {
   // $FF: synthetic method
   // $FF: bridge method
   public Object invoke(Object var1) {
      return Integer.valueOf(this.invoke((Database)var1));
   }

   public final int invoke(@NotNull Database it) {
      Intrinsics.checkParameterIsNotNull(it, "it");
      return db.delete("Customers", null, null);
   }
}
```
在你的 Android dex 文件中，每一个 lambda 表达式都被编译成一个 Function，这将最终增加3到4个方法。

好消息是，这些 Function 对象的新实例只在必要的时候才创建。在实践中，这意味着：

* 对捕获表达式来说，每当一个 lambda 做为参数传递的时候都会生成一个新的 Function 实例，执行完后就会进行垃圾回收。
* 对非捕获表达式（纯函数）来说，会创建一个单例的 Function 实例并且在下次调用的时候重用。

由于我们示例中的调用代码使用了一个非捕获的 lambda，因此它被编译为一个单例而不是内部类：
```
this.transaction(db, (Function1)MyClass$myMethod$1.INSTANCE);
```
避免反复调用那些正在调用捕获 lambdas的标准的（非内联）高阶函数以减少垃圾回收器的压力。

此外，还需要考虑装箱的开销

与 Java8 大约有43个不同的专业方法接口来尽可能地避免装箱和拆箱相反，Kotnlin 编译出来的 Function 对象只实现了完全通用的接口，有效地使用任何输入输出值的 Object 类型。
```
/** A function that takes 1 argument. */
public interface Function1<in P1, out R> : Function<R> {
    /** Invokes the function with the specified argument. */
    public operator fun invoke(p1: P1): R
}
```
这意味着调用一个做为参数传递给高阶函数的方法时，如果输入值或者返回值涉及到基本类型（如 Int 或 Long），实际上调用了系统的装箱和拆箱。这在性能上可能有着不容忽视的影响，特别是在 Android 上。

在上面编译好的 lambda 中，你可以看到结果被装箱成了 Integer 对象。然后调用者代码马上将其拆箱。

>当写一个标准（非内联）的高阶函数（涉及到以基本类型做为输入或输出值的函数做为参数）时要小心一点。反复调用这个参数函数会由于装箱和拆箱的操作对垃圾回收器造成更多压力。


幸好，使用 lambda 表达式时，Kotlin 有一个非常棒的技巧来避免这些成本：将高阶函数声明为内联。这将会使编译器将函数体直接内联到调用代码内，完全避免了方法调用。对高阶函数来说好处更大，因为作为参数传递的 lambda 表达式的函数体也会被内联起来。实际的影响有：

* 声明 lambda 时不会有 Function 对象被实例化；
* 不需要针对 lambda 输入输出的基本类型值进行装箱和拆箱；
* 方法总数不会增加；
* 不会执行真正的函数调用。对那些多次被使用的注重 CPU （计算）的方法来说可以提高性能。
* 
将我们的 transaction() 函数声明为内联后，调用代码变成了：
```
db.beginTransaction();
int var5;
try {
   int result$iv = db.delete("Customers", null, null);
   db.setTransactionSuccessful();
   var5 = result$iv;
} finally {
   db.endTransaction();
}
```
关于这个杀手锏特性的一些限制：

* 内联函数不能直接调用自己，也不能通过其他内联函数来调用；
* 一个类中被声明为公共的内联函数只能访问这个类中公共的方法和成员变量；
* 代码量会增加。多次内联一个长函数会使生成的代码量明显增多，尤其这个长方法又引用了另外一个长的内联方法。

>**如果可能的话，就将一个高阶函数声明为内联。保持简短，如有必要可以将大段的代码块移至非内联的方法中。你还可以将调用自代码中影响性能的关键部分的函数内联起来。


### 更多内容
局部函数 (暂时没有写过，待后续跟进)

### 参考引文
[](https://github.com/xitu/gold-miner/blob/master/TODO/exploring-kotlins-hidden-costs-part-3.md)
* [Exploring Kotlin’s hidden costs — Part 1](https://medium.com/@BladeCoder/exploring-kotlins-hidden-costs-part-1-fbb9935d9b62)
* [Exploring Kotlin’s hidden costs — Part 2](https://medium.com/@BladeCoder/exploring-kotlins-hidden-costs-part-2-324a4a50b70)
* [Exploring Kotlin’s hidden costs — Part 3](https://medium.com/@BladeCoder/exploring-kotlins-hidden-costs-part-3-3bf6e0dbf0a4)
* [深入探索Java 8 Lambda表达式](http://www.infoq.com/cn/articles/Java-8-Lambdas-A-Peek-Under-the-Hood)
* [Java 8 的新特性和改进总览](http://www.oschina.net/translate/everything-about-java-8)