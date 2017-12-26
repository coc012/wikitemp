版权信息
原始链接：[ConstraintLayout 终极秘籍](http://blog.chengyunfeng.com/?p=1030)（上）

---
个人笔记：


---
原文


[ConstraintLayout](https://developer.android.com/reference/android/support/constraint/ConstraintLayout.html) 终于[正式发布 1.0 版本](http://tools.android.com/recent/constraintlayout10isnowavailable)了,是时候来详细介绍下这个 Android 布局的终极武器了。

## 为何需要 ConstraintLayout

Android 上面布局嵌套层级直接影响 UI 界面绘制的效率，如果 UI 嵌套层级太多会导致界面有[性能问题](http://blog.venmo.com/hf2t3h4x98p5e13z82pl8j66ngcmry/performance-tuning-on-android)，目前对于复杂的界面，使用 [RelativeLayout](https://developer.android.com/reference/android/widget/RelativeLayout.html) 也无法解决。所以 Android UI 团队就在去年 Google IO 开发者大会上发布了一个新的布局控件 — ConstraintLayout。

ConstraintLayout 可以看做 RelativeLayout 的升级版。可以有更多的手段来控制里面的子 View 的布局，所以对于复杂的布局用 ConstraintLayout 一个布局容器即可实现。

## 设置开发环境

ConstraintLayout 是一个新的 Support 库，支持 Android 2.3 (API level 9) 以及以后的版本。在 Android Studio 中使用 ConstraintLayout 之前需要先下载最新的 ConstraintLayout 库，步骤如下：

1.  在 Android Studio 中选择菜单 Tools > Android > SDK Manager
2.  点击 SDK Tools Tab 页
3.  滚动到最下面找到 Support Repository 部分，然后勾选 ConstraintLayout for Android 和 Solver for ConstraintLayout 来安装新的 1.0 版本。 （勾选界面右下方的 ** Show Package Details ** 可以查看已经安装的版本。可以取消不需要的版本来卸载旧版本。）
4.  点击 OK 或者 Apply 可以安装需要的版本。
5.  安装完以后，在你项目模块的 build.gradle 中添加依赖项：

~~~Java
dependencies  {
    compile  'com.android.support.constraint:constraint-layout:1.0.0'
}
~~~


1.  选择 Sync Project with Gradle Files 来同步 Gradle 信息。

下载 ConstraintLayout 的截图
![image](http://pic.goodev.org/wp-files/2017/02/cl/cs_down.png)

然后就可以开始在项目中使用 ConstraintLayout 了。

## ConstraintLayout 布局属性详解

ConstraintLayout 中的 Constraint 为名词（翻译为：约束;限制;强制），所以顾名思义该布局在每个在子 View 上添加各种约束条件来控制每个子 View 所处的位置以及显示的尺寸。ConstraintLayout 在 1.0 版本有如下 54 个布局属性：

1.  android_maxHeight
2.  android_maxWidth
3.  android_minHeight
4.  android_minWidth
5.  android_orientation
6.  layout_constraintBaseline_creator
7.  layout_constraintBaseline_toBaselineOf
8.  layout_constraintBottom_creator
9.  layout_constraintBottom_toBottomOf
10.  layout_constraintBottom_toTopOf
11.  layout_constraintDimensionRatio
12.  layout_constraintEnd_toEndOf
13.  layout_constraintEnd_toStartOf
14.  layout_constraintGuide_begin
15.  layout_constraintGuide_end
16.  layout_constraintGuide_percent
17.  layout_constraintHeight_default
18.  layout_constraintHeight_max
19.  layout_constraintHeight_min
20.  layout_constraintHorizontal_bias
21.  layout_constraintHorizontal_chainStyle
22.  layout_constraintHorizontal_weight
23.  layout_constraintLeft_creator
24.  layout_constraintLeft_toLeftOf
25.  layout_constraintLeft_toRightOf
26.  layout_constraintRight_creator
27.  layout_constraintRight_toLeftOf
28.  layout_constraintRight_toRightOf
29.  layout_constraintStart_toEndOf
30.  layout_constraintStart_toStartOf
31.  layout_constraintTop_creator
32.  layout_constraintTop_toBottomOf
33.  layout_constraintTop_toTopOf
34.  layout_constraintVertical_bias
35.  layout_constraintVertical_chainStyle
36.  layout_constraintVertical_weight
37.  layout_constraintWidth_default
38.  layout_constraintWidth_max
39.  layout_constraintWidth_min
40.  layout_editor_absoluteX
41.  layout_editor_absoluteY
42.  layout_goneMarginBottom
43.  layout_goneMarginEnd
44.  layout_goneMarginLeft
45.  layout_goneMarginRight
46.  layout_goneMarginStart
47.  layout_goneMarginTop
48.  layout_optimizationLevel
49.  layout_marginStart
50.  layout_marginEnd
51.  layout_marginLeft
52.  layout_marginTop
53.  layout_marginRight
54.  layout_marginBottom

不要被数量吓怕了，其实很简单。这 48 个属性一共可以分为 8 组，下面会按类型逐步分解。

### 一、ConstraintLayout 本身使用的属性

*   android_maxHeight
*   android_maxWidth
*   android_minHeight
*   android_minWidth

这四个属性就是 Android 里面常见的控制 View 最大尺寸和最小尺寸的属性，可以设置到 ConstraintLayout 上来控制 ConstraintLayout 的尺寸信息。 这个比较简单就不做介绍了

### 二、Guideline 使用的属性

下面四个属性为应用到 [Guideline](https://developer.android.com/reference/android/support/constraint/Guideline.html) 上面的， Guideline 是用来辅助定位的一个不可见的元素，后面会详细介绍。

*   android_orientation 控制 Guideline 是横向的还是纵向的
*   layout_constraintGuide_begin 控制 Guideline 距离父容器开始的距离
*   layout_constraintGuide_end 控制 Guideline 距离父容器末尾的距离
*   layout_constraintGuide_percent 控制 Guideline 在父容器中的位置为百分比

### 三、相对定位属性

下面 13 个属性是用来控制子 View 相对位置的，这些属性和 RelativeLayout 的布局属性非常类似，用来控制子 View 的某一个属性相对于另外一个 View 或者 父容器的位置。

*   layout_constraintBaseline_toBaselineOf
*   layout_constraintTop_toBottomOf
*   layout_constraintTop_toTopOf
*   layout_constraintBottom_toBottomOf
*   layout_constraintBottom_toTopOf
*   layout_constraintStart_toEndOf
*   layout_constraintStart_toStartOf
*   layout_constraintEnd_toEndOf
*   layout_constraintEnd_toStartOf
*   layout_constraintLeft_toLeftOf
*   layout_constraintLeft_toRightOf
*   layout_constraintRight_toLeftOf
*   layout_constraintRight_toRightOf

上面这些属性命名非常有规律，例如前面的 layout 说明这是一个布局属性，中间的 constraintXXX为对当前 View 上的某个属性做的约束，而最后的 toXXOf 为当前 View 约束的对象以及约束的方位。

如下图：

![image](http://pic.goodev.org/wp-files/2017/02/cl/relative-positioning.png)

是通过如下的布局来实现的：
~~~xml
<Button android:id="@+id/buttonA" ... />
<Button android:id="@+id/buttonB" ...
    app:layout_constraintLeft_toRightOf="@+id/buttonA" />
~~~

所实现的效果是 ButtonB 左边位于 ButtonA 的右边。这样布局管理器就会把 ButtonB 的左边和 ButtonA 的右边对齐， A 的右边和 B 的左边位于同一个垂直位置。是不是非常简单！

注意，所相对的对象可以是临近的其他 子 View 也可以是 ConstraintLayout 自己，如果要把 ButtonB 的左边 和 父容器（ConstraintLayout）的左边对齐，则可以使用下面的方式(注意引用的控件为 parent )：

~~~xml
<Button android:id="@+id/buttonB" ...
    app:layout_constraintLeft_toLeftOf="parent" />
~~~


下图是每个 View 的上下左右以及 baseline 示意图：

![image](http://pic.goodev.org/wp-files/2017/02/cl/relative-positioning-constraints.png)

### 四、Margin （View 的边距）

如果没有 Margin 则两个 View 会紧紧挨着，比如上面的 ButtonB 和 ButtonA 挨着。而很多情况下，设计的效果都要求 View 之间有间隔，这个时候就要使用 Margin 属性了。

下图为 Margin 的说明：

![image](http://pic.goodev.org/wp-files/2017/02/cl/relative-positioning-margin.png)

下面为控制 View margin 的属性：
– layout_marginStart
– layout_marginEnd
– layout_marginLeft
– layout_marginTop
– layout_marginRight
– layout_marginBottom

margin 值只能为大于等于0的数字，这些都是很常见的属性，不做详细介绍了。

而下面 6 个是控制当前 View 所参考的 View 状态为 GONE 的时候的 margin 值：
– layout_goneMarginBottom
– layout_goneMarginEnd
– layout_goneMarginLeft
– layout_goneMarginRight
– layout_goneMarginStart
– layout_goneMarginTop

比如 ButtonB 左边相对于 ButtonA 右边对齐，ButtonA 左边相对于父容器左边对齐。如果 ButtonA 的状态为 GONE（不可见的），则 ButtonB 就相对于父容器左边对齐了。如果有个需求是，当 ButtonA 不可见的时候， ButtonB 和父容器左边需要一个边距 16dp。 这个时候就需要使用上面的 layout_goneMarginLeft 或者 layout_goneMarginStart 属性了，如果设置了这个属性，当 ButtonB 所参考的 ButtonA 可见的时候，这个边距属性不起作用；当 ButtonA 不可见（GONE）的时候，则这个边距就在 ButtonB 上面起作用了。

另外还有一个用途就是方便做 View 动画，可以先设置 ButtonA 为 GONE，同时可以保持 ButtonB 的布局位置不变。

### 五、居中和偏移(bias)

ConstraintLayout 处理看起来冲突的约束比较有意思。例如：

~~~xml

android.support.constraint.ConstraintLayout  ...>

    Button android:id="@+id/button"  ...

        app:layout_constraintLeft_toLeftOf="parent"

        app:layout_constraintRight_toRightOf="parent/>

android.support.constraint.ConstraintLayout/>
~~~

上面约束 `Button` 的左边和父容器左边对齐，右边和父容器右边对齐，除非父容器`ConstraintLayout`和 `Button` 的宽度一样，才能满足这个条件，否则的话是无法满足这个条件的。这样情况下，`ConstraintLayout` 是如何处理的呢？

这种情况下，`ConstraintLayout` 就像使用两个作用力分别从左边和右边来拉住这个 Button，就像 Button 左右一边一个弹簧固定到父容器左右。最终的效果就是 Button 在父容器中水平居中。对于垂直方向上的约束是类似的规则。

居中布局示意图：

![image](http://pic.goodev.org/wp-files/2017/02/cl/centering-positioning.png)

#### Bias

像上面提到的这种对称相反布局约束会把 View 居中对齐，使用 Bias 可以改变两边的权重（类似于 LinearLayout 中的 weight 属性）：

*   layout_constraintHorizontal_bias
*   layout_constraintVertical_bias

Bias 示意图：

![image](http://pic.goodev.org/wp-files/2017/02/cl/centering-positioning-bias.png)

如果没有设置 bias 值，则左右两边的取值为各占 50%，如果把左边的 bias 值修改为 30%（0.3），则左边空白的边距就小一点，而右边空白的距离就大一点，例如下面的代码就可以实现类似上图的效果：

~~~xml
android.support.constraint.ConstraintLayout  ...>

    Button android:id="@+id/button"  ...

        app:layout_constraintHorizontal_bias="0.3"

        app:layout_constraintLeft_toLeftOf="parent"

        app:layout_constraintRight_toRightOf="parent/>

/android.support.constraint.ConstraintLayout>
~~~

### 六、子 View 的尺寸控制

ConstraintLayout 中子 View 的宽度和高度还是通过 android:layout_width 和 android:layout_height 来指定，可以有三种不同的取值：
– 使用确定的尺寸，比如 48dp
– 使用 WRAP_CONTENT ，和其他地方的 WRAP_CONTENT 一样
– 使用 0dp，这个选项等于 “MATCH_CONSTRAINT”，也就是和约束规则指定的宽(高)度一样

例如下图：

![image](http://pic.goodev.org/wp-files/2017/02/cl/dimension-match-constraints.png)
(a) 设置为wrap_content；(b) 设置为 0dp，则 View 的宽度为整个父容器的宽度；(c) 是设置了 margin的情况下的宽度。

注意： MATCH_PARENT 属性无法在 ConstraintLayout 里面的 子 View 上使用。

#### 控制子 View 的宽高比

*   layout_constraintDimensionRatio 控制子View的宽高比

    除了上面三种设置 子 View 的尺寸以外，还可以控制 子 View 的宽高比。如果要使用宽高比则需要至少设置一个尺寸约束为 0dp，然后设置 `layout_constraintDimentionRatio` 属性：

~~~xml

Button android:layout_width="wrap_content"

    android:layout_height="0dp"

    app:layout_constraintDimensionRatio="1:1"  />

 ~~~

上面的代码会设置 Button 的高度和宽度一样。

比率的取值有两种形式：
– float 值，代表宽度/高度 的比率
– “宽度:高度”这种比率值

如果宽度和高度都是 MATCH_CONSTRAINT (0dp) 也可以使用宽高比。这种情况，系统会使用满足所有约束条件和比率的最大尺寸。要根据其中一种尺寸来约束另外一种尺寸，则可以在比率值的前面添加 `W` 或者 `H`来分别约束宽度或者高度。例如，如果一个尺寸被两个目标约束（比如宽度为0dp，在父容器中居中），你通过使用字符 `W` 或者 `H` 来指定那个边被约束。
例如：

~~~xml

Button android:layout_width="0dp"

    android:layout_height="0dp"

    app:layout_constraintDimensionRatio="H,16:9"

    app:layout_constraintBottom_toBottomOf="parent"

    app:layout_constraintTop_toTopOf="parent"/>

 ~~~

上面的 layout_constraintDimensionRatio 取值为 `H,16:9`，前面的 H 表明约束高度，所以结果就是 Button 的宽度和父容器宽度一样，而高度值符合 16:9 的比率。

除了基本的 View 尺寸控制以为，还有如下几个精细控制 View 尺寸的属性（注意：下面这些属性只有宽度或者高度设置为 0dp (MATCH_CONSTRAINT) 的情况下才有效）：

*   layout_constraintWidth_default
*   layout_constraintHeight_default 取值为 spread 或者 wrap，默认值为 spread ，占用所有的符合约束的空间；如果取值为 Wrap ，并且view 的尺寸设置为 wrap_content 且受所设置的约束限制其尺寸，则 这个 view 最终尺寸不会超出约束的范围。
*   layout_constraintHeight_max 取值为具体的尺寸
*   layout_constraintHeight_min 取值为具体的尺寸
*   layout_constraintWidth_max 取值为具体的尺寸
*   layout_constraintWidth_min 取值为具体的尺寸

下图为各个取值的示意图：

![image](http://pic.goodev.org/wp-files/2017/02/cl/match_parent.png)

### 七、链条布局（Chains）

Chains 为同一个方向（水平或者垂直）上的多个子 View 提供一个类似群组的概念。其他的方向则可以单独控制。

#### 创建一个 Chain

多个 View 相互在同一个方向上双向引用就创建了一个 Chain。什么是双向引用呢？ 比如在水平方向上两个 Button A 和 B，如果 A 的右边位于 B 的左边，而 B 的左边位于 A 的右边，则就是一个双向引用。如下图：

![image](http://pic.goodev.org/wp-files/2017/02/cl/chains.png)

> 注意： 在 Android Studio 编辑器中，先把多个 View 单向引用，然后用鼠标扩选多个 View，然后在上面点击右键菜单，选择 “Center Horizontally” 或者 “Center Vertically” 也可以快速的创建 Chain。

#### Chain heads

Chain 的属性由该群组的第一个 View 上的属性所控制（第一个 View 被称之为 Chain head）.

![image](http://pic.goodev.org/wp-files/2017/02/cl/chains-head.png)

水平群组，最左边的 View 为 head， 垂直群组最上面的 View 为 head。

#### Margins in chains

可以为 Chain 中的每个子 View 单独设置 Margin。对于 spread chains， 可用的布局空白空间是扣除 margin 后的空间。下面会详细解释。

#### Chain Style

下面几个属性是控制 Chain Style 的：
– layout_constraintHorizontal_chainStyle
– layout_constraintHorizontal_weight
– layout_constraintVertical_chainStyle
– layout_constraintVertical_weight

chainStyle 是设置到 Chain Head 上的，指定不同的 style 会改变里面所有 View 的布局方式，有如下四种 Style：

*   CHAIN_SPREAD 这个是默认的 Style， 里面的所有 View 会分散开布局
*   Weighted chain，在 CHAIN_SPREAD 模式下，如果有些 View 的尺寸设置为 MATCH_CONSTRAINT（0dp）,则这些 View 尺寸会占据所有剩余可用的空间，和 LinearLayout weight 类似。
*   CHAIN_SPREAD_INSIDE 和 CHAIN_SPREAD 类似，只不过两端的两个 View 和 父容器直接不占用多余空间，多余空间在 子 View 之间分散
*   CHAIN_PACKED 这种模式下，所有的子 View 都 居中聚集在一起，但是可以设置 bias 属性来控制聚集的位置。

下面是几种模式示意图：

![image](http://pic.goodev.org/wp-files/2017/02/cl/chains-styles.png)

如果多个子View尺寸设置为 MATCH_CONSTRAINT（0dp），则这些 View 会平均的占用多余的空间。通过 layout_constraintXXX_weight 属性，可以控制每个 View 所占用的多余空间的比例。例如，对于只有两个 View 的一个水平 Chain，如果每个View 的宽度都设置为 MATCH_CONSTRAINT， 第一个 View 的 weight 为 2；第二个 View 的 weight 为 1，则第一个 View 所占用的空间是 第二个 View 的两倍。

### 八、UI 编辑器所使用的属性

下面几个属性是 UI 编辑器所使用的，用了辅助拖拽布局的，在实际使用过程中，可以不用关心这些属性。

*   layout_optimizationLevel
*   layout_editor_absoluteX
*   layout_editor_absoluteY
*   layout_constraintBaseline_creator
*   layout_constraintTop_creator
*   layout_constraintRight_creator
*   layout_constraintLeft_creator
*   layout_constraintBottom_creator

## Guideline

Guideline 是 ConstraintLayout 中一个特殊的辅助布局的类。相当于一个不可见的 View，使用 ConstraintLayout 可以创建水平或者垂直的参考线，其他的 View 可以相对于这个参考线来布局。

*   垂直 Guideline 的宽度为 0， 高度为 父容器（ConstraintLayout）的高度
*   水平 Guideline 的高度为 0， 宽度为 父容器（ConstraintLayout）的宽度

参考线的位置是可以移动的。

*   layout_constraintGuide_begin 可以指定距离左（或者上）边开始的固定位置
*   layout_constraintGuide_end 可以指定距离右（或者下）边开始的固定位置
*   layout_constraintGuide_percent 可以指定位于布局中所在的百分比，比如距离左边 2% 的位置

下面是一个使用垂直 Guideline 的示例, Button 相对于 guideline 布局：

~~~xml

android.support.constraint.ConstraintLayout

        xmlns:android="http://schemas.android.com/apk/res/android"

        xmlns:app="http://schemas.android.com/apk/res-auto"

        xmlns:tools="http://schemas.android.com/tools"

        android:layout_width="match_parent"

        android:layout_height="match_parent">

    android.support.constraint.Guideline

            android:layout_width="wrap_content"

            android:layout_height="wrap_content"

            android:id="@+id/guideline"

            app:layout_constraintGuide_begin="100dp"

            android:orientation="vertical"/>

    Button

            android:text="Button"

            android:layout_width="wrap_content"

            android:layout_height="wrap_content"

            android:id="@+id/button"

            app:layout_constraintLeft_toLeftOf="@+id/guideline"

            android:layout_marginTop="16dp"

            app:layout_constraintTop_toTopOf="parent"  />

/android.support.constraint.ConstraintLayout>

 ~~~

上面就是 ConstraintLayout 所有布局属性文件了，看完后是不是感觉 ConstraintLayout 也非常简单吧。只不过被 RelativeLayout 复杂那么一点点而已。

## 通过代码来设置 ConstraintLayout 属性

上面只是 XML 布局文件中使用的属性，只能在 XML 布局文件中使用，但是现在针对 UI 做动画的时候，需要通过代码来动态设置 View 的布局属性。下面就来看看如何通过代码来设置这些属性。

### ConstraintSet

[ConstraintSet](https://developer.android.com/reference/android/support/constraint/ConstraintSet.html) 是用来通过代码管理布局属性的集合对象，可以通过这个类来创建各种布局约束，然后把创建好的布局约束应用到一个 ConstraintLayout 上，可以通过如下几种方式来创建 ConstraintSet：

*   手工创建：
    c = new ConstraintSet(); c.connect(….);
*   从 R.layout.* 对象获取
    c.clone(context, R.layout.layout1);
*   从 ConstraintLayout 中获取
    c.clone(clayout);

    然后通过 applyTo 函数来应用到ConstraintLayout 上

~~~xml

mConstraintSet.applyTo(mConstraintLayout);  // set new constraints
~~~

ConstraintSet 支持所有属性设置，每个函数如何使用请参考 API 文档，这里就不再介绍了。
Guideline 也有几个函数可以设置其位置。

本文出自 云在千峰，转载时请注明出处及相应链接。

本文永久链接: http://blog.chengyunfeng.com/?p=1030

---


在[前面一篇文章](http://blog.chengyunfeng.com/?p=1030)中我们介绍了 [ConstraintLayout](https://developer.android.com/reference/android/support/constraint/ConstraintLayout.html) 布局的相关属性。由于 ConstraintLayout 属性众多，如果还是直接在 XML 布局文件中手工编写布局代码则无疑写代码的效率会很低，为了方便大家更快捷的编写 UI 布局代码，Android Studio 中的布局编辑器功能越来越强大，布局编辑器配合 ConstraintLayout 无疑会让你写代码的效率提高很多。 下面来看看布局编辑器有哪些功能。

## 布局编辑器功能介绍

下图为布局编辑器 UI:

![image](http://pic.goodev.org/wp-files/2017/02/cl/cs_editor.jpg)

注意上图是基于 Android Studio 2.2 的截图，在新的 2.3版本中布局有稍微变化，个别图标也不一样。

布局编辑器主要有 5 个功能区域：
1\. Palette 显示了可用的 View 和 Layout，可以直接把这些控件拖动到布局编辑器中
2\. Component Tree 线上当前布局编辑器中的布局层级结构，在这里点击一个 View 在编辑器中就会选中这个 View。
3\. Toolbar 布局编辑器的功能选项，
4\. Design Editor 布局编辑器主窗口，线上当前的布局效果，有两种线上模式，Design 和 Blueprint
5\. Properties 可以修改当前选中的 View 的各种属性

当打开布局 xml 文件的时候，默认情况下会打开 Design 编辑器，如果你想直接修改 XML 代码则可以点击下面的 Text tab 来切换。

在 Text 编辑模式下，点击右边的 Preview 也同样可以查看预览效果：

![image](http://pic.goodev.org/wp-files/2017/02/cl/cs_editor2.png)

### 工具栏介绍

下面是 Design 编辑器的工具栏：

![image](http://pic.goodev.org/wp-files/2017/02/cl/cs_editor_tools.jpg)

工具栏第一行为通用功能，前面三个为Design 和 blueprint 预览模式切换，后面几个分别介绍如下：
1\. Design 和 blueprint 预览模式切换，可以只显示一种或者两种同时显示
2\. 屏幕方向切换
3\. 设备类型和屏幕尺寸切换，可以直接预览当前 UI 在不同屏幕设备上显示的效果
4\. API 版本可以选择预览 UI 布局的系统版本
5\. App 主题可以选择要使用的主题。 注意：这个功能只能使用支持的布局样式，很多主题是不能直接在这里使用的。
6\. 切换语言，还可以点击 Edit Translations 选项直接编辑多语言资源
7\. 创建其他种类的布局，比如当前有个适用于竖屏的布局，你可以通过这个按钮来创建一个新的应用于横屏的布局

工具栏第二行为 ConstraintLayout 特有的工具栏，后面会介绍。

## 把现有布局转换为 ConstraintLayout

这是一个贴心功能，可以直接把现有的其他布局一键转换为 ConstraintLayout：
1\. 在 Android Studio 中打开布局文件，在 Design 编辑模式下
2\. 点击右下角的 Component Tree 窗口，选中需要转换的布局，然后右键点击 “Convert layout to ConstraintLayout” 选项即可

## 设置 ConstraintLayout 约束

当把一个 View 拖动到布局编辑器中的时候，点击选中这个 View 的状态如下：

![image](http://pic.goodev.org/wp-files/2017/02/cl/69825db037949875.png)

上面显示了控制 View 各种约束的控制手柄：

*   Resize Handle ，在 View 四个角的四个方块手柄为控制 View 尺寸大小的手柄，可以拖动调整 View 尺寸
*   Side Constraint Handle， 在 View 四边上的四个圆形手柄为控制 View 位置的，拖动一个手柄可以指定该边所相对布局的位置。
*   Baseline Constraint Handle 是在 View 中间的一个长条装的圆角长方形，这个是指定 View baseline 相对布局位置的

例如下图，把 Button2 的左边圆形手柄拖动指向 Button1 的右边圆形手柄，同时设置直接的间距为 56dp，则表示 Button2 的左边位于 Button1 的右边，Button2 左边的 margin 为 56dp：

![image](http://pic.goodev.org/wp-files/2017/02/cl/9bc79b0252ae3bf6.png)

## 修改 View 属性

由于很多属性在布局编辑器主设计窗口中无法直接修改，如果需要修改一个 View 的属性，可以选中这个 View，然后在 Properties 窗口中修改，

![image](http://pic.goodev.org/wp-files/2017/02/cl/layout-editor-properties_2-2_2x.png)

点击 Properties 工具栏最左边的两个箭头的图标，可以显示所有可以修改的属性。

## ConstraintLayout 布局工具栏

当编辑 ConstraintLayout 的时候，在 编辑器工具栏下方还有一些 ConstraintLayout 特有的工具选项：

![image](http://pic.goodev.org/wp-files/2017/02/cl/layout-editor-margin-callout_2-2_2x.png)

这些选项从左到右分别为：
– 是否显示所有 View 的约束，如果选中则当鼠标放在布局编辑器的时候，可以看到所有 View 之间的约束
– 是否打开 autoconnect 功能，如果打开这个功能，则拖动添加新的 View 的可以，布局编辑器可以自动的在控件所停留的位置添加对应的约束
– 删除所有 View 上的约束
– 推理没添加的约束，点击这个按钮可以为哪些没有添加约束的 view 自动更加当前布局的位置添加一个合适的约束规则
– 修改默认 view 之间的 margin
– 设置 View 的尺寸缩放类型
– 设置 View 的对齐方式
– 最后一个为添加 垂直和水平 Guideline 的功能

合理使用这些功能，可以让你更高效率的编写 UI 代码。

本文出自 云在千峰，转载时请注明出处及相应链接。

本文永久链接: http://blog.chengyunfeng.com/?p=1031

