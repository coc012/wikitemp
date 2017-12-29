文档当前状态：**beta0.5**
* [x] 选题收集：2017/11/20
* [x] 初稿整理：
* [ ] 补充校对：
* [ ] 入库存档：
---


[TOC]

---
### **一.Nexus私服——简介**

#### Nexus介绍

Nexus是一个强大的Maven仓库管理器，它极大地简化了本地内部仓库的维护和外部仓库的访问。

如果使用了公共的Maven仓库服务器，可以从Maven中央仓库下载所需要的构件（Artifact），但这通常不是一个好的做法。

正常做法是在本地架设一个Maven仓库服务器，即利用Nexus私服可以只在一个地方就能够完全控制访问和部署在你所维护仓库中的每个Artifact。
*   Nexus在代理远程仓库的同时维护本地仓库，以降低中央仓库的负荷,节省外网带宽和时间，Nexus私服就可以满足这样的需要。
*   Nexus是一套“开箱即用”的系统不需要数据库，它使用文件系统加Lucene来组织数据。
*   Nexus使用ExtJS来开发界面，利用Restlet来提供完整的REST APIs，通过m2eclipse与Eclipse集成使用。
*   Nexus支持WebDAV与LDAP安全身份认证。
*   Nexus还提供了强大的仓库管理功能，构件搜索功能，它基于REST，友好的UI是一个extjs的REST客户端，它占用较少的内存，基于简单文件系统而非数据库。

#### 为什么要构建Nexus私服？

如果没有Nexus私服，我们所需的所有构件都需要通过maven的中央仓库和第三方的Maven仓库下载到本地，而一个团队中的所有人都重复的从maven仓库下载构件无疑加大了仓库的负载和浪费了外网带宽，如果网速慢的话，还会影响项目的进程。很多情况下项目的开发都是在内网进行的，连接不到maven仓库怎么办呢？开发的公共构件怎么让其它项目使用？这个时候我们不得不为自己的团队搭建属于自己的maven私服，这样既节省了网络带宽也会加速项目搭建的进程，当然前提条件就是你的私服中拥有项目所需的所有构件。

#### 在本地构建nexus私服的好处

1）加速构建；
2）节省带宽；
3）节省中央maven仓库的带宽；
4）稳定（应付一旦中央服务器出问题的情况）；
5）控制和审计；
6）能够部署第三方构件；
7）可以建立本地内部仓库；
8）可以建立公共仓库
这些优点使得Nexus日趋成为最流行的Maven仓库管理器。
先看下这张图应该大家就明白了：


[![234.png](https://blog.52itstyle.com/usr/uploads/2017/06/334573725.png)](https://blog.52itstyle.com/usr/uploads/2017/06/334573725.png)

这样就相当于在我们本地的局域网搭建了一个类似中央仓库的服务器，我们开始将中央仓库的一些资料下载到私服务器上，然后平时我们的maven项目就是直接访问局域网内的私服即可，既节省了网络带宽也会加速项目搭建的进程。
 
 此外，在局域网内搭建Maven私服，除了能从私服下载已缓存的jar包，还能将内部通用模块发布在私服上供其他同事使用。对内部项目部署很有帮助。这样对我们开发来说，对公司来说都是非常好的选择。下边简单说一下Nexus私服的搭建与简单使用。



### **二.Nexus私服——部署安装**

#### **2.1 环境准备**

这里以windows10平台为例，linux、mac平台同样支持。
整体步骤：准备环境、下载Nexus3、

>需求环境：jdk1.8、maven3.3.9（具体需求环境可以在官网可以看到，上述环境配置过程请自行百度）

##### **2.1.1：检查 Java 安装**

现在打开控制台，执行下面的 `java` 命令。

| 操作系统 | 任务 | 命令 |
| --- | --- | --- |
| Windows | 打开命令控制台 | `c:\> java -version` |
| Linux | 打开命令终端 | `$ java -version` |
| Mac | 打开终端 | `machine:~ joseph$ java -version` |

我们来验证一下所有平台上的输出：

| 操作系统 | 输出 |
| --- | --- |
| Windows | java version "1.6.0_21"  Java(TM) SE Runtime Environment (build 1.6.0_21-b07) Java HotSpot(TM) Client VM (build 17.0-b17, mixed mode, sharing) |
| Linux | java version "1.6.0_21"Java(TM) SE Runtime Environment (build 1.6.0_21-b07)  Java HotSpot(TM) Client VM (build 17.0-b17, mixed mode, sharing) |
| Mac | java version "1.6.0_21"  Java(TM) SE Runtime Environment (build 1.6.0_21-b07)  Java HotSpot(TM)64-Bit Server VM (build 17.0-b17, mixed mode, sharing) |

如果你没有安装 Java，从以下网址安装 Java 软件开发套件（SDK）：[](http://www.oracle.com/technetwork/java/javase/downloads/index.html)[http://www.oracle.com/technetwork/java/javase/downloads/index.html](http://www.oracle.com/technetwork/java/javase/downloads/index.html)。我们假定你安装的 Java 版本为1.6.0_21。

##### **2.1.2：设置 Java 环境**

设置 `JAVA_HOME` 环境变量，并指向你机器上的 Java 安装目录。例如：

| 操作系统 | 输出 |
| --- | --- |
| Windows | Set the environment variable JAVA_HOME to C:\Program Files\Java\jdk1.6.0_21 |
| Linux | `export JAVA_HOME=/usr/local/java-current` |
| Mac | `export JAVA_HOME=/Library/Java/Home` |

将 Java 编译器地址添加到系统路径中。

| 操作系统 | 输出 |
| --- | --- |
| Windows | 将字符串“;C:\Program Files\Java\jdk1.6.0_21\bin”添加到系统变量“Path”的末尾 |
| Linux | export PATH=PATH:JAVA_HOME/bin/ |
| Mac | not required |

使用上面提到的 java -version 命令验证 Java 安装。

##### **2.1.3：下载 Maven 文件**

从以下网址下载 Maven 3.2.5：[](http://maven.apache.org/download.html)[http://maven.apache.org/download.html](http://maven.apache.org/download.html)

##### **2.1.4：解压 Maven 文件**

解压文件到你想要的位置来安装 Maven 3.2.5，你会得到 apache-maven-3.2.5 子目录。

| 操作系统 | 位置 (根据你的安装位置而定) |
| --- | --- |
| Windows | `C:\Program Files\Apache Software Foundation\apache-maven-3.2.5` |
| Linux | `/usr/local/apache-maven` |
| Mac | `/usr/local/apache-maven` |

##### **2.1.5：设置 Maven 环境变量**

添加 M2_HOME、M2、MAVEN_OPTS 到环境变量中。

| 操作系统 | 输出 |
| --- | --- |
| Windows | 使用系统属性设置环境变量。M2_HOME=C:\Program Files\Apache Software Foundation\apache-maven-3.2.5 ,  M2=%M2_HOME%\bin     ,MAVEN_OPTS=-Xms256m -Xmx512m |
| Linux | 打开命令终端设置环境变量。export M2_HOME=/usr/local/apache-maven/apache-maven-3.2.5,export M2=$M2_HOME/bin, export MAVEN_OPTS=-Xms256m -Xmx512m |
| Mac | 打开命令终端设置环境变量。export M2_HOME=/usr/local/apache-maven/apache-maven-3.2.5,export M2=$M2_HOME/bin,export MAVEN_OPTS=-Xms256m -Xmx512m |

##### **2.1.6：添加 Maven bin 目录到系统路径中**

现在添加 M2 变量到系统“Path”变量中

| 操作系统 | 输出 |
| --- | --- |
| Windows | 添加字符串 “;%M2%” 到系统“Path”变量末尾 |
| Linux | export PATH=M2:PATH |
| Mac | export PATH=M2:PATH |

##### **2.1.7：验证 Maven 安装**

现在打开控制台，执行以下 mvn 命令。

| 操作系统 | 输出 | 命令 |
| --- | --- | --- |
| Windows | 打开命令控制台 | `c:\> mvn --version` |
| Linux | 打开命令终端 | `$ mvn --version` |
| Mac | 打开终端 | `machine:~ joseph$ mvn --version` |

最后，验证以上命令的输出，应该是像下面这样：

| 操作系统 | 输出 |
| --- | --- |
| Windows | Apache Maven 3.2.5 (r801777; 2009-08-07 00:46:01+0530)Java version: 1.6.0_21Java home: C:\Program Files\Java\jdk1.6.0_21\jre |
| Linux | Apache Maven 3.2.5 (r801777; 2009-08-07 00:46:01+0530)  Java version: 1.6.0_21  Java home: C:\Program Files\Java\jdk1.6.0_21\jre |
| Mac | Apache Maven 3.2.5 (r801777; 2009-08-07 00:46:01+0530)  Java version: 1.6.0_21  Java home: C:\Program Files\Java\jdk1.6.0_21\jre |

恭喜！你完成了所有的设置，开始使用 Apache Maven 吧。

**环境检验**

maven:在cmd下出现如下：
~~~

C:\Users\hwj>**mvn --version**
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T00:41:47+08:00)
Maven home: E:\apache-maven-3.3.9\bin\..
Java version: 1.8.0_60, vendor: Oracle Corporation
Java home: E:\Program Files\Java\jdk1.8.0_60\jre
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 10", version: "10.0", arch: "amd64", family: "dos"
~~~
jdk:在cmd下出现如下：
~~~
C:\Users\hwj>**java -version**
java version "1.8.0_60"
Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
Java HotSpot(TM) 64-Bit Server VM (build 25.60-b23, mixed mode)
~~~
表示环境已经搭好了。


#### **2.2 Nexus3部署**

链接：[https://www.sonatype.com/download-oss-sonatype](https://www.sonatype.com/download-oss-sonatype) 

下载完了解压后就是两个文件夹：

>nexus-3.2.1-01、sonatype-work

在nexus-3.2.1-01 下的bin下有个nexus.exe，但是你就是死活打不开（闪退），那是因为打开方式不对。在nexus-3.2.1-01\etc下有个nexus-default.properties文件，打开里面修改其中application-port=8081为其他端口，8081这个端口比较抢手吧，经常被占用，这里我就**修改为8083**了，这个时候就可以来打开nexus.exe了。

正确的打开方式：在cmd中切换到nexus.exe所在目录，然后敲入nexus.exe /run即可，例如我的就是C:\Users\hwj>E:\nexus\nexus-3.2.1-01\bin\nexus.exe /run
~~~
nexus.exe /run
~~~

之后等待一下看到，表示启动成功了。
~~~
Started Sonatype Nexus OSS 3.2.1-01
~~~


此外，当然也可以作为一个Service运行，详情看官网[service-windows](https://books.sonatype.com/nexus-book/3.0/reference/install.html#service-windows)：
~~~
nexus.exe /install
~~~




在浏览器输入localhost:8083即可看到此页面(如果部署到服务器上，就是服务器的ip:8083)

![](http://img.blog.csdn.net/20170223141443240?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaHV3ZWlqaWFuNQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

默认的账号admin,密码admin123.

### **三.Nexus私服**
>仓库管理这块主要涉及：功能介绍、服务器缓存中央仓库Maven、上传更新私有类库

这里假设你把 nexus 安装在了本机，仓库地址为 [http://localhost:8081/repository/test/](http://localhost:8081/repository/test/) 。如果把 nexus 安装在了局域网服务器，基本类似，只是仓库地址中的 localhost 改为局域网服务器 ip 。  
       从状态上来看nexus已经启动起来了,Nexus启动之后就可以在浏览器访问类似 192.168.9.190:9999/nexus/ 地址,其中ip为当前服务器ip,端口Nexus默认为8081,可以在conf/nexus.properties文件中修改端口.进入web管理页面后右上角登陆,Nexus默认的账号密码是admin:admin123,


#### **3.1 Nexus界面简介**
使用默认的管理员admin/admin123登录，进入管理界面：

[![](https://xnstatic-1253397658.file.myqcloud.com/nexus02.png)](https://xnstatic-1253397658.file.myqcloud.com/nexus02.png)

##### **3.1.1 用户和角色**

可以点击上面的“设置”图标，在“设置”里可以添加用户、角色，对接LDAP等的设置。

[![](https://xnstatic-1253397658.file.myqcloud.com/nexus03.png)](https://xnstatic-1253397658.file.myqcloud.com/nexus03.png)

[](https://xnstatic-1253397658.file.myqcloud.com/nexus04.png)

##### **3.1.2 Nexus搜索页**

这个不需要登录就可以访问，用来查询jar包。支持模糊查询

[![](https://xnstatic-1253397658.file.myqcloud.com/nexus07.png)](https://xnstatic-1253397658.file.myqcloud.com/nexus07.png)

##### **3.1.3 Blob Stores**

文件存储的地方，创建一个目录的话，对应文件系统的一个目录，可供仓库上传文件使用，如图所示：

[![](https://xnstatic-1253397658.file.myqcloud.com/nexus08.png)](https://xnstatic-1253397658.file.myqcloud.com/nexus08.png)

##### **3.1.4 Nexus的调度任务**

默认安装好之后是没有索引和jar文件的，因为你要自己定义任务去执行。

Nexus提供了一系列可配置的调度任务来方便用户管理系统。用户可以设定这些任务运行的方式，例如每天、每周等。调度任务会在适当的时候在后台运行。

要建立一个调度任务，单击左边导航菜单中的Tasks，点击`Create Task`，然后选择一个任务类型。

[![](https://xnstatic-1253397658.file.myqcloud.com/nexus11.png)](https://xnstatic-1253397658.file.myqcloud.com/nexus11.png)

以下几种常用类型的调度任务：

*   Execute script：执行自定义脚本
*   Purge开头：清理一些不使用的资源。
*   Rebuild repository index：为仓库重新编纂索引，从远仓库下载最新的索引。
*   Rebuild Maven repository metadata：基于仓库内容重新创建仓库元数据文件，同时重新创建每个文件的校验和md5与sha1。
*   Remove snapshots from Maven repository：把快照删了，这个是在稳定版发布后清除

比如我新建一个重构索引的任务，然后选择aliyun仓库，让它把远程索引取下来，手动执行。不过最好别这样做，因为需要很大的硬盘空间。

最好是让它自己去维护，请求一个依赖的时候如果私服没有会自动去远仓库取的。



#### **3.3 仓库Repository**
**重点来啦！重点来啦！重点来啦！**
现在点击左侧的repositories查看现有的仓库列表.

![](https://user-gold-cdn.xitu.io/2017/10/17/ccd528278ca0944b96107c48a2bf29b2?imageView2/0/w/1280/h/960/ignore-error/1)

##### **3.3.1 仓库管理**
最核心的是仓库管理

[![](https://xnstatic-1253397658.file.myqcloud.com/nexus05.png)](https://xnstatic-1253397658.file.myqcloud.com/nexus05.png)

默认的这几个仓库，先解释一下：
1.  **maven-central**：maven中央库，默认从https://repo1.maven.org/maven2/拉取jar；
2.  **maven-releases**：私库发行版jar，初次安装请将`Deployment policy`设置为`Allow redeploy`
3.  **maven-snapshots**：私库快照（调试版本）jar
4.  **maven-public**：仓库分组，把上面三个仓库组合在一起对外提供服务，在本地maven基础配置`settings.xml`中使用。

Nexus默认的仓库类型有以下四种：

1.  **group(仓库组类型)**：又叫组仓库，用于方便开发人员自己设定的仓库；
2.  **hosted(宿主类型)**：内部项目的发布仓库（内部开发人员，发布上去存放的仓库）；
3.  **proxy(代理类型)**：从远程中央仓库中寻找数据的仓库（可以点击对应的仓库的Configuration页签下Remote Storage属性的值即被代理的远程仓库的路径）；
4.  **virtual(虚拟类型)**：虚拟仓库（这个基本用不到，重点关注上面三个仓库的使用）；

Policy(策略): 表示该仓库为发布(Release)版本仓库还是快照(Snapshot)版本仓库；

由于访问中央仓库有时候会比较慢，这里我添加一个阿里云的代理仓库，然后优先级放到默认中央库之前,， 阿里云的maven仓库url为`http://maven.aliyun.com/nexus/content/groups/public`

[![](https://xnstatic-1253397658.file.myqcloud.com/nexus09.png)](https://xnstatic-1253397658.file.myqcloud.com/nexus09.png)

然后再public组里面讲这个`aliyun-proxy`仓库加入，排在`maven-central`之前即可。

[![](https://xnstatic-1253397658.file.myqcloud.com/nexus10.png)](https://xnstatic-1253397658.file.myqcloud.com/nexus10.png)

**Nexus仓库分类的概念**

1）Maven可直接从宿主仓库下载构件,也可以从代理仓库下载构件,而代理仓库间接的从远程仓库下载并缓存构件

2）为了方便,Maven可以从仓库组下载构件,而仓库组并没有时间的内容(下图中用虚线表示,它会转向包含的宿主仓库或者代理仓库获得实际构件的内容)

[![](https://xnstatic-1253397658.file.myqcloud.com/nexus06.png)](https://xnstatic-1253397658.file.myqcloud.com/nexus06.png)

###### **3.3.1.1  Proxy**
这里就是代理的意思，代理中央Maven仓库，当PC访问中央库的时候，先通过Proxy下载到Nexus仓库，然后再从Nexus仓库下载到PC本地。
这样的优势只要其中一个人从中央库下来了，以后大家都是从Nexus私服上进行下来，私服一般部署在内网，这样大大节约的宽带。
创建Proxy的具体步骤:
1. --点击“Create Repositories”按钮

![](http://images2015.cnblogs.com/blog/907596/201612/907596-20161221110640089-1389328954.png)

2. --选择要创建的类型

![](http://images2015.cnblogs.com/blog/907596/201612/907596-20161221110758542-1042948386.png)

3. --填写详细信息
Name：就是为代理起个名字
Remote Storage: 代理的地址，Maven的地址为: https://repo1.maven.org/maven2/
Blob Store: 选择代理下载包的存放路径

![](http://images2015.cnblogs.com/blog/907596/201612/907596-20161221111233651-1321037653.png)

###### **3.3.1.2  Hosted**
hosted 是宿主仓库 ,是自己的私有库地址，这个就是自己的。这个有 releases 和snapshots 两种类型，你如果自己创建的时候，需要指定 ，一个是正式发布地址，一个是开发中地址。  

Hosted是宿主机的意思，就是怎么把第三方的Jar放到私服上。
Hosted有三种方式，Releases、SNAPSHOT、Mixed
Releases: 一般是已经发布的Jar包
Snapshot: 未发布的版本
Mixed：混合的
Hosted的创建和Proxy是一致的，具体步骤和上面基本一致。如下：

![](http://images2015.cnblogs.com/blog/907596/201612/907596-20161221111325104-723208432.png)

![](http://images2015.cnblogs.com/blog/907596/201612/907596-20161221111344573-1949446719.png)

![](http://images2015.cnblogs.com/blog/907596/201612/907596-20161221111419870-2005652543.png)

**注意事项：**
Deployment Pollcy: 需要把策略改成“Allow redeploy”。

![](http://images2015.cnblogs.com/blog/907596/201612/907596-20161221111455589-1303191395.png)

###### **3.3.1.3  Group**
group 管理组 ，组是Nexus一个强大的特性，它允许你在一个单独的URL中组合多个仓库。比如这里默认组合了：maven-central、maven-releases和maven-snapshots ，一般直接引用这个地址就好了。
       
#### **3.4 开启远程索引(镜像中央maven)**
新搭建的neuxs环境只是一个空的仓库，需要手动和远程中心库进行同步，nexus默认是关闭远程索引下载，最重要的一件事情就是开启远程索引下载。登陆nexus系统，默认用户名密码为admin/admin123

*   点击左边Administration菜单下面的Repositories，找到右边仓库列表中的Apache Snapshots，Central，然后在configuration下把Download Remote Indexes修改为true 
    ![这里写图片描述](http://img.blog.csdn.net/20170125162023705?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMTcwNDM5NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
*   仓库上分别右键，选择Repari Index，这样Nexus就会去下载远程的索引文件。 
    ![这里写图片描述](http://img.blog.csdn.net/20170125162037658?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMTcwNDM5NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这样设置以后, Nexus会自动从远程中央仓库下载索引文件, 为了检验索引文件自动下载是否生效,可以却换到Browse Index查看

**增加阿里云镜像**(大部分情况下还是比较快)

~~~
Repositories –> Add –>ProxyRepository
阿里的maven镜像：
http://maven.aliyun.com/nexus/content/groups/public/
Checksum Policy 可以改为ignore

~~~

![这里写图片描述](http://img.blog.csdn.net/20170125162209824?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMTcwNDM5NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
Repositories –> Public Repositories 
将右侧栏全部加入左侧栏即可 
![这里写图片描述](http://img.blog.csdn.net/20170125162004173?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMTcwNDM5NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


#### **3.5 上传私有类库**

假设我们有一个有关 android 通用工具类的公共库 CommonUtils.jar 需要上传到仓库，供团队成员使用。

##### **3.5.1 配置仓库和类库的相关信息**
[参考这里](http://zmywly8866.github.io/2016/01/05/android-private-maven-repository.html)配置仓库和类库的相关信息，在项目 gradle.properties 添加以下属性

~~~
# maven local config
maven_local_url=http://localhost:8081/repository/test/
maven_local_username=admin
maven_local_password=admin123

# common utils
maven_pom_version=1.0.0
maven_pom_groupid=im.het
maven_pom_artifactId=commonutils
maven_pom_packaging=jar
maven_pom_description=android common utils
maven_pom_archives_file=libs/CommonUtils.jar
~~~

说明 ： 

*   maven_local_url maven仓库中相应repository的地址

*   maven_local_username 上传类库到仓库的用户名

*   maven_local_password 上传类库到仓库的密码

*   maven_pom_version 要上传的类库的版本号

*   maven_pom_groupid  类库的分组标记，一般使用公司名或包名即可

*   maven_pom_artifactId 类库的名称

*   maven_pom_packaging 类库的格式可以支持 jar ，aar , so 等

*   maven_pom_description 类库描述

*   maven_pom_archives_file 类库文件在项目中的位置（相对于 build.gradle）

#####  **3.5.2 gradle新增上传task**
在相应类库所在的 module 的 build.gradle 增加上传类库的 task

~~~
apply plugin: 'maven'

uploadArchives {
    repositories {
        mavenDeployer {        
            repository(url: maven_local_url) {
                authentication(userName:maven_local_username ,password: maven_local_password)
            }

            pom.project {
                version maven_pom_version
                artifactId maven_pom_artifactId
                groupId maven_pom_groupid
                packaging maven_pom_packaging
                description maven_pom_description
            }
        }
    }
}

artifacts {
    archives file(maven_pom_archives_file)
}
~~~

说明 ：

*   artifacts 中可以指定要上传的 jar 或 aar 的路径，为统一配置都由 gradle.properties 的属性 maven_pom_archives_file 指定

*   如果只需要上传项目编译时产生的 aar，artifacts 可以省略，因为 artifacts 默认就包含了编译产生的 aar 或 apk 

#####  **3.5.3 执行上传task** 
添加task后，可以使用命令行或者GUI
运行上传类库的 task 即可把相应类库上传到仓库

~~~
gradle uploadArchives
~~~
 在Android Studio的Gradle project插件中选中 对应 模块运行uploadArchives任务就可以将该模块的aar文件以 指定的版本部署到Nexus上,供其他项目引用.

![](http://img.blog.csdn.net/20150923141820997?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

如果上传成功，一般可以在[这里](http://localhost:8081/#browse/browse/assets:test)看到相关类库了。

下面是我们上传类库 commonutils 到 test repo 后的文件列表，可以看到其中有 aar 文件存在，这是因为 artifacts 中默认包含了编译产生的 aar 文件 ，可以在这里删除 aar 相关文件，因为我们只希望有 jar 的存在。

![](http://upload-images.jianshu.io/upload_images/711773-66a026eaac93294f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

browse-repo-test.png

##### **3.5.4 更新类库**

如果类库需要更新版本，基本操作同上。

比如类库 commonutils 有了新版本为 2.0.0 ，先修改 gradle.properties 配置如下 

~~~
maven_pom_version=2.0.0
...
maven_pom_archives_file=libs/CommonUtils2.0.jar
~~~

再运行上传类库的 task 即可。


### **四.Android Studio使用Nexus私服**
#### **4.1 配置私服Url**
##### **4.1.1 单个项目配置私服**
在需要使用类库的项目的 根build.gradle 文件里面加入以下即可在项目中使用 私服上的 类库了。
~~~
allprojects {
    repositories {
        jcenter()
        mavenLocal()
    }
    dependencies{
        repositories {
            maven { 
                url 'http://localhost:8081/repository/test/'      
            }
        }
    }
}
~~~

##### **4.1.2 所有项目配置私服**
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

如果公司暂时还不能提供私服，可以使用一些大公司的。
目前已知、可用nexus私服：
* 阿里云：http://maven.aliyun.com/nexus/content/groups/public/

#### **4.2 项目模块依赖**

模块的build.gradle中，常规写法：
~~~
dependencies {
    compile 'im.het:commonutils:1.2.1'
}
~~~

gradle 支持依赖库动态版本，如果你想每次从 maven 仓库中拉取最新的库，可以这样下面这样：
~~~
//不推荐，每次编译都会去服务器，检查新版
//接口可能会变更，编译报错
dependencies {
    compile 'im.het:commonutils:1.+'
}
~~~
也可以这样配置
~~~
dependencies {
    compile 'im.het:commonutils:+'
}
~~~
用”+”通配符来匹配最新的符合要求的版本，这样做的好处是如果 maven 仓库上有新版本库了，不用修改类库的依赖配置版本，gradle 编译的时候会自动拉取最新版本。但这样也会带来两个麻烦:

* 有可能新版本类库的接口改了，导致编译失败，这个时候需要修改代码做接口适配
* 会使 gradle 构建速度变慢，因为每次 gradle 编译完整的项目，都会去 maven 仓库上尝试拉取最新版本的类库，这样会拖慢编译速度，尤其在网络差的时候，不过我们这里的仓库一般是在本机或局域网，拉取最新版本并下载的这点时间一般很快可以忽略。

### **常见问题列表**
**使用私有仓库解决依赖后，我想拷贝其中的 jar 包，怎么找？**

在工程下的 External Libraries 可以看到相应的类库，这些类库默认是放在 gradle 缓存目录的 （在 linux 环境中，一般是在 ~/.gradle/caches 下面 ）。 可以右键点击 Reavel in Finder 打开所在文件的目录，拷贝其中类库即可。



### **参考资料**
* [maven私服nexus3.x环境配置](https://www.xncoding.com/2017/09/02/tool/nexus.html)
* [Windows下搭建基于Nexus的Android Maven私服(一)](http://blog.csdn.net/huweijian5/article/details/56670569)
* [Windows下搭建基于Nexus的Android Maven私服(二)](http://blog.csdn.net/huweijian5/article/details/56834199)
* [利用Nexus搭建Maven私服(包含aar上传)](http://cdn2.jianshu.io/p/e5ae84aebab9)
* [Android 搭建maven私服管理类库](http://www.jianshu.com/p/1b48489eb23a)
* [Android 项目部署之Nexus私服搭建和应用](http://blog.csdn.net/l2show/article/details/48653949)
* [Android Studio上使用Nexus搭建Maven私服并上传JCenter](http://www.jianshu.com/p/207c3a467167)
* [使用 Android Studio ＋ Nexus 搭建 Maven 私服（二）](http://www.jianshu.com/p/a8aac4d95214)
* [使用nexus搭建maven私服————开启远程索引](http://blog.h5min.cn/u011704394/article/details/54730044)
