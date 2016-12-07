###EditText基础
[TOC]

###**1.常用设置**
~~~
android:focusable=false// 无法输入内容
android:singleLine=true//设置单行输入，一旦设置为true，则文字不会自动换行。
android:textColor =#ff8c00;//字体颜色 
android:textStyle=normal;//正常字体 bold; //粗体 i talic斜体 bold|italic粗体+斜体
android:capitalize = characters;//输入内容为大写字母（注意：可以用inputType的某个属性代替）
android:textColorHighlight=#cccccc;//被选中文字的底色，默认为蓝色
android:textColorHint=#ffff00//设置提示信息文字的颜色，默认为灰色
android:textScaleX=1.5//控制字与字之间的间距
android:background=@null;//背景，这里没有，指透明
android:layout_gravity=center_vertical;//设置控件显示的位置：默认top，这里居中显示，还有bottom
android:gray=top; //多行中指针在第一行第一位置
android:capitalize //首字母大写
android：phoneNumber //输入电话号码
android:autoLink=”all” //设置文本超链接样式当点击网址时，跳向该网址
android:cursorVisible  //设定光标为显示/隐藏，默认显示。
android:hint="请输入数字！"//设置显示默认的提示信息
android:focusable="false"// 无法输入内容
android:singleLine="true"//设置单行输入，一旦设置为true，则文字不会自动换行。
android:textColor = "#ff8c00"//字体颜色
android:textStyle="normal"//正常字体
android:capitalize = "characters"//输入内容为大写字母（注意：可以用inputType的某个属性代替）
android:textColorHighlight="#cccccc"//被选中文字的底色，默认为蓝色
android:textColorHint="#ffff00"//设置提示信息文字的颜色，默认为灰色
android:textScaleX="1.5"//控制字与字之间的间距
android:background="@null"//背景，这里没有，指透明
android:layout_gravity="center_vertical"//设置控件显示的位置：默认top，这里居中显示，还有bottom
android:gray="top" //多行中指针在第一行第一位置
android:capitalize //首字母大写
android：phoneNumber //输入电话号码
android:autoLink=”all” //设置文本超链接样式当点击网址时，跳向该网址
android:cursorVisible  //设定光标为显示/隐藏，默认显示。
~~~
###**2.inputType常用设置**
~~~
android:inputType=textCapCharacters 字母大写
android:inputType=textCapWords 首字母大写
android:inputType=textCapSentences 仅第一个字母大写
android:inputType=textMultiLine 多行输入
android:inputType=textPassword 密码
android:inputType=number 数字
android:inputType=numberSigned 带符号数字格式
android:inputType=numberDecimal 带小数点的浮点格式
android:inputType=datetime 时间日期
android:inputType=date 日期键盘
android:inputType=time 时间键盘
android:inputType="textCapCharacters" 字母大写
android:inputType="textCapWords" 首字母大写
android:inputType="textCapSentences" 仅第一个字母大写
android:inputType="textMultiLine" 多行输入
android:inputType="textPassword" 密码
android:inputType="number" 数字
android:inputType="numberSigned" 带符号数字格式
android:inputType="numberDecimal" 带小数点的浮点格式
android:inputType="datetime" 时间日期
android:inputType="date" 日期键盘
android:inputType="time" 时间键盘
~~~

####**3.常用的数字输入设置：**
~~~
android:numeric=integer   //只可以输入正整数
android:numeric=decimal  //可以输入小数
android:numeric=signed   //表示可以输入整数（正整数或者负整数）
android:inputType=numberDecimal //可以输入小数，正小数（即只可以加一个小数点的正数）
android:maxLength=11   //最多可以输入11位数字
android:singleLine=true //单行输入
android:password=true   //密码输入框，可以使得输入的内容在1秒内变成*字样
android:inputType=number //设置只能输入数字（相当于是输入正整数），并且默认的弹出框是数字弹出框
android:numeric="integer"  //只可以输入正整数
android:numeric="decimal"  //可以输入小数
android:numeric="signed"   //表示可以输入整数（正整数或者负整数）
android:inputType="numberDecimal" //可以输入小数，正小数（即只可以加一个小数点的正数）
android:maxLength="11"   //最多可以输入11位数字
android:singleLine="true" //单行输入
android:password="true"   //密码输入框，可以使得输入的内容在1秒内变成*字样
android:inputType="number" //设置只能输入数字（相当于是输入正整数），并且默认的弹出框是数字弹出框
~~~
 4）EditText中，android:maxLines和android:minLines的区别：
例如：//开始EditText里面没内容时，默认控件大小为1行
* **android:minLines="3"**  
 使用minLines的EditText是至它至少显示3行内容（包括内容为空时）当输入的内容超过3行后，它形状的大小根据输入内容的多少而改变。
 
*  **android:maxLines="3"**  
使用maxLines的EditText最大行数为3行，当输入的内容超过3行后，它形状的大小不会根据输入内容的多少而改变，反正它显示的内容就是3行 。

#### **3.监听器**
#### TextWatcher
~~~
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        EditText  editText = (EditText) findViewById(R.id.et);
       
        //监听EditText框中的变化
        mEditText.addTextChangedListener(new TextWatcher() {
            //文本变化之前
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {}

            // 文本变化中
            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {  }

            //文本变化之后
            @Override
            public void afterTextChanged(Editable s) { }
        });
    }
}
~~~

#### **4.如何设置输入框只能输入指定的字符**
android 1.5以后添加了软件虚拟键盘的功能，所以在输入提示中将会有对应的软键盘模式。android中inputType属性在EditText输入值时启动的虚拟键盘的风格有着重要的作用。这也大大的方便的操作。有时需要虚拟键盘只为字符或只为数字。所以inputType尤为重要。
#### **android 中inputType详解：**
~~~
android:digits="0123456789xyzXYZ#*?"  //引号里面输入你想设置的输入内容
android:inputType="phone"文本类型，多为大写、小写和数字符号。
android:inputType="none"
android:inputType="text"
android:inputType="textCapCharacters" 字母大写
android:inputType="textCapWords" 首字母大写
android:inputType="textCapSentences" 仅第一个字母大写
android:inputType="textAutoCorrect" 自动完成
android:inputType="textAutoComplete" 自动完成
android:inputType="textMultiLine" 多行输入
android:inputType="textImeMultiLine" 输入法多行（如果支持）
android:inputType="textNoSuggestions" 不提示
android:inputType="textUri" 网址
android:inputType="textEmailAddress" 电子邮件地址
android:inputType="textEmailSubject" 邮件主题
android:inputType="textShortMessage" 短讯
android:inputType="textLongMessage" 长信息
android:inputType="textPersonName" 人名
android:inputType="textPostalAddress" 地址
android:inputType="textPassword" 密码
android:inputType="textVisiblePassword" 可见密码
android:inputType="textWebEditText" 作为网页表单的文本
android:inputType="textFilter" 文本筛选过滤
android:inputType="textPhonetic" 拼音输入 //数值类型
android:inputType="number" 数字
android:inputType="numberSigned" 带符号数字格式
android:inputType="numberDecimal" 带小数点的浮点格式
android:inputType="phone" 拨号键盘
android:inputType="datetime" 时间日期
android:inputType="date" 日期键盘
android:inputType="time" 时间键盘
~~~
### 4.其他
#### **1.字母大小写转换**
~~~java
/**
 * 字母小写字母自动转大写
 * 使用方法：
 * editext.setTransformationMethod(new AllCapTransformationMethod ());
 */
public class AllCapTransformationMethod extends ReplacementTransformationMethod {
    @Override
    protected char[] getOriginal() {
        char[] aa = { 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z' };
        return aa;
    }

    @Override
    protected char[] getReplacement() {
        char[] cc = { 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z' };
        return cc;
    }
}
~~~
**END**