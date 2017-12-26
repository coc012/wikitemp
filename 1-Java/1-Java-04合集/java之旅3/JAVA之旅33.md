

# JAVA之旅（三十三）——TCP传输，互相（伤害）传输，复制文件，上传图片，多并发上传，多并发登录

* * *

> 我们继续网络编程

## 一.TCP

> 说完UDP，我们就来说下我们应该重点掌握的TCP了

*   TCP传输 

    *   Socket和ServiceSocket
    *   建立客户端和服务端
    *   建立连接后，通过Socket中的IO流进行数据的传输
    *   关闭Socket

> 同样的，我们的客户端和服务端都是两个独立的应用
> 
> 我们通过查阅API文档发现，该对象在建立的时候，就可以去连接指定主机，因为tcp是面向连接的，所以在建立socket服务时，就要有服务存在，并成功连接，形成通路后，在该通道进行数据传输
> 
> 所以我们用代码来看下他的步骤

### 客户端

~~~
package com.lgl.hellojava;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.UnknownHostException;

public class TcpClient {

    public static void main(String[] args) {
        try {
            //1.创建客户端的服务，传地址和端口
            Socket s = new Socket("192.168.1.102",10000);
            //2.为了发送数据，应该获得socket流中的输出流
            OutputStream out = s.getOutputStream();
            out.write("你好".getBytes());
            s.close();
        } catch (UnknownHostException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

~~~

### 服务端

~~~
package com.lgl.hellojava;

import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * 定义端点接收数据打印出来
 * 服务端：
 * 1.建立服务端的socket服务，servicesocket,并监听一个端口
 * 2.获取连接过来的客户端对象，通过accept方法，这个方法是阻塞的，没有连接就会等
 * 3.客户端如果发过来数据，那么服务端要使用对应的客户端对象，并获取到该对象的读取流
 * 4.关闭服务端（可选操作）
 * @author LGL
 *
 */
public class TcpService {
    public static void main(String[] args) {
        try {
            //1.建立连接，监听端口
            ServerSocket ss = new ServerSocket(10000);
            //2.连接客户端对象
            Socket accept = ss.accept();
            //获取ip
            String ip = accept.getInetAddress().getHostAddress();
            //3.获取客户端发送过来的数据
            InputStream in = accept.getInputStream();
            //4.开始读取
            byte [] buf = new byte[1024];
            int len = in.read(buf);
            System.out.println(new String(buf,0,len));
            //5.关闭
            ss.close();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

~~~

## 二.TCP互相传输

> 我们在来写一个实例去说明，他们的互访动作，这里为了写起来方便，就写在一个类中了

~~~
package com.lgl.hellojava;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.UnknownHostException;

/**
 * 客户端发送信息，服务端收到，反馈信息
 * 
 * @author LGL
 *
 */
public class Tcp {

    public static void main(String[] args) {
        try {
            Socket s = new Socket("192.168.1.102", 10005);
            OutputStream out = s.getOutputStream();
            out.write("我是客户端".getBytes());
            InputStream in = s.getInputStream();
            byte[] buf = new byte[1024];
            int len = in.read(buf);
            System.out.println(new String(buf, 0, len));
            s.close();
        } catch (UnknownHostException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
/**
 * 服务端
 * @author LGL
 *
 */
class Server {
    public static void main(String[] args) {
        try {
            ServerSocket ss = new ServerSocket(10005);
            Socket s = ss.accept();
            InputStream in = s.getInputStream();
            byte[] buf = new byte[1024];
            int len = in.read(buf);
            System.out.println(new String(buf, 0, len));

            OutputStream out = s.getOutputStream();
            out.write("收到后反馈".getBytes());
            s.close();
            ss.close();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

~~~

## 三.复制文件

> 同样的这里也是使用的流，我们具体来看下怎么去操作，我们同样的，写在一个类中

~~~
package com.lgl.socket;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.UnknownHostException;

public class FileClient {

    public static void main(String[] args) {
        try {
            Socket s = new Socket("192.168.1.102", 10006);
            BufferedReader bufr = new BufferedReader(new FileReader("test.txt"));
            PrintWriter pw = new PrintWriter(s.getOutputStream(), true);
            String line = null;
            while ((line = bufr.readLine()) != null) {
                pw.println(line);
            }
            pw.print("over");
            BufferedReader bufIn = new BufferedReader(new InputStreamReader(
                    s.getInputStream()));
            String str = bufIn.readLine();
            System.out.println(str);

            bufr.close();
            s.close();
        } catch (UnknownHostException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

class FileServer {
    public static void main(String[] args) {
        try {
            ServerSocket ss = new ServerSocket(10006);
            Socket s = ss.accept();

            BufferedReader bufIn = new BufferedReader(new InputStreamReader(
                    s.getInputStream()));
            PrintWriter out = new PrintWriter(new FileWriter("test1.txt"), true);
            String line = null;
            while ((line = bufIn.readLine()) != null) {
                if ("over".equals(line))
                    break;
                out.println(line);
            }
            PrintWriter pw = new PrintWriter(s.getOutputStream(), true);
            pw.println("上传成功");

            out.close();
            s.close();
            ss.close();

        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

~~~

## 四.上传图片

> 我们再来看下图片是怎么上传的，我们先来分析下步骤

### 客户端

*   1.服务端点
*   2.读取客户端已有的图片数据
*   3.通过socket，发送给服务端
*   4.读取服务端反馈的信息
*   5.关闭资源

~~~
**
 * 客户端
 * 
 * @author LGL
 *
 */
public class PicClient {

    public static void main(String[] args) {
        try {
            Socket s = new Socket("192.168.1.102", 10009);
            FileInputStream fis = new FileInputStream("1.png");
            OutputStream out = s.getOutputStream();
            byte[] buf = new byte[1024];
            int len = 0;
            while ((len = fis.read(buf)) != -1) {
                out.write(buf, 0, len);
            }
            //告訴服务端数据写完
            s.shutdownInput();
            InputStream in = s.getInputStream();
            byte[] bufn = new byte[1024];
            int num = in.read(bufn);
            System.out.println(new String(bufn, 0, num));
            fis.close();
            s.close();
        } catch (UnknownHostException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

~~~

### 服务端

> 直接看代码

~~~

/**
 * 服務端
 * @author LGL
 *
 */
class PicServer {
    public static void main(String[] args) {
        try {
            ServerSocket ss = new ServerSocket(10009);
            Socket s = ss.accept();
            InputStream in = s.getInputStream();
            FileOutputStream fos = new FileOutputStream("2.png");
            byte[] buf = new byte[1024];
            int len = 0;
            while ((len = in.read(buf)) != -1) {
                fos.write(buf, 0, len);
            }

            OutputStream out = s.getOutputStream();
            out.write("上传成功".getBytes());
            fos.close();
            s.close();
            ss.close();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
~~~

> 其实跟I/O区别真不大，但是概念一定要了解清楚

## 五.多并发上传

> 多并发这个概念就是多人互动了，这对服务器的负荷还是有考究的，这里呢，我们就模拟一下，多人上传图片的场景，我们是怎么做的？我们还是在上传图片的那份代码上更改
> 
> 首先我们可以确定的是，这是服务端的代码

*   这个服务端有个局限性，当A客户端连接之后，被服务端获取到，服务端就在执行代码了，这个时候如果B客户端连接只有等待，这就是我们需要多并发的原因了，为了让多个客户端同时连接，服务端最好就是讲每个客户端封装到一个单独的线程中，这样就可以同时处理多个客户端请求

> 如何定义线程？

*   只要明确了每个客户端要在服务端执行的代码即可

~~~

/**
 * 服務端
 * 
 * @author LGL
 *
 */
class PicServer {
    public static void main(String[] args) {
        try {
            ServerSocket ss = new ServerSocket(10009);
            while (true) {
                Socket s = ss.accept();
                new Thread(new PicThread(s)).start();
            }
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
/**
 * 并发线程
 * @author LGL
 *
 */
class PicThread implements Runnable {

    private Socket s;

    public PicThread(Socket s) {
        this.s = s;
    }

    @Override
    public void run() {
        try {
            String ip = s.getInetAddress().getHostAddress();
            System.out.println("ip：" + ip);

            long millis = System.currentTimeMillis();
            File file = new File(millis + ".png");
            InputStream in = s.getInputStream();
            FileOutputStream fos = new FileOutputStream(file);
            byte[] buf = new byte[1024];
            int len = 0;
            while ((len = in.read(buf)) != -1) {
                fos.write(buf, 0, len);
            }

            OutputStream out = s.getOutputStream();
            out.write("上传成功".getBytes());
            fos.close();
            s.close();
        } catch (Exception e) {

            throw new RuntimeException("上传失败");
        }

    }

}
~~~

> 其实我写的代码还是有点烂的，但是思想在就好，我们得先把思想学会了

## 六.多并发登录

> 上面说的多并发的上传，实在服务端端，现在我们来说下登录，是作用在客户端

~~~
package com.lgl.socket;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.UnknownHostException;

public class LoginClient {

    public static void main(String[] args) {
        try {
            Socket s = new Socket("192.168.1.102", 10008);
            BufferedReader bufr = new BufferedReader(new InputStreamReader(
                    System.in));
            PrintWriter out = new PrintWriter(s.getOutputStream(), true);
            BufferedReader bufIn = new BufferedReader(new InputStreamReader(
                    s.getInputStream()));
            for (int i = 0; i < 3; i++) {
                String line = bufr.readLine();
                if (line == null) {
                    break;
                }
                out.println(line);

                String info = bufIn.readLine();
                System.out.println("info:" + info);
                if (info.contains("欢迎")) {
                    break;
                }
            }
            bufr.close();
            s.close();
        } catch (UnknownHostException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

/**
 * 服务端
 * 
 * @author LGL
 *
 */
class LoginServer {
    public static void main(String[] args) {
        try {
            ServerSocket ss = new ServerSocket(10008);
            while (true) {
                Socket s = ss.accept();
                new Thread(new UserThread(s)).start();
            }
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

/**
 * 并发登陆
 * 
 * @author LGL
 *
 */
class UserThread implements Runnable {

    private Socket s;

    public UserThread(Socket s) {
        this.s = s;
    }

    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            try {
                BufferedReader bufrIn = new BufferedReader(
                        new InputStreamReader(s.getInputStream()));
                String name = bufrIn.readLine();

                // 模拟读取数据库的用户名
                BufferedReader bufr = new BufferedReader(new FileReader(
                        "user.txt"));

                PrintWriter out = new PrintWriter(s.getOutputStream(), true);

                String line = null;

                boolean flag = false;
                while ((line = bufr.readLine()) != null) {
                    if (line.equals(name)) {
                        flag = true;
                        break;
                    }
                }
                if (flag) {
                    System.out.println("已登录");
                    out.print("欢迎");
                } else {
                    System.out.println("重新登录");
                    out.println("用户名不存在");
                }
                s.close();

            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }

        }
    }

}
~~~

> OK,这些代码中可能会存在一些错误，因为代码并没有去实际的验证中，我写的时候也是跟着思想去走的，这样写代码是极为友好的，这就是TCP的冰山一角了，不过关于这些，还有很多知识点，我们要做的就是把思想给掌握了，万变不理其中
> 
> 好的，最近写文的时间，有点懈怠了，看来要发力了，嘻嘻，

## 有兴趣的加群：555974449 一起来玩玩吧！

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。