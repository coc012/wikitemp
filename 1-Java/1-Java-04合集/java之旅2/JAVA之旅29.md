

# JAVA之旅（二十九）——文件递归，File结束练习，Properties，Properties存取配置文件，load，Properties的小练习

* * *

> 我们继续学习File

## 一.文件递归

> 我们可以来实现一个文件管理器，简单的，但是在此之前，我们先来做点小案例

~~~
package com.lgl.hellojava;

import java.io.File;

public class HelloJJAVA {
    public static void main(String[] args) {

        File dir = new File("E:\\AndroidDelepoer");

        showDir(dir);
    }

    private static void showDir(File dir) {
        System.out.println("目录：" + dir);
        File[] fils = dir.listFiles();
        for (int i = 0; i < fils.length; i++) {
            if (fils[i].isDirectory()) {
                showDir(fils[i]);
            } else {
                // 列出根目录
                System.out.println("files" + fils);
            }
        }

    }
}

~~~

> 因为目录中海油目录，只要使用同一个列出目录功能的函数完成即可，在列出过程中出现的还是目录的话，还可以再此的功能，，也就是函数自身调用自身，这种表现形式，或者手法称为**递归**

~~~
    //递归
    private static void method(){
        method();
    }
~~~

> 你可以看

~~~
private static void toBin(int num) {
        while (num > 0) {
            toBin(num / 2);
             System.out.println(num % 2);
            // num = num / 2;
        }
    }
~~~

## 二.File结束练习

> File讲到这里也是差不多的讲完了，感觉还不错，有很多知识点，我们就用一个小练习来结束这个知识点吧！

*   将一个指定目录下的java文件的绝对路径，存储到一个文本文件中，建立一个java文件列表的文件

~~~
package com.lgl.hellojava;

import java.io.BufferedWriter;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;

public class HelloJJAVA {
    public static void main(String[] args) {

        /**
         * 思路： 
         * 1.对指定的目录进行递归 
         * 2.获取递归过程所有的java文化的路径 
         * 3.将这些路径存储到集合中 
         * 4.将集合中的数据写入到一个文件中
         */

        File file = new File("f:\\");
        java.util.List<File> list = new ArrayList<File>();
        fileToList(file, list);
//      System.out.println(list.size());

        File name = new File(file,"HelloJAVA.java");
        writeToFile(list, name.toString());

    }

    public static void fileToList(File dir, java.util.List<File> list) {
        File[] files = dir.listFiles();
        for (File file : files) {
            if (file.isDirectory()) {
                fileToList(file, list);
            } else {
                // 判断java文件
                if (file.getName().endsWith(".java")) {
                    list.add(file);
                }
            }
        }
    }

    // 写入文件
    public static void writeToFile(java.util.List<File> list,
            String javaFileList) {
        BufferedWriter bufw = null;
        try {
            bufw = new BufferedWriter(new FileWriter(javaFileList));
            for (File f : list) {
                String path = f.getAbsolutePath();
                bufw.write(path);
                bufw.newLine();
                bufw.flush();
            }
        } catch (IOException e) {
            try {
                throw e;
            } catch (IOException e1) {
                // TODO Auto-generated catch block
                e1.printStackTrace();
            }
        }
    }
}

~~~

## 三.Properties

> Properties我们之前就早有接触了，他是hastable的子类，也就是说它具备map集合的特点，而且他里面存储的键值对都是字符串

*   该对象的特点，可以用于键值对形式的配置文件

> 这也是一个工具类，我们可以来学习学习，先来演示一下使用情况吧，我们从set,get说起

~~~
package com.lgl.hellojava;

import java.util.Properties;
import java.util.Set;

public class HelloJJAVA {
    public static void main(String[] args) {

        SetAndGet();
    }

    // 设置和获取元素
    private static void SetAndGet() {
        Properties prop = new Properties();
        prop.setProperty("张三", "20");

        System.out.println(prop);

        String property = prop.getProperty("张三");
        System.out.println(property);

        Set<String> stringPropertyNames = prop.stringPropertyNames();
        for (String s : stringPropertyNames) {
            // 打印姓名
            System.out.println(s);
            // 打印值
            System.out.println(prop.getProperty(s));
        }
    }

}

~~~

> 一览无余，打印的结果

![这里写图片描述](http://img.blog.csdn.net/20160713204719522)

## 四.读取配置文件

> 我们配置文件如果已经存在的话，我们就直接去读取了

*   我们来演示一下如何将流中的数据存储到集合中，想要通过键值对的形式保存起来

> 说白了就是读取本地的一个文件，然后通过键值对保存起来，我们用代码来实现

~~~
/**
     * 思路
     * 1.用一个流和info.txt文件关联
     * 2.读取遗憾数据，将该行数据进行去切割
     * 等号左边的作为键，右边的就是值
     */
    private static void ReadTxt(){
        try {
            BufferedReader bufr = new BufferedReader(new FileReader("info.txt"));
            String line  = null;
            Properties properties = new Properties();
            while((line = bufr.readLine()) != null){
                System.out.println(line);
                String [] arr = line.split("=");
                System.out.println(arr[0]+"...."+arr[1]);

                //存
                properties.setProperty(arr[0], arr[1]);
            }
            bufr.close();

            System.out.println(properties);
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

~~~

> 这样我们输出的就是

![这里写图片描述](http://img.blog.csdn.net/20160713210040814)

## 五.load

> JDK1.6以后出现的load就取代了上面的哪个方式，我们一起来实现一下吧

~~~
private static void loadDemo(){
        try {
            FileInputStream fish = new FileInputStream("info.txt");
            Properties properties = new Properties();
            //将流中的数据加载进集合
            properties.load(fish);
            System.out.println(properties);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
~~~

> 这样就可以达到效果了

## 六.Properties的小练习

*   用于记录应用程序运行的次数，如果使用次数已到，那么给出注册提示，这个很容易想到的是计算器，可是这个程序中，是自增，而且随着程序而存在的，如果程序退出了，这个计数也同样的消失，下一次启动程序又是从0开始了，这样不是我们想要的

> 我们现在要做的是程序退出之后数据任然存在而且继续自增，所以我们要创建配置文件去记录使用次数
> 
> Ok,我们用键值对的形式保存

~~~
package com.lgl.hellojava;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Properties;

public class HelloJJAVA {
    public static void main(String[] args) {

        try {
            Properties properties = new Properties();
            File file = new File("count.ini");
            if (!file.exists()) {
                file.createNewFile();
            }
            FileInputStream fis = new FileInputStream(file);

            properties.load(fis);

            int count = 0;
            String value = properties.getProperty("time");

            if (value != null) {
                count = Integer.parseInt(value);
                if (count >= 5) {
                    System.out.println("你余额不足呀！");
                }
            }

            count++;
            properties.setProperty("time", count + "");

            FileOutputStream fos = new FileOutputStream(file);
            properties.store(fos, "");

            fos.close();
            fis.close();

        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

    }

}

~~~

> 我们得到的结果

![这里写图片描述](http://img.blog.csdn.net/20160713213807449)

> OK，我们本篇就先到这里，我们下篇在聊

## 有兴趣的可以加群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。