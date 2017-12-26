抬头
---
* 属性相关
	* var、val
	* 默认实现的get、set
	* 幕后字段
	* 常量放在那里
	* lateinit
	* 委托
* 方法相关
* 对象相关
	* **object**

---
[TOC]


### object声明（object declarations）
#### **使用object声明方便地实现单例模式**
~~~
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
    // ...
}
val allDataProviders: Collection<DataProvider>
    get() = // ...
}
~~~
以上就是一个对象声明，和声明一个变量一样。使用如下：

~~~
DataProviderManager.registerDataProvider(...)
~~~
如此就完成了一个方便的单例模式的构造。

另外object是可以有父类的：
~~~
object DefaultListener : MouseAdapter() {
     override fun mouseClicked(e: MouseEvent) {
        // ...
    }
    override fun mouseEntered(e: MouseEvent) {
        // ...
     }
}
~~~
注意：object不可以作为内部类使用

#### companion object
在类的内部使用object，它的性质就成了这个类的静态部分，需要使用companion object

~~~
class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
}
~~~
使用如下：
~~~
val instance = MyClass.create()
~~~
很明显，这就是一个静态方法的调用。（Kotlin中没有类似java里买呢static关键字的用法）

#### 静态方法有了，那么静态属性怎么实现？
companion object的名字是可以省略的，这时候直接把需要的静态属性和静态方法都放进去就OK了。
值得注意的是，静态内部类（就是直接在一个类里面用class声明的类）是不可以访问它的外部类的普通成员的，那么怎么办？访问其外部类的companion object的成员就OK了。
举例说明：
~~~

class NotificationModule(context: ReactApplicationContext?) : ReactContextBaseJavaModule(context) {

    companion object {
        private val TAG = "NotificationModule"
        private var mCachedBundle: Bundle? = null // 静态存储通知内容
   
        private fun sendEvent() {

        }
    }

    override fun getName(): String = "notification"

    class Receiver : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            mCachedBundle = intent?.extras
            Log.d(TAG, "onReceive: $mCachedBundle")
            when (intent?.action) {
                JPushInterface.ACTION_MESSAGE_RECEIVED -> {
                    try {
                        val message = intent.getStringExtra(JPushInterface.EXTRA_MESSAGE)
                        Log.d(TAG, "收到自定义消息： " + message)
                        mEvent = RECEIVE_CUSTOM_MESSAGE
                        if (mRAC != null) {
                            sendEvent()
                        }
                    } (e: Exception) {
                        e.printStackTrace()
                    }
                    }
                }
            }
        }

    }
}
~~~
#### object 表示（object expressions）
##### 使用object表达实现匿名类的实例化
在Java里，经常在需要立刻实现一个接口并override其方法的时候，就会使用的匿名类，例如：
~~~
builder.setNegativeButton(buttons.btnCancel.title, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialogInterface, int i) {
                  // ...
            }
        });
~~~
改写成Kotlin就是如下的样子：
~~~
builder.setNegativeButton(buttons.btnCancel.title, object : DialogInterface.OnClickListener() {
    override fun onClick(dialogInterface: DialogInterface, I: Int) { // ...}
}
~~~
##### just object
有时候仅仅就是需要一个object（不去继承某个类或者实现某个接口）

~~~
fun foo() {
    val adHoc = object {
        var x: Int = 0
        var y: Int = 0
    }
    print(adHoc.x + adHoc.y)
}
~~~
这就类似与JS里对object的灵活定义了。

#### 其他使用的注意事项
##### 匿名类作为方法返回值
匿名类一般只作为本地或者私有声明，如果是公开的且没有继承的，它可能就是作为Any返回，看例子：

~~~
class C {
    // Private function, so the return type is the anonymous object type
    private fun foo() = object {
        val x: String = "x"
    }
    // Public function, so the return type is Any
    fun publicFoo() = object {
        val x: String = "x"
    }
    fun bar() {
        val x1 = foo().x // Works
        val x2 = publicFoo().x // ERROR: Unresolved reference 'x'
    }
}
~~~
##### 访问外部变量不需要像Java一样严格为final
~~~
fun countClicks(window: JComponent) {
    var clickCount = 0
    var enterCount = 0
    window.addMouseListener(object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            clickCount++
        }
        override fun mouseEntered(e: MouseEvent) {
            enterCount++
        }
    })
// ...
}
~~~

#### object declaration (object 声明)与object expressions (object 表达式)的区别
object expressions是立即生效的，而object declaration是懒加载的，只有在使用的时候才生效
companion object的实例化是依赖于它所在的类的加载的