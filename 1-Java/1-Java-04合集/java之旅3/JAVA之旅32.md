

# JAVA之旅（三十二）——JAVA网络请求，IP地址,TCP/UDP通讯协议概述，Socket，UDP传输，多线程UDP聊天应用

* * *

> GUI写到一半电脑系统挂了，也就算了，最多GUI还有一个提示框和实例，我们暂时不讲了，我们直接来重点吧，关于JAVA的网络请求是怎么实现的？当然是HTTP协议，但是不可否认，他的概念和思想都是我们必须去涉及的，包括后面的tcp和socket等，好吧，我们开车吧！

## 一.JAVA网络请求概述

> 关于JAVA的网络请求，我们大致的可以分为以下几个分类

*   网络模式 

    *   OSI
    *   TCP/IP
*   网络通讯 

    *   IP地址
    *   端口号
    *   传输协议

> 拿这些都是干嘛的呢？我们接下来都会讲到
> 
> 首先我们应该思考的是他们通信的一个过程的步骤

*   1.找到对方IP
*   2.数据发送到指定应用程序上，为了识别，就有了端口的概念
*   3.定义通信协议（也就是后来的传输协议）国际协议/TCP/IP
*   4.三要素：IP，端口，协议

> OK，那我们就研究下网络模型，OSI和TCP/IP的区别 
> 其实理解起来也不难，我们看一下他的逻辑结构就知道了

*   OSI

    *   应用层
    *   表示层
    *   会话层
    *   传输层
    *   网络层
    *   数据链路层
    *   物理层
*   TCP/IP

    *   应用层
    *   传输层
    *   网络层
    *   主机-网络层

> 应用层，我们就在这里玩，TCP封装了就比较好用，他们都有使用规则，而我们常用的大概就是HTTP协议了

## 二.IP地址

> 通讯要素大致的就是这些，我们来说一下我们耳熟能详的IP地址，他是什么概念呢？

*   IP地址

    *   网络中设备的标识
    *   可用主机名
    *   本地回环地址：127.0.0.1，主机名：location
*   端口号

    *   用于标识进程的逻辑地址，不同进程的标识
    *   有效端口：0-65535，其中0-1024系统使用或者保留，我们熟知的8080
*   通讯协议

    *   通讯的规则
    *   常见的TCP，UDP

> 我们可用用代码获得哦，先看API文档,会发现JAVA给我们提供了一个类InetAddress
> 
> 我们可用直接去用代码使用

~~~
try {
    InetAddress localHost = InetAddress.getLocalHost();
    System.out.println(localHost.toString());
} catch (UnknownHostException e) {
    // TODO Auto-generated catch block
    e.printStackTrace();
}
~~~

> 可以得到

![这里写图片描述](http://img.blog.csdn.net/20160816223428992)

> 得到的本机的主机名和IP地址 
> 当然，你要单独获取也是没问题的

~~~
        try {
            InetAddress localHost = InetAddress.getLocalHost();
            String hostAddress = localHost.getHostAddress();
            String hostName = localHost.getHostName();
            System.out.println(localHost.toString());
        } catch (UnknownHostException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
~~~

## 三.TCP/UDP通讯协议概述

> 端口我们没什么可说的，我们直接说通讯协议，目前常见的就是TCP/UDP了，我们先来简单的说下他们的概念

*   TCP

    *   建立连接，形成传输数据的通道
    *   在连接中进行大数据量传输
    *   通过三次握手完成连接，是可靠协议
    *   必须建立连接，效率稍微低点
*   UDP

    *   将数据及源和目的封装在数据包中，不需要建立连接
    *   每个数据包的大小限制在64K内
    *   因无连接，是不可靠协议
    *   不需要建立连接，速度快

> 这些这么多，java肯定会给我们封装对象的，这个是毋庸置疑的，那我们接着往下看

## 四.Socket

> Socket就厉害了，我们先来看看他的概念

*   Socket就是为网络服务提供的一种机制
*   通信的两端都有socket
*   网络通信其实就是socket通信
*   数据在两个socket通过IO传输

> 我们现在先说概念，后期再实战

## 五.UDP传输

> UDP传输的socket服务该怎么建立？

*   DatagramSocket和DatagramPacket
*   建立发送端和接收端
*   建立数据包
*   调用socket的发送和接收方法
*   关闭socket

> 客户端和服务端是两个单独的服务，我们可用来用代码讲解下，用到的就是DatagramSocket和DatagramPacket
> 
> 所以这里应该是有两个，一个传输端，一个接收端

### 传输端

~~~

/**
 * 需求： 通过UDP传输方式将一段文字数据发送出去 
 * 思路： 
 * 1.建立UDP的socket服务 
 * 2.建立数据包 
 * 3.发送数据 
 * 4.关闭资源
 * 
 * @author LGL
 *
 */
public class UdpSend {
    public static void main(String[] args) {
        try {
            // 1.建立UDP的socket服务,通过DatagramSocket对象
            DatagramSocket dSocket = new DatagramSocket();
            // 2.确定数据，封装成数据包
            byte[] data = "udp".getBytes();
            DatagramPacket dp = new DatagramPacket(data, data.length,
                    InetAddress.getByName("192.168.1.102"), 8080);
            // 3.发送数据
            dSocket.send(dp);
            // 4.关闭资源
            dSocket.close();
        } catch (SocketException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
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

### 接收端

~~~
/**
 * 需求：接收指定端口发送过来的数据 
 * 思路： 
 * 1.定义socket服务
 * 2.定义数据包，存储接收到的字节数据，因为数据包对象中有更多功能可以提取字节数据中的不同数据信息
 * 3.通过socket的receive方法收到的数据存储到数据包中 
 * 4.将这些不同的数据取出，打印 
 * 5.关闭资源
 * 
 * @author LGL
 *
 */
class UdpRece {
    public static void main(String[] args) {
        try {
            // 1.创建服务，建立端点
            DatagramSocket dSocket = new DatagramSocket(8080);
            // 2.定义数据包，存储数据
            byte[] buf = new byte[1024];
            DatagramPacket dp = new DatagramPacket(buf, buf.length);
            // 3.存储
            dSocket.receive(dp);
            // 4.获取其中的数据
            String ip = dp.getAddress().getHostAddress();
            String data = new String(dp.getData(), 0, dp.getLength());
            int port = dp.getPort();
            System.out.println(ip+"：" + data + ":" + port);
            //5.关闭资源
            dSocket.close();
        } catch (SocketException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

~~~

> 这样就可以通信了

## 六.多线程UDP聊天应用

> 既然上面有模有样的写出来了，那我们可以动手写一个应用了，我们继续来看，我不开多个进程，我写一个进程，两个线程来实现聊天

~~~
package com.lgl.hellojava;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.SocketException;

/**
 * 编写一个聊天应用程序 有收数据和发数据的部分，所以用到多线程的技术，一个接一个发 收和发的动作不一致，所以有两个Runnable
 * 
 * @author LGL
 *
 */
public class UdpSpeak {

    public static void main(String[] args) {

        try {
            DatagramSocket sendSocket = new DatagramSocket();
            DatagramSocket receSocket = new DatagramSocket(10000);

            new Thread(new send(sendSocket)).start();
            new Thread(new rece(receSocket)).start();

        } catch (SocketException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

/**
 * 发送
 * 
 * @author LGL
 *
 */
class send implements Runnable {

    private DatagramSocket socket;

    public send(DatagramSocket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            BufferedReader bufr = new BufferedReader(new InputStreamReader(
                    System.in));
            String line = null;
            while ((line = bufr.readLine()) != null) {
                if ("close".equals(line)) {
                    break;
                }
                byte[] buf = line.getBytes();
                DatagramPacket dp = new DatagramPacket(buf, buf.length,
                        InetAddress.getByName("192.168.1.102"), 10000);
                socket.send(dp);
            }
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

}

/**
 * 接收
 * 
 * @author LGL
 *
 */
class rece implements Runnable {

    private DatagramSocket socket;

    public rece(DatagramSocket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        while (true) {
            try {
                byte[] buf = new byte[1024];
                DatagramPacket dp = new DatagramPacket(buf, buf.length);
                socket.receive(dp);

                String ip = dp.getAddress().getHostAddress();
                String data = new String(dp.getData(), 0, dp.getLength());
                int port = dp.getPort();
                System.out.println(ip + "：" + data + ":" + port);

            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }

}
~~~

> OK,搞定，其实主要还是要了解他的思想，编码什么的不重要的
> 
> 好了，本篇主要是以UDP和概念为起点，而且UDP用的较少，我们一般不是常接触，真正要用的是TCP，所以会重点掌握，那本篇，我们先到这里就好了

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。