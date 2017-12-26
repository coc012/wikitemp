

# JAVA之旅（二十七）——字节流的缓冲区，拷贝mp3，自定义字节流缓冲区，读取键盘录入，转换流InputStreamReader，写入转换流，流操作的规律

* * *

> 我们继续来聊聊I/O

## 一.字节流的缓冲区

> 这个，和我们的字符流基本上没有什么差别，我们来拷贝mp3，看例子

~~~
// 通过字节流的缓冲区拷贝图片
    public static void copyMp3() {

        try {
            FileInputStream fi = new FileInputStream("audio.mp3");
            BufferedInputStream buf = new BufferedInputStream(fi);

            FileOutputStream fio = new FileOutputStream("audioCapy.mp3");
            BufferedOutputStream buo = new BufferedOutputStream(fio);

            int ch = 0;

            while ((ch = buf.read()) != -1) {
                buo.write(ch);
            }

            buf.close();
            buo.close();
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
~~~

> 这样，就直接拷贝了

![这里写图片描述](http://img.blog.csdn.net/20160709085335274)

## 二.自定义字节流缓冲区

> 我们队缓冲区已经了解很多了，这样的话，我们来尝试解析他的原理然后自定义一个字节流的缓冲区出来，来看看对不对

~~~
class MyBufferedImputStream {

    private InputStream in;
    private byte[] buf = new byte[1024];
    private int pos = 0;
    private int count = 0;

    public MyBufferedImputStream(InputStream in) {
        this.in = in;
    }

    // 从缓冲区一次读一个字节
    public int myRead() throws IOException {
        // 通过in对象读取硬盘上的数据，存储在buf
        if (count == 0) {
            count = in.read(buf);
            if (count < 0)
                return -1;
            byte b = buf[pos];
            count--;
            pos++;
            return b;
        } else if (count > 0) {
            byte b = buf[pos];
            pos++;
            count--;
            return b;
        }
        return -1;
    }
    //关闭流
    public void myClose() throws IOException {
        in.close();
    }
}

~~~

> 思路是比较清晰的，想知道对不对，小伙伴赶紧去试试

## 三.读取键盘录入

> 这个其实早就要讲，现在讲就有点晚了，就是键盘输入文字读取

~~~
package com.lgl.hellojava;

import java.io.IOException;
import java.io.InputStream;

public class HelloJJAVA {
    public static void main(String[] args) throws IOException {
        /**
         * 通过键盘录入数据 当录入一行数据后，打印 发现over,停止
         */
        InputStream in = System.in;
        StringBuilder sb = new StringBuilder();
        while (true) {
            int ch = in.read();
            if (ch == '\r')
                continue;
            if (ch == '\n') {
                String s = sb.toString();
                if ("over".equals(s))
                    break;
                System.out.println(s);
                // delte all
                sb.delete(0, sb.length());
            } else
                sb.append(ch);

        }
    }
}

~~~

> 当我们写完之后就发现，这个写法我们之前是有写过的，就是readLine的原理，这样的话，我们可以对其进行改造一下，但是这里就产生了一个新的问题，一个是字符流，一个是字节流，那这里也就产生了一个思考，能不能将字节流转换成字符流，再去使用它缓冲区的readLine方法呢？

## 四.转换流InputStreamReader

> java中需要转换流就会使用到转换流，使用到了InputStreamReader，你会发现十分的方便的

~~~
package com.lgl.hellojava;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;

public class HelloJJAVA {
    public static void main(String[] args) throws IOException {
        //获取键盘录入对象
        InputStream in = System.in;
        //转换
        InputStreamReader isr = new InputStreamReader(in);
        //提高效率
        BufferedReader bur = new BufferedReader(isr);
        String line = null;
        while((line = bur.readLine()) != null){
            if(line.equals("over"))
                break;
            System.out.println(line.toString());
        }
    }
}

~~~

> 我们来演示一下

![这里写图片描述](http://img.blog.csdn.net/20160709095031952)

## 五.写入转换流

> 我们转换流的read学完了，我们就来学习一下write.我们继续增强上面的方法

~~~
package com.lgl.hellojava;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.OutputStreamWriter;

public class HelloJJAVA {
    public static void main(String[] args) throws IOException {
        //获取键盘录入对象
        InputStream in = System.in;
        //转换
        InputStreamReader isr = new InputStreamReader(in);
        //提高效率
        BufferedReader bur = new BufferedReader(isr);

        OutputStream os = System.out;
        OutputStreamWriter osw = new OutputStreamWriter(os);
        BufferedWriter bufw = new BufferedWriter(osw);

        String line = null;
        while((line = bur.readLine()) != null){
            if(line.equals("over"))
                break;
            bufw.write(line.toString());
            bufw.newLine();
            bufw.flush();
        }
    }
}

~~~

> OK,实现的功能也是正常的了；

## 六.流操作的规律

> 我们写了这么多流，我们来总结一下规律

*   1 

    *   源：键盘录入
    *   目的：控制台
*   2 ： 需求：想把键盘录入的数据存储到一个文件中 

    *   源：键盘
    *   目的：文件
*   3.需求：想要将一个文件的数据打印在控制台上 

    *   源：文件
    *   目的：控制台

> 流操作的基本规律

*   最痛苦的就是不知道流对象要用哪一个
*   通过两个明确来完成 

    *   1.明确源和目的 

        *   源：输入流 InputStream Reader
        *   目的：输出流 OutputStream writer
    *   2.明确操作的数据是否是纯文本 

        *   是：字符流
        *   不是：字节流
    *   3.当体系明确后，再明确要使用哪个具体的对象 

        *   通过设备来进行区分
        *   源设备：内存，硬盘，键盘
        *   目的：内存，硬盘，控制台。

> 前面两个是比较重要的，也可以明确出来，第三个就是加分项了
> 
> I/O就先到这里了，我们下篇继续聊，同时开始讲File了

## 有兴趣的可以加群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。