

# JAVA之旅（二十五）——文件复制,字符流的缓冲区，BufferedWriter，BufferedReader，通过缓冲区复制文件，readLine工作原理，自定义readLine

* * *

> 我们继续IO上个篇幅讲

## 一.文本复制

> 读写都说了，我们来看下其他的操作，我们首先来看复制

*   复制原理：其实就是将C盘下的文件数据存储到D盘的一个文件中

> 实现的步骤： 
> 1.在D盘创建一个文件，用于存储文件中的数据 
> 2.定义读取流和文件关联 
> 3.通过不断的读写完成数据的存储 
> 关闭资源

~~~
package com.lgl.hellojava;

import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {

        copy_1();
        copy_2();
    }

    // 从c盘读一个字符，就往D盘写一个字符
    public static void copy_1() {
        try {
            // 创建目的地
            FileWriter fw = new FileWriter("copy_1.txt");
            // 与已有文件关联
            FileReader fr = new FileReader("copy_1.txt");
            int ch = 0;
            while ((ch = fr.read()) != -1) {
                // 读一个 写一个
                fw.write(ch);
            }
            fw.close();
            fr.close();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

    public static void copy_2() {
        FileWriter fw = null;
        FileReader fr = null;

        try {
            fw = new FileWriter("copy_2.txt");
            fr = new FileReader("copy_2.txt");

            char[] buf = new char[1024];

            int len = 0;
            while ((len = fr.read(buf)) != -1) {
                fw.write(buf, 0, len);
            }
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } finally {
            if (fr != null) {
                try {
                    fr.close();
                } catch (IOException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }

            if (fw != null) {
                try {
                    fw.close();
                } catch (IOException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        }
    }
}

~~~

> 这里做了两种方式的拷贝方式，其实都是整理好思路，读和写的一个过程罢了！

## 二.字符流的缓冲区

> 字符流的缓冲区，提高了对数据的读写效率，他有两个子类

*   BufferedWriter
*   BufferedReader 

> 缓冲区要结合柳才可以使用 
> 在流的基础上对流的功能进行了增强

### 1.BufferedWriter

![这里写图片描述](http://img.blog.csdn.net/20160702170355221)

> 缓冲区的出现是提高流的效率而出现的，所以在创建缓冲区之前，必须先有流对象，我们看例子

~~~
package com.lgl.hellojava;

import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {

        try {
            // 创建一个字符写入流对象
            FileWriter fw = new FileWriter("buffer.txt");
            // 为了提高写入流的效率加入了缓冲技术
            BufferedWriter bufw = new BufferedWriter(fw);
            //写入数据
            bufw.write("hello");
            //换行
            bufw.newLine();

            //只要用到了缓冲区，就需要刷新
            bufw.flush();

            //缓冲区关闭的就是关联的流
            bufw.close();

        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

    }
}

~~~

> 使用都是比较基础的，大家也是可以看到

### 2.BufferedReader

> 高效读取

![这里写图片描述](http://img.blog.csdn.net/20160702170544941)

> 我们直接看代码

~~~
package com.lgl.hellojava;

import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {

        try {
            // 创建一个读取流对象和文件相关联
            FileReader fr = new FileReader("buffer.txt");
            // 为了提高效率，加入缓冲技术
            BufferedReader bfr = new BufferedReader(fr);

            String line = null;
            while((line = bfr.readLine()) != null){
                System.out.println(line);
            }
            bfr.close();
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

> 这样就可以全部出来了

## 三.通过缓冲区复制文件

> OK，我们还是复制文件这个问题，现在我们有缓冲区，我们要怎么样复制文件？

~~~
package com.lgl.hellojava;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {
        /**
         * 缓冲区文件复制
         */
        BufferedReader bufr = null;
        BufferedWriter bufw = null;

        try {
            bufr = new BufferedReader(new FileReader("buffer.txt"));
            bufw = new BufferedWriter(new FileWriter("buffercopy.txt"));

            String line = null;

            while((line = bufr.readLine()) != null){
                bufw.write(line);
            }

            //关闭流
            bufr.close();
            bufw.close();
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

> 这样，就可以复制文件了

## 四.readLine工作原理

> 我们注意到我们要使用这个方法readline,无论是读一行还是读多个字符，其实都是在硬盘上一个一个读取，所以最终使用的还是read方法一个读一个的方法

*   其实他内存中有一个数组，你读完之后并没有立马读，而是临时存储起来，这就是缓冲区，

![这里写图片描述](http://img.blog.csdn.net/20160702173638937)

> 当读到换行，才去返回一行数据，就这样一行一行的读取，这就是他的工作原理

## 五.自定义readLine

> 我们了解了readLine的工作原理，那我们就可以尝试去更改他了，自定义一个怎么样？我们尝试一下

~~~
package com.lgl.hellojava;

import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

public class HelloJJAVA {
    public static void main(String[] args) {
        /**
         * 自定义readLine
         */
        FileReader fr;
        try {
            fr = new FileReader("buffer.txt");
            MyBufferReader my = new MyBufferReader(fr);
            String line = null;

            while ((line = my.myReadLine()) != null) {
                System.out.println(line);
            }
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

    }
}

class MyBufferReader {

    private FileReader fr;

    public MyBufferReader(FileReader fr) {
        this.fr = fr;
    }

    // 一次读取一行的方法
    public String myReadLine() throws IOException {

        // 定义临时容器
        StringBuilder sb = new StringBuilder();
        int ch = 0;
        while ((ch = fr.read()) != -1) {

            if (ch == '\r') {
                continue;
            } else if (ch == '\n') {
                return sb.toString();
            } else {
                sb.append((char) ch);
            }
        }
        if(sb.length() != 0){
            return sb.toString();
        }
        return null;
    }

    public void close() throws IOException {
        fr.close();
    }
}

~~~

> 仔细看实现思路，静静的看，没错，我们也是可以实现的，好的，我们本篇到这里也OK，算是结束了，我们下一篇继续会将IO的，毕竟这是一个大知识点！

## 有兴趣可以加群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。