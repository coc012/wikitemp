文档当前状态：**alpha0.1**
* [x] 选题收集：2018/01/09
* [ ] 初稿整理：
* [ ] 补充校对：
* [ ] 入库存档：
---
参考源：
[Android：手把手带你 使用 熟悉而陌生的Application类](https://juejin.im/post/5a5413f4f265da3e497fe8b9)

---


# 前言

*   `Applicaiton`类在 `Android`开发中非常常见，可是你真的了解`Applicaiton`类吗？
*   本文将全面解析`Applicaiton`类，包括特点、方法介绍、应用场景和具体使用，希望你们会喜欢。

* * *

# 目录

![示意图](https://user-gold-cdn.xitu.io/2018/1/9/160d86decf202880?imageView2/0/w/1280/h/960/ignore-error/1)

* * *

# 1\. 定义

*   代表应用程序（即 `Android App`）的类，也属于`Android`中的一个系统组件
*   继承关系：继承自 `ContextWarpper` 类

![示意图](https://user-gold-cdn.xitu.io/2018/1/9/160d86ded05ff21b?imageView2/0/w/1280/h/960/ignore-error/1)

* * *

# 2\. 特点

### 2.1 实例创建方式：单例模式

*   每个`Android App`运行时，会首先自动创建`Application` 类并实例化 `Application` 对象，且只有一个

> 即 `Application`类 是单例模式（`singleton`）类

*   也可通过 继承 `Application` 类自定义`Application` 类和实例

### 2.2 实例形式：全局实例

即不同的组件（如`Activity、Service`）都可获得`Application`对象且都是同一个对象

### 2.3 生命周期：等于 Android App 的生命周期

`Application` 对象的生命周期是整个程序中最长的，即等于`Android App`的生命周期

* * *

# 3\. 方法介绍

那么，该 `Application` 类有什么作用呢？下面，我将介绍`Application` 类的方法使用

![示意图](data:image/svg+xml;utf8,)

### 3.1 onCreate（）

*   调用时刻： `Application` 实例创建时调用

> `Android`系统的入口是`Application`类的 `onCreate（）`，默认为空实现

*   作用
    1.  初始化 应用程序级别 的资源，如全局对象、环境配置变量、图片资源初始化、推送服务的注册等

> 注：请不要执行耗时操作，否则会拖慢应用程序启动速度

1.  数据共享、数据缓存 设置全局共享数据，如全局共享变量、方法等

> 注：这些共享数据只在应用程序的生命周期内有效，当该应用程序被杀死，这些数据也会被清空，所以只能存储一些具备 **临时性的共享数据**

*   具体使用

~~~
// 复写方法需要在Application子类里实现

private static final String VALUE = "Carson";
    // 初始化全局变量
    @Override
    public void onCreate()
    {
        super.onCreate();

        VALUE = 1;

    }
}

~~~

### 3.2 registerComponentCallbacks（） & unregisterComponentCallbacks（）

*   作用：注册和注销 `ComponentCallbacks2`回调接口

> 本质上是复写 `ComponentCallbacks2`回调接口里的方法从而实现更多的操作，具体下面会详细介绍

*   具体使用

~~~
registerComponentCallbacks(new ComponentCallbacks2() {
// 接口里方法下面会继续介绍
            @Override
            public void onTrimMemory(int level) {

            }

            @Override
            public void onLowMemory() {

            }

            @Override
            public void onConfigurationChanged(Configuration newConfig) {

            }
        });

~~~

### 3.3 onTrimMemory（）

*   作用：通知 应用程序 当前内存使用情况（以内存级别进行识别）

> `Android 4.0` 后提供的一个API

![示意图](data:image/svg+xml;utf8,)

*   应用场景：根据当前内存使用情况进行自身的内存资源的不同程度释放，以避免被系统直接杀掉 & 优化应用程序的性能体验

> 1.  系统在内存不足时会按照`LRU Cache`中从低到高杀死进程；优先杀死占用内存较高的应用
> 2.  若应用占用内存较小 = 被杀死几率降低，从而快速启动（即热启动 = 启动速度快）
> 3.  可回收的资源包括： a. 缓存，如文件缓存，图片缓存 b. 动态生成 & 添加的View

典型的应用场景有两个：

![示意图](data:image/svg+xml;utf8,)

*   具体使用

~~~
registerComponentCallbacks(new ComponentCallbacks2() {

@Override
  public void onTrimMemory(int level) {

  // Android系统会根据当前内存使用的情况，传入对应的级别
  // 下面以清除缓存为例子介绍
    super.onTrimMemory(level);
  .   if (level >= ComponentCallbacks2.TRIM_MEMORY_MODERATE) {

        mPendingRequests.clear();
        mBitmapHolderCache.evictAll();
        mBitmapCache.evictAll();
    }

        });

~~~

*   可回调对象 & 对应方法

~~~
Application.onTrimMemory()
Activity.onTrimMemory()
Fragment.OnTrimMemory()
Service.onTrimMemory()
ContentProvider.OnTrimMemory()

~~~

**特别注意：`onTrimMemory()`中的`TRIM_MEMORY_UI_HIDDEN`与`onStop（）`的关系**

*   `onTrimMemory()`中的`TRIM_MEMORY_UI_HIDDEN`的回调时刻：当应用程序中的所有UI组件全部不可见时
*   `Activity`的`onStop（）`回调时刻：当一个Activity完全不可见的时候
*   使用建议：
    1.  在 `onStop（）中`释放与 `Activity`相关的资源，如取消网络连接或者注销广播接收器等
    2.  在`onTrimMemory()`中的`TRIM_MEMORY_UI_HIDDEN`中释放与`UI`相关的资源，从而保证用户在使用应用程序过程中，`UI`相关的资源不需要重新加载，从而提升响应速度

> 注：`onTrimMemory`的`TRIM_MEMORY_UI_HIDDEN`等级是在`onStop（）`方法之前`调用`的

### 3.4 onLowMemory（）

*   作用：监听 `Android`系统整体内存较低时刻
*   调用时刻：`Android`系统整体内存较低时

~~~
registerComponentCallbacks(new ComponentCallbacks2() {

  @Override
            public void onLowMemory() {

            }

        });

~~~

*   应用场景：`Android 4.0`前 检测内存使用情况，从而避免被系统直接杀掉 & 优化应用程序的性能体验

> 类似于 `OnTrimMemory（）`

*   特别注意：`OnTrimMemory（）` & `OnLowMemory（）` 关系
    1.  `OnTrimMemory（）`是 `OnLowMemory（）` `Android 4.0`后的替代 `API`
    2.  `OnLowMemory（）` = `OnTrimMemory（）`中的`TRIM_MEMORY_COMPLETE`级别
    3.  若想兼容`Android 4.0`前，请使用`OnLowMemory（）`；否则直接使用`OnTrimMemory（）`即可

### 3.5 onConfigurationChanged（）

*   作用：监听 应用程序 配置信息的改变，如屏幕旋转等
*   调用时刻：应用程序配置信息 改变时调用
*   具体使用

~~~
registerComponentCallbacks(new ComponentCallbacks2() {

            @Override
            public void onConfigurationChanged(Configuration newConfig) {
              ...
            }

        });

~~~

*   该配置信息是指 ：`Manifest.xml`文件下的 `Activity`标签属性`android:configChanges`的值，如下：

~~~
<activity android:name=".MainActivity">
      android:configChanges="keyboardHidden|orientation|screenSize"
// 设置该配置属性会使 Activity在配置改变时不重启，只执行onConfigurationChanged（）
// 上述语句表明，设置该配置属性可使 Activity 在屏幕旋转时不重启
 </activity>

~~~

### 3.6 registerActivityLifecycleCallbacks（） & unregisterActivityLifecycleCallbacks（）

*   作用：注册 / 注销对 应用程序内 所有`Activity`的生命周期监听
*   调用时刻：当应用程序内 `Activity`生命周期发生变化时就会调用

> 实际上是调用`registerActivityLifecycleCallbacks（）`里 `ActivityLifecycleCallbacks`接口里的方法

*   具体使用

~~~
// 实际上需要复写的是ActivityLifecycleCallbacks接口里的方法
registerActivityLifecycleCallbacks(new ActivityLifecycleCallbacks() {
            @Override
            public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
                Log.d(TAG,"onActivityCreated: " + activity.getLocalClassName());
            }

            @Override
            public void onActivityStarted(Activity activity) {
                Log.d(TAG,"onActivityStarted: " + activity.getLocalClassName());
            }

            @Override
            public void onActivityResumed(Activity activity) {
                Log.d(TAG,"onActivityResumed: " + activity.getLocalClassName());
            }

            @Override
            public void onActivityPaused(Activity activity) {
                Log.d(TAG,"onActivityPaused: " + activity.getLocalClassName());
            }

            @Override
            public void onActivityStopped(Activity activity) {
                Log.d(TAG, "onActivityStopped: " + activity.getLocalClassName());
            }

            @Override
            public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
            }

            @Override
            public void onActivityDestroyed(Activity activity) {
                Log.d(TAG,"onActivityDestroyed: " + activity.getLocalClassName());
            }
        });

<-- 测试：把应用程序从前台切到后台再打开，看Activcity的变化 -->
 onActivityPaused: MainActivity
 onActivityStopped: MainActivity
 onActivityStarted: MainActivity
 onActivityResumed: MainActivity

~~~

### 3.7 onTerminate（）

调用时刻：应用程序结束时调用

> 但该方法只用于`Android`仿真机测试，在`Android`产品机是不会调用的

* * *

# 4\. 应用场景

从`Applicaiton`类的方法可以看出，`Applicaiton`类的应用场景有：（已按优先级排序）

*   初始化 应用程序级别 的资源，如全局对象、环境配置变量等
*   数据共享、数据缓存，如设置全局共享变量、方法等
*   获取应用程序当前的内存使用情况，及时释放资源，从而避免被系统杀死
*   监听 应用程序 配置信息的改变，如屏幕旋转等
*   监听应用程序内 所有Activity的生命周期

* * *

# 5\. 具体使用

*   若需要复写实现上述方法，则需要自定义 `Application`类
*   具体过程如下

### 步骤1：新建Application子类

即继承 `Application` 类

~~~
public class CarsonApplication extends Application
  {
    ...
    // 根据自身需求，并结合上述介绍的方法进行方法复写实现

    // 下面以onCreate()为例
  private static final String VALUE = "Carson";
    // 初始化全局变量
    @Override
    public void onCreate()
    {
        super.onCreate();

        VALUE = 1;

    }

  }

~~~

### 步骤2：配置自定义的Application子类

在`Manifest.xml`文件中 `<application>`标签里进行配置

*Manifest.xml*

~~~
<application

        android:name=".CarsonApplication"
        // 此处自定义Application子类的名字 = CarsonApplication

</application>

~~~

### 步骤3：使用自定义的Application类实例

~~~
private CarsonApplicaiton app;

// 只需要调用Activity.getApplication（） 或Context.getApplicationContext（）就可以获得一个Application对象
app = (CarsonApplication) getApplication();

// 然后再得到相应的成员变量 或方法 即可
app.exitApp();

~~~

至此，关于 `Applicaiton` 类已经讲解完毕。

* * *

# 6\. 总结

*   我用一张图总结上述文章

![示意图](https://user-gold-cdn.xitu.io/2018/1/9/160d86ded045a134?imageView2/0/w/1280/h/960/ignore-error/1)

*   下面我将继续对 `Android`中的知识进行深入讲解 ，有兴趣可以继续关注[Carson_Ho的安卓开发笔记](https://link.juejin.im?target=https%3A%2F%2Fjuejin.im%2Fuser%2F58d4d9781b69e6006ba65edc)

* * *

# 请点赞！因为你的鼓励是我写作的最大动力！

