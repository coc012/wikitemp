

## 为什么网上这么多dagger2教程，我还写了这篇文章。

1.  找了很多Dagger2相关的博客，我看的脑浆炸裂……
2.  Dagger2给我们带来了什么，大多数博文也没有说明
3.  手动写写，加深印象，骗骗粉丝 （手动滑稽）
4.  部分Dagger2的运作机制是我个人的臆测，比如Dagger2编译入口，不过应该八九不离十吧，测试了挺多次的，没有@Component的话是不会编译的=。=

## 一、Dagger2使用Q&A

**Q1：dagger2是什么，有什么用？**
A1：dagger2是一个基于JSR-330标准的依赖注入框架，在编译期间自动生成代码，负责依赖对象的创建。

**Q2：什么是JSR-330**
A2：JSR即Java Specification Requests，意思是java规范提要。
而JSR-330则是 Java依赖注入标准
关于JSR-330可以阅读这篇文章[Java 依赖注入标准（JSR-330）简介](http://blog.csdn.net/dl88250/article/details/4838803)，随便看下就好了，不是重点。

**Q3：用dagger2提供依赖有什么好处**
A:3：为了进一步解耦和方便测试，我们会使用依赖注入的方式构建对象。（可以看这篇文章[使用Dagger2前你必须了解的一些设计原则](http://www.jianshu.com/p/cc1427e385b5)）
但是，在Activity中有可能出现这样的情况。

~~~
public class LoginActivity extends AppCompatActivity {

    LoginActivityPresenter presenter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        OkHttpClient okHttpClient = new OkHttpClient();
        RestAdapter.Builder builder = new RestAdapter.Builder();
        builder.setClient(new OkClient(okHttpClient));
        RestAdapter restAdapter = builder.build();
        ApiService apiService = restAdapter.create(ApiService.class);
        UserManager userManager = UserManager.getInstance(apiService);

        UserDataStore userDataStore = UserDataStore.getInstance(
                getSharedPreferences("prefs", MODE_PRIVATE)
        );

        //Presenter is initialized here
        presenter = new LoginActivityPresenter(this, userManager, userDataStore);
    }
}
~~~

其实我们需要的只是`LoginActivityPresenter`对象，但是因为使用依赖注入的原因，我们不得不在LoginActivity中初始化一大堆Presenter所需要的依赖。

现在不仅依赖于`LoginActivityPresenter`，还依赖`OkHttpClient ，UserManager ，RestAdapter`等。它们之中任何一个的构造改变了，或者Presenter构造改变了，我们都需要反复修改LoginActivity中的代码。

而dagger框架就解决了这种问题，使用dagger2框架后相同代码如下：

~~~
public class LoginActivity extends AppCompatActivity {

    @Inject
    LoginActivityPresenter presenter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        //Satisfy all dependencies requested by @Inject annotation
        getDependenciesGraph().inject(this);
    }
}
~~~

LoginActivity瞬间清爽了。dagger2框架可以让依赖注入独立于组件之外，不管Presenter的依赖怎么改，都不会对LoginActivity的代码照成任何影响，**这就是dagger2框架的好处了**

## 二、Dagger2 API

~~~
public @interface Component {
    Class<?>[] modules() default {};
    Class<?>[] dependencies() default {};
}

public @interface Subcomponent {
    Class<?>[] modules() default {};
}

public @interface Module {
    Class<?>[] includes() default {};
}

public @interface Provides {
}

public @interface MapKey {
    boolean unwrapValue() default true;
}

public interface Lazy<T> {
    T get();
}
~~~

还有在Dagger 2中用到的定义在 [JSR-330](https://jcp.org/en/jsr/detail?id=330) （Java中依赖注入的标准）中的其它元素：

~~~
public @interface Inject {
}

public @interface Scope {
}

public @interface Qualifier {
}
~~~

## 三、@Inject和@Component

先来看一段没有使用dagger的依赖注入Demo
MainActivity依赖Pot， Pot依赖Rose

~~~
public class Rose {
    public String whisper()  {
        return "热恋";
    }
}
~~~

~~~
public class Pot {

    private Rose rose;

    @Inject
    public Pot(Rose rose) {
        this.rose = rose;
    }

    public String show() {
        return rose.whisper();
    }
}
~~~

~~~
public class MainActivity extends AppCompatActivity {

    private Pot pot;

    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        Rose rose = new Rose();
        pot = new Pot(rose);

        String show = pot.show();
        Toast.makeText(MainActivity.this, show, Toast.LENGTH_SHORT).show();
    }
}
~~~

使用Dagger2进行依赖注入如下：

~~~
public class Rose {

    @Inject
    public Rose() {}

    public String whisper()  {
        return "热恋";
    }
}
~~~

~~~
public class Pot {

    private Rose rose;

    @Inject
    public Pot(Rose rose) {
        this.rose = rose;
    }

    public String show() {
        return rose.whisper();
    }
}
~~~

~~~
@Component
public interface MainActivityComponent {
    void inject(MainActivity activity);
}
~~~

~~~
public class MainActivity extends AppCompatActivity {

    @Inject
    Pot pot;

    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // 这个类是重新编译后Dagger2自动生成的，所以写这行代码之前要先编译一次
        // Build --> Rebuild Project
        DaggerMainActivityComponent.create().inject(this);
        String show = pot.show();
        Toast.makeText(MainActivity.this, show, Toast.LENGTH_SHORT).show();
    }
}
~~~

Dagger2生成的代码保存在这里：

![](http://upload-images.jianshu.io/upload_images/2202079-cdf20511b8e40939.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Dagger2 apt.png

源码待会分析，现在先来了解下`@Inject`和`@Component`两个API，想要使用Dagger2进行依赖注入，至少要使用到这两个注解。
`@Inject`用于标记需要注入的依赖，或者标记用于提供依赖的方法。
`@Component`则可以理解为注入器，在注入依赖的目标类`MainActivity`使用Component完成注入。

### @Inject

依赖注入中第一个并且是最重要的就是`@Inject`注解。JSR-330标准中的一部分，标记那些应该被依赖注入框架提供的依赖。在Dagger 2中有3种不同的方式来提供依赖：

1.  **构造器注入，@Inject标注在构造器上其实有两层意思。**
    ①告诉Dagger2可以使用这个构造器构建对象。如`Rose`类
    ②注入构造器所需要的参数的依赖。 如`Pot`类，构造上的Rose会被注入。
    构造器注入的局限：如果有多个构造器，我们只能标注其中一个，无法标注多个。

2.  **属性注入**
    如`MainActivity`类，标注在属性上。被标注的属性不能使用`private`修饰，否则无法注入。
    属性注入也是Dagger2中使用最多的一个注入方式。

3.  **方法注入**

    ~~~
    public class MainActivity extends AppCompatActivity {

     private Pot pot;

     protected void onCreate(@Nullable Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
         DaggerMainActivityComponent.create().inject(this);
         String show = pot.show();
         Toast.makeText(MainActivity.this, show, Toast.LENGTH_SHORT).show();
     }

     @Inject
     public void setPot(Pot pot) {
         this.pot = pot;
     }
    }
    ~~~

    标注在public方法上，Dagger2会在构造器执行之后立即调用这个方法。
    方法注入和属性注入基本上没有区别， 那么什么时候应该使用方法注入呢？
    比如该依赖需要this对象的时候，使用方法注入可以提供安全的this对象，因为方法注入是在构造器之后执行的。
    比如google mvp dagger2中，给View设置Presenter的时候可以这样使用方法注入。

    ~~~
    /**
      * Method injection is used here to safely reference {@code this} after the object is created.
      * For more information, see Java Concurrency in Practice.
      */
     @Inject
     void setupListeners() {
         mTasksView.setPresenter(this);
     }
    ~~~

### @Component

`@Inject`注解只是JSR-330中定义的注解，在`javax.inject`包中。
这个注解本身并没有作用，它需要依赖于注入框架才具有意义，用来标记需要被注入框架注入的方法，属性，构造。

而Dagger2则是用`Component`来完成依赖注入的，`@Component`可以说是Dagger2中最重要的一个注解。

~~~
@Component
public interface MainActivityComponent {
    void inject(MainActivity activity);
}
~~~

以上是定义一个Component的方式。使用接口定义，并且`@Component`注解。
命名方式推荐为：`目标类名+Component`，在编译后Dagger2就会为我们生成`DaggerXXXComponent`这个类，它是我们定义的`xxxComponent`的实现，在目标类中使用它就可以实现依赖注入了。

**Component中一般使用两种方式定义方法。**

1.  `void inject(目标类 obj);`Dagger2会从目标类开始查找@Inject注解，自动生成依赖注入的代码，调用inject可完成依赖的注入。
2.  `Object getObj();` 如：`Pot getPot();`
    Dagger2会到Pot类中找被@Inject注解标注的构造器，自动生成提供Pot依赖的代码，这种方式一般为其他Component提供依赖。（一个Component可以依赖另一个Component，后面会说）

**Component和Inject的关系如下：**

![](http://upload-images.jianshu.io/upload_images/2202079-51b78542dd3c8575.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Jsr330和Dagger2.png

Dagger2框架以Component中定义的方法作为入口，到目标类中寻找JSR-330定义的@Inject标注，生成一系列提供依赖的Factory类和注入依赖的Injector类。
而Component则是联系Factory和Injector，最终完成依赖的注入。

**我们看下源码（请对应上面的Dagger2 apt图一起看）：**

**Rose_Factory和Pot_Factory分别对应Rose类和Pot类的构造器上的@Inject注解。**
而Factory其实是个Provider对象

~~~
public interface Provider<T> {

    /**
     * Provides a fully-constructed and injected instance of {@code T}.
     *
     * @throws RuntimeException if the injector encounters an error while
     *  providing an instance. For example, if an injectable member on
     *  {@code T} throws an exception, the injector may wrap the exception
     *  and throw it to the caller of {@code get()}. Callers should not try
     *  to handle such exceptions as the behavior may vary across injector
     *  implementations and even different configurations of the same injector.
     */
    T get();
}
~~~

~~~
public interface Factory<T> extends Provider<T> {}
~~~

为什么这里要使用枚举作为提供Rose对象的Provide我也不太清楚，反正能提供就对了=。=

~~~
public enum Rose_Factory implements Factory<Rose> {
  INSTANCE;

  @Override
  public Rose get() {
    return new Rose();
  }

  public static Factory<Rose> create() {
    return INSTANCE;
  }
}
~~~

Pot对象依赖Rose，所以直接将RoseProvide作为参数传入了。

~~~
public final class Pot_Factory implements Factory<Pot> {
  private final Provider<Rose> roseProvider;

  public Pot_Factory(Provider<Rose> roseProvider) {
    assert roseProvider != null;
    this.roseProvider = roseProvider;
  }

  @Override
  public Pot get() {
    return new Pot(roseProvider.get());
  }

  public static Factory<Pot> create(Provider<Rose> roseProvider) {
    return new Pot_Factory(roseProvider);
  }
}
~~~

**MainActivity上的@Inject属性或方法注解，则对应MainActivity_MembersInjector类**

~~~
public interface MembersInjector<T> {

  /**
   * Injects dependencies into the fields and methods of {@code instance}. Ignores the presence or
   * absence of an injectable constructor.
   *
   * <p>Whenever the object graph creates an instance, it performs this injection automatically
   * (after first performing constructor injection), so if you're able to let the object graph
   * create all your objects for you, you'll never need to use this method.
   *
   * @param instance into which members are to be injected
   * @throws NullPointerException if {@code instance} is {@code null}
   */
  void injectMembers(T instance);
}
~~~

~~~
public final class MainActivity_MembersInjector implements MembersInjector<MainActivity> {
  private final Provider<Pot> potProvider;

  public MainActivity_MembersInjector(Provider<Pot> potProvider) {
    assert potProvider != null;
    this.potProvider = potProvider;
  }

  public static MembersInjector<MainActivity> create(Provider<Pot> potProvider) {
    return new MainActivity_MembersInjector(potProvider);
  }

  @Override
  public void injectMembers(MainActivity instance) {
    if (instance == null) {
      throw new NullPointerException("Cannot inject members into a null reference");
    }
    instance.pot = potProvider.get();
  }

  public static void injectPot(MainActivity instance, Provider<Pot> potProvider) {
    instance.pot = potProvider.get();
  }
}
~~~

最后是DaggerMainActivityComponent类，对应@Component注解就不多说了。这是Dagger2解析JSR-330的入口。
它联系Factory和MainActivity两个类完成注入。

~~~
public final class DaggerMainActivityComponent implements MainActivityComponent {
  private Provider<Pot> potProvider;

  private MembersInjector<MainActivity> mainActivityMembersInjector;

  private DaggerMainActivityComponent(Builder builder) {
    assert builder != null;
    initialize(builder);
  }

  public static Builder builder() {
    return new Builder();
  }

  public static MainActivityComponent create() {
    return builder().build();
  }

  @SuppressWarnings("unchecked")
  private void initialize(final Builder builder) {

    this.potProvider = Pot_Factory.create(Rose_Factory.create());

    this.mainActivityMembersInjector = MainActivity_MembersInjector.create(potProvider);
  }

  @Override
  public void inject(MainActivity activity) {
    mainActivityMembersInjector.injectMembers(activity);
  }

  public static final class Builder {
    private Builder() {}

    public MainActivityComponent build() {
      return new DaggerMainActivityComponent(this);
    }
  }
}
~~~

只使用几个注解，Dagger2就默默中为我们做了这么多事情，太感动了……
看完这个，相信大家已经完全理解了@Inject和@Component两个注解的作用了，要区分的是，@Inject是JSR330定义的，而@Component是Dagger2框架自己定义的。

## 四、@Module和@Provides

使用@Inject标记构造器提供依赖是有局限性的，比如说我们需要注入的对象是第三方库提供的，我们无法在第三方库的构造器上加上@Inject注解。
或者，我们使用依赖倒置的时候，因为需要注入的对象是抽象的，@Inject也无法使用，因为抽象的类并不能实例化，比如：

~~~
public abstract class Flower {
    public abstract String whisper();
}
~~~

~~~
public class Lily extends Flower {

    @Inject
    Lily() {}

    @Override
    public String whisper() {
        return "纯洁";
    }
}
~~~

~~~
public class Rose extends Flower {

    @Inject
    public Rose() {}

    public String whisper()  {
        return "热恋";
    }
}
~~~

~~~
public class Pot {

    private Flower flower;

    @Inject
    public Pot(Flower flower) {
        this.flower = flower;
    }

    public String show() {
        return flower.whisper();
    }
}
~~~

修改下Demo，遵循依赖倒置规则。但是这时候Dagger就报错了，因为Pot对象需要Flower，而Flower是抽象的，无法使用@Inject提供实例。

![](http://upload-images.jianshu.io/upload_images/2202079-798acb053f54eb7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

抽象的依赖.png

这时候就需要用到Module了。

清除Lily和Rose的@Inject

~~~
public class Lily extends Flower {

    @Override
    public String whisper() {
        return "纯洁";
    }
}
~~~

~~~
public class Rose extends Flower {

    public String whisper()  {
        return "热恋";
    }
}
~~~

@Module标记在类上面，@Provodes标记在方法上，表示可以通过这个方法获取依赖。

~~~
@Module
public class FlowerModule {
    @Provides
    Flower provideFlower() {
        return new Rose();
    }
}
~~~

在@Component中指定Module

~~~
@Component(modules = FlowerModule.class)
public interface MainActivityComponent {
    void inject(MainActivity activity);
}
~~~

其他类不需要更改，这样就完成了。

那么Module是干嘛的，我们来看看生成的类。

![](http://upload-images.jianshu.io/upload_images/2202079-60ef03623d0d85c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Module.png

可以看到，被@Module注解的类生成的也是Factory。

~~~
public final class FlowerModule_FlowerFactory implements Factory<Flower> {
  private final FlowerModule module;

  public FlowerModule_FlowerFactory(FlowerModule module) {
    assert module != null;
    this.module = module;
  }

  @Override
  public Flower get() {
    return Preconditions.checkNotNull(
        module.provideFlower(), "Cannot return null from a non-@Nullable @Provides method");
  }

  public static Factory<Flower> create(FlowerModule module) {
    return new FlowerModule_FlowerFactory(module);
  }
}
~~~

@Module需要和@Provide是需要一起使用的时候才具有作用的，并且@Component也需要指定了该Module的时候。

@Module是告诉Component，可以从这里获取依赖对象。Component就会去找被@Provide标注的方法，相当于构造器的@Inject，可以提供依赖。

还有一点要说的是，@Component可以指定多个@Module的，如果需要提供多个依赖的话。
并且Component也可以依赖其它Component存在。

## 五、@Qualifier和@Named

@Qualifier是限定符，而@Named则是基于String的限定符。

当我有两个相同的依赖（都继承某一个父类或者都是先某一个接口）可以提供给高层时，那么程序就不知道我们到底要提供哪一个依赖，因为它找到了两个。
这时候我们就可以通过限定符为两个依赖分别打上标记，指定提供某个依赖。

接着上一个Demo，例如：Module可以提供的依赖有两个。

~~~
@Module
public class FlowerModule {

    @Provides
    Flower provideRose() {
        return new Rose();
    }

    @Provides
    Flower provideLily() {
        return new Lily();
    }
}
~~~

![](http://upload-images.jianshu.io/upload_images/2202079-1c4a2b616d4e8781.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

多个Provider

这时候就可以用到限定符来指定依赖了，我这里用@Named来演示。

~~~
@Module
public class FlowerModule {

    @Provides
    @Named("Rose")
    Flower provideRose() {
        return new Rose();
    }

    @Provides
    @Named("Lily")
    Flower provideLily() {
        return new Lily();
    }
}
~~~

我们是通过@Inject Pot的构造器注入Flower依赖的，在这里可以用到限定符。

~~~
public class Pot {

    private Flower flower;

    @Inject
    public Pot(@Named("Rose") Flower flower) {
        this.flower = flower;
    }

    public String show() {
        return flower.whisper();
    }
}
~~~

而@Qualifier的作用和@Named是完全一样的，不过更推荐使用@Qualifier，因为@Named需要手写字符串，容易出错。

@Qualifier不是直接注解在属性上的，而是用来自定义注解的。

~~~
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface RoseFlower {}
~~~

~~~
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface LilyFlower {}
~~~

~~~
@Module
public class FlowerModule {

    @Provides
    @RoseFlower
    Flower provideRose() {
        return new Rose();
    }

    @Provides
    @LilyFlower
    Flower provideLily() {
        return new Lily();
    }
}
~~~

~~~
public class Pot {

    private Flower flower;

    @Inject
    public Pot(@RoseFlower Flower flower) {
        this.flower = flower;
    }

    public String show() {
        return flower.whisper();
    }
}
~~~

* * *

我们也可以使用Module来管理Pot依赖，当然还是需要@Qualifier指定提供哪一个依赖

~~~
@Module
public class PotModule {

    @Provides
    Pot providePot(@RoseFlower Flower flower) {
        return new Pot(flower);
    }
}
~~~

然后MainAcitivtyComponent需要增加一个Module

~~~
@Component(modules = {FlowerModule.class, PotModule.class})
public interface MainActivityComponent {
    void inject(MainActivity activity);
}
~~~

## 六、@Component的dependence和@SubComponent

[参考：SubComponent和Dependence区别](http://stackoverflow.com/questions/29587130/dagger-2-subcomponents-vs-component-dependencies)

上面也说过，Component可以依赖于其他Component，可以使用@Component的dependence，也可以使用@SubComponent，这样就可以获取其他Component的依赖了。

如：我们也用Component来管理FlowerModule和PotModule，并且使用dependence联系各个Component。
这次我就将代码贴完整点吧。

~~~
public abstract class Flower {
    public abstract String whisper();
}
~~~

~~~
public class Lily extends Flower {

    @Override
    public String whisper() {
        return "纯洁";
    }
}
~~~

~~~
public class Rose extends Flower {

    public String whisper()  {
        return "热恋";
    }
}
~~~

~~~
@Module
public class FlowerModule {

    @Provides
    @RoseFlower
    Flower provideRose() {
        return new Rose();
    }

    @Provides
    @LilyFlower
    Flower provideLily() {
        return new Lily();
    }
}
~~~

Component上也需要指定@Qualifier

~~~
@Component(modules = FlowerModule.class)
public interface FlowerComponent {
    @RoseFlower
    Flower getRoseFlower();

    @LilyFlower
    Flower getLilyFlower();
}
~~~

~~~
public class Pot {

    private Flower flower;

    public Pot(Flower flower) {
        this.flower = flower;
    }

    public String show() {
        return flower.whisper();
    }
}
~~~

PotModule需要依赖Flower，需要指定其中一个子类实现，这里使用RoseFlower

~~~
@Module
public class PotModule {

    @Provides
    Pot providePot(@RoseFlower Flower flower) {
        return new Pot(flower);
    }
}
~~~

~~~
@Component(modules = PotModule.class,dependencies = FlowerComponent.class)
public interface PotComponent {
    Pot getPot();
}
~~~

~~~
@Component(dependencies = PotComponent.class)
public interface MainActivityComponent {
    void inject(MainActivity activity);
}
~~~

而在MainActivity则需要创建其依赖的Component

~~~
public class MainActivity extends AppCompatActivity {

    @Inject
    Pot pot;

    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        DaggerMainActivityComponent.builder()
                .potComponent(DaggerPotComponent.builder()
                        .flowerComponent(DaggerFlowerComponent.create())
                        .build())
                .build().inject(this);

        String show = pot.show();
        Toast.makeText(MainActivity.this, show, Toast.LENGTH_SHORT).show();
    }
}
~~~

这就是Component的dependencies的用法了，我们Component不需要重复的指定Module，可以直接依赖其它Component获得。

分析下源码，看下Component的dependencies做了什么事情。

~~~
public final class DaggerPotComponent implements PotComponent {
  private Provider<Flower> getRoseFlowerProvider;

  private Provider<Pot> providePotProvider;

  private DaggerPotComponent(Builder builder) {
    assert builder != null;
    initialize(builder);
  }

  public static Builder builder() {
    return new Builder();
  }

  @SuppressWarnings("unchecked")
  private void initialize(final Builder builder) {

    this.getRoseFlowerProvider =
        new Factory<Flower>() {
          private final FlowerComponent flowerComponent = builder.flowerComponent;

          @Override
          public Flower get() {
            return Preconditions.checkNotNull(
                flowerComponent.getRoseFlower(),
                "Cannot return null from a non-@Nullable component method");
          }
        };

    this.providePotProvider =
        PotModule_ProvidePotFactory.create(builder.potModule, getRoseFlowerProvider);
  }

  @Override
  public Pot getPot() {
    return providePotProvider.get();
  }

  public static final class Builder {
    private PotModule potModule;

    private FlowerComponent flowerComponent;

    private Builder() {}

    public PotComponent build() {
      if (potModule == null) {
        this.potModule = new PotModule();
      }
      if (flowerComponent == null) {
        throw new IllegalStateException(FlowerComponent.class.getCanonicalName() + " must be set");
      }
      return new DaggerPotComponent(this);
    }

    public Builder potModule(PotModule potModule) {
      this.potModule = Preconditions.checkNotNull(potModule);
      return this;
    }

    public Builder flowerComponent(FlowerComponent flowerComponent) {
      this.flowerComponent = Preconditions.checkNotNull(flowerComponent);
      return this;
    }
  }
}
~~~

PotComponent依赖FlowerComponent，其实就是将FlowerComponent的引用传递给PotComponent，这样PotComponent就可以使用FlowerComponent中的方法了。
注意看getRoseFlowerProvider这个Provider，是从 `flowerComponent.getRoseFlower()`获取到的

* * *

如果使用Subcomponent的话则是这么写， 其他类不需要改变，只修改Component即可

~~~
@Component(modules = FlowerModule.class)
public interface FlowerComponent {

    PotComponent plus(PotModule potModule);
}
~~~

~~~
@Subcomponent(modules = PotModule.class)
public interface PotComponent {
    MainActivityComponent plus();
}
~~~

~~~
@Subcomponent
public interface MainActivityComponent {
    void inject(MainActivity activity);
}
~~~

~~~
public class MainActivity extends AppCompatActivity {

    @Inject
    Pot pot;

    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        DaggerFlowerComponent.create()
                .plus(new PotModule())  // 这个方法返回PotComponent
                .plus()                 // 这个方法返回MainActivityComponent
                .inject(this);

        String show = pot.show();
        Toast.makeText(MainActivity.this, show, Toast.LENGTH_SHORT).show();
    }
}
~~~

FlowerComponent管理了PotComponent和MainActivityComponent，看起来不符合常理。

先来说说Component中的方法的第三种定义方式（上面说了两种）。

~~~
@Component
class AComponpent {
    XxxComponent plus(Module... modules)
}
~~~

~~~
@Subcomponent(modules = xxxxx)
class XxxComponent {

}
~~~

xxxComponent是该AComponpent的依赖，被@Subcomponent标注。
而modules参数则是xxxComponent指定的Module。
在重新编译后，Dagger2生成的代码中，Subcomponent标记的类是Componpent的内部类。
像上面的Demo，MainActivityComponent是PotComponent的内部类，而PotComponent又是FlowerComponent的内部类。

* * *

但是用Subcomponent怎么看怎么别扭，各个Component之间联系太紧密，不太适合我们Demo的使用场景。
**那什么时候该用@Subcomponent呢？**
Subcomponent是作为Component的拓展的时候。
像我写的Demo中，Pot和Flower还有MainActivity只是单纯的依赖关系。就算有，也只能是Flower作为Pot的Subcomponent，而不是Demo中所示，因为我需要给大家展示Dagger的API，强行使用。

**比较适合使用Subcomponent的几个场景：**
很多工具类都需要使用到Application的Context对象，此时就可以用一个Component负责提供，我们可以命名为AppComponent。
需要用到的context对象的SharePreferenceComponent，ToastComponent就可以它作为Subcomponent存在了。

而且在AppComponent中，我们可以很清晰的看到有哪些子Component，因为在里面我们定义了很多`XxxComponent plus(Module... modules)`

每个ActivityComponent也是可以作为AppComponent的Subcomponent，这样可以更方便的进行依赖注入，减少重复代码。

**Component dependencies和Subcomponent区别**

1.  Component dependencies 能单独使用，而Subcomponent必须由Component调用方法获取。
2.  Component dependencies 可以很清楚的得知他依赖哪个Component， 而Subcomponent不知道它自己的谁的孩子……真可怜
3.  使用上的区别，Subcomponent就像这样`DaggerAppComponent.plus(new SharePreferenceModule());`
    使用Dependence可能是这样`DaggerAppComponent.sharePreferenceComponent(SharePreferenceComponent.create())`

**Component dependencies和Subcomponent使用上的总结**

Component Dependencies：

1.  你想保留独立的想个组件（Flower可以单独使用注入，Pot也可以）
2.  要明确的显示该组件所使用的其他依赖

Subcomponent：

1.  两个组件之间的关系紧密
2.  你只关心Component，而Subcomponent只是作为Component的拓展，可以通过Component.xxx调用。

[Dagger 2 subcomponents vs component dependencies](http://stackoverflow.com/questions/29587130/dagger-2-subcomponents-vs-component-dependencies)

## 七、@Scope和@Singleton

@Scope是用来管理依赖的生命周期的。它和@Qualifier一样是用来自定义注解的，而@Singleton则是@Scope的默认实现。

~~~
/**
 * Identifies a type that the injector only instantiates once. Not inherited.
 *
 * @see javax.inject.Scope @Scope
 */
@Scope
@Documented
@Retention(RUNTIME)
public @interface Singleton {}
~~~

Component会帮我们注入被@Inject标记的依赖，并且可以注入多个。
但是每次注入都是重新new了一个依赖。如

~~~
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    @Inject
    Pot pot;

    @Inject
    Pot pot2;

    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        DaggerMainActivityComponent.builder()
                .potComponent(DaggerPotComponent.builder()
                                .flowerComponent(DaggerFlowerComponent.create()).build())
                .build().inject(this);

        Log.d(TAG, "pot = " + pot.hashCode() +", pot2 = " + pot2.hashCode());

        String show = pot.show();
        Toast.makeText(MainActivity.this, show, Toast.LENGTH_SHORT).show();
    }
}
~~~

打印的地址值不一样，是两个对象。
`D/MainActivity: pot = com.aitsuki.architecture.pot.Pot@240f3ff5, pot2 = com.aitsuki.architecture.pot.Pot@2c79118a`

假设我们需要Pot对象的生命周期和app相同，也就是单例，我们需要怎么做？这时候就可以用到@Scope注解了。

我们来使用默认的@Scope实现——@Singleton
需要在@Provide和@Component中同时使用才起作用，为什么呢，待会会说明。

~~~
@Module
public class PotModule {

    @Provides
    @Singleton
    Pot providePot(@RoseFlower Flower flower) {
        return new Pot(flower);
    }
}
~~~

~~~
@Singleton
@Component(modules = PotModule.class, dependencies = FlowerComponent.class)
public interface PotComponent {
    Pot getPot();
}
~~~

然后我们再运行下项目，报错了

![](http://upload-images.jianshu.io/upload_images/2202079-b60d581820901e83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

@Scope报错

那是因为我们的MainActivityComponent依赖PotComponent，而dagger2规定子Component也必须标注@Scope。
但是我们不能给MainActivityComponent也标注@Singleton，并且dagger2也不允许。因为单例依赖单例是不符合设计原则的，我们需要自定义一个@Scope注解。

定义Scope是名字要起得有意义，能一眼就让你看出这个Scope所规定的生命周期。
比如ActivityScope 或者PerActivity，生命周期和Activity相同。

~~~
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface ActivityScope {}
~~~

~~~
@ActivityScope
@Component(dependencies = PotComponent.class)
public interface MainActivityComponent {
    void inject(MainActivity activity);
}
~~~

`D/MainActivity: pot = com.aitsuki.architecture.pot.Pot@240f3ff5, pot2 = com.aitsuki.architecture.pot.Pot@240f3ff5`
这时候我们看到两个pot对象的地址值是一样的，@Scope注解起作用了。

那么我再新建一个Activity，再次注入pot打印地址值。

~~~
public class SecondActivity extends AppCompatActivity {

    private static final String TAG = "SecondActivity";

    @Inject
    Pot pot3;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        DaggerSecondActivityComponent.builder()
                .potComponent(DaggerPotComponent.builder().flowerComponent(DaggerFlowerComponent.create()).build())
                .build().inject(this);

        Log.d(TAG, "pot3 = " + pot3);
    }
}
~~~

~~~
@ActivityScope
@Component(dependencies = PotComponent.class)
public interface SecondActivityComponent {
    void inject(SecondActivity activity);
}
~~~

在MainActivity初始化时直接跳转到SecondActivity

~~~
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    @Inject
    Pot pot;

    @Inject
    Pot pot2;

    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        DaggerMainActivityComponent.builder()
                .potComponent(DaggerPotComponent.builder()
                                .flowerComponent(DaggerFlowerComponent.create()).build())
                .build().inject(this);

        Log.d(TAG, "pot = " + pot +", pot2 = " + pot2);

        String show = pot.show();
        Toast.makeText(MainActivity.this, show, Toast.LENGTH_SHORT).show();

        startActivity(new Intent(this, SecondActivity.class));
    }
}
~~~

`D/MainActivity: pot = com.aitsuki.architecture.pot.Pot@240f3ff5, pot2 = com.aitsuki.architecture.pot.Pot@240f3ff5`
`D/SecondActivity: pot3 = com.aitsuki.architecture.pot.Pot@1b7661c7`

可以看到，在SecondActivity中，Pot对象地址和MainActivity中的不一样了。
为什么呢？不是叫@Singleton么，为什么使用了它Pot还不是单例的，Dagger2你逗我！

* * *

那么现在我可以说说@Scope的作用了，它的作用只是保证依赖在@Component中是唯一的，可以理解为“局部单例”。
**@Scope是需要成对存在的，在Module的Provide方法中使用了@Scope，那么对应的Component中也必须使用@Scope注解，当两边的@Scope名字一样时（比如同为@Singleton）, 那么该Provide方法提供的依赖将会在Component中保持“局部单例”。
而在Component中标注@Scope，provide方法没有标注，那么这个Scope就不会起作用，而Component上的Scope的作用也只是为了能顺利通过编译，就像我刚刚定义的ActivityScope一样。**

@Singleton也是一个自定义@Scope，它的作用就像上面说的一样。但由于它是Dagger2中默认定义的，所以它比我们自定义Scope对了一个功能，就是编译检测，防止我们不规范的使用Scope注解，仅此而已。

在上面的Demo中，Pot对象在PotComponent中是“局部单例”的。
而到了SecondActivity，因为是重新Build了一个PotComponent，所以Pot对象的地址值也就改变了。

**那么，我们如何使用Dagger2实现单例呢？**
很简单，做到以下两点即可。

1.  依赖在Component中是单例的（供该依赖的provide方法和对应的Component类使用同一个Scope注解。）
2.  对应的Component在App中只初始化一次，每次注入依赖都使用这个Component对象。（在Application中创建该Component）

如：

~~~
public class App extends Application {

    private PotComponent potComponent;

    @Override
    public void onCreate() {
        super.onCreate();
        potComponent = DaggerPotComponent.builder()
                .flowerComponent(DaggerFlowerComponent.create())
                .build();
    }

    public PotComponent getPotComponent() {
        return potComponent;
    }
}
~~~

然后修改MainActivity和SecondActivity的Dagger代码如下

~~~
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    @Inject
    Pot pot;

    @Inject
    Pot pot2;

    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        DaggerMainActivityComponent.builder()
                .potComponent(((App) getApplication()).getPotComponent())
                .build().inject(this);

        Log.d(TAG, "pot = " + pot +", pot2 = " + pot2);

        String show = pot.show();
        Toast.makeText(MainActivity.this, show, Toast.LENGTH_SHORT).show();

        startActivity(new Intent(this, SecondActivity.class));
    }
}
~~~

~~~
public class SecondActivity extends AppCompatActivity {

    private static final String TAG = "SecondActivity";

    @Inject
    Pot pot3;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        DaggerSecondActivityComponent.builder()
                .potComponent(((App) getApplication()).getPotComponent())
                .build().inject(this);

        Log.d(TAG, "pot3 = " + pot3);
    }
}
~~~

运行后的log输出
`D/MainActivity: pot = com.aitsuki.architecture.pot.Pot@240f3ff5, pot2 = com.aitsuki.architecture.pot.Pot@240f3ff5`
`D/SecondActivity: pot3 = com.aitsuki.architecture.pot.Pot@240f3ff5`
现在Pot的生命周期就和app相同了。

你也可以试试自定义一个@ApplicationScope，替换掉@Singleton，结果是一样的，这里就不演示了。

稍微总结下@Scope注解：
**Scope是用来给开发者管理依赖的生命周期的，它可以让某个依赖在Component中保持 “局部单例”（唯一），如果将Component保存在Application中复用，则可以让该依赖在app中保持单例。 我们可以通过自定义不同的Scope注解来标记这个依赖的生命周期，所以命名是需要慎重考虑的。**
@Singleton告诉我们这个依赖时单例的
@ActivityScope告诉我们这个依赖的生命周期和Activity相同
@FragmentScope告诉我们这个依赖的生命周期和Fragment相同
@xxxxScope ……

## 八、MapKey和Lazy

### @MapKey

这个注解用在定义一些依赖集合（目前为止，Maps和Sets）。让例子代码自己来解释吧：
定义：

~~~
@MapKey(unwrapValue = true)
@interface TestKey {
    String value();
}
~~~

提供依赖：

~~~
@Provides(type = Type.MAP)
@TestKey("foo")
String provideFooKey() {
    return "foo value";
}

@Provides(type = Type.MAP)
@TestKey("bar")
String provideBarKey() {
    return "bar value";
}
~~~

使用：

~~~
@Inject
Map<String, String> map;

map.toString() // => „{foo=foo value, bar=bar value}”
~~~

@MapKey注解目前只提供两种类型 - String和Enum。

### Lazy

Dagger2还支持Lazy模式，通过Lazy模拟提供的实例，在@Inject的时候并不初始化，而是等到你要使用的时候，主动调用其.get方法来获取实例。
比如：

~~~
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    @Inject
    Lazy<Pot> potLazy;

    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        DaggerMainActivityComponent.builder()
                .potComponent(((App) getApplication()).getPotComponent())
                .build().inject(this);

        Pot pot = potLazy.get();
        String show = pot.show();
        Toast.makeText(MainActivity.this, show, Toast.LENGTH_SHORT).show();
    }
}
~~~

## 九、项目实战

略……

233333333，直接去看Google的MVP模式吧，上面有例子，也可以去看看其他博客。
我也不知道写不写哈，有点小忙，就算写也可能是国庆过后了。

## 十、完结

看完这篇博文之后，感觉如何？博主表示写的算是很详细，很清晰易懂了。不懂的可以跟着思路敲一下哦，不动手，永远不会知道Dagger2其实并没有想象的那么难用……

作者：AItsuki
链接：http://www.jianshu.com/p/24af4c102f62
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。