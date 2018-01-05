文档当前状态：**alpha0.1**
* [x] 选题收集：2018/01/05
* [ ] 初稿整理：
* [ ] 补充校对：
* [ ] 入库存档：
---


#### **adb shell相关**

* **结束指定包名的进程**
~~~
adb shell am force-stop <PACKAGE>

//示例：已知包名：com.android.browser
adb shell am force-stop com.android.browser
~~~

* **使用adb shell查看电池信息**
如果想获取有关电池的一些信息，可以直接用adb命令来获取，方便自动化直接调用。

~~~
//如下命令，通过cat可以获取全部Battery信息：
adb shell cat/sys/class/power_supply/battery/uevent

//获取电池容量：
adb shell cat/sys/class/power_supply/battery/charge_full_design

//获取当前电量百分比：
adb shell cat/sys/class/power_supply/battery/capacity

//获取当前电流（说明：充电时电流可能为负值。）：
adb shell cat/sys/class/power_supply/battery/current_now


~~~