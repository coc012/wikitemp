版权信息
原文链接：[Android View的事件分发机制和滑动冲突解决](http://blog.csdn.net/qian520ao/article/details/77429593)

---
### 初探View事件

前言View的事件分发和滑动冲突处理是老生常谈的知识了，因为最近撸了一个[仿QQ侧滑删除](https://github.com/qdxxxx/SwipeMenuContainer "optional title")，所以对该View事件有了更深入的总结。老铁们是时候走一波[star](https://github.com/qdxxxx/SwipeMenuContainer "optional title")了。 
我们常说的View事件是指： 从手指亲密接触屏幕的那一刻到手指离开屏幕的这个过程，该事件序列以down事件为起点，move事件为过程，up事件为终点。 
一次down-move-up这一个事件过程我们称为一个事件序列。所以我们今天研究的对象就是MotionEvent。

### 事件分发

#### 理论知识

*   public boolean dispatchTouchEvent(MotionEvent ev) 
    用来分发事件，即事件序列的大门，如果事件传递到当前View的`onTouchEvent`或者是子View的`dispatchTouchEvent`，即该方法被调用了。 
    return true: 表示消耗了当前事件，有可能是当前View的`onTouchEvent`或者是子View的`dispatchTouchEvent`消费了，事件终止，不再传递。 
    return false: 调用父ViewGroup或则Activity的`onTouchEvent`。 （不再往下传）。①另外如果不消耗ACTION_DOWN事件，那么down,move,up事件都与该View无关，交由父类处理(父类的`onTouchEvent`方法) 
    return super.dispatherTouchEvent: 则继续往下(子View)传递，或者是调用当前View的onTouchEvent方法;

*   public boolean onInterceptTouchEvent(MotionEvent ev) 
    在`dispatchTouchEvent`内部调用，顾名思义就是判断是否拦截某个事件。(注：ViewGroup才有的方法，View因为没有子View了，所以不需要也没有该方法) 
    return true: ViewGroup将该事件拦截，交给自己的`onTouchEvent`处理。②而且这一个事件序列（当前和其它事件）都只能由该ViewGroup处理，并且不会再调用该`onInterceptTouchEvent`方法去询问是否拦截。 
    return false: 继续传递给子元素的`dispatchTouchEvent`处理。 
    return super.dispatherTouchEvent: 事件默认不会被拦截。

*   public boolean onTouchEvent(MotionEvent ev) 
    在`dispatchTouchEvent`内部调用 
    return true: 事件消费，当前事件终止。 
    return false: 交给父View的`onTouchEvent`。 
    return super.dispatherTouchEvent: 默认处理事件的逻辑和返回 false 时相同。

其实上面的关系可以用以下代码简单描述。

~~~
public boolean dispatchTouchEvent(MotionEvent ev){
    boolean consume = false;//是否消费事件
    if(onInterceptTouchEvent(ev)){//是否拦截事件
        consume = onTouchEvent(ev);//拦截了，交给自己的View处理
    }else{
        consume = child.dispatchTouchEvent(ev);//不拦截，就交给子View处理
    }

    return consume;//true：消费事件，终止。false:交给父onTouchEvent处理。并不再往下传递当前事件。
}
~~~

有图有真相

![View事件分发](http://img.blog.csdn.net/20170820165410164?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcWlhbjUyMGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

有点类似[责任链设计模式](http://blog.csdn.net/qian520ao/article/details/73558275 "optional title")

#### 实战讲解

##### 验证View的事件分发

*   创建CustomViewGroup继承FrameLayout
*   创建CustomView继承View

xml

~~~

<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="qdx.viewtouchevent.MainActivity">

//最外层为activity（白色背景）
    <qdx.viewtouchevent.CustomViewGroup
        android:layout_width="300dp"
        android:layout_height="400dp"
        android:layout_gravity="right"
        android:background="#84bf96">
//CustomViewGroup（绿色背景）包含CustomView（黄色背景）
        <qdx.viewtouchevent.CustomView
            android:layout_width="150dp"
            android:layout_height="300dp"
            android:layout_gravity="right"
            android:background="#f2eada" />

    </qdx.viewtouchevent.CustomViewGroup>

</FrameLayout>

~~~

![无事件处理者](http://img.blog.csdn.net/20170820190556008?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcWlhbjUyMGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

如上图所示，down事件由activity->ViewGroup->View，因为View并没有处理down事件，所以事件消费情况为false，并且最后由View->ViewGroup->activity传递。 

##### 验证不消耗ACTION_DOWN事件

我们再来验证①另外如果不消耗ACTION_DOWN事件，那么down,move,up事件系列都与该View无关，交由父类处理(父类的`onTouchEvent`方法) 
根据上面文字描述，因为我们的`CustomViewGroup`和`CustomView`都没有去处理任何事件，即当前序列的所有事件都return false，所以我们也无法接收/处理其他事件(move,up)

![ 不消耗ACTION_DOWN事件](http://img.blog.csdn.net/20170820192518176?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcWlhbjUyMGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

我们再将customView设置为可点击状态，即消费touch事件。`setClickable(true);`

![子View消费touch事件](http://img.blog.csdn.net/20170820214522582?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcWlhbjUyMGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##### 验证 ViewGroup事件拦截

viewGroup将事件拦截后，②而且这一个事件序列（当前和其它事件）都只能由该ViewGroup处理，并且不会再调用该`onInterceptTouchEvent`方法去询问是否拦截。

![View事件拦截](http://img.blog.csdn.net/20170820232415501?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcWlhbjUyMGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

通过上面的几个验证，我们越来越接近真相，用通俗的话来解释就是：

> 老板发现BUG解决，一开始是由上级往下级问话。（[类似责任链设计模式](http://blog.csdn.net/qian520ao/article/details/73558275 "optional title")）
> 
> 例如突然间出现了BUG，老板问小组A有没有空处理一下BUG(即分发ACTION_DOWN)，小组A说没时间(return false)，那么老板就不会把这个序列的BUG（ACTION_MOVE和ACTION_UP）交给小组A。如果再次出现BUG，老板还会再次询问小组A。①
> 
> 如果你举手揽了这个BUG（即拦截），那么这一事件的BUG都交由你解决，并且相同序列的BUG老板不会问话，直接找你处理。②

#### 源码分析ViewGroup

源码分析这一块主要还是基于《Android开发艺术探索》这本书的引导和理解做出的总结。PS：这本书性价比很高，涵盖知识面广。

##### Activity的事件分发

Activity的事件分发还关系到View的绘制和加载机制，等待下一篇来更详细认识这个知识点。

~~~
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }

        //最终获取到顶级View(ViewGroup)分发事件
        //（getWindow().getDecorView().findViewById(android.R.id.Content)）.getChildAt(0)
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }

        //如果所有的View都没有处理事件，则由Activity亲自出马
        return onTouchEvent(ev);
    }
~~~

##### ViewGroup的事件拦截

~~~
            public boolean dispatchTouchEvent(MotionEvent ev) {
            ......

            final int action = ev.getAction();
            final int actionMasked = action & MotionEvent.ACTION_MASK;

            if (actionMasked == MotionEvent.ACTION_DOWN) {
                cancelAndClearTouchTargets(ev);

                //清除FLAG_DISALLOW_INTERCEPT，并且设置mFirstTouchTarget为null
                resetTouchState(){
                    if(mFirstTouchTarget!=null){mFirstTouchTarget==null;}
                    mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;
                    ......
                };
            }
            final boolean intercepted;//ViewGroup是否拦截事件

            //mFirstTouchTarget是ViewGroup中处理事件(return true)的子View
            //如果没有子View处理则mFirstTouchTarget=null,ViewGroup自己处理
            if (actionMasked == MotionEvent.ACTION_DOWN|| mFirstTouchTarget != null) {
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    intercepted = onInterceptTouchEvent(ev);//onInterceptTouchEvent
                    ev.setAction(action);
                } else {
                    intercepted = false;

                    //如果子类设置requestDisallowInterceptTouchEvent（true）
                    //ViewGroup将无法拦截MotionEvent.ACTION_DOWN以外的事件
                }
            } else {
                intercepted = true;

                //actionMasked != MotionEvent.ACTION_DOWN并且没有子View处理事件，则将事件拦截
                //并且不会再调用onInterceptTouchEvent询问是否拦截
            }

            ......
            ......
}
~~~

我们将上面的结论再次写下来，方便对照。 
①另外如果不消耗ACTION_DOWN事件，那么down,move,up事件都与该View无关，交由父类处理(父类的`onTouchEvent`方法)（dispatchTouchEvent） 
②而且这一个事件序列（当前和其它事件）都只能由该ViewGroup处理，并且不会再调用该`onInterceptTouchEvent`方法去询问是否拦截。（onInterceptTouchEvent return true）

*   首先我们分析上面第21行代码： ViewGroup在两种情况下会拦截事件（ACTION_DOWN || mFirstTouchTarget != null）所以反过来也就是说 I : 当ACTION_MOVE和ACTION_UP事件到来时，如果没有子元素处理事件（mFirstTouchTarget==null），则ViewGroup的onInterceptTouchEvent不会再被调用，而且同一序列中的其它事件都会默认交给它处理（第34行 intercepted=true）；与上面所说的①②呼应。
*   紧接着22行： ViewGroup`disallowIntercept`（不拦截）的判定是`FLAG_DISALLOW_INTERCEPT`标记位，这个标记是通过子View`requestDisallowInterceptTouchEvent`方法设置的。所以我们可以得出这么一个结论II : 当子View处理了ACTION_DOWN事件(mFirstTouchTarget =该子View)，而且设置了FLAG_DISALLOW_INTERCEPT标记位，那么ViewGroup将无法拦截除了ACTION_DOWN以外的其它事件。（在11行代码ACTION_DOWN时清除了FLAG_DISALLOW_INTERCEPT标记位，所以ViewGroup无论如何都可以选择是否拦截处理ACTION_DOWN）

上面变着花样的又一次验证了①②个知识点，不得不说read the fuck source code让我们可以找到一个处理滑动冲突的方法：子View处理DOWN事件并且设置`FLAG_DISALLOW_INTERCEPT`标记位，就可以不让ViewGroup拦截DOWN以外的事件。

##### ViewGroup的事件分发

~~~
        public boolean dispatchTouchEvent(MotionEvent ev) {
        final View[] children = mChildren;

        for (int i = childrenCount - 1; i >= 0; i--) {
            final int childIndex = getAndVerifyPreorderedIndex(childrenCount, i, customOrder);
            final View child = getAndVerifyPreorderedView(preorderedList, children, childIndex);

            ......

            if (!canViewReceivePointerEvents(child) || !isTransformedTouchPointInView(x, y, child, null)) 
            {
                ev.setTargetAccessibilityFocus(false);
                //如果子View没有播放动画，而且点击事件的坐标在子View的区域内，继续下面的判断
                continue;
            }
            //判断是否有子View处理了事件
            newTouchTarget = getTouchTarget(child);

            if (newTouchTarget != null) {
                //如果已经有子View处理了事件，即mFirstTouchTarget!=null，终止循环。
                newTouchTarget.pointerIdBits |= idBitsToAssign;
                break;
            }

            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                //点击dispatchTransformedTouchEvent代码发现其执行方法实际为
                //return child.dispatchTouchEvent(event); （因为child!=null）
                //所以如果有子View处理了事件，我们就进行下一步：赋值

                ......

                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                //addTouchTarget方法里完成了对mFirstTouchTarget的赋值
                alreadyDispatchedToNewTouchTarget = true;

                break;
            }
        }
    }

    private TouchTarget addTouchTarget(@NonNull View child, int pointerIdBits) {
        final TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
        target.next = mFirstTouchTarget;
        mFirstTouchTarget = target;
        return target;
    }

    private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
            View child, int desiredPointerIdBits) {
            ......

            if (child == null) {
            //如果没有子View处理事件，就自己处理
                handled = super.dispatchTouchEvent(event);
            } else {
           //有子View，调用子View的dispatchTouchEvent方法
                handled = child.dispatchTouchEvent(event);

            ......

            return handled;
    }

~~~

上面为ViewGroup对事件的分发，主要有2点

1.  如果有子View，则调用子View的dispatchTouchEvent方法判断是否处理了事件，如果处理了便赋值mFirstTouchTarget，赋值成功则跳出循环。
2.  ViewGroup的事件分发最终还是调用View的`dispatchTouchEvent`方法，具体如上代码所述。

至此View的事件分发机制已经演练完毕，如果事件分发机制理解深入的话，那么处理滑动冲突便是手到擒来了。

### View的滑动冲突

关于View的滑动冲突我们就开门见山吧，因为上述的事件分发已经有足够的理论知识了，我们可以单刀赴会了。

![这里写图片描述](http://img.blog.csdn.net/20170822233424600?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcWlhbjUyMGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![View的滑动冲突](http://img.blog.csdn.net/20170822232321861?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcWlhbjUyMGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

针对上图，这个是比较普遍的滑动冲突事件，我们先拿它来开刀。

好记性不如烂笔头，我们再次把结论搬到战场上 
①另外如果不消耗ACTION_DOWN事件，那么down,move,up事件都与该View无关，交由父类处理(父类的`onTouchEvent`方法)（dispatchTouchEvent） 
②而且这一个事件序列（当前和其它事件）都只能由该ViewGroup处理，并且不会再调用该`onInterceptTouchEvent`方法去询问是否拦截。（onInterceptTouchEvent return true）

I : 当ACTION_MOVE和ACTION_UP事件到来时，如果没有子元素处理事件（mFirstTouchTarget==null），则ViewGroup的onInterceptTouchEvent不会再被调用，而且同一序列中的其它事件都会默认交给它处理（第34行 intercepted=true）；

#### 外部拦截

外部拦截顾名思义就是由父ViewGroup对事件拦截处理（所以重写`onInterceptTouchEvent`方法即可），子View只能眼巴巴的处理父View“吃剩”的事件。主要有以下几点。

*   父类不能拦截ACTION_DOWN，也就是说必须返回false，根据上述①②和 I 可得。
*   父类在ACTION_MOVE的时候根据需求，判断是否拦截。
*   ACTION_UP事件建议返回false或者`super.onInterceptTouchEvent`，因为如果已经拦截的话，那么并不会调用`onInterceptTouchEvent`方法再次询问。如果不拦截，而且返回true，子View可能就无法触发onClick等相关事件。

ViewGroup ： 需要重写`onInterceptTouchEvent`，判断是否拦截即可。 
但是有一种情况：用户正在水平滑动（事件已拦截给ViewGroup），但是水平滑动停止前用户再进行竖直滑动，下面代码我用`isSolve`进行简单的处理。

~~~
    private boolean isIntercept;
    private boolean isSolve;//是否完成了拦截判断，如果决定拦截，那么同系列事件就不能设置为不拦截

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mPointGapF.x = ev.getX();
                mPointGapF.y = ev.getY();
                return false;//down的时候拦截后，就只能交给自己处理了

            case MotionEvent.ACTION_MOVE:
                if (!isSolve) {//是否已经决定拦截/不拦截？
                    isIntercept = (Math.abs(ev.getX() - mPointGapF.x) > Math.abs(ev.getY() - mPointGapF.y)*2);//如果是左右滑动，且水平角度小于30°，就拦截
                    isSolve = true;
                }
                return isIntercept;//如果是左右滑动，就拦截
        }
        return super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_MOVE:
                scrollBy((int) (mPointGapF.x - ev.getX()), 0);

                mPointGapF.x = ev.getX();
                mPointGapF.y = ev.getY();
                break;
        }
        return super.onTouchEvent(ev);
    }
~~~

子View ： 和子View没有多大关系，只需要处理自身的移动操作即可。

~~~
public boolean onTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mPointGapF.x = ev.getX();
                mPointGapF.y = ev.getY();
                break;
            case MotionEvent.ACTION_MOVE:
                scrollBy(0, (int) (mPointGapF.y - ev.getY()));
                mPointGapF.x = ev.getX();
                mPointGapF.y = ev.getY();
                break;
        }
        return true;
    }
~~~

![外部拦截冲突](http://img.blog.csdn.net/20170823135102485?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcWlhbjUyMGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![外部拦截冲突](http://img.blog.csdn.net/20170823134034862?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcWlhbjUyMGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 内部拦截

II : 当子View处理了ACTION_DOWN事件(mFirstTouchTarget =该子View)，而且设置了FLAG_DISALLOW_INTERCEPT标记位，那么ViewGroup将无法拦截除了ACTION_DOWN以外的其它事件。

ViewGroup ： 只需在`onInterceptTouchEvent`MotionEvent.ACTION_DOWN时候不拦截，其他时候都需要拦截，否则父类的`onTouchEvent`就不能处理任何事件了。

~~~
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                return false;//down的时候拦截后，就只能交给自己处理了
        }
        return true;//如果不拦截，父类的onTouchEvent方法就无事件可以处理。
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_MOVE:
                scrollBy((int) (mPointGapF.x - ev.getX()), 0);

                mPointGapF.x = ev.getX();
                mPointGapF.y = ev.getY();
                break;
        }
        return super.onTouchEvent(ev);
    }
~~~

子View ： 需要在ACTION_DOWN事件设置getParent().requestDisallowInterceptTouchEvent(true)，并且在ACTION_MOVE的时候通过判断是否禁止父类的拦截。

~~~
    private boolean isSolve;
    private boolean isIntercept;

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                isIntercept = false;
                isSolve = false;
                mPointGapF.x = ev.getX();
                mPointGapF.y = ev.getY();
                getParent().requestDisallowInterceptTouchEvent(true);
                break;

            case MotionEvent.ACTION_MOVE:
                if (!isSolve) {
                    isSolve = true;
                    isIntercept = (Math.abs(ev.getX() - mPointGapF.x) < Math.abs(ev.getY() - mPointGapF.y) * 2);
                    getParent().requestDisallowInterceptTouchEvent(isIntercept);
                }
                break;
        }
        return super.dispatchTouchEvent(ev);
    }
~~~

最终的效果图和外部拦截的效果一致，这里就不再次贴出来了。 

### 总结

通过理论和实战，更清晰的了解了事件的分发机制，从而这些理论知识使得我们更有效的处理滑动冲突事件，所以以后只要再遇见滑动冲突事件，再次巩固View的事件分发，万变不离其宗，定能手到擒来解决这一问题！