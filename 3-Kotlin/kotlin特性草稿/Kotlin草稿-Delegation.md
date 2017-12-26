抬头
---
属性相关
* var、val
* 默认实现的get、set
* 幕后字段
* 常量放在那里
* lateinit
* **委托**
---
# 代理模式

在了解Delegation之前，有必要先复习一下代理模式，回顾一下它的使用场景。
不清楚的读者可以移步[这篇文章](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fitachi85%2Farticle%2Fdetails%2F50912632)。
这里我要重点引用这篇文章对于应用场景的总结：

> 远程代理：为一个对象在不同的地址空间提供局部代表，这样系统可以将Server部分的事项隐藏。
> 虚拟代理：使用一个代理对象表示一个十分耗资源的对象并在真正需要时才创建。
> 安全代理：用来控制真实对象访问时的权限。
> 智能指引：当调用真实的对象时，代理处理另外一些事，比如计算真实对象的引用计数，当该对象没有引用时，可以自动释放它；或者访问一个实际对象时，检查是否已经能够锁定它，以确保其他对象不能改变它。

# [](https://juejin.im/entry/5a1a10a96fb9a0450e75d286?utm_source=gold_browser_extension#Class-Delegation "Class Delegation")Class Delegation

官方文档给了我们这样一个例子：

> ~~~
> >interface Base {
> >    fun print()
> >}
> >class BaseImpl(val x: Int) : Base {
> >    override fun print() { print(x) }
> >}
> >class Derived(b: Base) : Base by b
> >fun main(args: Array<String>) {
> >    val b = BaseImpl(10)
> >    Derived(b).print() // prints 10
> >}
> >
> 
> ~~~

这里通过短句by确定了b这个动态代理，b作为Derived类的对象，编译器会为它生成所有Base的接口方法。
然后在真正需要代理的时候，把被代理的类的实例作为参数来实例化代理类，然后调用接口方法，则可以实现动态代理。
这个显然比Java利用反射来实现代理要方便得多。

# [](https://juejin.im/entry/5a1a10a96fb9a0450e75d286?utm_source=gold_browser_extension#Delegated-Properties "Delegated Properties")Delegated Properties

有一些属性，我们每次需要的时候可以实现他们，但是有种方法可以只需要实现一次。有这样几个场景：

*   lazy properties: 只有第一次访问时才需要计算的属性
*   observable properties: 监听变化的属性
*   把属性存在一个map里，而不是分散的feild
    为了满足以上需求，Kotlin推出了代理属性delegate properties:

    ~~~
    class Example {
        var p: String by Delegate()
    }

    ~~~

与成员p对应的get和set方法都会被Delegate的get和set方法所代理。Delegate不需要实现什么接口，但是必须提供get方法，如果是var类型的，还必须提供set方法。eg：

~~~
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name} in $thisRef.'")
    }
}

~~~

现在属性p就变成了Delegate的一个实例，读取p就会调用Delegate的getValue方法，第一个参数表示代理p属性所在的类的实例，第二个参数则是p属性本身。例如：

~~~
val e = Example()
println(e.p) // will print "Example@33a17727, thank you for delegating ‘p’ to me!"
e.p = "NEW" // will print "NEW has been assigned to ‘p’ in Example@33a17727."

~~~

值得注意的是，getValue和setValue方法前必须添加operater，这是因为Delegate类实际是实现了系统标准库的接口，所以必须保持一致。

## [](https://juejin.im/entry/5a1a10a96fb9a0450e75d286?utm_source=gold_browser_extension#标准代理库 "标准代理库")标准代理库

Kotlin标准库提供了几个常用的标准代理。老实讲，除了lazy目前其他的灵活代理我还没有发现使用场景。发现了一定回来做补充。

### [](https://juejin.im/entry/5a1a10a96fb9a0450e75d286?utm_source=gold_browser_extension#lazy "lazy")lazy

lazy是一个方法。它可以通过传入一个lamda函数返回一个Lazy实例用于代理属性。第一次get调用，会执行传入lazy()的lamda函数，并且记录返回值，后续的调用只会返回第一次记录的值。
例如：

~~~
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}
fun main(args: Array<String>) {
    println(lazyValue)
    println(lazyValue)
}

~~~

打印结果是这样的：

~~~
computed!
Hello
Hello

~~~

如果你想要线程安全，使用 blockingLazy(): 它还是按照同样的方式工作，但保证了它的值只会在一个线程中计算，并且所有的线程都获取的同一个值。

用途：在我们生命类的成员的时候，很多时候还不需要初始化，这时，我们就可以用以初始化的构造函数作为lazy的参数，然后形成代理属性。比如：

~~~
private val bannerAdapter: BannerAdapter by lazy { BannerAdapter() }
val viewPager: ViewPager by lazy { ViewPager(context) }
private val indicators: LinearLayout by lazy { LinearLayout(context) }
private val tvTitle: JumpShowTextView by lazy { JumpShowTextView(context) }
private val tvSlogan: JumpShowTextView by lazy { JumpShowTextView(context) }

~~~

### [](https://juejin.im/entry/5a1a10a96fb9a0450e75d286?utm_source=gold_browser_extension#observable "observable")observable

Delegates.observable() 有两个参数：初始值和用于修改的handler。每次给这个属性派值（生效）的时候，handler都会被调用。这个Handler有三个参数：被指派的属性，旧值和新值：

~~~
import kotlin.properties.Delegates
class User {
    var name: String by Delegates.observable("<no name>") {
        prop, old, new ->
        println("$old -> $new")
    }
}
fun main(args: Array<String>) {
    val user = User()
    user.name = "first"
    user.name = "second"
}

~~~

这个例子打印如下：

~~~
<no name> -> first
first -> second

~~~

### [](https://juejin.im/entry/5a1a10a96fb9a0450e75d286?utm_source=gold_browser_extension#map "map")map

Delegates.mapVal() 拥有一个 map 实例并返回一个可以从 map 中读其中属性的代理。在应用中有很多这样的例子，比如解析 JSON 或者做其它的一些 “动态”的事情：

~~~
class User(val map: Map<String, Any?>) {
	val name: String by Delegates.mapVal(map)
	val age: Int     by Delegates.mapVal(map)
}

~~~

在这个例子中，构造函数持有一个 map :

~~~
val user = User(mapOf (
	"name" to "John Doe",
	"age" to 25
))

~~~

代理从这个 map 中取指(通过属性的名字)：

~~~
println(user.name) // Prints "John Doe"
println(user.age)  // Prints 25

~~~

var 可以用 mapVar