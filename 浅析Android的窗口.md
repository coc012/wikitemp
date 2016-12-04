![](http://img1.tuicool.com/UBRZbij.jpg!web)

**一、窗口的概念**

在开发过程中，我们经常会遇到，各种跟窗口相关的类，或者方法。但是，在 Android 的框架设计中，到底什么是窗口？窗口跟 Android Framework 中的 Window 类又是什么关系？以手Q 的主界面为例，如下图所示，上面的状态栏是一个窗口，手Q 的主界面自然是一个窗口，而弹出的 PopupWindow 也是一个窗口，我们经常使用的 Toast 也是一个窗口。像 Dialog，ContextMenu，以及 OptionMenu 等等这些都是窗口。这些窗口跟 Window 类的关系是什么，或者窗口跟 Window 类描述的是同一个概念吗？

![](http://img2.tuicool.com/QJj6rmu.png!web)

其实窗口的概念，从不同的角度来看，其含义是不一样的。我们知道，WindowManagerService（后面简称 WmS）管理所有的窗口。但是对于WmS来讲，一个窗口其实就是一个 View 类，而不是 Window 类。WmS 负责管理这些 View 的 Z-order，显示区域，以及把消息派发到对应的 View 中。View 本身并不能直接从 WmS 中接收消息，而是通过实现了 IWindow 接口的 ViewRootImpl.W 类来实现，以下是这些类的关系：

![](http://img1.tuicool.com/3EF732E.png!web)

所以这里窗口分为两层概念：

（1）WmS 眼中的，窗口是可以显示用来显示的 View。对于 WmS 而言，所谓的窗口就是一个通过 WindowManagerGlobal.addView()添加的 View 罢了；

（2）Window 类是一个针对窗口交互的抽象，也就是对于 WmS 来讲所有的用户消息是直接交给 View/ViewGroup 来处理的。而 Window 类把一些交互从 View/ViewGroup 中抽离出来，定义了一些窗口的行为，例如菜单，以及处理系统按钮，如“Home”，“Back”等等。由此可见，Window 描述的窗口只是在通用窗口的基础上，再抽象了一层，把符合某种规范的窗口统一了一下。Window 所描述的窗口，应该是通用窗口的一个子集。例如 PopupWindow 是一个窗口，但是分析其源码可以知道，该类并没有创建任何 Window 对象。而 Dialog 则是通过 PolicyManager.makeNewWindow(mContext) 创建了一个 Window 对象来管理窗口。当一个 Dialog 显示时，我们可以通过按 back 把它 dismiss 了，但是 PopupWindow 则不行，需要自己去处理。

**二、窗口类型**

添加一个窗口是通过 WindowManagerGlobal.addView()来完成的，分析 addView 方法的参数，有三个参数是必不可少的，view，params，以及 display。而 display 一般直接取 WindowMnagerImpl 中的 mDisplay，表示要输出的显示设备。view 自然表示要显示的 View，而 params 是 WindowManager.LayoutParams，用来描述这个 view 的些窗口属性，其中一个重要的参数 type，用来描述窗口的类型。

[Java] *纯文本查看* *复制代码*

~~~
public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow) {
        if (view == null) {
            throw new IllegalArgumentException("view must not be null");
        }
        if (display == null) {
            throw new IllegalArgumentException("display must not be null");
        }
        if (!(params instanceof WindowManager.LayoutParams)) {
            throw new IllegalArgumentException("Params must be WindowManager.LayoutParams");
        }
         .....
      }
~~~

分析 WindowManager 对于 type 可赋值类型的描述可知，Framework 中定义了三种类型的窗口：

（1）应用窗口

Activity 对应的窗口就是应用窗口， 所有 Activity 默认的窗口类型是 TYPE _BASE _APPLICATION。WindowManager 的 LayoutParams 的默认构建方法的实现，可以看到默认类型是 TYPE _ APPLICATION。 Dialog 的窗口类型是 TYPE _ APPLICATION，而很多 Dialog 的子类，修改了窗口类似，如 ContextMenu，本质是用 Dialog 来实现的，但是在添加窗口前，修改了 type 类型，赋值为 TYPE _ APPLICATION _ ATTACHED _ DIALOG。从这个我们可以看到，WmS 并没有把应用窗口与子窗口区分得那么清楚。

[Java] *纯文本查看* *复制代码*

~~~
public LayoutParams() {
        super(LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT);
        type = TYPE_APPLICATION;
        format = PixelFormat.OPAQUE;
    }
~~~

![](http://img2.tuicool.com/nAfmueB.png!web)

（2）子窗口子窗口是指该窗口必须要有一个父窗口，父窗口可以是一个应用类型窗口，也可以是任何其他类型的窗口。例如前面手Q 界面中，点击右上角的按钮显示一个 PopupWindow，它就是一个子窗口，其类型一般 TYPE _ APPLICATION _ PANEL。既然称为子窗口，其与父窗口的关系是比较容易理解的。B 是 A 的子窗口，当 A 不可见时，B 也会不可见的。如果A不可见时添加B，B 也是不可见的，直到 A 可见为止，B 跟随一起可见。

![](http://img2.tuicool.com/qY36buA.png!web)

（3）系统窗口

系统窗口跟应用窗口不同，不需要对应 Activity。跟子窗口不同，不需要有父窗口。一般来讲，系统窗口应该由系统来创建的，例如发生异常，ANR时的提示框，又如系统状态栏，屏保等。但是，Framework 还是定义了一些，可以被应用所创建的系统窗口，如 TYPE_ TOAST，TYPE _INPUT _ METHOD，TYPE _WALLPAPTER 等等。

![](http://img2.tuicool.com/RBRfeia.png!web)

token 的含义

相信大家对于 token 这个词并不陌生，在开发过程中经常遇到，例如 Bad Token 的异常。到底在 Android 框架中，token 代表什么？分析源码，我们发现，大多数 token 的对象，都表示一个 IBinder 对象。提到 IBinder，大家一点也不陌生，就是 Android 的 IPC 通信机制。在创建窗口过程中，涉及到的 IPC 通信，无非包含两方面，一个是 WmS 用来跟应用所在的进程进行通信的 ViewRootImpl.W 类的对象，另一个是指向一个 ActivityRecord 的对象，自然应该是WmS用来跟 AmS 进行通信的了。我们梳理了一下，token 以下几处的定义，分别来讲讲这里的 token 代表什么。

![](http://img1.tuicool.com/ENZZni.png!web)

分析一下 View 的 AttachInfo 的赋值。ViewRootImpl 在构建方法里，会初始化一个 AttachInfo 实例，把它的 Session，以及 W类对象赋值给 AttachInfo。分析可以看到，AttachInfo 中的 mWindowToken，与mWindow 都是指向 ViewRootImpl 中的 mWindow(W类实例)。当一个 View attach 到窗口后，ViewRootImpl会执行performTraversals，如果发现是首次调用会，会把自己的 mAttachInfo 传递给根 View（通过dispatchAttachedToWindow），告诉 View 树现在已经 attch to Window 了，马上可以显示了。根 View（一般是 ViewGroup）会把这个信息，遍历地传递给 View 树中的每一个子 View，这样每个 View 的 mAttachInfo 都被赋值为 ViewRootImp 的 mAttachInfo了。

[Java] *纯文本查看* *复制代码*

~~~
//分析一下 View 中的 AttachInfo 的赋值，以及 ViewRootImpl 中的 mAttachInfo
    public ViewRootImpl(Context context, Display display) {
        ...
        mWindow = new W(this);
        ...
         mAttachInfo = new View.AttachInfo(mWindowSession, mWindow, display, this, mHandler, this);
        ...
    }
    //看到mWindowToken其实就是IWindow实例
    AttachInfo(IWindowSession session, IWindow window, Display display,
                 ViewRootImpl viewRootImpl, Handler handler, Callbacks effectPlayer) {
           mSession = session;
           mWindow = window;
           mWindowToken = window.asBinder();
           mDisplay = display;
           mViewRootImpl = viewRootImpl;
           mHandler = handler;
           mRootCallbacks = effectPlayer;
    }
    // ViewRootImpl在第一次执行performTraversals时，会把自己的mAttachInfo传递给根View,然后由根View逐级传递下去
    private void performTraversals() {
       ...
        if (mFirst) {
            ...
            host.dispatchAttachedToWindow(mAttachInfo, 0);
            ...
        }else{
            ...
        }
    }
    //ViewGroup.java
    void dispatchAttachedToWindow(AttachInfo info, int visibility) {
       mGroupFlags |= FLAG_PREVENT_DISPATCH_ATTACHED_TO_WINDOW;
       super.dispatchAttachedToWindow(info, visibility);
       mGroupFlags &= ~FLAG_PREVENT_DISPATCH_ATTACHED_TO_WINDOW;
       final int count = mChildrenCount;
       final View[] children = mChildren;
       for (int i = 0; i < count; i++) {
           final View child = children[i];
           child.dispatchAttachedToWindow(info,
                   visibility | (child.mViewFlags & VISIBILITY_MASK));
      }
    }
    //View.java
    void dispatchAttachedToWindow(AttachInfo info, int visibility) {
       mAttachInfo = info;
       ...
     }   
[/i]
~~~

**WindowManager.LayoutParams 中的 type 与 token**

WindowManager.LayoutParams 用来描述一个窗口的特性，最终在添加窗口时，会传递给 WmS。而且 WmS 会保存在 WindowState 的 mAttrs 中。LayoutParams 有很多参数，但是跟窗口创建相关的参数，最重要的就是 type 与 token 了，这里我们可以通过分析 WmS 的 addWindow 代码的可以知道：

[Java] *纯文本查看* *复制代码*

~~~
[i]
    //WindowManagerService.java addWindow 
    ...
    //权限检查，需要用到type类型，会检查窗口类型是否合法，如果是系统窗口类型
    //还需要进行权限检查，详见PhoneWindowManager.java
    int res = mPolicy.checkAddPermission(attrs, appOp);
    ...
    //如果是子窗口类型，还会检查其父窗口是否存在，如果父窗口不存在，直接抛出异常
    if (type >= FIRST_SUB_WINDOW && type <= LAST_SUB_WINDOW) {
                attachedWindow = windowForClientLocked(null, attrs.token, false);
                if (attachedWindow == null) {
                     Slog.w(TAG, "Attempted to add window with token that is not a window: " + attrs.token + ".  Aborting.");
                     return WindowManagerGlobal.ADD_BAD_SUBWINDOW_TOKEN;
                 }
                 if (attachedWindow.mAttrs.type >= FIRST_SUB_WINDOW
                       && attachedWindow.mAttrs.type <= LAST_SUB_WINDOW) {
                    Slog.w(TAG, "Attempted to add window with token that is a sub-window: " + attrs.token + ".  Aborting.");
                    return WindowManagerGlobal.ADD_BAD_SUBWINDOW_TOKEN;
                 }
      }
      ...
     //如果是类型是TYPE_PRIVATE_PRESENTATION ，还会检查相应的显示设备
     if (type == TYPE_PRIVATE_PRESENTATION && !displayContent.isPrivate()) {
            Slog.w(TAG, "Attempted to add private presentation window to a non-private display.  Aborting.");
            return WindowManagerGlobal.ADD_PERMISSION_DENIED;
        }
     ...    
    //根据LayoutParams中的token，会检索WmS中保存的WindowToken，
    //由引可见，不同的窗口类型，其对应的token是有区别的。WmS要根据窗口类型来检查其传递过来的token是否合法。
    boolean addToken = false;
    WindowToken token = mTokenMap.get(attrs.token);
    if (token == null) {
       if (type >= FIRST_APPLICATION_WINDOW && type <= LAST_APPLICATION_WINDOW) {
                Slog.w(TAG, "Attempted to add application window with unknown token " + attrs.token + ".  Aborting.");
                return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
            }
    if (type == TYPE_INPUT_METHOD) {
                Slog.w(TAG, "Attempted to add input method window with unknown token " + attrs.token + ".  Aborting.");
                return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
            }
    if (type == TYPE_VOICE_INTERACTION) {
                Slog.w(TAG, "Attempted to add voice interaction window with unknown token "+ attrs.token + ".  Aborting.");
                return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
            }
    if (type == TYPE_WALLPAPER) {
                Slog.w(TAG, "Attempted to add wallpaper window with unknown token " + attrs.token + ".  Aborting.");
                return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
            }
    if (type == TYPE_DREAM) {
                Slog.w(TAG, "Attempted to add Dream window with unknown token " + attrs.token + ".  Aborting.");
                return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
            }
    if (type == TYPE_ACCESSIBILITY_OVERLAY) {
                Slog.w(TAG, "Attempted to add Accessibility overlay window with unknown token "  + attrs.token + ".  Aborting.");
                return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
            }
            token = new WindowToken(this, attrs.token, -1, false);
            addToken = true;
    } else if (type >= FIRST_APPLICATION_WINDOW && type <= LAST_APPLICATION_WINDOW) {
            AppWindowToken atoken = token.appWindowToken;
            if (atoken == null) {
                Slog.w(TAG, "Attempted to add window with non-application token " + token + ".  Aborting.");
                return WindowManagerGlobal.ADD_NOT_APP_TOKEN;
            } else if (atoken.removed) {
                Slog.w(TAG, "Attempted to add window with exiting application token "  + token + ".  Aborting.");
                return WindowManagerGlobal.ADD_APP_EXITING;
            }
            if (type == TYPE_APPLICATION_STARTING && atoken.firstWindowDrawn) {
                // No need for this guy!
                if (localLOGV) Slog.v(
                        TAG, "**** NO NEED TO START: " + attrs.getTitle());
                return WindowManagerGlobal.ADD_STARTING_NOT_NEEDED;
            }
        } else if (type == TYPE_INPUT_METHOD) {
            if (token.windowType != TYPE_INPUT_METHOD) {
                Slog.w(TAG, "Attempted to add input method window with bad token "  + attrs.token + ".  Aborting.");
                  return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
            }
        } else if (type == TYPE_VOICE_INTERACTION) {
            if (token.windowType != TYPE_VOICE_INTERACTION) {
                Slog.w(TAG, "Attempted to add voice interaction window with bad token "  + attrs.token + ".  Aborting.");
                  return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
            }
        } else if (type == TYPE_WALLPAPER) {
            if (token.windowType != TYPE_WALLPAPER) {
                Slog.w(TAG, "Attempted to add wallpaper window with bad token "   + attrs.token + ".  Aborting.");
                  return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
            }
        } else if (type == TYPE_DREAM) {
            if (token.windowType != TYPE_DREAM) {
                Slog.w(TAG, "Attempted to add Dream window with bad token " + attrs.token + ".  Aborting.");
                  return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
            }
        } else if (type == TYPE_ACCESSIBILITY_OVERLAY) {
            if (token.windowType != TYPE_ACCESSIBILITY_OVERLAY) {
                Slog.w(TAG, "Attempted to add Accessibility overlay window with bad token " + attrs.token + ".  Aborting.");
                return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
            }
        } else if (token.appWindowToken != null) {
            Slog.w(TAG, "Non-null appWindowToken for system window of type=" + type);
            // It is not valid to use an app token with other system types; we will
            // instead make a new token for it (as if null had been passed in for the token).
            attrs.token = null;
            token = new WindowToken(this, null, -1, false);
            addToken = true;
        }   
      ...
    //经过一系列的检查之后，最后会生成窗口在WmS中的表示WindowState，并且把LayoutParams赋值给WindowState的mAttrs
    win = new WindowState(this, session, client, token, attachedWindow, appOp[0], seq, attrs, viewVisibility, displayContent);  
    ...  
    if (addToken) {
     mTokenMap.put(attrs.token, token);
    } 
    ...
     mWindowMap.put(client.asBinder(), win);   
[/i]
~~~

**总结一下：**

（1） 窗口类型必须是指定合法范围内的，即应用窗口，子窗口，系统窗口中的一种，否则检查会失败；

（2） 如果是系统，需要进行权限检查

以下类型不需要特别声明权限

TYPE _ TOAST，TYPE _ DREAM，TYPE _ INPUT _ METHOD，TYPE _ WALLPAPER，TYPE _ PRIVATE _ PRESENTATION，TYPE _ VOICE _ INTERACTION，TYPE _ ACCESSIBILITY _ OVERLAY

以下类型需要声明使用权限：android.permission.SYSTEM _ ALERT _ WINDOW

TYPE _ PHONE，TYPE _ PRIORITY _ PHONE，TYPE _ SYSTEM _ ALERT，TYPE _ SYSTEM _ ERROR，TYPE _ SYSTEM _ OVERLAY

其他的系统窗口，需要声明权限：android.permission.INTERNAL _ SYSTEM _ WINDOW

**（3）** 如果是应用窗口，通过 token 检索出来的 WindowToken，一定不能为空，而且还必须是 Activity 的 mAppToken，同时对应的 Activity 还必须是没有被 finish。之前分析 Activity 的启动过程我们知道，Activity 在启动过程中，会先通过 WmS 的 addAppToken( )添加一个 AppWindowToken 到 mTokenMap 中，其中 key 就用了 IApplicationToken token。而 Activity 中的 mToken，以及 Activity 对应的 PhoneWindow 中的 mAppToken 就是来自 AmS 的 token (代码见 Activity 的 attach 方法)。

**（4）** 如果是子窗口，会通过 attrs.token 去通过 windowForClientLocked 查找其父窗口，如果找不到其父窗口，会抛出异常。或者如果找到的父窗口的类型还是子窗口类型，也会抛出异常。这里查找父窗口的过程，是直接取了 attrs.token 去 mWindowMap 中找对应的 WindowState，而 mWindowMap 中的 key 是 IWindow。所以，由此可见，创建一个子窗口类型，token 必须赋值为其父窗口的 ViewRootImpl 中的 W 类对象 mWindow。

**（5）** 如果是如下系统窗口，TYPE _ INPUT _ METHOD，TYPE _ VOICE _ INTERACTION，TYPE _ WALLPAPER，TYPE _ DREAM，TYPE _ ACCESSIBILITY _ OVERLAY，token 不能为空，而且通过 token 检索到的 WindowToken 的类型不能是其本身对应的类型。

**（6）** 如果是其他系统窗口，会直接把 attrs 中的 token 给清除了，不需要 token。因此其他类型的系统窗口，LayoutParams 中 token 是可以为空的。

**（7）** 检查通过后，如果需要创建新的 WindowToken，会以 attrs.token 为 key，add 到 mTokenMap 中。

**（8）** WindowState 创建后，会以 IWindow 为 key (对应应用进程中的 ViewRootImpl.W 类对象 mWindow，重要的事强调多遍！！)，添加到 mWindowMap 中。

由此可见，我们要成功添加一个窗口，对于 type 与 token 的赋值是有要求的，否则先不说能否正确显示，直接就创建失败了。那 type 与 token 是如何赋值的呢？最直接来讲，就是在调用 WindowManagerImpl 的 addView 方法前，把值赋好就可以了。但是，分析 FrameWork 所提供的一些窗口的显示，如 Dialog 等，并没有看到在调用 addView 之前，对 token 赋值呢。其窗口类型是应用窗口，根据前面所描述的检查，token 肯定不能为 null 的，而且还必须是 Activity 的 mAppToken，否则创建失败的。这里我们分析一下，在 token 没有赋值的情况下，调用 addView 会做哪些处理。代码就要回到 WindowManagerImpl 开始了。

[Java] *纯文本查看* *复制代码*

~~~
[i]
    //WindowManagerImpl.java addView
    //applyDefaultToken会检查，有没有设置默认token，如果有设置，而且没有设置父窗口的情况下，token又是null的话，直接把默认token赋值给token吧。
    @Override
    public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
       applyDefaultToken(params);
       mGlobal.addView(view, params, mDisplay, mParentWindow);
    }
    //WindowManagerGlobal.java addView
    //如果有设置父窗口，会通过adjustLayoutParamsForSubWindow来调整params。
    final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams)params;
    if (parentWindow != null) {
        parentWindow.adjustLayoutParamsForSubWindow(wparams);
    }
    //Window.java adjustLayoutParamsForSubWindow
    if (wp.type >= WindowManager.LayoutParams.FIRST_SUB_WINDOW &&
        wp.type <= WindowManager.LayoutParams.LAST_SUB_WINDOW) {
        //如果是子窗口类型，而且token为null，直接取父窗口的AttachInfo中的mWindowToken，其实就是父窗口对应的ViewRootImpl中的W类对象mWindow。
        if (wp.token == null) {
            View decor = peekDecorView();
            if (decor != null) {
                wp.token = decor.getWindowToken();
            }
        }
        ...
    }else {
    //如果token为null，直接取父窗口的mAppToken吧。
    if (wp.token == null) {
            wp.token = mContainer == null ? mAppToken :           mContainer.mAppToken;
      }
    }
[/i]
~~~

这里的父窗口并不一定是真正意义上的父窗口，有可能就是描述一个窗口的 PhoneWindow 对象本身。有可能 PhoneWindow a，需要添加 a 窗口时，这里 parentWindow 有可能就是 a 对象本身。这里  WindowManagerImpl 中的 mParentWindow，到底代表什么，跟 WindowManagerImpl 的创建有关。例如 Activity 添加窗口的分析可知：

[Java] *纯文本查看* *复制代码*

~~~
[i]
    //Activity.java attach方法, 给PhoneWindow设置WindowManager
    mWindow.setWindowManager(
            (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
            mToken, mComponent.flattenToString(),
            (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
    // Window.java 这里mWindowManager其实是一个WindowManagerImpl对象，而且是重新创建一个新对象的。这里调用了createLocalWindowManager来创建一个新的WindowManagerImpl，传递的parentWindow就是它自己的。这表示PhoneWindow创建一个WindowManagerImpl对象时，把它自己作为parentWindow传递给WindowManagerImpl。
    public void setWindowManager(WindowManager wm, IBinder appToken, String appName,
        boolean hardwareAccelerated) {
    mAppToken = appToken;
    mAppName = appName;
    mHardwareAccelerated = hardwareAccelerated
            || SystemProperties.getBoolean(PROPERTY_HARDWARE_UI, false);
    if (wm == null) {
        wm = (WindowManager)mContext.getSystemService(Context.WINDOW_SERVICE);
    }
    mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager(this);
    } 
    //WindowManagerImpl.java
    public WindowManagerImpl createLocalWindowManager(Window parentWindow){
    return new WindowManagerImpl(mDisplay, parentWindow);
    }        
[/i]
~~~

这里再总结一下，添加窗口时，WindowManager.Layout 的 token 的一些赋值情况：

（1） 无论是应用窗口，还是子窗口，以及部分系统窗口，token 是一定要赋值的。否则创建窗口会抛异常。对于应用窗口，token 必须是某个 Activity 的 mToken。对于子窗口而言，token 必须是其父窗口的 IWindow 对象。对于部分系统窗口而言，token 检索到的 WindowToken 的类型，不能再是其本身的类型了。

**（2）** 可以在调用 WindowManagerImpl 的 addView 创建窗口前，直接把 token 赋值好。

**（3）** 如果调用 WindowManagerImpl 的 addView 前，没有把 token 赋值好。这里会走一些默认逻辑，以保证 token 的合法性：

**A:** 检查有没有设置默认 token，如果有，而且父窗口为 null，直接取设置的默认 token 吧。目前还没有遇到用这种方法设置 token 的情况。

**B:** 如果有设置父窗口，则会调用父窗口的 adjustLayoutParamsForSubWindow 来检查 token。对于子窗口类型，会把父窗口的 IWindow 对象赋值给 token。对于应用窗口，或者系统窗口，直接取父窗口的 mAppToken。（父窗口是一个 Window 对象，其实例是 PhoneWindow。）

**三、窗口的创建与移除**

在分析窗口的创建与移除之前，我们先简单来介绍一下 Android 的 GUI 系统，它包含以下部分内容：

（1）窗口和图形系统—Window and View Manager System

（2）显示合成系统 — Surface Flinger

（3）用户输出系统 — InputManager System

（4）应用框架系统 — Activity Manager System

它们之间的关系，如下图所示：

![](http://img2.tuicool.com/FF7rmii.png!web)

简单来讲，Activity Manager System（重点是 ActivityManagerService，简称 AmS）负责 Activity 的启动，以及生命周期的管理。一个 Activity 对应一个应用窗口，这个窗口的创建以及管理是 Window and View Manager System 的职责。例如前面的截图，手机屏幕上显示了三个窗口，状态栏窗口，手Q 主界面窗口，以及一个 PopupWindow。View 系统管理每个窗口中复杂的布局，最终这个 View Hierarchy 最顶端的根 View 会被作为窗口，添加到 Window Manager System 中。Window Manager System 管理着所有这些添加的窗口，负责管理这些窗口的层次，显示位置等内容。每个窗口都有一块自己的Surface，Surface Flinger 负责把这些 Surface 合成一块 FrameBuffer。

![](http://img1.tuicool.com/FJBn2yr.png!web)

也就是说 Window Manager System 负责窗口的创建与移除，以及显示状态的管理。具体绘制是由 Suerface Flinger 来负责的。

**3.1 应用窗口的创建**

首先，我们来分析应用窗口的创建，这也是我们开发过程中，最先遇到的。从开发第一个 Hello World 的 Android 应用开始，我们就已经在接触应用窗口了。

我们知道每个 Activity 对应一个 PhoneWindow，当我们调用 setContentView 时，其实最终结果是把我们的 View 树作为子 View 添加到 PhoneWindow 的 DecorView 中。也就是每个 Activity 的根 View 其实是一个 DecorView。而最终这个 DecorView，又是在 ActivityThread 的 handleResumeActivity 方法中，通过 WindowMnagerImpl 的 addView 方法添加到 WmS 中去的。 PhoneWindow 的 setContentView 又做了哪些事情呢？只是通过 installDecor()方法，给窗口初始化一了些装饰。而所谓的装饰就是指界面上看到的标题栏，导航栏 ActionBar。而我们通过 Activity 的 setContentView 设置的 View，是作为窗口的内容(如下图所示，是作为ID为android.R.content 的 FrameLayout 的子 View)。这里，我们是不是就理解了，窗口的标题栏是如何被添加的。以及窗口的一些属性为什么要在 setContentView 调用之前被设置了。因为，generateLayout 方法负责根据窗口的属性，最终决定采用不同的布局来生成窗口的顶层布局。而 generateLayout 只在 mContentParent 为 null 的时候被调用，而 mContentParent 只有在第一次调用 setContentView 时为 null，此后就不再为 null 了。

![](http://img2.tuicool.com/M7fUrye.png!web)

从上面的图可以知道，PhoneWindow 只是负责处理一些应用窗口通用的逻辑。但是真正完成把一个 View，作为窗口添加到 WmS 的过程是由 WindowManager 来完成的。WindowManager 在应用程序端的实现是 WindowManagerImpl，也就是说，我们通过 Activity 的 getWindowManager 获取到的实际上是 WindowManagerImpl。因此在在 ActivityThread 的 handleResumeActivity 方法中，有调用 WindowManagerImpl 的 addView，把 PhoneWindow 准备好的 DecorView 作为窗口添加到 WmS 中。代码如下所示：

[Java] *纯文本查看* *复制代码*

~~~
[i]
    if (r.window == null && !a.mFinished && willBeVisible) {
            r.window = r.activity.getWindow();
            View decor = r.window.getDecorView();
            decor.setVisibility(View.INVISIBLE);
            ViewManager wm = a.getWindowManager();
            WindowManager.LayoutParams l = r.window.getAttributes();
            a.mDecor = decor;
            l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
            l.softInputMode |= forwardBit;
            if (a.mVisibleFromClient) {
                a.mWindowAdded = true;
                wm.addView(decor, l);
            }
[/i]
~~~

![](http://img1.tuicool.com/UJRv6nq.png!web)

窗口的添加过程如上图所示，我们知道 WmS 运行在单独的进程中。这里 IWindowSession 执行的 addtoDisplay 操作应该是 IPC 调用。每个应用窗口创建时，都会创建一个 ViewRootImpl 对象。分析了后面子窗口的创建，以及系统窗口的创建后，我们会知道其实任何一个窗口的创建，最终都是会创建一个 ViewRootImpl对象。ViewRootImpl 是一很重要的类，类似 ActivityThread 负责跟AmS通信一样，ViewRootImpl 的一个重要职责就是跟 WmS 通信，它通静态变量 sWindowSession（IWindowSession实例）与 WmS 进行通信。每个应用进程，仅有一个 sWindowSession 对象，它对应了 WmS 中的 Session 子类，WmS 为每一个应用进程分配一个 Session 对象。WindowState 类有一个 IWindow mClient 参数，是在构造方法中赋值的，是由 Session 调用 addWindow 传递过来了，对应了 ViewRootImpl 中的 W 类的实例。

![](http://img1.tuicool.com/iIrU3ue.png!web)

这里梳理了一下，这些类的对应关系。由此可以看到，创建一个窗口的本质过程，就是在 WmS 端创建一个用来表示这个窗口的 WindowState 对象。 WmS 负责管理这里些 WindowState 对象。

![](http://img2.tuicool.com/MFnQni.png!web)

到此为止，一个应用窗口的创建过程就结束了。

**总结一下，对于Acivity对应的应用窗口创建：**

**（1）** 其 type 类型是TYPE _ BASE _ APPLICATION，定义为应用窗口；

**（2）** 用来执行 addView 操作的 WindowManager，到底是哪个呢？分析前面的源码，是通过 Activity 的 getWindowManager 来获取的，而返回的是 Activity 中的 mWindowManager，而它的赋值又是在 attch 方法中，直接取的 Window 的 mWindowManager。Window 的 mWindowManager 又是在 attach 方法中，通过 mWindow 的 setWindowManager 来初始化的。通过前面的分析可以知道，最终 Window 中的 mWindowManager，是通过 createLocalWindowManager(this) 创建一个新的 WindowManagerImpl 对象，这个对象的 mParentWindow 就是 Activity 中的 mWindow。

**（3）** 基 token 呢？查看添加窗口过程，在调用 addView 方法之前，没有为 token 赋值呢？这里走的是默认的赋值逻辑。而 Activity 对应的 WindowManagerImpl 实例中，mParentWidnow 是其对应的 PhoneWindow。所以在默认检查逻辑中，会走它自己的 PhoneWindow 的 adjustLayoutParamsForSubWindow，把 mAppToken 赋值给 token。

到此为止，type 与 token 都合法了，可以创建窗口了。Window 中的 mAppToken 在 Activity 的 attach 方法中已经通过调用 Window 的 setWindowManager 赋值好了，就是 Activity 中的 mToken 的值。

**Dialog 对应的应用窗口的创建**

在我们的印象中，Dialog 是个子窗口。但是看 Dialog 的源码，我们会发现 Dialog 的类型是 TYPE _ APPLICATION，应用窗口。跟 Activity 对应的窗口一样，Dialog 有一个 PhoneWindow 的实例。在 Dialog 的构建方法中，会通过 PolicyManager.makeNewWindow(mContext)生成一个 Window 对象，其实就是 PhoneWindow 对象，一点跟 Activity 创建应用窗口很像。有一点不同的是，在调用 Window 的 setWindowManager 时，参数 appToken 与 appName 传递都是 null。而 Activity 是把自己的 mToken 给赋值过去的。

[Java] *纯文本查看* *复制代码*

~~~
[i]
    Dialog(Context context, int theme, boolean createContextThemeWrapper) {
    if (createContextThemeWrapper) {
        if (theme == 0) {
            TypedValue outValue = new TypedValue();
            context.getTheme().resolveAttribute(com.android.internal.R.attr.dialogTheme,
                    outValue, true);
            theme = outValue.resourceId;
        }
        mContext = new ContextThemeWrapper(context, theme);
    } else {
        mContext = context;
    }
    //这里的context一般是指Activity的，如果传递的不是Activity，在显示Dialog时会抛出异常android.view.WindowManager$BadTokenException: Unable to add window -- token null is not for an application
    mWindowManager = (WindowManager)context.getSystemService(Context.WINDOW_SERVICE);
    Window w = PolicyManager.makeNewWindow(mContext);
    mWindow = w;
    w.setCallback(this);
    w.setOnWindowDismissedCallback(this);
    //这里有一点要注意的是，Dialog调用setWindowManager时，没有传递token
    //回到前面有关token的描述，PhoneWindow的mAppToken是在这个方法被调用时赋值
    //的，由此说来Dialog的mAppToken是空的。这是跟应用窗口的差别所在。
    w.setWindowManager(mWindowManager, null, null);
    w.setGravity(Gravity.CENTER);
    mListenersHandler = new ListenersHandler(this);
    }
[/i]
~~~

接下来，当应用程序调用 show 方法时，Dialog 就会显示出来，由此可见，把 View 添加到窗口的过程应该是在 show 方法中执行的。

[Java] *纯文本查看* *复制代码*

~~~
[i]
    public void show() {
    ...
    mDecor = mWindow.getDecorView();

    if (mActionBar == null && mWindow.hasFeature(Window.FEATURE_ACTION_BAR)) {
        final ApplicationInfo info = mContext.getApplicationInfo();
        mWindow.setDefaultIcon(info.icon);
        mWindow.setDefaultLogo(info.logo);
        mActionBar = new WindowDecorActionBar(this);
    }

    WindowManager.LayoutParams l = mWindow.getAttributes();
    if ((l.softInputMode
            & WindowManager.LayoutParams.SOFT_INPUT_IS_FORWARD_NAVIGATION) == 0) {
        WindowManager.LayoutParams nl = new WindowManager.LayoutParams();
        nl.copyFrom(l);
        nl.softInputMode |=
                WindowManager.LayoutParams.SOFT_INPUT_IS_FORWARD_NAVIGATION;
        l = nl;
    }

    try {
        mWindowManager.addView(mDecor, l);
        mShowing = true;

        sendShowMessage();
    } finally {
    }
    }
[/i]
~~~

**分析到这里，有几点疑问需要回答**

**（1）** Dialog 的 type 是 TYPE _ APPLICATION，表示应用窗口。这个应用窗口跟 Activity 对应的应用窗口是什么不一样呢？在分析代码时，发现 Dialog 的 PhoneWindow 实例，在调用 setWindowManger 时，给参数 appToken 传递的是 null，也就是 PhoneWindow 对象的 mAppToken 是空的。而 Activity 的 PhoneWindow 的 mAppToken 等于其 mToken，一定不是空的。

**（2）** Activity 的 WindowManager 是通过 Context 的 getSystemService 来获取的。而且最终也是通过这个 WindowManger 来执行 addView 操作。这个 WindowManger 的实例，到底是如何创建的呢？

**（3）** Dialog 在 addView 前，也没有为 token 赋值。显然 token 的值需要走默认逻辑来赋值。而默认逻辑跟 WindowManager 实例中的 mParentWindow 有很大的关系的。显然 Dialog 中的 mWindowManager 的 mParentWindow 不能为空，否则 Dialog 的 token 就没有赋值，创建窗口时会抛异常的。还有一个重点哦，Dialog 是应用窗口类型，token必须是 Activity 的 mToken 哦。说到这里，你是不是已经明白呢？如果 mWindowManager 就是 Activity 中的 mWindowManager，一切问题都解决了。Dialog 的构建方法里，要传递一个 Context，平常我们传递 Activity 的实例，看看 Activity 的 getSystemService 的实现。如果传递一个 Application 的 Context 会有什么结果呢？

![](http://img1.tuicool.com/nuInya7.png!web)

[Java] *纯文本查看* *复制代码*

~~~
[i]
    //如果是一个Application的Context，直接会调用ContextWrapper的getSystemService,因为Application并没有重载这个方法, 而ContextWrapper会调用mBase的getSystemService，mBase其实就是ContextImpl的实例
    @Override
    public Object getSystemService(String name) {
        return mBase.getSystemService(name);
    }
    //ContextImpl.java
    @Override
    public Object getSystemService(String name) {
        ServiceFetcher fetcher = SYSTEM_SERVICE_MAP.get(name);
        return fetcher == null ? null : fetcher.getService(this);
    }
    //而WINDOW_SERVICE，对应去获取的，其实是创建一个WindowManagerImpl的实例，这个实例的mParentWindow是空的。如果mParentWindow是空的，说明Dialog的创建窗口时，token是没有被赋值的。在WmS中，会直接抛出异常：android.view.WindowManager$BadTokenException: Unable to add window -- token null is not for an application
    registerService(WINDOW_SERVICE, new ServiceFetcher() {
            Display mDefaultDisplay;
            public Object getService(ContextImpl ctx) {
                Display display = ctx.mDisplay;
                if (display == null) {
                    if (mDefaultDisplay == null) {
                        DisplayManager dm = (DisplayManager)ctx.getOuterContext().
                                getSystemService(Context.DISPLAY_SERVICE);
                        mDefaultDisplay = dm.getDisplay(Display.DEFAULT_DISPLAY);
                    }
                    display = mDefaultDisplay;
                }
                return new WindowManagerImpl(display);
            }});
[/i]
~~~

**3.2 子窗口的创建**

对于 WmS 来讲，无论什么样的窗口创建，最终都是通过 WindowManagerImpl 的 addView，来添加一个 View。只是对于不同类型的窗口，type 不同，token 的要求也有所不同。创建窗口的过程是本质是一样。常用的子窗口，有 PopupWindow，ContextMenu，OptionMenu。我们从具体的子窗口出发，来分析其中的差异，以及实现的原理。

**PopupWindow**

分析 PopupWindow 发现，它跟 Dialog 不同，并没有一个 Window 对象，在 invokePopup 方法中，直接把 mPopupView 通过 WindowManager 添加为一个窗口。

[Java] *纯文本查看* *复制代码*

~~~
[i]
    private void invokePopup(WindowManager.LayoutParams p) {
    if (mContext != null) {
        p.packageName = mContext.getPackageName();
    }
    mPopupView.setFitsSystemWindows(mLayoutInsetDecor);
    setLayoutDirectionFromAnchor();
    mWindowManager.addView(mPopupView, p);
    }
[/i]
~~~

其 WindowManager.LayoutParams 是通过 createPopupLayout(anchor.getWindowToken())来初始化的，其 type 类型是 WindowManager.LayoutParams.TYPE _ APPLICATION _ PANEL，属于子窗口类型。因此其 token 是取自外部传递的。无论是通过 showAsDropDown （ anchor.getWindowToken() ），还是通过 showAtLocation 始终都要给 PopupWindow 设置一个 token 的。

[Java] *纯文本查看* *复制代码*

~~~
[i]
    private WindowManager.LayoutParams createPopupLayout(IBinder token) {
    //这里new了一个Params，type的默认类型是TYPE_APPLICATION哦
    WindowManager.LayoutParams p = new WindowManager.LayoutParams();
    // 看到下面了，会修改type,修改为TYPE_APPLICATION_PANEL
    // token直接取赋值过来的token
    p.gravity = Gravity.START | Gravity.TOP;
    p.width = mLastWidth = mWidth;
    p.height = mLastHeight = mHeight;
    if (mBackground != null) {
        p.format = mBackground.getOpacity();
    } else {
        p.format = PixelFormat.TRANSLUCENT;
    }
    p.flags = computeFlags(p.flags);
    p.type = mWindowLayoutType;
    p.token = token;
    p.softInputMode = mSoftInputMode;
    p.setTitle("PopupWindow:" + Integer.toHexString(hashCode()));

    return p;
    }
    // 一般要显示PopupWindow都是通过showAtLocation
    //或者showAsDropDown来的，token就来自这两个方法。要不调用方，直接传递一个token，要不调用方法传递一个anchor，直接取anchor中的getWindowToken。
    public void showAtLocation(IBinder token, int gravity, int x, int y) {
    ...
    WindowManager.LayoutParams p = createPopupLayout(token);
    ...
    preparePopup(p);
    ...    
    invokePopup(p);
    }
    public void showAsDropDown(View anchor, int xoff, int yoff, int gravity) {
    ...
    WindowManager.LayoutParams p = createPopupLayout(anchor.getWindowToken());
    preparePopup(p);
    ...
    invokePopup(p);
    }
[/i]
~~~

**这里也总结一下，PopUpWindow 的显示：**

**（1）** 其 Type 类型是 TYPE _ APPLICATION _ PANEL

**（2）** 其 mWindowManager 是通过 Context 的 getSystemService 来获取的。而 Context 有可能是通过其构建方法传递过来的，或者通过传递的 contentView 去取的。

**（3）** 其 token 是在 addView 前赋值好的。无论是 showAtDropDown 还是 showAtLocation，都需要传递一个 View anchor 进来，token 直接取的 anchor.getWindowToken。View 的 getWindowToken 返回的是 ViewRootImpl 的 AttachInfo 中的 mWindowToken。

**ContextMenu （情景菜单）**

ContextMenu 是 Android 的一个标准交互，一般是长按一个 View 时，可以显示当前的情景菜单。当我们没有自己处理长按事件时，Android 默认在长按操作时，会调用 showContextMenu()，而 View 的 showContextMenu 又会不断调用 showContextMenuForChild，如果它有 Parent 的话，

[Java] *纯文本查看* *复制代码*

~~~
[i]
      //View.java
      public boolean performLongClick() {
      sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_LONG_CLICKED);

      boolean handled = false;
      ListenerInfo li = mListenerInfo;
      if (li != null && li.mOnLongClickListener != null) {
          handled = li.mOnLongClickListener.onLongClick(View.this);
      }
      if (!handled) {
          handled = showContextMenu();
      }
      if (handled) {
          performHapticFeedback(HapticFeedbackConstants.LONG_PRESS);
      }
      return handled;
    }
    //View.java
       public boolean showContextMenu() {
       return getParent().showContextMenuForChild(this);
    }
    //ViewGroup.java
       public boolean showContextMenuForChild(View originalView) {
       return mParent != null && mParent.showContextMenuForChild(originalView);
    }
[/i]
~~~

对于具有 PhoneWindow 的窗口而言，View 树的根视图是 DecorView，所以最终都会调用 DecorView 的同名方法，其处理如下：

[Java] *纯文本查看* *复制代码*

~~~
[i]
        @Override
    public boolean showContextMenuForChild(View originalView) {
        // Reuse the context menu builder
        if (mContextMenu == null) {
            mContextMenu = new ContextMenuBuilder(getContext());
            mContextMenu.setCallback(mContextMenuCallback);
        } else {
            mContextMenu.clearAll();
        }

        final MenuDialogHelper helper = mContextMenu.show(originalView,
                originalView.getWindowToken());
        if (helper != null) {
            helper.setPresenterCallback(mContextMenuCallback);
        } else if (mContextMenuHelper != null) {
            // No menu to show, but if we have a menu currently showing it just became blank.
            // Close it.
            mContextMenuHelper.dismiss();
        }
        mContextMenuHelper = helper;
        return helper != null;
    }
[/i]
~~~

情景菜单真正显示的地方，应该是在 ContextMenuBuilder 的 show 方法里的，看到了吧，这里会调用 originalView 的 createContextMenu方法，而 originalView 就是被长按的 View 呀。

[Java] *纯文本查看* *复制代码*

~~~
[i]
    public MenuDialogHelper show(View originalView, IBinder token) {
    if (originalView != null) {
        // Let relevant views and their populate context listeners populate
        // the context menu
        originalView.createContextMenu(this);
    }

    if (getVisibleItems().size() > 0) {
        EventLog.writeEvent(50001, 1);

        MenuDialogHelper helper = new MenuDialogHelper(this); 
        helper.show(token);

        return helper;
    }

    return null;
    }
[/i]
~~~

接着，调用 MenuDialogHelper 的 show 方法，来完成创建窗口的操作，到这里是不是对于情景菜单的显示逻辑很清楚了。

[Java] *纯文本查看* *复制代码*

~~~
[i]
    public void show(IBinder windowToken) {
    // Many references to mMenu, create local reference
    final MenuBuilder menu = mMenu;
    // Get the builder for the dialog
    final AlertDialog.Builder builder = new AlertDialog.Builder(menu.getContext());

    mPresenter = new ListMenuPresenter(builder.getContext(),
            com.android.internal.R.layout.list_menu_item_layout);

    mPresenter.setCallback(this);
    mMenu.addMenuPresenter(mPresenter);
    builder.setAdapter(mPresenter.getAdapter(), this);

    // Set the title
    final View headerView = menu.getHeaderView();
    if (headerView != null) {
        // Menu's client has given a custom header view, use it
        builder.setCustomTitle(headerView);
    } else {
        // Otherwise use the (text) title and icon
        builder.setIcon(menu.getHeaderIcon()).setTitle(menu.getHeaderTitle());
    }

    // Set the key listener
    builder.setOnKeyListener(this);

    // Show the menu
    mDialog = builder.create();
    mDialog.setOnDismissListener(this);

    WindowManager.LayoutParams lp = mDialog.getWindow().getAttributes();
    lp.type = WindowManager.LayoutParams.TYPE_APPLICATION_ATTACHED_DIALOG;
    if (windowToken != null) {
        lp.token = windowToken;
    }
    lp.flags |= WindowManager.LayoutParams.FLAG_ALT_FOCUSABLE_IM;

    mDialog.show();
    }
[/i]
~~~

我们再来看看 View 的 createContextMenu 做了什么事情，调用本身的 onCreateContextMenu，以及调用 mParent 的 createContextMenu，就这样一直遍历下去。由此可见，父视图的的情景菜单项会出现在每个一个子视图中。

[Java] *纯文本查看* *复制代码*

~~~
[i]
    public void createContextMenu(ContextMenu menu) {
    ContextMenuInfo menuInfo = getContextMenuInfo();
    // Sets the current menu info so all items added to menu will have
    // my extra info set.
    ((MenuBuilder)menu).setCurrentMenuInfo(menuInfo);

    onCreateContextMenu(menu);
    ListenerInfo li = mListenerInfo;
    if (li != null && li.mOnCreateContextMenuListener != null) {
        li.mOnCreateContextMenuListener.onCreateContextMenu(menu, this, menuInfo);
    }

    // Clear the extra information so subsequent items that aren't mine don't
    // have my extra info.
    ((MenuBuilder)menu).setCurrentMenuInfo(null);

    if (mParent != null) {
        mParent.createContextMenu(menu);
     }
    }
[/i]
~~~

总结来看，情景菜单本质上是一个 Dialog （这里要注意，这个 Dialog 的窗口类型，被赋值为：WindowManager.LayoutParams.TYPE _ APPLICATION _ ATTACHED _ DIALOG），Framework 的 UI 框架，提供了一种便利的方式借大家添加菜单项。

**总结一下，ContextMenu 的创建：**

**（1）** 本质是一个 Dialog，只是 type 修改为TYPE _ APPLICATION _ ATTACHED _ DIALOG，为子窗口类型；

**（2）** Dialog 的 Context 是取了 menu.getContext()，而 mMenu本 质应该是 DecorView。这个 DecorView 的 context 自然是 PhoneWindow 的 context了。如果长按的是 Activity 对应窗口中的View，自然 Context 是 Activity 了。

（3） final MenuDialogHelper helper = mContextMenu.show(originalView,

originalView.getWindowToken()); 可以看到传递的 token 是被按下的 View 的 WindowToken，这个就是一个 IWindow 对象。

**（4）** 其实只要 token 有被赋予正确的值，这里 Context 本身是什么类型，并不重要。无论是什么类型的 Context，获取 WindowService 时，都会返回一个 WinderManagerImpl 的实例。由于 token 已经被正确赋值了，就不需要 mParentWindow 的默认逻辑来赋值了。

**OptionMenu (选项菜单)**

选项菜单一般是用户按下”Menu”键后弹出的菜单，要启动 OptionMenu，一种是按下“Menu”键，另一种是调用 openOptionsMenu 方法。对于通用的窗口来说，所有的用户消息都由窗口中的视图来处理的，但是对于具有 Window 对象（即 PhoneWindow 对象）的窗口来说，它已经帮窗口处理了”Menu”按键消息。具体在 PhoneWindow 的 onKeyDown 方法中，会调用 onKeyDownPanel，在 onKeyUp 中会调用 onKeyUpPanel。最终来讲，要显示 OptionMenu 都是通过调用 openPanel 来实现的。

~~~
[i]
    private void openPanel(final PanelFeatureState st, KeyEvent event){

    //...此处省略了对于st的decorView的校验，以及处理
    st.isHandled = false;

    WindowManager.LayoutParams lp = new WindowManager.LayoutParams(
            width, WRAP_CONTENT,
            st.x, st.y, WindowManager.LayoutParams.TYPE_APPLICATION_ATTACHED_DIALOG,
            WindowManager.LayoutParams.FLAG_ALT_FOCUSABLE_IM
            | WindowManager.LayoutParams.FLAG_SPLIT_TOUCH,
            st.decorView.mDefaultOpacity);

    if (st.isCompact) {
        lp.gravity = getOptionsPanelGravity();
        sRotationWatcher.addWindow(this);
    } else {
        lp.gravity = st.gravity;
    }

    lp.windowAnimations = st.windowAnimations;

    wm.addView(st.decorView, lp);
    st.isOpen = true;
    // Log.v(TAG, "Adding main menu to window manager.");
    }
[/i]
~~~

分析 openPanel 的实现，我们知道 OptionMenu 并没有 PhoneWindow 对象，仅仅是一个 View，通过 PanelFeatureState 来管理的。显示 OptionMenu 所用到的 WindowManager 其实就是当前调用 openPanel 的 PhoneWindow 的 mWindowManager，跟 Dialog 一样，在执行 addView 过程中，OptionMenu的WindowManager.LayoutParams 的 tokey 会被赋值为 Activity 的 mAppToken。当窗口被真正添加时，会调整为对应的 ViewRootImpl 的 W 类对象。

要显示 OptionMenu，本质就是更新 PanelFeatureState 中的内容，Window.Callback 定义了一些，专门用来准备 Optionmenu 用的，以及响应 OptionMenu 的操作用的。通过 WindowCallback 的这些接口，Android Framework 把显示选项菜单的流程自己处理了，同时具体显示菜单内容的权限交给了 Activity，这就是我们在实现过程中，只需要重载这些接口就能实现显示选项菜单的原因所在了。而且 OptionMenu 的根视图也是 DecorView，

[Java] *纯文本查看* *复制代码*

~~~
[i]
     //Window.Callback中定义的，跟OptionMenu相关的接口
     public View onCreatePanelView(int featureId);
     public boolean onCreatePanelMenu(int featureId, Menu menu);
     public boolean onPreparePanel(int featureId, View view, Menu menu);
     public void onPanelClosed(int featureId, Menu menu);
     public boolean onMenuOpened(int featureId, Menu menu);
     public boolean onMenuItemSelected(int featureId, MenuItem item);
[/i]
~~~

**总结一下，OptionMenu 的创建：**

（1）其 type 类型是TYPE _ APPLICATION _ ATTACHED _ DIALOG，属于子窗口；

（2） 由于 OptionMenu 是在 PhoneWindow 的 openPanel 中添加窗口的，WindowManager 直接取了 PhoneWindow的WindowManager。PhoneWindow 中的 WindowManager 是通过((WindowManagerImpl)wm).createLocalWindowManager(this)来创建 的，可见其 mParentWindow 就是当前 PhoneWindow。

（3） 显示之前，没有明确为 token 赋值。在 addView 中会走默认的赋值逻辑，最终会调用当前 PhoneWindow 本身的 adjustLayoutParamsForSubWindow。类型是子窗口，会取 PhoneWindow 的 DecorView 的 windowToken 赋值给 token。DecorView 的 WinodwToken 就是 IWindow 的实例。

**总结一下子窗口的创建：**

（1）type 的取值在 [FIRST _ SUB _ WINDOW, LAST _ SUB _ WINDOW]；

（2）token 必须是父窗口的 IWindow 对象，父窗口必须存在的;

（3）当父窗口不可见时，子窗口也不可见；

思考一下，在 Activity 的 onCreate 方法中，可以创建普通的 Dialog 并显示。但是创建一个子窗口类型的 Dialog，并显示吗？ **3.3 系统窗口的创建**

系统窗口分4类：

（1）TYPE _ TOAST，不需要声明权限，也不需要 token;

（2）第二类，不需要声明权限，但是需要 token，而且对 token 有一定的要求；

（3）第三类，需要 android.Manifest.permission.SYSTEM _ ALERT _ WINDOW 权限；

（4）第四类，需要 android.Manifest.permission.INTERNAL _ SYSTEM _ WINDOW权限；

**Toast 窗口创建**

分析一下 Toast 窗口的创建，代码如下：

[Java] *纯文本查看* *复制代码*

~~~
[i]
    public void handleShow() {
        if (localLOGV) Log.v(TAG, "HANDLE SHOW: " + this + " mView=" + mView  + " mNextView=" + mNextView);
        if (mView != mNextView) {
            // remove the old view if necessary
            handleHide();
            mView = mNextView;
            // 这里优先取Application的Context，跟我们之前了解的，如果用Acivity的Context，容易导致Activity内存泄露有关
            Context context = mView.getContext().getApplicationContext();
            String packageName = mView.getContext().getOpPackageName();
            if (context == null) {
                context = mView.getContext();
            }
            //取WindowManager，这里取的WindowManagerImpl，一般来讲mParentWindow是空的。原因前面分析过了，Application的Context中，获取的WindowManagerImpl，mParentWindow就是空的。
            mWM = (WindowManager)context.getSystemService(Context.WINDOW_SERVICE);
            final Configuration config = mView.getContext().getResources().getConfiguration();
            final int gravity = Gravity.getAbsoluteGravity(mGravity, config.getLayoutDirection());
            mParams.gravity = gravity;
            if ((gravity & Gravity.HORIZONTAL_GRAVITY_MASK) == Gravity.FILL_HORIZONTAL) {
                mParams.horizontalWeight = 1.0f;
            }
            if ((gravity & Gravity.VERTICAL_GRAVITY_MASK) == Gravity.FILL_VERTICAL) {
                mParams.verticalWeight = 1.0f;
            }
            mParams.x = mX;
            mParams.y = mY;
            mParams.verticalMargin = mVerticalMargin;
            mParams.horizontalMargin = mHorizontalMargin;
            mParams.packageName = packageName;
            if (mView.getParent() != null) {
                if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
                mWM.removeView(mView);
            }
            // 直接addView了
            mWM.addView(mView, mParams);
            trySendAccessibilityEvent();
        }
    }
    //
    TN() {
        // XXX This should be changed to use a Dialog, with a Theme.Toast
        // defined that sets up the layout params appropriately.
        final WindowManager.LayoutParams params = mParams;
        params.height = WindowManager.LayoutParams.WRAP_CONTENT;
        params.width = WindowManager.LayoutParams.WRAP_CONTENT;
        params.format = PixelFormat.TRANSLUCENT;
        params.windowAnimations = com.android.internal.R.style.Animation_Toast;
        params.type = WindowManager.LayoutParams.TYPE_TOAST;
        params.setTitle("Toast");
        params.flags = WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON
                | WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
                | WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE;
    } 
[/i]
~~~

**3.4 窗口的移除**

任何一个窗口类型，窗口的移除都是通过调用 WindowManager 的 removeView 来完成的，具体流程如下：

![](http://img2.tuicool.com/zAvEve.png!web)ViewRootImpl 在收到要删除窗口的命令后，会执行以下操作，详细见源码分析：

（1）判断是否可以立即删除窗口，否则会等下次 UI 操作时执行；

（2）确认需要删除窗口时，会执行 doDie 方法，通过 dispatchDetachedFromWindow 通知 View 树，窗口要被删除了；

（3）dispatchDetachedFromWindow 执行以下操作

1、通过 dispatchDetachedFromWindow，通知 View 树，窗口已经移除了，你们已经 detach from window 了。

2、把窗口对应的 HardRender, Surface 给释放了；

3、通过 mWindowSession，通知 WmS，窗口要移除了，WmS 会把跟这个窗口相关的 WindowState，以及 WindowToken 给移除，同时更新其它窗口的显示

4、 通知 Choreographer,这个窗口不需要显示了，跟这个窗口相关的一些UI刷新操作，可以取消了。

（4）当根 View 收到 dispatchDetachedFromWindow 调用后，会遍历View树中的每一个 View，把这个通知传递下来。这样 View 的 mAttachInfo 会清除了，reset 为 null了。

[Java] *纯文本查看* *复制代码*

~~~
[i]
    //ViewRootImpl.java
    boolean die(boolean immediate) {
    //如果需要立即移除，立即执行doDie
    if (immediate && !mIsInTraversal) {
        doDie();
        return false;
    }
    if (!mIsDrawing) {
        destroyHardwareRenderer();
    } else {
       mWindowAttributes.getTitle());
    }
    //要不然，通过Handler发个消息，一会执行吧
    mHandler.sendEmptyMessage(MSG_DIE);
    return true;
    }
    // 在doDie中，会通过dispatchDetachedFromWindow，通知View树，窗口已经移除了
    void doDie() {
    ...
    synchronized (this) {
        if (mRemoved) {
            return;
        }
        mRemoved = true;
        if (mAdded) {
            dispatchDetachedFromWindow();
        }
        ...
    }
    ...
    }
    //dispatchDetachedFromWindow 执行以下操作
    //(1) 通过dispatchDetachedFromWindow，通知View树，窗口已经移除了，你们已经detach from window了。
    //(2) 把窗口对应的HardRender, Surface给释放了；
    //(3) 通过mWindowSession，通知WmS，窗口要移除了，WmS会把跟这个窗口相关的WindowState，以及WindowToken给移除，同时更新其它窗口的显示
    //(4) 通知Choreographer,这个窗口不需要显示了，跟这个窗口相关的一些UI刷新操作，可以取消了。
    void dispatchDetachedFromWindow() {
    if (mView != null && mView.mAttachInfo != null) {
        mAttachInfo.mTreeObserver.dispatchOnWindowAttachedChange(false);
        mView.dispatchDetachedFromWindow();
    }

    ...
    destroyHardwareRenderer();
    setAccessibilityFocus(null, null);
    mView.assignParent(null);
    mView = null;
    mAttachInfo.mRootView = null;
    mSurface.release();
    ...
    try {
        mWindowSession.remove(mWindow);
    } catch (RemoteException e) {
    }
    ...
    unscheduleTraversals();
    }
[/i]
~~~

**四、总结** **这里我们总结一下，Android 中有关窗口的相关内容：**

（1）在 Window System 中，分为两部分的内容，一部分是运行在系统服务进程（WmS 所在进程）的 WmS 及相关类，另一部分是运行在应用进程的 WindowManagerImpl, WindowManagerGlobal，ViewRootImpl 等相关类。WmS 用 WindowState 来描述一个窗口，而应用进程用 ViewRootImpl，WindowManager.LayoutParms 来描述一个窗口的相关内容。

（2）对于 WmS 来讲，窗口对应一个 View 对象，而不是 Window 对象。添加一个窗口，就是通过 WindowManager 的 addView 方法。同样的，移除一个窗口，就是通过 removeView 方法。更新一个窗口的属性，通过 updateViewLayout 方法。

（3）Window 类描述是一类具有某种通用特性的窗口，其实现类是 PhoneWindow。Activity 对应的窗口，以及 Dialog 对应的窗口，会对应一个 PhoneWindow 对象。PhoneWindow 类把一些操作的统一处理了，例如长按，按”Back”键等。

（4）Android Framework 把窗口分为三种类型，应用窗口，子窗口以及系统窗口。不同类型的窗口，在执行添加窗口操作时，对于 WindowManager.LayoutParams 中的参数 token 具有不同的要求。应用窗口，LayoutParams 中的 token，必须是某个有效的 Activity 的 mToken。而子窗口，LayoutParams 中的 token，必须是父窗口的 ViewRootImpl 中的 W 对象。系统窗口，有些系统窗口不需要 token，有些系统窗口的 token 必须满足一定的要求。

（5）只能通过 Context.getSystemServer 来获取 WindowManager（即获取一个 WindowManagerImpl 的实例）。如果这个 context 是 Activity，则直接返回了 Activity 的 mWindowManager，其 WindowManagerImpl.mParentWindow 就是这个 Activity 本身对应的 PhoneWindow。如果这个 context 是 Application，或者 Service，则直接返回一个 WindowManagerImpl 的实例，而且 mParentWindow 为 null。

（6）在调用 WindowManagerImpl 的 addView 之前，如果没有给 token 赋值，则会走默认的 token 赋值逻辑。默认的 token 赋值逻辑是这样的，如果 mParentWindow 不为空，则会调用其 adjustLayoutParamsForSubWindow 方法。在 adjustLayoutParamsForSubWindow 方法中，如果当前要添加的窗口是，应用窗口，如果其 token 为空，则会把当前 PhoneWindow 的 mToken 赋值给 token。如果是子窗口，则会把当前 PhonwWindow 对应的 DecorView 的 mAttachInfo 中的 mWindowToken 赋值给 token。而 View 中的 AttachInfo mAttachIno 来自 ViewRootImpl 的 mAttachInfo。因此这个 token 本质就是父窗口的 ViewRootImpl 中的 W 类对象。