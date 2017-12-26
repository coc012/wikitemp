之前已经整体的对组件化框架进行了概述，这篇文章只针对组件化的编译脚本的配置进行详述。

在概览的时候，我们提出了几个目标：可复用、热插拔、灵活发布。
那么，如何才能算是可灵活发布呢？
1、组件可以独立运行
2、组件可以独立发布
3、组件有独立的版本

### 组件可以独立运行
组件可以独立运行，就是说在有时候 Module 可以以libary运行，有时候可以以application运行，为了实现这些目标，以下的这些使我们需要改动的。
* 自动选择android plugin——application或libary
* 自动引入ApplicationId
* 自动匹配不同环境时的AndroidMenifest文件
* 代码隔离：组件在单独运行时的测试代码以及资源等应该与核心发布代码进行隔离
* 自动匹配依赖项

首先，为了区分不同运行环境：Application或Libary，我们需要在build.gradle 引入一个布尔值 **isApp**。

最开始，我们添加了两个配置文件 config.gradle和configpersonal.gradle:
* **config.gradle**：主要对各个模块的依赖版本号进行 统一管理，团队的全体成员应该同一份该文件；
* **configpersonal.gradle**：主要是对不同模块 进行 模式切换的统一配置文件，方便管理。初始化后，不应再commit该文件的改动，以避免不必要的冲突；

此外，因为settings.gradle 是gradle run最先运行的几个文件之一，暂时没有发现可以通过在其他文件中 配置一个布尔值，来自动 include对应的module，只能手动开关注释来处理。

好啦现在我们通过一份具体的build.gradle 里了解我们所需要的解决的问题。完整版本的build.gradle 可以查看最下面的build.gradle.

为了获取configpersonal.gradle中 指定module的运行模式，这里定义了 isApp的全局变量，用于获取configpersonal中的值。后续moudle的build.gradle都会依赖这个布尔值。

~~~
def isApp = rootProject.isApplication_ModuleD
~~~
首先，我们通过isApp 来选择对应的 android plugin：
~~~
if (isApp) {
    apply plugin: 'com.android.application'
} else {
    apply plugin: 'com.android.library'
}
~~~

接着处理  在libary中不能使用applicationId的问题：
~~~
...
android {
    compileSdkVersion rootProject.ext.android.compileSdkVersion
    buildToolsVersion rootProject.ext.android.buildToolsVersion

    defaultConfig {
        if (isApp) applicationId "com.example.app.moduled"
        ...
        }
}
~~~

再来处理 不同状态时，AndroidMenifest.xml和java、res的配置：
~~~
    //***************************不同build下，Module代码文件
    sourceSets {
        main {
            if (isApp) {
                manifest.srcFile 'src/main/debug/AndroidManifest.xml'
                java.srcDirs = ['src/main/java', 'src/main/java-debug']
                res.srcDirs = ['src/main/res', 'src/main/res-debug']
            } else {
                manifest.srcFile 'src/main/release/AndroidManifest.xml'
            }
        }
    }
~~~
相关说明：
* java文件夹存放Module的正常源代码文件
* java-debug存放Module单独运行时的额外源代码文件
* res存放Module的正常资源文件
* res-debug存放Module单独运行时的额外资源文件
* release/AndroidManifest.xml存放Module的正常配置信息
* debug/AndroidManifest.xml存放Module单独运行的配置信息，要包括AndroidManifest.xml的所有内容。

需要特别注意的地方：

1. java文件夹中的类不能调用java-debug中的类，因为java-debug中的类在作为library发布的时候是非源码文件夹，不参与编译，编译时会报错。而java-debug文件夹的类可以随意引用java中的类。java文件夹与java-debug文件夹不能出现重复的类。

2. res文件夹中的资源不能引用res-debug文件夹中的资源，原因同上，并且不能出现重复的资源。

3. debug/AndroidManifest.xml中必须包含所有release/AndroidManifest.xml中的配置信息，并且可以随意添加其他信息。原因同上。

最后我们在java-debug中添加我们的测试入口类TestActivity.java，在res-debug中添加测试layout文件activity_test.xml，在debug/AndroidManifest.xml中设置TestActivity为Main入口。



最后，不用状态时,依赖关系也需要分别处理 ，比如Arouter插件的问题：
~~~
    if(isApp) compile 'com.alibaba:arouter-api:x.x.x'
    annotationProcessor 'com.alibaba:arouter-compiler:x.x.x'
~~~


## 参考：
* [组件化开发：build.gradle配置](http://www.jianshu.com/p/9620a40c203f)
* [android模块化简单教程](http://www.wxdroid.com/index.php/4048.html)
和java、res的配置：
~~~
~~~
---

## 附件：

settings.gradle
~~~
//根据自己的需要打开/关闭对应的module

//used for team release
//include ':app', ':ModuleA', ':ModuleB', ':ModuleC', ':ModuleD'


// only for ModuleD debug used
include ':ModuleD'
~~~

---

config.gradle 
~~~
//公共配置信息，依赖版本，统一管理
ext {
	
    //通用build 版本
    android = [
            compileSdkVersion: 26,
            buildToolsVersion: "26.0.2",

            minSdkVersion    : 21,
            targetSdkVersion : 22,

            versionCode      : 10,
            versionName      : "6.0"
    ]


    //公共依赖
    supportV4_Ver = "25.2.0"
    supportV7_Ver = "25.2.0"
    supportV13_Ver = "25.2.0"

    constraint_layout_Ver = "1.0.2"

    leakcanary_Ver = "1.5"


    okhttp3_Ver = "3.8.0"
    gson_Ver = "2.3.1"
    glide_Ver = "3.7.0"
    eventbus_Ver = "3.0.0"
    rxjava_Ver = "2.0.8"
    rxandroid_Ver = "2.0.1"
    retrofit_Ver = "2.2.0"
    alibaba_arouter_Ver ="1.2.1.1"
    alibaba_arouter_compiler_Ver ="1.1.2.1"
}

~~~

---

configpersonal.gradle 
~~~
//个人配置项，用于个人定制项目
ext {

    /**
     * 移除不必要的gradle task ，加快build速度
     */
    isRemoveSomeTask = true


    /**
     * 本地编译 配置项目
     * isApplication_Main = true 表示 将 main Module 编译为application
     * 1.main 为false时，需要注释applicationid(调试自己module时，最后setting注释掉)
     * 2.main 为ture时，其依赖的子module 不能为true， 如libary、baseModule(缘由-application 不能依赖 application)
     * 3.子module为Application时，最后在setting中把 不必要的给注释掉
     */
    modulesBuildMode = [
            isApplication_app = true,
            isApplication_ModuleA = false,
            isApplication_ModuleB = false,
            isApplication_ModuleC = false,
            isApplication_ModuleD = false,
    ]
}
~~~

项目的root build.gradle
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
        maven { url 'http://10.10.4.43:8083/repository/maven-public/' }
//        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }

        flatDir {
            dirs '../Library/libs'
            dirs '../FingerPrintModule/libs'
            dirs '../UpdateLib/libs'

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

ModuleD的build.gradle
~~~
def isApp = rootProject.isApplication_ModuleD

if (isApp) {
    apply plugin: 'com.android.application'
} else {
    apply plugin: 'com.android.library'
}


android {
    compileSdkVersion rootProject.ext.android.compileSdkVersion
    buildToolsVersion rootProject.ext.android.buildToolsVersion

    defaultConfig {
        if (isApp) applicationId "com.example.app.moduled"

        minSdkVersion rootProject.ext.android.minSdkVersion
        targetSdkVersion rootProject.ext.android.targetSdkVersion
        
        versionCode rootProject.ext.android.versionCode
        versionName rootProject.ext.android.versionName

        //项目只是使用armeabi情况下，为避免产生不必要的.so文件夹
        //在gradle.properties中添加 android.useDeprecatedNdk=true，并在defaultConfig中添加以下代码
        // 如支持更多平台，abiFilters "armeabi", "armeabi-v7a", "x86"
        ndk {
            abiFilters "armeabi-v7a"
        }
        
        javaCompileOptions {
	    	annotationProcessorOptions {
				arguments = [ moduleName : project.getName() ]
	    	}
		}

    }

    //自定义BuildConfig，添加一个 字段 IS_DEBUG
    //主要是为了处理gradle插件2.x module中的BuildConfig.DEBUG 值异常的bug，3.0修复了这个问题
    //gradle插件 2.x使用以下进行依赖
    //debugCompile project(path: ':GlxssModule', configuration: 'debug')
    //releaseCompile project(path: ':GlxssModule', configuration: 'release')
    //gradle插件 3.0 后，仅使用一行即可，implementation或api
    //api project(':GlxssModule')
    
    publishNonDefault true
    buildTypes {
        debug {
            minifyEnabled false
            buildConfigField "boolean", "IS_DEBUG", "true"
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }

        release {
            minifyEnabled false
            buildConfigField "boolean", "IS_DEBUG", "false"
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }


    //***************************不同build下，Module代码文件
    sourceSets {
        main {
            if (isApp) {
                manifest.srcFile 'src/main/debug/AndroidManifest.xml'
                java.srcDirs = ['src/main/java', 'src/main/java-debug']
                res.srcDirs = ['src/main/res', 'src/main/res-debug']
            } else {
                manifest.srcFile 'src/main/release/AndroidManifest.xml'
            }
        }
    }
    
    //资源文件名前缀约束，避免发布release时可能重名的问题
    resourcePrefix "glxss_"
}

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')

    compile "com.android.support:appcompat-v7:$rootProject.ext.supportV7_Ver"
    compile "com.android.support.constraint:constraint-layout:$rootProject.ext.constraint_layout_Ver"

    compile "org.greenrobot:eventbus:$rootProject.ext.eventbus_Ver"
    
    if(isApp) compile 'com.alibaba:arouter-api:x.x.x'
    annotationProcessor 'com.alibaba:arouter-compiler:x.x.x'
}

~~~

