

# JAVA之旅（二十六）——装饰设计模式，继承和装饰的区别，LineNumberReader，自定义LineNumberReader，字节流读取操作，I/O复制图片

* * *

## 一.装饰设计模式

> 其实我们自定义readLine就是一种装饰模式

*   当想要对已有的对象进行功能增强时，可以定义一个类，将已有对象传入，并且提供加强功能，那么自定义的该类就称为装饰类

~~~
package com.lgl.hellojava;

public class HelloJJAVA {
    public static void main(String[] args) {

        Person p = new Person();
        p.eat();
        // 开始进行增强
        superPerson p1 = new superPerson(p);
        p1.superEat();
    }
}

class Person {
    public void eat() {
        System.out.println("吃饭");
    }
}

class superPerson {

    private Person p;

    public superPerson(Person p) {
        this.p = p;
    }

    public void superEat() {
        System.out.println("小菜+吃饭");
    }
}
~~~

> 这里的逻辑就是当我们吃饭这个功能需要增强的时候，我们应该装饰他

*   装饰类通常会通过构造方法接收被装饰的对象，并基于被装饰的对象的功能提供更强的功能

## 二.继承和装饰的区别

> 你现在知道了装饰模式，那你一定会疑问，和继承的道理类似，对吧，我们现在来说下他们的区别
> 
> 这里我们就不写代码了，我们看注释

~~~
package com.lgl.hellojava;

public class HelloJJAVA {
    public static void main(String[] args) {
        /**
         * MyReader:专门用于读取数据的类
         * MyTextReader:专门读取文本 两个向上抽取，形成继承体系
         */

        /**
         * 想实现更多的功能 
         * MyBufferReader 
         * myBufferTestReader
         */

        /**
         *谁需要加强就传谁进来
         * class MyBufferReader{
         * }
         */

    }
}

~~~

> 这个逻辑大概是这样的，我们有两个功能，一个读取文件，一个读取文本，他们其实是有共性的，你就把他们共性部分抽取出来，可是我现在在读取文本的时候我顺便想读取图片呢？其实，我们就是这样才产生的装饰者模式

*   装饰者模式比继承要灵活，避免了继承体系的臃肿，而且降低了类与类之间的关系
*   装饰类因为增强已有对象，具备功能和已有的想相同，只不过提供了更强的功能，所以装饰类和被装饰类通常属于一个体系中的

## 三.LineNumberReader

> 这也是一个子类

![这里写图片描述](http://img.blog.csdn.net/20160707214409225)

> 他也是一个包装类，我们看例子

~~~
package com.lgl.hellojava;

import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.io.LineNumberReader;

public class HelloJJAVA {
    public static void main(String[] args) {
        FileReader fr;
        try {
            fr = new FileReader("test.txt");
            LineNumberReader lnr = new LineNumberReader(fr);
            String line = null;
            while((line = lnr.readLine()) != null){
                System.out.println(lnr.getLineNumber()+":"+line);
            }
            lnr.close();
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

> 他输出的结果

![这里写图片描述](http://img.blog.csdn.net/20160707214921086)

> 他可以获取和设置行号

## 四.自定义LineNumberReader

> 我们可以根据他的原理自己也来实现一个，仔细看注释

~~~
package com.lgl.hellojava;

import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.io.Reader;

public class HelloJJAVA {
    public static void main(String[] args) {
        try {
            FileReader fr = new FileReader("test.txt");
            MyLineNumberReader my = new MyLineNumberReader(fr);
            String line = null;
            while ((line = my.MyReadLine()) != null) {
                System.out.println(my.getLineReader() + ":" + line);
            }
            my.MyClose();
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

class MyLineNumberReader {
    // 读取
    private Reader r;
    // 行号
    private int lineReader;

    // 构造方法
    public MyLineNumberReader(Reader r) {
        this.r = r;
    }

    // 提供对外方法
    public String MyReadLine() {
        // 行号自增
        lineReader++;
        StringBuilder sb = new StringBuilder();
        int ch = 0;
        try {
            while ((ch = r.read()) != -1) {
                if (ch == '\r')
                    continue;
                if (ch == '\n')
                    return sb.toString();
                else
                    sb.append((char) ch);
            }
            if (sb.length() != 0)
                return sb.toString();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return null;
    }

    public int getLineReader() {
        return lineReader;
    }

    public void setLineReader(int lineReader) {
        this.lineReader = lineReader;
    }

    public void MyClose() {
        try {
            r.close();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

}

~~~

> 这个思路是不是很清晰，实际上和LineNumberReader是类似的

## 五.字节流读取操作

> 字符流我们讲的差不多了，我们接着说字节，其实他们类似的，知识他操作的是字节而已

*   inputStream:读
*   outputStream:写

> 我们还是从例子开始

~~~
package com.lgl.hellojava;

import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {
        writeFile();

    }

    // 写文件
    public static void writeFile() {
        try {
            FileOutputStream fo = new FileOutputStream("demo.txt");
            fo.write("test".getBytes());
            fo.close();
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

> 这里我们可以看到，他写入数据不需要刷新，现在还没有涉及到缓存区，我们继续看，写已经写好了，现在我们开始读，对于读取数据，我们开头用到的两种方法

~~~
// 字符读数据
    public static void readFile() {
        try {
            FileInputStream fs = new FileInputStream("demo.txt");
            int ch = 0;
            while ((ch = fs.read()) != -1) {
                System.out.println((char) ch);
            }
            fs.close();
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

    // 字节读取
    public static void readFile1() {
        try {
            FileInputStream fs = new FileInputStream("demo.txt");
            byte[] buf = new byte[1024];
            int len = 0;
            while ((len = fs.read(buf)) != -1) {
                System.out.println(new String(buf, 0, len));
            }
            fs.close();
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
~~~

> 现在我们有了专门处理的字节流，我们可以这样做

~~~
public static void readFile2() {
        try {
            FileInputStream fs = new FileInputStream("demo.txt");
            int num = fs.available();
            System.out.println(num);
            fs.close();
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
~~~

> 我们发现直接用available就可以拿到字节了，原理其实是这段代码

~~~
public static void readFile2() {
        try {
            FileInputStream fs = new FileInputStream("demo.txt");
            byte[] buf = new byte[fs.available()];
            fs.read(buf);
            System.out.println(new String(buf));
            fs.close();
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
~~~

## 六.I/O复制图片

> ok,这里算是一个小练习，复制一张图片，我们理顺下思路

*   1.用字节读取流和图片关联
*   2.用字节流写入流对象创建一个图片文件，存储数据
*   3.通过循环读写，完成数据存储
*   4.关闭流

> OK，我们用代码说话

~~~
package com.lgl.hellojava;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {
        FileOutputStream fos = null;
        FileInputStream fis = null;

        try {
            // 复制
            fos = new FileOutputStream("copy_img.png");
            // 原图
            fis = new FileInputStream("img.png");

            byte[] buf = new byte[1024];

            int len = 0;

            while ((len = fis.read(buf)) != -1) {
                fos.write(buf, 0, len);
            }

            fis.close();
            fos.close();
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

> 这样。我们图片就拷贝过来了

![这里写图片描述](http://img.blog.csdn.net/20160707234857950)

> 好的，知识点今天就到这里

## 有兴趣的可以加群：555974449，咱们一起学习，一起进步！

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。