

# AVA之旅（三十一）——JAVA的图形化界面，GUI布局，Frame，GUI事件监听机制，Action事件，鼠标事件

* * *

> 有段时间没有更新JAVA了，我们今天来说一下JAVA中的图形化界面，也就是GUI

## 一.GUI的概述

> GUI全称叫做Graphical User Intergace(图形用户接口)，用图形的方式，来显示计算机操作的界面，这样更加方便直观，与用户交互
> 
> 说道交互，其实系统跟用户有两种交互，一种是GUI，一种叫做CLI，也就是命令行，全称叫做Command User Intergace,这个需要一些学习成本，倒是不怎么推荐，比如创建文件夹之类的，要是你用CLI那就有点麻烦了
> 
> 回到JAVA，java中为GUI提供的对象都存在java.Awt和javax.Swing两个包中，这两个是什么意思呢？

*   java.Awt：abstract Window ToolKit(抽象窗口工具包)，需要调用本地系统方法实现功能，属于重量级控件
*   javax.Swing：在Awt的基础上，建立的一套图形化界面系统。其中提供了更多的组件，而且完全由java实现，增强了移植性，属于轻量级控件

> 我们来看看大致的组件

![这里写图片描述](http://img.blog.csdn.net/20160730171659782)

## 二.GUI布局

> 我们来学习这些控件之前，我们要学习他的布局，这些组件应该按照什么样的样式排放，这就是布局，常见的布局管理器有以下这几种

*   FlowLayout(流式布局管理器) 

    *   从左往右的顺序排列
    *   Panel默认的布局管理器
*   BorderLayout(边界布局管理器) 

    *   东南西北中
    *   Frame默认的布局管理器
*   GridLayout(网格布局管理器) 

    *   规则的矩阵
*   CardLayout(卡片布局管理器) 

    *   选项卡
*   GridBagLayout(网格包布局管理器) 

    *   非规矩的矩阵

## 三.Frame

> 我们来玩一下这个布局

~~~
package com.lgl.hello;

import java.awt.Frame;

public class Test {
    public static void main(String[] args) {

        Frame f = new Frame("GUI");
        //设置宽高
        f.setSize(300, 200);
        //设置显示位置
        f.setLocation(720, 560);
        //显示
        f.setVisible(true);
    }

}

~~~

> 运行的结果

![这里写图片描述](http://img.blog.csdn.net/20160730175107330)

> 紧接着，我们往里面放控件

~~~
package com.lgl.hello;

import java.awt.Button;
import java.awt.FlowLayout;
import java.awt.Frame;

public class Test {
    public static void main(String[] args) {

        //默认边界布局
        Frame f = new Frame("GUI");
        //设置布局管理器
        f.setLayout(new FlowLayout());
        //设置宽高
        f.setSize(300, 200);
        //设置显示位置
        f.setLocation(720, 560);

        //按钮
        Button b = new Button("Button");
        f.add(b);

        //显示
        f.setVisible(true);
    }

}

~~~

> 运行的结果

![这里写图片描述](http://img.blog.csdn.net/20160730175514732)

> 既然如此，我们就给他设置点击事件了

## 四.GUI事件监听机制

> 我们怎么去监听他的事件？我们先来看下流程图

![这里写图片描述](http://img.blog.csdn.net/20160730182015857)

*   1.事件源
*   2.事件
*   3.监听器
*   4.事件处理

> 我们就直接看代码了，我们先监听这个窗体右上角的关闭按钮

~~~
    // 窗体监听
        f.addWindowListener(new WindowAdapter() {
            @Override
            public void windowClosing(WindowEvent e) {
                // 关闭窗口
                System.exit(0);
            }
        });
~~~

## 五.Action事件

> 我们继续来看，我们先按传统的四位给写好布局

~~~
package com.lgl.hello;

import java.awt.Button;
import java.awt.FlowLayout;
import java.awt.Frame;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class Test {

    public static void main(String[] args) {

        new Test1();
    }
}

class Test1{

    // 定义组件
    private Frame f;
    private Button b;

    // 构造方法
    public Test1() {
        init();
    }

    // 初始化
    private void init() {
        //初始化坐标
        f = new Frame("My GUI");
        //设置坐标
        f.setBounds(300, 100, 600, 500);
        //设置布局
        f.setLayout(new FlowLayout());
        //初始化按钮
        b = new Button("Button");
        //添加到布局
        f.add(b);
        //显示之前加载一下
        myEvent();
        //显示
        f.setVisible(true);
    }

    //监听器
    private void myEvent() {
        f.addWindowListener(new WindowAdapter() {
            @Override
            public void windowClosing(WindowEvent e) {
                System.exit(0);
            }
        });

        //添加按钮事件

    }
}

~~~

> 仔细看代码，我们现在才是添加按钮事件

~~~
    // 添加按钮事件
        b.addActionListener(new ActionListener() {

            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("点击事件");
            }
        });
~~~

> 这样按钮也就具备了点击事件

## 六.鼠标事件

> 什么都有事件，那这样我们来监听一下鼠标的事件

~~~
    // 鼠标事件
        b.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseEntered(MouseEvent e) {
                System.out.println("鼠标进入了");
            }

            @Override
            public void mouseExited(MouseEvent e) {
                System.out.println("鼠标出去了");
            }
        });

~~~

> 十分的简单是吧，那相对的，还有一个键盘事件，提示框什么的，我们这个篇幅就到这里了，下篇文章我们再详细的介绍！

## 有兴趣的可以加群：555974449

版权声明：本文为博主原创文章，博客地址：http://blog.csdn.net/qq_26787115，未经博主允许不得转载。