

近来迁移了一些项目到`[Android](http://lib.csdn.net/base/android "Android知识库")<span class="Apple-converted-space"> </span>Studio`，采用`Gradle`构建确实比原来的Ant方便许多。但是编译时下载依赖的网速又着实令人蛋疼不已。

如果能切换到国内的`Maven`镜像仓库，如开源中国的`Maven`库，又或者是换成自建的`Maven`私服，那想必是极好的。

一个简单的办法，修改项目根目录下的`build.gradle`，将`jcenter()`或者`mavenCentral()`替换掉即可：

~~~
allprojects {
    repositories {
        maven{ url 'http://maven.oschina.net/content/groups/public/'}
    }
}
~~~

但是架不住项目多，难不成每个都改一遍么？ 
自然是有省事的办法，将下面这段Copy到名为`init.gradle`文件中，并保存到`USER_HOME/.gradle/`文件夹下即可。

~~~
allprojects{
    repositories {
        def REPOSITORY_URL = 'http://maven.oschina.net/content/groups/public'
        all { ArtifactRepository repo ->
            if(repo instanceof MavenArtifactRepository){
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
~~~

`init.gradle`文件其实是Gradle的初始化脚本(Initialization Scripts)，也是运行时的全局配置。

如果碰到如下错误，多尝试几次就好了：

~~~
FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring root project 'fresco'.
> Could not resolve all dependencies for configuration ':classpath'.
   > Could not download httpcore.jar (org.apache.httpcomponents:httpcore:4.1)
      > Could not get resource 'https://jcenter.bintray.com/org/apache/httpcomponents/httpcore/4.1/httpcore-4.1.jar'.
         > SSL peer shut down incorrectly

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED
~~~

