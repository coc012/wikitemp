文档当前状态：**beta0.2**
* [x] 选题收集：2017/10/25
* [x] 初稿整理：
* [ ] 补充校对：
* [ ] 入库存档：
---


[TOC]

---

## gradle配置

Android工程的每个module都有一个自己私有的**build.gradle**（绿色部分），而整个项目的根目录中也有一个**build.gradle**（灰色部分），我们这里谈论的全局配置基本都是在根build.gradle中进行的。

![image_1bp80no6ubmgo4itssrg7kfam.png-117.1kB][1]

### 设定UTF-8

一个项目的根目录的build.gradle决定了项目的全局配置，对于编码这种所有module的通用配置自然就是在这里定义的：

```
allprojects {
    repositories {
        jcenter()
        mavenCentral()
    }

    tasks.withType(JavaCompile){
        options.encoding = "UTF-8"
    }
}
```

**题外话：**
> UTF-8是Unicode的实现方式之一，IDE默认的编码也是UTF-8。

### 支持Google仓库

```
buildscript {

    repositories {
        // ...
        google()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.0'
    }
}


allprojects {
    repositories {
        google()
        jcenter()
    }
}
```
我们可以在项目根目录中的**build.gradle**给单个项目或全部工程启用google的仓库，配置后我们就可以让其自动下载最新的Android plugin了，再也无需我们手动干预。

![image_1bp7vae1vc1sbosedr1vaf1i209.png-18kB][2]

目前所有的support包都是通过google仓库进行远程依赖，如果不配置仓库的依赖就必然会出现support库依赖异常：
> Could not find com.android.support:appcompat-v7:25.4.0.

如果我们想要看下google这个仓库的地址，可以打印一下它的url：
```
buildscript {
    repositories {
        google()
        jcenter()
        mavenCentral()
    }
    
    repositories.each {
        println it.getUrl() // 输出url
    }
}
```

输出：
```
file:/D:/android-studio-ide-171.4195411-windows/android-studio/gradle/m2repository/
https://dl.google.com/dl/android/maven2/
https://jcenter.bintray.com/
https://repo1.maven.org/maven2/
```

### 支持Groovy

在根目录的build.gradle中：
```
allprojects {
    // ...
}

apply plugin: 'groovy'

dependencies {
    compile localGroovy()
}
```

这个是可选配置，配置后可以减少一些Groovy的warnning。但如果你的项目中用到了自己写的Gradle插件，那么添加**apply plugin: 'groovy'**就是必须的了。

### 配置Java版本

如果工程中的大多数module都是支持到Java7，那么可以在根目录中的build.gradle中配置最低Java版本：
```
allprojects {
    repositories {
        jcenter()
    }
    tasks.withType(JavaCompile) {
        sourceCompatibility = JavaVersion.VERSION_1_7
        targetCompatibility = JavaVersion.VERSION_1_7
    }
}
```
对于某个支持到Java8的module，当然可以在它里面的build.gradle配置Java8的支持：
```
android {
    // ...
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

配置好后可以通过项目的图形界面进行查看：

![image_1bp8s6d521v9gk2u10gcfq31qnu3q.png-46.3kB][3]

如果你的项目比较老，可以考虑使用[Gradle Retrolambda Plugin](https://github.com/evant/gradle-retrolambda)来引用Java8的语法。

### 变量、依赖版本 统一管理

当我们在开发多module工程的时候，最麻烦的就是为每个module管理**targetSdkVersion**。老的module的minSdkVersion很低，新的module因为是Android Studio自动建立的，经常会把targetSdkVersion升级到最新。我们十分希望全部的module的targetSdkVersion都能进行统一的管理，这时我们就可以考虑定义全局变量了。

#### 写法一

在project根目录下的build.gradle定义全局变量:
```
ext {
    minSdkVersion = 16
    targetSdkVersion = 24
}
buildscript {
    // ...
}
```
然后在各module的build.gradle中可以通过**rootProject.ext**来引用：
```
android {
    defaultConfig {
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
    }
}
```
这里添加**rootProject.ext**是因为这个变量定义在根build.gradle中的，如果是在私有build.gradle文件中定义的话就不用加了。


#### 写法二(推荐)
除了在build.gradle中定义全局变量外，我们还可以新增一个config.gradle文件的方式实现(config文件一般私有 不提交到版本库上)。
config.gradle可以这样写:
```
ext {
    signingConfig = [
            storePassword: "xxxxx",
            keyAlias     : "xxxxx",
            keyPassword  : "xxxx"
    ]

    android = [
            compileSdkVersion: 26,
            buildToolsVersion: "26.0.0",

            minSdkVersion    : 16,
            targetSdkVersion : 22,

            versionCode      : 32,
            versionName      : "3.4"
    ]

    supportV4_Ver = "25.3.1"
    supportV7_Ver = "25.3.1"
    supportV13_Ver = "25.3.1"

    constraint_layout = "1.0.2"

    leakcanary_Ver = "1.5"
    glide_Ver = "3.7.0"
    eventbus_Ver = "3.0.0"
    
}
```

在根gradle中：apply from: "config.gradle"
在模块的gradle中，
```
android {
    compileSdkVersion rootProject.ext.android.compileSdkVersion
    buildToolsVersion rootProject.ext.android.buildToolsVersion

    defaultConfig {
        applicationId 'com.xin.newcar2b'
        minSdkVersion rootProject.ext.android.minSdkVersion
        targetSdkVersion rootProject.ext.android.targetSdkVersion
        versionCode rootProject.ext.android.versionCode
        versionName rootProject.ext.android.versionName
    }
    
    signingConfigs {
        myConfig {
            storeFile file("../attachfiles/xxxx.jks")
            storePassword rootProject.ext.signingConfig.storePassword
            keyAlias rootProject.ext.signingConfig.keyAlias
            keyPassword rootProject.ext.signingConfig.keyPassword
        }
    }
}

dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')

    implementation "com.android.support:appcompat-v7:$rootProject.ext.supportV7_Ver"

    implementation "com.github.bumptech.glide:glide:$rootProject.ext.glide_Ver"
}
```

这样写的好处，除了能多人开发提供更友好的依赖管理外，修改版本号 并不需要重新sync,提升了开发效率,如果团队对 版本号有严格限制，这个方法也能较好的实现。
需要提的一点是，一旦用这种方式管理了support库，那么Android Studio的升级提示就完全失效了，其余类似的官方库也是同理。当然 和上面的好处相比，这并不算什么。


## 自定义Task

### 自定义apk保存路径与命名、自动保存对应的混淆mapping文件

在开发的过程中，Android Studio默认会生成app开头的apk。但如果我们有多个团队，打包机器肯定肯定会要求多个团队的apk有明显的名称区分，所以我们需要在输出的时候让apk的名称有意义：

```
static def packageTime() {
    return new Date().format("yyMMddHHmmss", TimeZone.getTimeZone("GMT+8"))
}

android {
    buildTypes {
       // ...
    }
    
    //自动重命名 生成的apk包、并且保存对应的混淆 mapping文件
     applicationVariants.all { variant ->
        variant.outputs.all { output ->
            //自动重命名 生成的apk包
            def outputFile = output.outputFile
            def packageTime = packageTime()
            if (outputFile != null && outputFile.name.endsWith('.apk')) {
                def fileName
                if (variant.buildType.name == "release") {
                    fileName = "appname_release_v${defaultConfig.versionName}_${defaultConfig.versionCode}_${packageTime}.apk"
                } else {
                    fileName = "appname_debuger_v${defaultConfig.versionName}_${defaultConfig.versionCode}_${packageTime}.apk"
                }
                outputFileName = fileName
            }

            //自动保存mapping文件、避免发版遗失
            if (variant.getBuildType().isMinifyEnabled()) {
                variant.assemble.doLast {
                    copy {
                        from variant.mappingFile
                        into "${projectDir}/mappings"
                        rename { String fileName ->
                            "mapping_appname_v${defaultConfig.versionName}_${defaultConfig.versionCode}_${packageTime}.txt"
                        }
                    }
                }
            }
        }
    }
}
```

上面的方式，会自动保存apk和对应的maping文件(相同的时间戳)，apk发布后，可以寻找相同时间戳的mapping上传对相关的平台上，便于bug的追踪定位。

### 更改AAR的输出的位置

在插件项目的开发过程中，在调试模式时输出的是apk，在插件模式中时aar。宿主App会在运行时自动加载SD卡根目录中的aar，将其当作一个资源来进行管理。

在Android Studio中我们的aar都是输出在outputs目录（最新的Android Studio的输出路径可能变更）下的，每次输出后都扔到手机里很麻烦。我们可以通过copy命令将输出的文件复制到想要的路径中，节约人力成本。

```
android.libraryVariants.all { variant ->
    variant.outputs.all { output ->
        if (output.outputFile != null
                && output.outputFile.name.endsWith('.aar')
                && output.outputFile.size() != 0) {

            copy {
                from output.outputFile
                into "${rootDir}/libs/"
            }
        }
    }
}
```

这里的**${rootDir}**关键字是项目的跟路径。通过这个路径变量，我们可以屏蔽不同开发者电脑路径不同产生的差异性。

## 编译提速
gradle笨重缓慢的build速度令人发指，除了提升cpu、ram、ssd外，还可以使用以下方式提速。
### 使用新版gradle 依赖接口替代老的
android studio 3 中开始使用新版的依赖 接口，一般状况下 对编译提速有很大的帮助。
新旧 接口 对应关系。
* compile 替换为 implementation，
* provided 替换为 compileOnly，
* releaseCompile 替换为 releaseImplementation
* debugCompile 替换为 debugImplementation，
* testCompile 替换为 testImplementation

对于一些跨层次依赖(app module中使用了了辅助module的依赖库中的接口)，此时使用implementation 代替 compile 会报错，你可以使用api 来替代 compile ，同compile相比， 除了名称，功能几乎一样。


```
dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    implementation "com.android.support:appcompat-v7:$rootProject.ext.supportV7_Ver"
```
至于为什么为提速？
当本module依赖的lib(也可以是module)发生变化时，由于本module对外暴露的接口并不发生变化，在构建工程时gradle将会只重新编译本module，所有依赖于本module的module并不会发生编译。
详细的讲解情况可以参考这里：[安卓工程依赖方式：Implementation vs API dependency](http://blog.csdn.net/cysion1989/article/details/73442034)
### 跳过不必要的Task
我们在项目构建的过程中会有很多的task，有些是Android默认的，有些是我们自定义的。在很多时候我们并不需要运行test相关的Task，我们可以通过**task.enable**来强制跳过它。

这里有两个配置：全局统一配置、局部配置。一般直接用统一配置，如果某些module需要定制，那就在每个module中写。
**全局配置**
~~~
// Top-level build file where you can add configuration options common to all sub-projects/modules.
apply from: "config.gradle"
apply from: "configpersonal.gradle"
buildscript {
    repositories {
        //maven 私服地址，如不能使用，尝试切换到maven.aliyun
        maven { url 'http://10.10.4.43:8083/repository/maven-public/' }
//        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'
    }
}



allprojects {
    repositories {
        //maven 私服地址，如不能使用，尝试切换到maven.aliyun
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }

        flatDir {
            dirs '../Library/libs'
        }
    }

    //全局配置，移除不必要的gradle task，加快编译
    if (rootProject.ext.isRemoveSomeTask) gradle.taskGraph.whenReady {
        tasks.each { task ->
            if (task.name.contains("lint")
                    //如果instant run不生效，把clean这行干掉
                    //||task.name.equals("clean")
                    //如果项目中有用到aidl则不可以舍弃这个任务
                    || task.name.contains("Aidl")
                    //用不到测试的时候就可以先关闭
                    || task.name.contains("mockableAndroidJar")
                    || task.name.contains("UnitTest")
                    || task.name.contains("AndroidTest")
                    //用不到NDK和JNI的也关闭掉
                    || task.name.contains("Ndk")
            //|| task.name.contains("Jni")
            ) {
                task.enabled = false
            }
        }
    }
}
~~~

***局部配置**
将以下代码放到每个module中的 android{} 内

```
android {

    //自定义任务 减少build时间
    tasks.whenTaskAdded { task ->
        if (task.name.contains("lint")
                //如果instant run不生效，把clean这行干掉
                //||task.name.equals("clean")
                //如果项目中有用到aidl则不可以舍弃这个任务
                ||task.name.contains("Aidl")
                //用不到测试的时候就可以先关闭
                ||task.name.contains("mockableAndroidJar")
                ||task.name.contains("UnitTest")
                ||task.name.contains("AndroidTest")
                //用不到NDK和JNI的也关闭掉
                || task.name.contains("Ndk")
                || task.name.contains("Jni")
        ) {
            task.enabled = false
        }
    }
}
```

之前：

![image_1bp8i68921q101o70pk929a12221g.png-162.8kB][5]

之后：

![image_1bp8iah1jv618c53561vu310di3d.png-174.5kB][6]

每一个Task都有inputs和outputs，如果在执行一个Task时，如果它的输入和输出与前一次执行时没有发生变化（通过快照来判断），那么Gradle便会认为该Task是没变的，Gradle将不予执行，这就是所谓的增量构建。为了更好的说明这点，我们可以定义一个查看输出/输出详细信息的Task：

```
gradle.taskGraph.afterTask { task ->
    StringBuffer taskDetails = new StringBuffer()
    taskDetails << """"-------------\nname:$task.name"""
    taskDetails << "\nInputs:\n"
    task.inputs.files.each{ inp ->
        taskDetails << " ${inp}\n"
    }
    taskDetails << "Outputs:\n"
    task.outputs.files.each{ out ->
        taskDetails << " ${out.absolutePath}\n"
    }
    println taskDetails
}
```

示例：
```
-------------
:lib:compileDebugRenderscript UP-TO-DATE
"-------------
name:compileDebugRenderscript
Inputs:
 D:\studio\kaleExample\lib\src\main\rs
 D:\studio\kaleExample\lib\src\debug\rs
Outputs:
 D:\studio\kaleExample\lib\build\intermediates\rs\debug\lib
 D:\studio\kaleExample\lib\build\intermediates\rs\debug\obj
 D:\studio\kaleExample\lib\build\generated\res\rs\debug
 D:\studio\kaleExample\lib\build\generated\source\rs\debug
```

顺便一提，一个任务如果没有定义输出的话, 那么Gradle永远都没用办法判断是UP-TO-DATE。

### 禁用lint
todo: 暂时可以看这里[禁用lint来解决](http://www.jianshu.com/p/326c91e344a8)

### 定制脚本
todo：可以参考 自带脚本installDebug的实现  ，来写一个相同的

### 抽离Task脚本

一个成熟的项目必然有着庞大的app.gradle，脚本多了自然就想要抽离出去。我们可以建立一个xxx.gradle来存放这些脚本，引用者只需要通过**apply from**依赖即可。

taskcode.gradle：

```
buildscript {
    repositories {
        jcenter()
    }
}

ext.autoVersionName = { ->
    def branch = new ByteArrayOutputStream()
    exec {
        commandLine 'git', 'rev-parse', '--abbrev-ref', 'HEAD'
        standardOutput = branch
    }
    def cmd = 'git describe --tags'
    def version = cmd.execute().text.trim()

    return branch.toString().trim() == "master" ? version :
            version.split('-')[0] + '-' + branch.toString().trim() // v1.0.1-dev
}


ext.autoVersionCode = {
    def cmd = 'git tag --list'
    def code = cmd.execute().text.trim()
    return code.toString().split("\n").size()
}


tasks.whenTaskAdded { task ->
    if (task.name.contains('AndroidTest')) {
        task.enabled = false
    }
}


android {
    applicationVariants.all { variant ->
        variant.assemble.doLast {
            //If this is a 'release' build, reveal the compiled apk in finder/explorer
            if (variant.buildType.name.contains('release')) {
                def path = null
                variant.outputs.each { output ->
                    path = output.outputFile
                }
                if (path != null) {
                    if (System.properties['os.name'].toLowerCase().contains('mac os x')) {
                        ['open', '-R', path].execute()
                    } else if (System.properties['os.name'].toLowerCase().contains('windows')) {
                        ['explorer', '/select,', path].execute()
                    }
                }
            }
        }
    }
}
```

build.gradle：

```
apply plugin: 'com.android.application'
// ...
apply from: 'taskcode.gradle'
```

这样配置后，上面的的**taskcode.gradle**中的脚本就能和之前一样引用进来了，十分方便。

## 动态定制

### 动态设置BuildConfig

在测试开发阶段，开发人员并不会修改老的版本号，每次打包提测给不同的测试人员的时候就会遇到不知道当前是在什么节点上的包。这种问题十分难查，有时候是开发人员自己失误没有merge，有时候是测试人员失误打错包了。

为了解决这个问题，我们可以在App的详情页面增加一个commit值，让测试和开发人员可以迅速定位当前包的节点。

![image_1bpauii7v1c6hgv11oaj15immnh4k.png-39.2kB][7]

项目中的BuildConfig文件随着编译环境的不同BuildConfig的内容也是不同的，所以我们可以利用它来做一些事情。

```
/**
 * Automatically generated file. DO NOT MODIFY
 */
package com.example.kale;

public final class BuildConfig {
  public static final boolean DEBUG = Boolean.parseBoolean("true");
  public static final String APPLICATION_ID = "com.example.kale";
  public static final String BUILD_TYPE = "debug";
  public static final String FLAVOR = "dev";
  public static final int VERSION_CODE = 1;
  public static final String VERSION_NAME = "1.0";
}
```

我们可以看到这里有构建的Type和Flavor，VersionCode和VersionName也是可以直接拿到。Gradle提供了一个buildConfigField的DSL，在编译的时候我们可以直接设置其中的一些参数，从而在项目中利用这些参数进行逻辑判断。

```
android {
   defaultConfig {
        // String中的引号记得加转义符
        buildConfigField 'String', 'API_URL', '"http://www.kale.com/api"'
        buildConfigField "boolean", "IS_FOR_TEST", "true"
        buildConfigField "String" , "LAST_COMMIT" , "\""+ revision() + "\""
        
        resValue "string", "build_host", hostName()
    }
}

def hostName() {
    return System.getProperty("user.name") + "@" + InetAddress.localHost.hostName
}

def revision() {
    def code = new ByteArrayOutputStream()
    exec {
        // 执行：git rev-parse --short HEAD
        commandLine 'git', 'rev-parse', '--short', 'HEAD'
        standardOutput = code
    }
    return code.toString().substring(0, code.size() - 1) // 去掉最后的\n
}
```

BuildConfig的参数会变成静态变量：

```
public final class BuildConfig {
  public static final boolean DEBUG = Boolean.parseBoolean("true");
  public static final String APPLICATION_ID = "com.example.kale";
  public static final String BUILD_TYPE = "debug";
  public static final String FLAVOR = "dev";
  public static final int VERSION_CODE = 1;
  public static final String VERSION_NAME = "1.0";
  
  // Fields from default config.
  public static final String API_URL = "http://www.kale.com/api";
  public static final boolean IS_FOR_TEST = true;
  public static final String LAST_COMMIT = "2b07344";
}
```

res生成的参数会在**build/generated/res/resValue/.../generated.xml**中看到，在代码中可以通过getString来拿到：

![image_1bpau5ijj64rdr217n11mf91ii047.png-25.4kB][8]

现在我们可以通过LAST_COMMIT来拿到最近的commit的SHA，并且在有多个测试Flavor的时候，可以通过IS_FOR_TEST来判断了。

### 填充Manifest中的值

我们在开发第三方库的时候可能需要根据引用者的不同来定义Manifest中的值，但是Manifest本身就是一个写死的xml文件，并非拥有Java类那种灵活性。比如我开发一个了第三方登录分享的库（[ShareLoginLib](https://github.com/tianzhijiexian/ShareLoginLib/)），这个库的Manifest中必须配置一个腾讯的id，但是这个id肯定是根据使用的app来定义的。这就是变和不变的矛盾，为了解决这个问题，我希望将变化的部分抽离出去，让Mainfest中的变化元素变成变量。

[[代码地址]](https://github.com/tianzhijiexian/ShareLoginLib/blob/master/share/src/main/AndroidManifest.xml#L71)

```xml
<!-- 腾讯的认证activity -->
<activity
    android:name="com.tencent.tauth.AuthActivity"
    android:launchMode="singleTask"
    android:noHistory="true"
    >
    <intent-filter>
        <!-- 仅仅是用来占位的key -->
        <data android:scheme="${tencentAuthId}" />
    </intent-filter>
</activity>
```

我们用**${tencentAuthId}**来做占位，使用者在编译的时候会动态设置**tencentAuthId**这个的值：

[[代码地址]](https://github.com/tianzhijiexian/ShareLoginLib/blob/master/app/build.gradle#L35)

```
defaultConfig {
    manifestPlaceholders = [
            "tencentAuthId": "tencent123456",
    ]
}
```

如果你想要一次性填充某些或者所有Flavor的Apk中的Manifest，使用遍历是最快速的方案：

```
android {
    // ...
    productFlavors {
        google {
        }
        baidu {
        }
    }
    productFlavors.all { flavor ->
        manifestPlaceholders.put("UMENG_CHANNEL",name)
    }
}
```

通过这种方式我们就可以把所有的灵活改变的东西都变为动态配置，让本身很死板的xml文件具有动态化的特性。

### 让BuildType支持继承

一个复杂项目的buildType是很多的。如果我们想要新增加一个buildType，又想要新的buildType继承之前配置好的参数，那么用**init.with()**就很适合了：

```
buildTypes {
    release {
        zipAlignEnabled true
        minifyEnabled true
        shrinkResources true // 是否去除无效的资源文件
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.txt'
        signingConfig signingConfigs.release
    }
    
    rtm.initWith(buildTypes.release) // 继承release的配置
    rtm {
        zipAlignEnabled false // 覆盖release中的一些配置
    }
}
```

### 让Flavor支持继承

在开发的过程中，我们有开发版本、提测版本、内部测试版本、预发版本、正式版本等多个版本。为了标识这些版本，我们就需要用到Flavor来区分了。

Flavor也是一个多维度的，可以类比为中国-上海-黄埔，变为渠道就是：
```
flavorDimensions "china", "shanghai", "huangpu" // 按照先后进行排序
```
每个Flavor可以定义自己属于的维度：
```
flavorDimensions "china", "shanghai", "huangpu"

productFlavors {
    country {
        dimension "china"
    }
    city {
        dimension "shanghai"
    }
    town {
        dimension "huangpu"
    }
```

![image_1bpb67drbrpg4e3m6f1hulcvk51.png-18.8kB][9]

说的实际一点：
> [是否免费]+[渠道]+[针对用户]+[Debug/Release]

```
flavorDimensions("isfree", "channel", "sex")

productFlavors {
    // 是否免费的维度
    free { dimension "isfree" }
    paid { dimension "isfree" }

    // 渠道维度
    googleplay { dimension "channel" }
    wandoujia { dimension "channel" }

    // 用户维度
    male { dimension "sex" }
    female { dimension "sex" }
}
```

![image_1bpb9n0mlnn91hu718gc80ubbp8m.png-38.3kB][10]

这其实就是间接实现了Flavor的继承，有了这种维度的帮助，我们可以实现父Flavor做通用配置，子Flavor做差异化配置。比方说我们有多个内部测试渠道，但对于开发者来说内部测试的代码都是几乎一样的，所以只需要一个**IS_FOR_TEST**的变量来标识：

```
forJackTest {
    buildConfigField "boolean", "IS_FOR_TEST", "true"

}
forTonyTest {
    buildConfigField "boolean", "IS_FOR_TEST", "true"

}
forSamTest {
    buildConfigField "boolean", "IS_FOR_TEST", "true"
}
```

这种重复写多次的变量肯定是有更优解的，而**dimension**就给了我们一个优雅的处理方式：

```
flavorDimensions("innertest", "channel")

productFlavors {
    innertest{
        dimension "innertest"
        buildConfigField "boolean", "IS_FOR_TEST", "true"
    }
    forJackTest {
        dimension "channel"
    }
    forTonyTest {
        dimension "channel"
    }
    forSamTest {
        dimension "channel"
    }
}
```

![image_1bpfi2buhvi71g28sdcmps1eod9.png-26.7kB][11]

在Java代码中我们可以很容易的进行这种二维的判断：

```
if (BuildConfig.IS_FOR_TEST) {
    switch (BuildConfig.FLAVOR_channel) {
        case "forJackTest":
            break;

        case "forTonyTest":
            break;

        case "forSamTest":
            break;
    }
}
```

**题外话：**
> FLAVOR_channel是系统自动生成的，FLAVOR_channel这种大小写混合的写法特别奇怪，但官方推荐的flavorDimensions中定义的都是小写字母，所以这点可以暂时不用管它。

### 测试App有独特Icon

测试人员的手机经常被借来借去，他们很难知道当前手机上的包是否是他们想要的版本。为了方便测试人员区分包和避免测错包的情况，我们希望开发版本和测试版本的图标和app的名字是不同的，这样一眼就可以分辨出是正式包还是测试包了。

![image_1bpb8ners1nov10ai108c1nlrg287s.png-252.1kB][12]

不同的Flavor在目录结构中映射不同的文件夹。我们可以在src中建立以Flavor命名的包，然后在里面做一些某个Flavor私有的操作.

![image_1bpfjitgi1vap12fl1kb6ihm1o6f1g.png-33.2kB][13]

为了区别于正式包的Icon，我们得给InnerTest建立自己私有的启动Icon：

![image_1bpfjcjga5bi14cs11ar17cm9ci13.png-89.2kB][14]

在FemaleApplication这个类推荐继承自原始App的Application类，它只需要做自己的差异化工作就好：

```
package com.example.kale;

public class InnerTestApplication extends MyApplication {

    @Override
    public void onCreate() {
        super.onCreate();
        // do something for test
        
        // 差异化工作，比如Debug功能
        Stetho.initialize(
                Stetho.newInitializerBuilder(this)
                        .enableDumpapp(Stetho.defaultDumperPluginsProvider(this))
                        .enableWebKitInspector(
                                Stetho.defaultInspectorModulesProvider(this)).build());
        HttpHelper.getInstance().openDebugMode();
    }
}
```

然后，在Manifest中我们可以进行需要项目的替换：
```
<?xml version="1.0" encoding="utf-8"?>
<manifest package="com.example.kale"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    >

    <application
        android:name=".GooglePlayApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="GooglePlay-Example"

        tools:replace="android:name,android:icon,android:label"
        />

</manifest>
```

很多时候我们可以把那些不必打包到正式版本中的类定义在InnerTest的src下，这样测试环境可以引用而且有时候还可以省去了no-op的依赖。这种方法可玩性很大，大家可以多多思考把玩。

### 不同渠道不同包名

```
flavorDimensions("innertest", "channel")

productFlavors {
    innertest {
        dimension "innertest"
        applicationIdSuffix '.test' // 包名后缀
        buildConfigField "boolean", "IS_FOR_TEST", "true"
    }
    dev {
        dimension "channel"
        applicationIdSuffix '.dev' // 包名后缀
        minSdkVersion 21 // 设置某个Flavor的minSdkVersion
        // ...
        versionNameSuffix "-minApi21" // 版本名后缀

    }
}
```

以dev渠道为例，通过**applicationIdSuffix**可以给原始包名增加后缀，通过**versionNameSuffix**可以给原始版本名字增加后缀。

最终得到：

名称| 内容 | 来源
--|--|--
基础包名 | com.example.kale | applicationId的值
包名|com.example.kale.test.dev |test来自innertest，dev来自channel
版本名| 1.0-minApi21| -minApi21来自dev

不同包名不同的签名就决定了不同的App，通过这种方式我们可以让手机上同时安装测试版本和正式版本。因为后缀会根据Flavor的维度层层添加，所以我们甚至可以**把基本包名定为com或org，然后根据输出的方案拼接包名**，大大增加了打包的灵活性。

对于某些不想打包的Flavor或者维度，我们可以利用variantFilter进行操作，下面的代码会将“minApi21”和“demo”的类型直接跳过：

```
android {

 buildTypes {...}

 flavorDimensions "api", "mode"
 productFlavors {
    demo {...}
    full {...}
    minApi24 {...}
    minApi23 {...}
    minApi21 {...}
  }

  variantFilter { variant ->
    def names = variant.flavors*.name
    // To check for a build type instead, use variant.buildType.name == "buildType"
    if (names.contains("minApi21") && names.contains("demo")) {
      // Gradle ignores any variants that satisfy the conditions above.
      setIgnore(true)
    }
  }
}
```

### 自动升级版本号

#### 自动填写versionName

我们知道build.gradle中管理了versionCode和versionName：

```
android {  
    // ...
    defaultConfig {
        // ...
        versionCode 1
        versionName "1.0"
    }
}
```
versionName和git的tag是有相关性的，我们希望可以将每次的tag和当前的versionName进行关联，实现自动化设置版本名称的功能。

![image_1bpi867g2oie1t21bl71vi8qbam.png-8.8kB][15]

我们执行**git describe --tag**就可以看到最近的一次tag，所以靠这个就可以实现自动版本名了：
```
def autoVersionName() {
    def cmd = 'git describe --tags'  
    def version = cmd.execute().text.trim()
    return version.toString()
}
```

如果不是在master分支，那么得到的结果就可能是：v1.0.1-1-g0cb4465，所以我们可以处理一下：

```
def autoVersionName() {
    def branch = new ByteArrayOutputStream()
    exec {
        commandLine 'git', 'rev-parse', '--abbrev-ref', 'HEAD'
        standardOutput = branch
    }
    def cmd = 'git describe --tags'
    def version = cmd.execute().text.trim()
    
    return branch.toString().trim() == "master" ? version : 
        version.split('-')[0] + '-' + branch.toString().trim() // v1.0.1-dev
}
```

**题外话：**
> 上面是tag和verionName完全相同时的例子，如果你的tag和versionName不同，你可以在autoVersionName()利用String的Api对原始的tag进行处理，处理后返回即可。

#### 实现versionCode自增

有了通过tag来映射versionName的经验后，我们可以考虑通过tag数量来映射versionCode。每一次发版我们就会打一个tag，tag的数量也会增加1个，和我们版本号的递增逻辑是符合的。

```
def autoVersionCode() {
    def cmd = 'git tag --list'  
    def code = cmd.execute().text.trim()
    return code.toString().split("\n").size()
}
```

最终结果：

```
android {  
    // ...
    defaultConfig {
        // ...
        versionCode autoVersionCode() // 4
        versionName autoVersionName() // v1.0.1
    }
}
```

这里有一点需要注意，打tag是在即将发版的时候才进行的，如果我们想要在调试的时候先升级一下versionCode的话，那肯定不能走这套自动化方案。在实际中我推荐在dev的Flavor中将版本名和版本号手动填写，在正式版中用tag做自动化处理。最后再写个脚本在打tag的时候自动修改开发版本的versionCode，一切都变得轻松许多。

### 隐藏Release签名

```
signingConfigs {
    storeFile file('../test_key.jks')
    storePassword 'test123'
    keyAlias 'kale'
    keyPassword 'test123'
}
```

通常情况下我们是这么配置签名的，但这样的话安全性就是一个问题。签名会被自动提交到git仓库中，所有仓库的只读权限的人员都可以看到，十分不安全。我们的目标应该是少数人掌握release的签名，所有人可以有debug版本的签名。这样的话，在别的部门想要看下这个工程的代码做参考的时候，项目组长就可以放心大胆的给别人开权限了。

很多人推荐在**gradle.properties**中存放配置信息：
```
STORE_FILE_PATH ../test_key.jks
STORE_PASSWORD test123
KEY_ALIAS kale
KEY_PASSWORD test123
```
```
signingConfigs {
    release {
        storeFile file(STORE_FILE_PATH)
        storePassword STORE_PASSWORD
        keyAlias KEY_ALIAS
        keyPassword KEY_PASSWORD
    }
}
```

但是gradle.properties也是被git管理的文件，如果你ignore掉了gradle.properties，就会出现文件找不到的错误，所以我**强烈不建议在gradle.properties中存放正式版的签名信息**。

我们可以参考[ShareLoginLib](https://github.com/tianzhijiexian/ShareLoginLib)的方式，可以建立一个**signing.properties**的文件，然后在里面写上信息：

```
STORE_FILE_PATH = ../signing.keystore
STORE_PASSWORD = jack2017
KEY_ALIAS = jack
KEY_PASSWORD = jack@2017
```

在**gradle.properties**中写上debug的签名：

```
STORE_FILE_PATH ../test_key.jks
STORE_PASSWORD test123
KEY_ALIAS kale
KEY_PASSWORD test123
```

build.gradle：

```
Properties props = new Properties()
File f = file(rootProject.file("signing.properties"))

// 如果这个签名文件存在则用，如果不存在就从gradle.properties中取
if (!f.exists()) {
    f = file(rootProject.file("gradle.properties"))
}
props.load(new FileInputStream(f))

android {
    signingConfigs {
        release {
            storeFile file(props['STORE_FILE_PATH'])
            storePassword props['STORE_PASSWORD']
            keyAlias props['KEY_ALIAS']
            keyPassword props['KEY_PASSWORD']
        }
    }
}
```

因为signing.properities是被ignore掉的，所以这个文件不会被git管理，增加了签名的可控性。

还有一种写法是将签名放入环境变量：

```
android {
    // ...
    signingConfigs {
        def appStoreFile = System.getenv("STORE_FILE")
        def appStorePassword = System.getenv("STORE_PASSWORD")
        def appKeyAlias = System.getenv("KEY_ALIAS")
        def appKeyPassword = System.getenv("KEY_PASSWORD")
        
        // 四要素中的任何一个没有获取到，就使用默认的签名信息
        if(!appStoreFile||!appStorePassword||!appKeyAlias||!appKeyPassword){
            appStoreFile = "debug.keystore"
            appStorePassword = "android"
            appKeyAlias = "androiddebugkey"
            appKeyPassword = "android"
        }
        release {
            storeFile file(appStoreFile)
            storePassword appStorePassword
            keyAlias appKeyAlias
            keyPassword appKeyPassword
        }
    }
}
```

这里用到了System.getenv()方法，你可以参考java中System下的getenv()来理解，就是当前机器的环境变量。比如**System.getenv().get("ADB")**在我机器上得到的就是：H:\Android\sdk\platform-tools;H:\Android\sdk\tools。



### 自动打开apk的目录

开发人员通常情况下是不Build Apk的，开发都是直接run app，但是测试人员经常要编译各种版本，很少去run app。Android Studio在每次生成apk后都会提示去打开本地目录，那么我们能否在编译成功Release版本的时候自动打开本地目录呢？

![image_1bpi4qa3eblt1gu71n6a15olb1i13.png-23kB][16]

![image_1bpi4p7gj1q1m1nkba0abc6fko9.png-9.5kB][17]

下面的脚本通过**applicationVariants**来监听apk生成的时机，如果是Rlease版本就打开文件管理器：

```
android {
    applicationVariants.all { variant ->
        variant.assemble.doLast {
            //If this is a 'release' build, reveal the compiled apk in finder/explorer
            if (variant.buildType.name.contains('release')) {
                def path = null
                variant.outputs.each { output ->
                    path = output.outputFile
                }
                if (path != null) {
                    if (System.properties['os.name'].toLowerCase().contains('mac os x')) {
                        ['open', '-R', path].execute()
                    } else if (System.properties['os.name'].toLowerCase().contains('windows')) {
                        ['explorer', '/select,', path].execute()
                    }
                }
            }
        }
    }
}
```
### 组件化相关的配置
在组件化的模式中，我们每个业务部门都是一个独立的项目，所以在独自开发的时候我们的项目都是一个独立的App；在需要整体打包测试的时候就需要每个部门的项目变成library了。通过上述的代码，我们可以很轻易的用配置文件的方式进行环境切换，更方便进行CI的配置化处理。

除了在build.gradle中定义全局变量外，我们还可以在**gradle.properties**中定义变量：

```
# When configured, Gradle will run in incubating parallel mode.
# This option should only be used with decoupled projects. More details, visit
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
# org.gradle.parallel=true

isFusion = false
```

使用时：

```
if (!isFusion.toBoolean()) {
    apply plugin: 'com.android.application'
} else {
    apply plugin: 'com.android.library'
}
```



## 远程依赖

### 配置仓库

Gradle管理依赖是它的一大特点，想当年还在Eclipse时代的时候，所有的依赖都必须打包成jar，资源文件还得依次复制到工程中，十分难以管理。

无论是远程依赖还是本地依赖，配置依赖的仓库总是我们的第一步：

```
buildscript {
  repositories {
    maven { url 'https://maven.fabric.io/public' }
  }

  dependencies {
    // These docs use an open ended version so that our plugin
    // can be updated quickly in response to Android tooling updates

    classpath 'io.fabric.tools:gradle:1.+'
    classpath 'com.antfortune.freeline:gradle:0.8.'
    classpath 'me.tatarka:gradle-retrolambda:3.2.5'
  }
}
```

配置多个maven仓库：
```
allprojects {
    repositories {
        jcenter()
        maven {
            url="http://maven.mbd.qiyi.domain/nexus/content/repositories/mbd-vertical/"
        }
        maven {
            url "https://jitpack.io"
        }
        maven {
            url 'http://repo.xxxx.net/nexus/'
            name 'maven name'
            credentials {
                username = 'username'
                password = 'password'
            }
        }
    }
}
```

其中name和credentials是可选项，视具体情况而定。
### 使用nexus私服 加速 依赖
当我们添加新的依赖时，gradle或去中央仓库download相关的文件到本地，众所周知的原因，编译时下载依赖的网速又着实令人蛋疼不已。除了一些特殊的穿越技巧外，更推荐使用nexus私服。
这方面，有条件的公司可以自己搭建nexus私服，也可以使用国内的Maven镜像仓库，如[~~开源中国的Maven库~~](http://maven.oschina.net/index.html)(已停止维护），或[阿里云的Maven服务](http://maven.aliyun.com/nexus/)，又或者是换成自建的Maven私服，那想必是极好的。

一个简单的办法，修改项目根目录下的build.gradle，将`jcenter()`或者`mavenCentral()`替换掉即可：

```
allprojects {
    repositories {
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public/'}
    }
}
```

但是架不住项目多，难不成每个都改一遍么？

自然是有省事的办法，将下面这段Copy到名为`init.gradle`文件中，并保存到 `USER_HOME/.gradle/`文件夹下即可。

```
allprojects {
    repositories {
        def REPOSITORY_URL = 'http://maven.aliyun.com/nexus/content/groups/public/'
        all { ArtifactRepository repo ->
            if (repo instanceof MavenArtifactRepository && repo.url != null) {
                def url = repo.url.toString()
                if (url.startsWith('https://repo1.maven.org/maven2') || url.startsWith('https://jcenter.bintray.com/')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $REPOSITORY_URL."
                    remove repo
                }
            }
        }
        maven {
            url REPOSITORY_URL
        }
    }
}
```

`init.gradle`文件其实是Gradle的`初始化脚本`(Initialization Scripts)，也是运行时的全局配置。
更详细的介绍请参阅 [http://gradle.org/docs/current/userguide/init_scripts.html](http://gradle.org/docs/current/userguide/init_scripts.html)

目前已知、可用nexus私服：
* 阿里云：http://maven.aliyun.com/nexus/content/groups/public/

### 基础Api

![image_1bpi8nt3m1ulotlo160ac7lc8q2d.png-82.7kB][18]

Gradle提供了多种依赖方式的Api：

配置 | 解释
-- | --
api|编译时依赖和运行时依赖
implementation | 基础依赖方式，运行时依赖，对于依赖结构做了优化
compileOnly | 类似于**provided**，仅仅在编译时进行依赖，不会将依赖打包到app中
runtimeOnly | 类似于**apk**，它仅仅将依赖打包到apk中，在编译时无法获得依赖的类
annotationProcessor | 类似于**apt**，是注解处理器的依赖
testImplementation | Java测试库的依赖，仅仅在测试环境生效
androidTestImplementation | Android测试库的依赖，仅仅在测试环境生效
[Flavor]Api | 针对于某个Flavor的依赖，写法是Flavor的名称+依赖方式

举例：
```
dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    // 基础依赖方式
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jre7:$kotlin_version"
    implementation 'com.android.support:appcompat-v7:' + rootProject.ext.support_version
    implementation 'com.android.support.constraint:constraint-layout:1.0.2'
    
    // 依赖注解，注解处理器不会打包到apk中
    compileOnly 'com.baoyz.treasure:treasure:0.7.4'
    annotationProcessor 'com.baoyz.treasure:treasure-compiler:0.7.4'
    annotationProcessor 'com.google.dagger:dagger-compiler:<version-number>'
    
    // buildTypes是debug的时候才能被依赖
    debugImplementation 'com.github.nekocode.ResourceInspector:resinspector:0.5.3'
    
    testImplementation 'junit:junit:4.12'
    
    androidTestImplementation 'com.android.support.test:runner:1.0.0'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.0'
    
    // 编写时无法访问lib中的资源，lib会被打包到apk中
    runtimeOnly project(':lib')
}
```

### 组合依赖

有时候一些库是一并依赖的，删除的时候也是要一并剔除的，如果像上面一样多条引用的话，很容易不知道哪些库是要一并删除的。为了解决这个问题，我们可以像下面这样进行统一引入：

```
implementation([
        'com.github.tianzhijiexian:logger:2e5da00f0f', // logger和timber总是结合使用的
        'com.jakewharton.timber:timber:4.1.2'
])
```

这样整合起来的库就成了一组，开发者一眼就知道这些库是有相关性的，在删除库的时候十分方便。

```
implementation([
    'io.reactivex.rxjava2:rxjava:2.1.3', 
    'io.reactivex.rxjava2:rxandroid:2.0.1'
])
```

rxandroid本身是自带rxjava的依赖的，但是rxjava的升级很快，rxandroid十分稳定，几乎不怎么升级。在保证Api稳定的前提下，我们通过这种聚合依赖的方式可以很方便升级rxjava，让核心代码的升级不被rxandroid限制。

### 依赖传递

我们配置Crashlytics的依赖的时候一般会这样写：

```
api('com.crashlytics.sdk.android:crashlytics:2.6.8@aar') {
    transitive = true;
}
```

为什么要写**transitive = true**呢？其实@符号的作用是仅仅下载文件本身，不下载它自身的依赖，等于关闭了以来传递。如果你要支持依赖传递，那么就必须要写**transitive = true**。

更多配置方案可参考：[Crashlytics for Android \- Fabric Install](https://fabric.io/kits/android/crashlytics/install)

### 动态版本号(不推荐)


如果想要自己的依赖库永远保持最新版本，那么就可以利用**版本名+**或**-SNAPSHOT**的方式来做：

```
implementation 'com.android.support:appcompat-v7:23.0.+'

implementation 'com.kale.business:CommonAdapter:1.0.6-SNAPSHOT'
```

同时还要记得开启offline功能：

![image_1bpialnaf9g51jfi1umu1vjp1ubf2q.png-84.4kB][19]

Gradle默认24小时自动检查一次更新，我们可通过resolutionStrategy来修改检查周期：

```
configurations.all {
    // check for updates every build
    resolutionStrategy.cacheDynamicVersionsFor 0, 'seconds'
    resolutionStrategy.cacheChangingModulesFor 0, 'seconds'
}
dependencies {
	implementation 'com.android.support:appcompat-v7:23.0.+'
    implementation 'com.kale.business:CommonAdapter:1.0.6-SNAPSHOT'
}
```

在实际中我**不建议采用动态版本的方式做依赖**。动态依赖就必然会出现版本不稳定的情况，你无法确定所有项目组的成员是否都保持了一致的依赖版本，而且一旦依赖版本的最新版出现了bug，你会不自觉的将bug引入进来，十分难以排查。对于这种不受到版本控制系统管理的危险方案，请不要随意尝试。
此外，每次run时 都会向服务器请求最新版本，当前的网络环境会让这一过程漫长的令人发指。

### 强制版本号

有时候第三方的lib中用到了很高版本的support包，而那个高版本的support包可能有一个bug，我们肯定不想因为它而引入这个bug。事实上Gradle的默认机制是有高版本则用高版本，这就让我们处于了一种进退两难的境地。幸好，**configurations**提供了强制约束库版本的能力。

我们先在根build.gradle中配置一个task：

```
subprojects {
    task allDeps(type: DependencyReportTask) {}
}
```
使用命令行**gradlew alldeps**得到输出：

![image_1bpnccniu9oi1ffr15869q5b8k19.png-146.9kB][20]


配置强制的support版本号：

```
android {
    configurations.all {
        // 指定某个库的版本
        resolutionStrategy.force "com.android.support:appcompat-v7:25.4.0"
        
        // 一次指定多个库的版本
        resolutionStrategy {
            force 'com.android.support.test.espresso:espresso-core:3.0.0',
                     "com.android.support:appcompat-v7:25.4.0"
        }
    }
}
```

![image_1bpnch7h511ak1g66p1god8i1e1m.png-155kB][21]

此外，我们还可以通过**force**来强制指定某个库的版本号：

```
implementation group: 'com.android.support', name: 'appcompat-v7', version: '26.0.2', force: true
```

### exclude关键字

如果我们引用的库多了，各个库之间可能会出现相互引用。Gradle的默认处理是进行依赖分析的时候自动将多个相同库的最高版本定位最终依赖。但有时候会出现一个jar包打包了库A，而我们依赖的库B也有库A。在实际中，我们经常会通过**exclude**关键字来剔除某些依赖：

```
implementation('com.android.support:appcompat-v7:23.2.0') {
    exclude group: 'com.android.support', module: 'support-annotations' // 写全称
    exclude group: 'com.android.support', module: 'support-compat'
    exclude group: 'com.android.support', module: 'support-v4'
    exclude group: 'com.android.support', module: 'support-vector-drawable'
}
```

剔除整个组织的库（一下子剔除所有support库）：

```
implementation('com.facebook.fresco:animated-webp:0.13.0') {
    exclude group: 'com.android.support' // 仅仅写组织名称
}
```

exclude的参数有group和module，可以分别单独使用。如果你想要全局剔除某个库，可以在**configurations**中进行配置：
```
configurations {
   all*.exclude group: 'org.hamcrest', module: 'hamcrest-core'
}
```


顺便一提，大厂部门众多，不同部门的库很容易就出现了依赖冲突。早期Google的Espresso库就和support-annotations有冲突，所以Android官方给出了下面的方案：

```
// Espresso UI Testing
androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
    exclude group: 'com.android.support', module: 'support-annotations'
})
```
现在3.x的版本就没有这方面的问题了：
```
androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.1'
```
### 依赖冲突
强制版本号和exclude关键字  其实 提了下 依赖冲突 的处理。都是先打印依赖树，查看冲突的位置后，进行exclude。
两种方法可以用来打印依赖树：
```
//其中app是指你的module名字(gradle alldeps 也可以)，命名行运行：
gradle -q app:dependencies   
//或者 在对应 module中 apply plugin: 'project-report'，命名行运行：
gradle htmlDependencyReport
```
首先在你需要插在的module比如app的下的build.gradle 文件上加上apply plugin: 'project-report'
然后在项目的根目录下执行gradle命令./gradlew htmlDependencyReport 之后会在Build目录下面生成report文件夹，里面生成的有html，里面会有compile的标签，打开即可看到相关的依赖包情况。
搜索CompileClasspath  或者 RuntimeClasspath  查看


示例，如果多余 v4 和design包 可以这样
```
compile ('com.wdullaer:materialdatetimepicker:3.2.2') {
    exclude group: 'com.android.support', module: 'support-v4'
    exclude group: 'com.android.support', module: 'design'
}
```
也可以整个移除
```
compile ('com.android.support:design:22.2.1')
{
    exclude group: 'com.android.support'
}
```

### 动态依赖第三方库

#### 用变量判断

在Dev版本的时候我们可能会依赖很多测试库，整合很多开发插件，App很容易就突破了65535的方法数限制。但是在Release版本中方法数却很少，可能只需要一个Dex（一个Dex大概是10M）。最好的方式是我们可以通过判断当前的开发状态来决定是否需要依赖multidex这个库。

除了可以通过BuildType、Flavor来判断不同的开发环境外，我们还可以通过内部变量的逻辑判断来做：

```
def needMultidex = true

android {
    buildTypes {
        release {
            multiDexEnabled = false // 关闭multiDex
            // ...
        }
        
        debug {
            multiDexEnabled true // 开启multiDex
            // ...
        }
    }
}

dependencies {
    if (!needMultidex.toBoolean()) {
        implementation fileTree(dir: 'libs', include: ['*.jar'])
        // ....
    } else {
        implementation 'com.android.support:multidex:1.0.0'
        // ...
    }
}
```

#### 用Flavor实现

如果这里的needMultidex修改的很频繁，每次打包都需要改代码，那么这样的方案就不太合理了。在这种情况下，我建议通过Flavor来做依赖配置：

![image_1bppsjsu67kmdep10d81d801qda9.png-18.2kB][22]

```
debugImplementation 'com.android.support:multidex:1.0.0'
```

通过Flavor的方式可以让我们通过切换Build Variants的方式来切换环境，可以将环境切换和代码分开，解决硬编码的情况。但是如果我们不同环境的差异性很大，仍旧会出现不方便管理的情况。

以插件化方案举例子，我们定义两个Flavor：

```
android {
    productFlavors {
        // 插件
        plugin {
            buildConfigField "boolean", "IS_PLUGIN", "true"
        }
        // 独立App
        single {
            buildConfigField "boolean", "IS_PLUGIN", "false"
        }
    }
}

// 这里的baseLibxxx在插件的时候是不需要打包的，而独立App的时候是需要打包的
dependencies {
    pluginImplementation project(':pluginLib')
    pluginCompileOnly project(':baseLib01')
    pluginCompileOnly project(':baseLib02')
    pluginCompileOnly project(':baseLib03')
    pluginCompileOnly project(':baseLib04')

    singleImplementation 'com.android.support:multidex:1.0.0'
    singleImplementation project(':singleLib')
    singleImplementation project(':baselib01')
    singleImplementation project(':baselib02')
    singleImplementation project(':baselib03')
    singleImplementation project(':baselib04')
}
```

#### 用回变量

通过区分Flavor的方式固然可以实现根据环境来依赖不同的东西，但如果更复杂一些呢？涉及到Task呢？对于复杂的需求，我们只有通过建立判断逻辑来解决了。

增加判断逻辑的好处是省去了一个Flavor，顺便增加了动态程度：

```
// 每次切换插件/独立App模式都得要改一次代码，十分麻烦
ext { IS_PLUGIN = true; }

apply plugin: 'com.android.application'

// 判断是否引入某个插件
if (IS_PLUGIN) {
    apply plugin: "build-time-tracker"
    buildtimetracker {
    reporters {
       csv {
           output "build/times.csv"
           append true
           header false
       }

       summary {
           ordered false
           threshold 50
           barstyle "unicode"
       }

       csvSummary {
           csv "build/times.csv"
       }
    } 
}

android {
    defaultConfig {
        // 判断是否使用multiDex
        if (IS_PLUGIN) {
            multiDexEnabled false
        } else {
            multiDexEnabled true
        }
    }
    dexOptions {
        // 判断内存配置
        if (!IS_PLUGIN) {
            javaMaxHeapSize "4g"
            jumboMode = true
        }
    }
    buildTypes {
        debug {
            buildConfigField("boolean", "IS_PLUGIN", "$IS_PLUGIN")
        }
        release {
            buildConfigField("boolean", "IS_PLUGIN", "$IS_PLUGIN")
        }
    }
    sourceSets {
        main {
            // 判断资源
            if (!IS_PLUGIN) {
                jniLibs.srcDirs = ['libs']
            }
           res.srcDirs += ['src/main/res' , 'src/main/res-v7-appcompat']
        }
    }
}

dependencies {
    // 这里还是出现了大量的相似依赖，说明可以进一步的进行优化
    if (IS_PLUGIN) {
        compileOnly fileTree(dir: 'libs-common', include: ['*.jar'])
        compileOnly fileTree(dir: 'libs', include: ['*.jar'])
        compileOnly 'com.android.support:support-annotations:23.0.1'
    } else {
        implementation fileTree(dir: 'libs-compile', include: ['*.jar'])
        implementation fileTree(dir: 'libs-common', include: ['*.jar'])
        implementation(name: 'lintaar-release', ext: 'aar')

        implementation 'com.android.support:multidex:1.0.0'
    }
}
```

**优化依赖**

有些库在插件的宿主中是有的，但是调试的时候是独立的App，所以只需要在调试时依赖。为了聚合这些类似的库，我们可以将其封装为数组，最终进行一次性的依赖判断：

```
dependencies {
    ext.libs =
            ['com.baoyz.treasure:treasure:0.7.4',
            'com.squareup.okhttp3:okhttp:3.9.0',
            'io.reactivex.rxjava2:rxjava:2.1.3']

    if (isPlugin()) {
        compileOnly(libs) // 不将依赖打入App中
    } else {
        implementation(libs)
    }

    if (isPlugin()) {
        implementation project(':releaselib')
    } else {
        implementation project(':debuglib')
    }
}
```

**优化配置**

如果我们切换一次独立App/插件模式，那么**IS_PLUGIN**字段就得修改一遍。一个项目组内有些同事在调试插件模式，一些同事在调试独立App，那么每次的Git提交就很容易在这个字段上冲突。为了解决这个问题，我们可以通过Gradle的打包命令，在执行命令行的时候动态设置这个字段，让所有的修改和代码分离：

```
// 并不定义PLUGIN这个变量
//ext.PLUGIN = true

ext.isPlugin = {
    try {
        if (PLUGIN.toBoolean()) {
            return true
        }
    } catch (Exception ignore) {
    }
    return false
}
```

默认是独立App模式，执行命令行时可进行修改：

```
gradlew clean -P PLUGIN=true installInnertestDevDebug
```

![image_1bpq3onor1m5i8715l0u88mmv9.png-62.1kB][23]

总的来说，如果你的需求不是很复杂，那么推荐用Flavor的方式，如果你的需求十分复杂，对于动态化和灵活性的要求很高，那么建议通过变量的方式来做。


## 本地依赖

### 引用aar

有时候我们有部分代码需要给多个项目组共用，在不方便上传仓库的时候，可以做一个本地的aar依赖。

1.把aar文件放在某目录内，比如就放在app的libs目录内

![image_1bpkjae8p1n1l117i1jnv1k141hmn9.png-15.1kB][25]

2.在app的build.gradle文件中添加：
```
apply plugin: 'com.android.application'

repositories {
    flatDir {
        dirs 'libs' // this way we can find the .aar file in libs folder
    }
}
```

3.之后在其他项目中添加下面的代码后就引用了该aar

```
dependencies {
    // name不用加aar的后缀
    implementation(name:'lib-release', ext:'aar')
}
```

目前暂不知晓如何依赖根目录中的aar文件。

### 依赖module/jar

#### module

依赖module：

```
implementation project(':lib')
```

如果的module在多级目录中，那么首先要在settings.gradle中进行配置：

```
include ':app', ':lib', ':libraries:lib01', ':libraries:lib02'
```

![image_1bpkouc9iah616lv2go473lig2n.png-6.1kB][26]

依赖方式：

```
implementation project(':libraries:lib01')
implementation project(':libraries:lib02')
```

#### jar

依赖指定路径下的全部jar文件

![image_1bpko53ob1a6hd5ivnd1b621c5k1t.png-39kB][27]

```
dependencies {
    // 依赖当前module的libs目录下的所有jar
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    
    // 依赖外部目录，librarymodule中libs目录下的所有jar
    implementation fileTree(dir: '../somedir/libstore', include: '*.jar')
}
```
如果用这种模糊依赖的话，我们只需要把要依赖的jar放入某个目录中就好，但是这就有难以被版本控制系统管理的问题。一般情况下，**我建议通过指定依赖的方式来做**： 

```
// 依赖当前目录下的某个jar
implementation files('libs/guava-19.0.jar') // 指定依赖某个jar

// 依赖其他目录下的某个jar
implementation files('../somedir/libstore/gson-2.8.1.jar')
```

为了方便管理和维护，放入jar文件的时候记得带上版本号：

![image_1bpkn7e2vo919ik1k9ubct1g8613.png-8.8kB][28]

**题外话：**
> jar所在的目录的名字可以随便定义的，不局限于libs和是否在当前工程，见名之意即可。

### 自建仓库

除了直接依赖aar或某个module外，我们可以将自己的module变成本地依赖的方式提供出去。

一个仓库通常具有如下参数：
```
<?xml version="1.0" encoding="UTF-8"?>
<metadata>

  <groupId>com.kale.github.example</groupId>
  <artifactId>LocalLib</artifactId>
  
  <versioning>
    <release>1.1.1</release>
    <versions>
      <version>1.1.1</version>
    </versions>
    <lastUpdated>20170910025547</lastUpdated>
  </versioning>
</metadata>
```

这里的groupId和库名字一定要和公司的其余项目组协调定义，一般情况下**一个公司的库的groupId都是一致的，这个是要写入wiki的**。

可以参考Gson的信息：

![image_1bpkuj8ffenb1vta1bdf1gc8121n58.png-40.5kB][29]

#### 生成库

1.在根路径下的gradle.properties添加：

```
#org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
# When configured, Gradle will run in incubating parallel mode.
# This option should only be used with decoupled projects. More details, visit
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
# org.gradle.parallel=true

# 组织信息
GROUP_ID=com.kale.github.example

# Licence信息（一般用apache的就行）
PROJ_LICENCE_NAME=The Apache Software License, Version 2.0
PROJ_LICENCE_URL=http://www.apache.org/licenses/LICENSE-2.0.txt
PROJ_LICENCE_DEST=repo
```

2.在module（library）的build.gradle中定义发布配置：

```
apply plugin: 'com.android.library'

apply plugin: 'maven'

uploadArchives {
    repositories.mavenDeployer {
        pom.artifactId = 'LocalLib'
        pom.groupId = GROUP_ID
        pom.version = '1.1.1'
        repository(url: "file:///${rootDir}/localstorage/locallib")
    }
}
```

3.执行发布命令，等待文件生成完毕：
```
// 我演示的library叫做locallib
./gradlew -p <Library name> clean build uploadArchives --info
```

![image_1bpksi0upj281djvece1g9p1b2i34.png-9.9kB][30]

![image_1bpksmia45d21edq1tic5hens73h.png-35.7kB][31]

#### 依赖库

依赖本地库的方式将在依赖React Native的时候讲，这里直接列代码：

```
repositories {
    // ...
    maven {
        url "$rootDir/localstorage/locallib/"
    }
}

implementation 'com.kale.example:LocalLib:1.1.1'
```

顺便一提，你也可以在Libary的根目录下新建gradle.properties文件来填写配置参数：

```
ARTIFACTID = androidLib
LIBRARY_VERSION = 2.2.2

LOCAL_REPO_URL = file:///D:/kale/my/local/repo // 可以是绝对路径，但一定是file:开头的
```

在build.gradle中：

```
apply plugin: 'com.android.library'
apply plugin: 'maven'

uploadArchives{
    repositories.mavenDeployer{
        repository(url:LOCAL_REPO_URL)
        pom.groupId = GROUP_ID
        pom.artifactId = ARTIFACTID
        pom.version = LIBRARY_VERSION
    }
}
```

### 本地依赖React Native

FaceBook的React Native因为更新速度很快，在它不支持远程依赖的时候，我们可以考虑将项目作为一个仓库进行配置，而仓库的地址就是本地的目录。

1.先将库文件放入一个module的libs目录中：

![](https://user-gold-cdn.xitu.io/2016/11/29/b946c41a4c8cd065a4698f2c504071bc.png)

2.配置maven的url为本地地址：

```
allprojects {
    repositories {
        maven {
            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
            url "$rootDir/module_name/libs/android" // 路径是根据放置的目录来定的
        }
    }
}
```

3.正常使用：

```
dependencies {
    implementation 'com.facebook.react:react-native:0.32.0'
}
```

这里用到了**$rootDir**来屏蔽多个开发者机器环境的差异性，保证了项目的兼容性。

#### 依赖冲突

我们依赖本地jar的时候可能会出现jar中也打包了别的库代码的情况，如果是aar我们可以通过gradle来做处理，但在面对依赖冲突的时候，jar文件就变得令人棘手了。

[shevek/jarjar](https://github.com/shevek/jarjar)是一个再次打包工具，它可以为我们提供一次性更换包名的功能，是一个解决一来冲突的利器。

它还提供了gradle的脚本来操作你依赖的jar文件：
```
dependencies {
	// Use jarjar.repackage in place of a dependency notation.
	compile jarjar.repackage {
		from 'com.google.guava:guava:18.0'

		classDelete "com.google.common.base.**"

		classRename "com.google.**" "org.private.google.@1"
	}
}
```

这回我们尝试通过手动的方式来操作gson.jar，我们希望把原本的**com.google.gson**的包换为**com.gg.gson**。

1.先建立一个**rule.txt**的文本文件，内容：

```
rule  com.google.gson.** com.gg.gson.@1
```

2.执行命令：

```
java -jar jarjar.jar process rule.txt gson.jar gg.jar
```

执行后我们可以看到在当前目录生成了一个gg.jar的文件，分析后就可以发现其内容已经变了：

![image_1bpo183ba1jhu1u7g5ibmln2m423.png-20.9kB][32]

jarjar并不提供修改META-INF的功能，但这并不影响我们使用。

如果你想要删除特定包或特定的类，那么就在rule.txt中加入**zap**命令。

```
rule  com.google.gson.** com.gg.gson.@1

zap com.google.gson.reflect.TypeToken // 删除某个类

zap com.google.gson.stream.**

zap com.google.gson.annotations.**

zap com.google.gson.internal.**
```

原始的gson：

![image_1bpo1tnunvgabiki9jsl813m12g.png-29.3kB][33]

删除后：

![image_1bpo1u8rk179i1991qurps81igr2t.png-16.8kB][34]

除了上面提到的rule、zap外还是有keep。首先zap会删除需要删除的所有类，然后执行rule替换符合要求的类，最后如果配置了keep的话，将不符合规则的所有类的移除，只保留keep指定的包。总结来说，这三条命令的执行优先级是：zap > rule > keep。

需要注意的是：jarjar无法支持反射，**如果jar包内有使用反射调用的情况，替换操作是十分危险的**。

另一个插件[dinuscxj/ClassPlugin](https://github.com/dinuscxj/ClassPlugin)还提供了替换依赖中的类的功能，有兴趣可以尝试一下。

**题外话：**
> 对于aar文件，我们只有将aar解压后对解压的jar进行处理，最后再打包成aar。

## 更多待续。。。

## 尾注
#### 附加说明：
* **版权**。文章大部分内容 来着 网络，基于自己的使用情况，做了些许增删改动，算是一个归纳笔记，仅供自己复习或内部交流；
* **gradle适用版本**。笔记时间跨度较长，文章中并不是同一个version的android studio上测试（1.5、2.1、2.3、3.0），实际使用可能会有部分差异，但大体是可以，如果有差异的话，可以稍微google下，应该都可以解决。
* **勘误、贡献**。文章中存在的错误或者尚未提到的gradle使用技巧，欢迎加入PR来勘误补充；

#### 参考
* [gradle知识点总结分享](http://www.jianshu.com/p/d935357588bb)
* [Gradle插件用户指南(译)](http://rinvay.github.io/android/2015/03/26/Gradle-Plugin-User-Guide(Translation)/)
* [Gradle 修改 Maven 仓库地址](https://yrom.net/blog/2015/02/07/change-gradle-maven-repo-url/)
* [gradledoc 项目报告插件](http://gradledoc.qiniudn.com/1.12/userguide/project_reports_plugin.html)
* [gradledoc翻译](https://github.com/msdx/gradledoc)
* [gradle 中文用户手册](https://github.com/GradleCN/gradledoc)

  [1]: http://static.zybuluo.com/shark0017/fuy6g22wbov6hr6dm2nq9vwg/image_1bp80no6ubmgo4itssrg7kfam.png
  [2]: http://static.zybuluo.com/shark0017/ea6679q508r7cheld5c9hkgm/image_1bp7vae1vc1sbosedr1vaf1i209.png
  [3]: http://static.zybuluo.com/shark0017/7qcl5lcqbjzxwog1h18vnbxz/image_1bp8s6d521v9gk2u10gcfq31qnu3q.png
  [4]: http://static.zybuluo.com/shark0017/3am0o8lsp1c4082hvhpkpen8/image_1bp8f00691rmc1q9len5snt19n913.png
  [5]: http://static.zybuluo.com/shark0017/6884pwtvgnne7n64ajytmkot/image_1bp8i68921q101o70pk929a12221g.png
  [6]: http://static.zybuluo.com/shark0017/knc24xyxfch0hmrurhvaqjrm/image_1bp8iah1jv618c53561vu310di3d.png
  [7]: http://static.zybuluo.com/shark0017/uuzh3f48fceteeztskf2yn2u/image_1bpauii7v1c6hgv11oaj15immnh4k.png
  [8]: http://static.zybuluo.com/shark0017/k3qvytsb56sjcgdhohxshnkw/image_1bpau5ijj64rdr217n11mf91ii047.png
  [9]: http://static.zybuluo.com/shark0017/u54dmheosm4d1m7cdn2n0t6n/image_1bpb67drbrpg4e3m6f1hulcvk51.png
  [10]: http://static.zybuluo.com/shark0017/tl0jxghf9wwbiobknxkj2qhn/image_1bpb9n0mlnn91hu718gc80ubbp8m.png
  [11]: http://static.zybuluo.com/shark0017/bol18npfie0wc40s82au53n9/image_1bpfi2buhvi71g28sdcmps1eod9.png
  [12]: http://static.zybuluo.com/shark0017/wyiq7vcmyx2phn4npuy94nxi/image_1bpb8ners1nov10ai108c1nlrg287s.png
  [13]: http://static.zybuluo.com/shark0017/emufnh1e2wqseluodeoa4oag/image_1bpfjitgi1vap12fl1kb6ihm1o6f1g.png
  [14]: http://static.zybuluo.com/shark0017/0vp6ktmp2m9tws2ix0iybprx/image_1bpfjcjga5bi14cs11ar17cm9ci13.png
  [15]: http://static.zybuluo.com/shark0017/1o6pec8yj5i6fo7mwn93654j/image_1bpi867g2oie1t21bl71vi8qbam.png
  [16]: http://static.zybuluo.com/shark0017/og5eyv2460q2n814eeavg3oa/image_1bpi4qa3eblt1gu71n6a15olb1i13.png
  [17]: http://static.zybuluo.com/shark0017/3r89uk61i3dp4y2cqjaovaip/image_1bpi4p7gj1q1m1nkba0abc6fko9.png
  [18]: http://static.zybuluo.com/shark0017/rktt8zmd7ssmp7yo9j4ect6y/image_1bpi8nt3m1ulotlo160ac7lc8q2d.png
  [19]: http://static.zybuluo.com/shark0017/3zugs7m6dzlcud6zrwbzzht5/image_1bpialnaf9g51jfi1umu1vjp1ubf2q.png
  [20]: http://static.zybuluo.com/shark0017/na34vhd03qwmz2g0milxs31h/image_1bpnccniu9oi1ffr15869q5b8k19.png
  [21]: http://static.zybuluo.com/shark0017/x7rj8qs0wq5hdhd93z971w8g/image_1bpnch7h511ak1g66p1god8i1e1m.png
  [22]: http://static.zybuluo.com/shark0017/uv3na0un1t96igl38nb2gsyb/image_1bppsjsu67kmdep10d81d801qda9.png
  [23]: http://static.zybuluo.com/shark0017/xw6ziprqbf437jepmw63xuzu/image_1bpq3onor1m5i8715l0u88mmv9.png
  [24]: http://static.zybuluo.com/shark0017/61baz9yoc6amv1bsun6po4wv/image_1bpijpqvh1n23sjva9b14fp85v9.png
  [25]: http://static.zybuluo.com/shark0017/r2f3hm71dsdi4jvofmvldj7h/image_1bpkjae8p1n1l117i1jnv1k141hmn9.png
  [26]: http://static.zybuluo.com/shark0017/et59xla3vypuoknehfzxjl6z/image_1bpkouc9iah616lv2go473lig2n.png
  [27]: http://static.zybuluo.com/shark0017/8csawzbedyc48f1xspepn8ao/image_1bpko53ob1a6hd5ivnd1b621c5k1t.png
  [28]: http://static.zybuluo.com/shark0017/uoek1cgf1wx07guilsssc6hr/image_1bpkn7e2vo919ik1k9ubct1g8613.png
  [29]: http://static.zybuluo.com/shark0017/pu0cgsdr7i5g0fie6c2fs0f4/image_1bpkuj8ffenb1vta1bdf1gc8121n58.png
  [30]: http://static.zybuluo.com/shark0017/wehwhgf1myalfhtjbazigvev/image_1bpksi0upj281djvece1g9p1b2i34.png
  [31]: http://static.zybuluo.com/shark0017/4oi27gbrr9b17c1vyfnkwkbo/image_1bpksmia45d21edq1tic5hens73h.png
  [32]: http://static.zybuluo.com/shark0017/4ytqyxbs6xmufp2mf5aec4hc/image_1bpo183ba1jhu1u7g5ibmln2m423.png
  [33]: http://static.zybuluo.com/shark0017/3e1wewyzaiod5zhimd2ovkrn/image_1bpo1tnunvgabiki9jsl813m12g.png
  [34]: http://static.zybuluo.com/shark0017/03w4zw2sbuh6v52hl0jn4ye4/image_1bpo1u8rk179i1991qurps81igr2t.png
