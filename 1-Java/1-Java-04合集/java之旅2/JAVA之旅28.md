

# JAVA之旅（二十八）——File概述，创建，删除，判断文件存在，创建文件夹，判断是否为文件/文件夹，获取信息，文件列表,文件过滤

* * *

> 我们可以继续了,今天说下File

## 一.File概述

> 文件的操作是非常重要的,我们先来看下他的基本概念

*   用于将文件或者文件夹封装成对象
*   方便对文件和文件夹的操作
*   File对象可以作为参数传递给流的构造函数

> 我们写个小例子先

~~~
package com.lgl.hellojava;

import java.io.File;

public class HelloJJAVA {
    public static void main(String[] args) {
        // 创建File对象,不存在也没事
        File file = new File("a.txt");
        //目录 文件名
        File file2 = new File("F:\\isblog\\Demo","a.txt");

        //封装什么就打印什么
        System.out.println(file);
    }
}

~~~

> 其实就是一个类的使用

## 二.创建删除

> 是文件肯定有操作方法

*   1.创建
*   2.删除
*   3.判断
*   4.获取信息

### 1.创建

> 忽然如此，我们用实际的例子来说明岂不妙哉？

~~~
package com.lgl.hellojava;

import java.io.File;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {
        // 创建File对象
        File file = new File("a.txt");

        try {
            //创建
            file.createNewFile();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

~~~

> 在指定的位置创建文件，如果文件已经存在，就不创建，并且返回false,和输出流不一样，输出流对象已建立文件，文件就已经存在，会覆盖

### 2.删除

> 删除我就不说了，直接这样

~~~
file.delete();
~~~

> 他还有一个方法比较好玩

~~~
file.deleteOnExit();
~~~

> 在程序退出之后删除文件

## 三.判断文件存在

> 判断文件是否存在

~~~
package com.lgl.hellojava;

import java.io.File;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {
        // 创建File对象
        File file = new File("a.txt");
        // 判断是否存在，不存在则创建
        if (!file.exists()) {
            try {
                file.createNewFile();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }
}

~~~

> 这样我们就可以去判断文件是否存在且不存在就去创建文件了。

## 四.创建文件夹

> 我们继续来看怎么去创建文件夹，其实也很简单

~~~
package com.lgl.hellojava;

import java.io.File;

public class HelloJJAVA {
    public static void main(String[] args) {
        // 创建File对象
        File file = new File("liuguilin");
        file.mkdir();
    }
}

~~~

> OK,这样的话，就创建了，这里注意mkdir只能创建一级目录，而mkdirs可以创建多级文件夹目录

## 五.判断是否为文件/文件夹

> 有时候还是需要的

~~~
package com.lgl.hellojava;

import java.io.File;

public class HelloJJAVA {
    public static void main(String[] args) {
        File file = new File("liuguilin");
        //是否为文件
        System.out.println(file.isFile());
        //是否为文件夹
        System.out.println(file.isDirectory());
    }
}

~~~

> 他返回的是boolean值来确定是否存在，但是这里也要记住一电，就是一定要确定这个文件是否存在，所以我们的流程可以这样写

~~~
package com.lgl.hellojava;

import java.io.File;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {
        File file = new File("liuguilin");
        // 判断文件是否存在
        if (file.exists()) {
            // 再去判断文件还是文件夹
            if (file.isFile()) {
                System.out.println("文件");
            } else if (file.isDirectory()) {
                System.out.println("文件夹");
            }
        } else {
            try {
                file.createNewFile();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }
}

~~~

> 这样逻辑是比较清晰的

## 六.获取信息

> 获取的话，我们是怎么去获取信息的呢？毫无疑问，是get，比如getNmae之类的，我们用代码里的注释来说明是比较好的

~~~
package com.lgl.hellojava;

import java.io.File;

public class HelloJJAVA {
    public static void main(String[] args) {
        File file = new File("liuguilin.txt");
        File file2 = new File("haha.txt");
        // 项目路径下+文件名
        System.out.println("路径：" + file.getPath());
        // 全路径
        System.out.println("绝对路径：" + file.getAbsolutePath());
        // 最后一次修改时间
        System.out.println("时间：" + file.lastModified());
        // 绝对路径中的文件父目录,如果是相对路径，返回的为空
        System.out.println("父目录：" + file.getParent());
        // 把内容拷贝到另一个文本中并且删除自身
        System.out.println(file.renameTo(file2));
    }
}

~~~

> 运行的结果

![这里写图片描述](http://img.blog.csdn.net/20160710190329157)

## 七.文件列表

> 列出可用的系统目录，我们看代码

~~~
package com.lgl.hellojava;

import java.io.File;

public class HelloJJAVA {
    public static void main(String[] args) {
        File[] listRoots = File.listRoots();
        for (File f : listRoots) {
            // 打印磁盘目录
            System.out.println(f);
        }
    }
}

~~~

> 这样我们就可以得到有效盘符了

![这里写图片描述](http://img.blog.csdn.net/20160710190619458)

> 我们可以进行改进，我们打印C盘下的所有文件

~~~
package com.lgl.hellojava;

import java.io.File;

public class HelloJJAVA {
    public static void main(String[] args) {
    //必须封装了一个目录，该目录还必须存在
        File f = new File("c:\\");
        String[] list = f.list();
        for (String fi : list) {
            System.out.println(fi);
        }
    }
}

~~~

> 得到的肯定就是所有文件的列表咯

![这里写图片描述](http://img.blog.csdn.net/20160710191108315)

## 八.文件过滤

> 我们做文件夹的时候经常会用到的一个小知识点，就是过滤文件

~~~
package com.lgl.hellojava;

import java.io.File;
import java.io.FilenameFilter;

public class HelloJJAVA {
    public static void main(String[] args) {
        File f = new File("c:\\");
        String[] list = f.list(new FilenameFilter() {
            // 过滤
            @Override
            public boolean accept(File dir, String name) {
                // 只返回txt后缀的文件
                return name.endsWith(".txt");
            }
        });
        for (String fi : list) {
            // 过滤
            System.out.println(fi);
        }
    }

}

~~~

> 需要监听，然后过滤，当然，他还有一些其他的子类listFiles就不讲了，详细的翻阅下API
> 
> 我们本篇博文就先到这里

## 有兴趣的可以加群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。