
[原文地址](http://www.gcssloop.com/course/jitpack-sources-javadoc)

很早之前写过一篇[用JitPack发布Android开源库](http://www.gcssloop.com/course/PublishLibraryByJitPack/)的文章，有小伙伴反馈说**发布到JitPack上的开源库没有文档注释，使用起来很不方便**，这是我的失误，上一篇文章只是讲解了如何使用JitPack发布开源库，最终发布的只有arr(即编译好的动态链接库)，不仅没有文档注释(Javadoc)，也没有源码(sources)，本次就教大家如何在发布同时添加上注释和源码。

**由于JitPack本身就是一个自定义Maven仓库，所以与上传Maven的配置方式基本一样。**

### 配置项目的 build.gradle

项目的 build.gradle 配置和上一篇一样，没有变化。

~~~
buildscript { 
  dependencies {
    // 重点就是下面这一行(上面两行是为了定位这一行的添加位置)
    classpath 'com.github.dcendents:android-maven-gradle-plugin:1.3' 
~~~

### 配置 Library 的 build.gradle

完整示例(重点内容已经用注释标出):

~~~
apply plugin: 'com.android.library'
apply plugin: 'com.github.dcendents.android-maven' // 添加这个

group='com.github.GcsSloop'	// 指定group，com.github.<用户名>

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.3"

    defaultConfig {
        minSdkVersion 7
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.4.0'
}

//---------------------------------------------

// 指定编码
tasks.withType(JavaCompile) {
    options.encoding = "UTF-8"
}

// 打包源码
task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}

task javadoc(type: Javadoc) {
    failOnError  false
    source = android.sourceSets.main.java.sourceFiles
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    classpath += configurations.compile
}

// 制作文档(Javadoc)
task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}

artifacts {
    archives sourcesJar
    archives javadocJar
}
~~~

### 发布参照上一篇文章： [使用JitPack发布开源库](http://www.gcssloop.com/course/PublishLibraryByJitPack/)

### 查看在线文档

如果你在JitPack配置了文档和源码支持，在引用同时就包含了源码和文档，不仅如此，你也可以在线查看。

查看地址是:

`https://jitpack.io/com/github/USER/REPO/VERSION/javadoc/`

例如我的一个开源库：

`https://jitpack.io/com/github/GcsSloop/ViewSupport/v1.2.2/javadoc/`

在线API文档样式：![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f7o8gabelfj31400mbjy0.jpg)

* * *

> #### 如果你觉得我的文章对你有帮助的话，捐赠一些晶石，鼓励我继续研究!
> 
> ¥ 捐赠晶石 
> 
> 
> 
> ![微信](http://www.gcssloop.com/assets/images/wechat.png) ![支付宝](http://www.gcssloop.com/assets/images/alipay.png)
> 
> 感谢所有支持我的魔法师，[点击这里查看捐赠者名单。](http://www.gcssloop.com/contribute)
> 
> 

