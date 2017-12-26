

# JAVA之旅（三十四）——自定义服务端，URLConnection，正则表达式特点，匹配，切割，替换，获取，网页爬虫

* * *

> 我们接着来说网络编程，TCP

## 一.自定义服务端

> 我们直接写一个服务端，让本机去连接，可以看到什么样的效果

~~~
package com.lgl.socket;

import java.io.IOException;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;

public class BrowserServer {

    //http://192.168.1.103:11000/

    public static void main(String[] args) {
        try {
            ServerSocket ss = new ServerSocket(11000);
            Socket s = ss.accept();
            System.out.println(s.getInetAddress().getHostName() + ":" + s.getInetAddress().getHostAddress());
            PrintWriter out = new PrintWriter(s.getOutputStream(), true);
            out.println("Hello Client");
            s.close();
            ss.close();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

}

~~~

> 我们运行了之后直接访问[http://192.168.1.103:11000/](http://192.168.1.103:11000/)就知道什么效果

![这里写图片描述](http://img.blog.csdn.net/20160821154226135)

> 我们控制台也打印出我们的地址来了

![这里写图片描述](http://img.blog.csdn.net/20160821154323802)

> 比较有意思的是，既然是网页打开，那么他是支持html的，我们来输出这句

~~~
out.println("<font color='red' size='30'>Hello Client");
~~~

> 你就可以看到

![这里写图片描述](http://img.blog.csdn.net/20160821154620140)

## 二.URLConnection

> 先看URL的用法

~~~
package com.lgl.socket;

import java.net.MalformedURLException;
import java.net.URL;

public class URLDemo {

    public static void main(String[] args) {
        try {
            URL url = new URL("http://192.168.1.102/myweb/test.html？name=zhangsan&age=18");
            // 协议
            System.out.println(url.getProtocol());
            // 主机
            System.out.println(url.getHost());
            // 端口
            System.out.println(url.getPort());
            // 路径
            System.out.println(url.getPath());
            // 查询部
            System.out.println(url.getQuery());
        } catch (MalformedURLException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

}

~~~

> 得到的结果

![这里写图片描述](http://img.blog.csdn.net/20160823215453693)

> 继续来看

~~~
// 返回一个url连接对象
URLConnection openConnection = url.openConnection();
System.out.println(openConnection);
InputStream inputStream = openConnection.getInputStream();
byte[] buf = new byte[1024];
int len = inputStream.read(buf);
System.out.println(new String(buf, 0, len));
~~~

> 其实可以读取流，我们从流中拿到我们想要的东西

## 三.正则表达式特点

> 正则表达式：你可以理解为符合一定规则的表达式，正则我们虽然用的不多，但是确实比较适用的，我们主要来看他的做用

*   专门操作字符串

> 我们直接来看下使用方法
> 
> 我们现在有一个需求

*   对QQ号码进行效验，要求5-15位，不能开头，只能是数字

> 先看一下我们的传统方式是怎么去计算的

~~~
public class Test {

    public static void main(String[] args) {
        /**
         * 对QQ号码进行效验，要求5-15位，不能开头，只能是数字
         */
        String qq = "11299923";
        int len = qq.length();
        // 长度
        if (len > 5 && len <= 15) {
            // 不能0开头
            if (!qq.startsWith("0")) {
                // 全部是数字
                char[] charArray = qq.toCharArray();
                boolean flag = false;
                for (int i = 0; i < charArray.length; i++) {
                    if (!(charArray[i] >= '0' && charArray[i] <= '9')) {
                        flag = true;
                        break;
                    }
                }
                if (flag) {
                    System.err.println("QQ:" + qq);
                } else {
                    System.out.println("非纯数字");
                }
            } else {
                System.out.println("0开头不符合规范");
            }
        } else {
            System.out.println("QQ长度有问题");
        }
    }
}

~~~

> 这是一件非常麻烦的事情的，而我们来看下正则表达式，是怎么表示的

~~~
public class Test1 {

    public static void main(String[] args) {

        String qq = "789152";
        /**
         * 我只要告诉你对与错就行
         */
        String regex = "[1-9][0-9]{4,14}";
        boolean flag = qq.matches(regex);
        if (flag) {
            System.out.println("QQ:" + qq);
        } else {
            System.out.println("错误");
        }
    }
}

~~~

> 非常的强大，只要几行代码就可以显示，牛啊，这符号定义我们稍后解答

## 四.匹配

> 正则很厉害，我们来看下他的作用

*   特点：用一些特定的符号来表示一些代码操作，这样就简化了书写，学习正则表达式就是用来学习一些特殊符号的使用

![这里写图片描述](http://img.blog.csdn.net/20160824223314946)

*   1.匹配：matches

> 我们来看下这段代码

~~~
        String str = "c";
        /**
         * 这个字符串只能是bcd中的其中一个，而且只能是一个字符
         */
        String reg = "[bcd]";
        boolean flag = str.matches(reg);
        System.out.println(flag);
~~~

> 含义理解清楚，其实就比较顺眼了一点点了，我们继续

~~~
        /**
         * 这个字符的第二位是a-z就行
         */
        String reg1 = "[bcd][a-z]";
        boolean flag1 = str.matches(reg);
        System.out.println(flag1);
~~~

> 到现在是否是有点概念？我们继续，如果我现在想我第一个是个字母第二个是个数字，该怎么去拼？

~~~
    String reg2 = "[a-zA-Z][0-9]";
    boolean flag2 = str.matches(reg2);
    System.out.println(flag2);
~~~

> 大致的讲解一下，因为我也不是很熟，嘿嘿

## 五.切割

> 这个切割，在string也是一个切割split，而我们的正则，也是有的，我们继续看

~~~
public class Test2 {

    public static void main(String[] args) {

        String str = "zhangsan,lisi,wangwu";
        String reg = ",";
        String[] split = str.split(reg);
        for (String s : split) {
            System.out.println(s);
        }
    }
}

~~~

> 我们输出

![这里写图片描述](http://img.blog.csdn.net/20160827162818963)

## 六.替换

> 正则表达式就是string的操作，我们看下替换

~~~
public class Test2 {

    public static void main(String[] args) {
        // 将数字连续超过五个替换成#号
        replaceAll("fwfsda777777fs74666677s", "\\d{5,}", "#");
    }

    public static void replaceAll(String str, String reg, String newStr) {
        str = str.replaceAll(reg, newStr);
        System.out.println(str);
    }
}

~~~

> 得到的结果

![这里写图片描述](http://img.blog.csdn.net/20160827163958152)

# 七.获取

*   1.将正则表达式封装成对象
*   2.让正则表达式和要操作的对象进行关联
*   3.关联后，获取正则匹配引擎
*   4.通过引擎对符合规则的子串进行操作，比如取出

~~~
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class Test2 {

    public static void main(String[] args) {

        String string  = " hello java android c cc ccc cccc ccccc";
        //test
        String reg = "[a-z]";

        //将规则封装成对象
        Pattern p = Pattern.compile(reg);
        //让正则对象和要作用的字符串相关联,获取匹配器对象
        Matcher matcher = p.matcher(string);
        System.out.println(matcher.matches());
    }

}

~~~

> 体现了一个模式而已，我们可用通过这个模式去获取字符串

## 八.网页爬虫

> 爬虫我们再熟悉不过了，也俗称蜘蛛，其实就是获取一些数据罢了，我们也是可以用到我们正则中的获取功能的

~~~
import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class Test2 {

    public static void main(String[] args) {

    }

    /**
     * 获取指定文档中的邮箱地址
     */
    public static void getEmail() {
        try {
            BufferedReader bufr = new BufferedReader(
                    new FileReader("email.txt"));
            String line = null;
            String emailreg = "\\w+@\\w+(\\.\\w+)+";
            Pattern p = Pattern.compile(emailreg);
            while ((line = bufr.readLine()) != null) {
                System.out.println(line);
                // 判断邮箱
                Matcher m = p.matcher(line);
                while (m.find()) {
                    System.out.println(m.group());
                    // 这样就拿到所有的邮箱了
                }
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

~~~

> 这样我们的所有邮箱号码就拿到了，当然，这只是一个简单的爬虫概念，爬虫博大精深，我们要学习的话还是要系统的了解一下才好！！！
> 
> 好的，我们的java之旅也到这里over了，我们本篇也结束了

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。