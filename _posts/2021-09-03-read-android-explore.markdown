---
layout: post
title: "读书摘要-Android 开发艺术探索"
published: true
categories: [Android]
tags: [快速复习,读书笔记]
---

# Android 开发艺术探索
{: toc }
## 第2章 IPC

### 3. IPC 基础概念介绍

IPC 有许多种方式：借助文件系统实现IPC（自建文件，SharePreference（有权限限制，需要expose），content provider，sqlite也是会将数据库暴露为文件节点），socket （Unix domain socket），Android使用虚拟设备方式实现的Binder方式

文件系统：文件IO，Linux文件系统包装互斥访问，但要实现内容同步，数据格式，协议 ，而且是辅存读写，等等问题，优点：操作简单，编码统一，通用性好

socket：需要自己实现相应的权限管理，可见性暴露，优点：通用性好，各种平台都支持，标准化

Binder：缺点：Android支持，不够通用，优点：速度较快，mmap，而且是Android系统架构的部分，配合AIDL 这样的接口定义语言，实现多语言支持，且是Android framework层广泛支持和适用的

Messager：Android 的Java工具，使用了Binder，操作简单

数据打包还可以使用第三方的Protobuf，语言中立 平台中立 版本管理匹配

 RPC等工具

### 4. Binder介绍（适用RPC，CORBA）

数据打包：序列化：Parcelable Serializable

收发端：BpBinder BnBinder

数据流动channel：Binder



1. Serializable 是Java支持的序列化接口

实现简单：implement Serializable接口，添加private static final long serialVersionUID = xxxxx

使用：ObjectOutputStream ObjectInputStream 来读取和保存 进行打包；序列化前后两个对象内容完全相同，但是不是同一个对象，就是说 equal 但是不 ==

serialVersionUID工作机制：序列化时，将UID写入文件，反序列时，将文件的UID与当前类的UID比对，如果匹配说明没问题，不匹配说明对象与类的版本不一致，可能类有了更新，就会使得反序列化失败；是版本匹配作用，一般使用自动生产的hash值作为版本号

评估：开销大，中间结果是在辅存，涉及到大量IO

2. Parcelable接口：Android提供的，可通过Intent和Binder传递

实现：implements Parcelable接口，实现相应打包和拆包方法，打包是 writeToParcel函数，拆包是使用：Parcelable.Creator<T> CREATOR的内部辅助对象；辅助字段：describeContent，返回对象的内容描述

Parcel对象是序列化后的数据封装，数据包，可在Binder媒介上传输 

评估：中间结果是在内存上的，方便后期写到socket或者binder，所以效率很高，

3. Binder是用内存实现的数据通道，使用虚拟设备并注册为/dev/binder来暴露接口

Binder不仅表示 数据channel，还指代一系列方便使用的工具类，AIDL接口定义语言：就是为了方便Binder使用，用binder实现接口，并统一 binder收发两端的协议接口

从上层而言，就是关注 数据的打包发送 和 接受拆包，以及定义数据交互协议的 aidl

Android 工具根据编写的aidl自动生成 对应的Java接口，接口implements了IInterface，IInterface类使用组合的方式，持有了Stub（extends Binder） 和 Stub#Proxy （implements AIDL的接口）两个内部类，Stub实现了Binder的收发数据功能，Proxy提供使用内部binder对象实现预定义的接口的功能；主要有 asInterface asBinder

Stub中实现功能的是onTransact的函数，code函数编码，reply数据包和data要发送的数据包

Proxy的实现的aidl接口函数，内部使用 binder.transact收发数据，用Parcelable 反序列化为对象或 序列化

### Bundle ：基于Parcel和Binder

很多时候我们不直接使用Binder aidl，而是使用Bundle，更加方便使用

Bundle实现了Parcelable接口

### SharePreference

目录： /data/data/package_name/shared_prefs/

使用xml文件约束的键值对，Android对其有一定的缓存策略，就是SharePreference的读写事务，面对高并发读写，会丢失数据，而且不适合大数据读写



### Messenger：使用Binder实现的工具：理解为简单数据包发送的Binder

使用：Service创建Messenger对象，在onBind回调中返回这个Messenger的底层Binder就好；客户端绑定服务端的Service，用服务端返回的Binder对象创建一个Messenger收发器辅助对象就好，打包实体为Message对象，setData使用的是Bundle；

使用Message对象创建方法为 Message.obtain()，说明也是复用了消息对象，对象池的思想

Messenger简化了使用Binder收发简单数据的使用门槛，但是其只适用于发送简单的数据，而且Messsenger是串行收发数据的方式，对于大量数据共享，就不适用

使用AIDL的方式使用Binder，定义接口，本地代理，更多的是一种架构上的完整性，代码层感知不到数据是来源于本地还是远程，是一种RPC的方式，来支持一种CORBA架构理念

### ContentProvider：使用Binder，理解为使用了标准化SQL接口的Binder

操作的接口为CRUD，要在Manefest注册，authorites 属性是自我标识，permission是访问者需要有的权限

content 是URI的协议头，content:authorities 就是Intent找到目标组件的URI地址

其CRUD方法是存在多线程并发访问的，所以CRUD方法的实现必须要处理竞争访问，线程同步的问题

Android的四大组件就是一整套接口，Android会为遵循接口的实体提供对应的服务，ContentProvider就是使用Binder为跨进程数据传输为基本功能保证内容提供的一项接口

### 场景选择

首先进行场景分类，分类指标：1. 是否跨线程 2. 是否有并发访问 3. 时延迟要求 4. 是否要持久化 5. 接口复杂度 6. 架构要求：CS架构，简单的共享内存，还是数据邮箱模型

根据各种工具的特点进行选取

## View 视图

### 1. view的位置

1. 基本参数

（top right bottom left ）clockwise， 相对坐标；使用相对坐标，有利于基于层级的视图树 计算相应的层级关系

width = left -right ; height = bottom - top

代码体现就是 getXXX()

2. 额外参数：x y 左上角的坐标，translationX  translationY，相对于父容器的便宜量

x = left + translationX

y = top + translationY

**在view平移的过程中，基本参数是不发生改变的，改变的是 额外参数**

### 2. MotionEvent TouchSlop VelocityTracker GestureDetector Scroller

MotionEvent（EVENT_type x y rawx rawy）（事件类型 相对于当前view左上角的坐标  相对于屏幕左上角的坐标）

是手指接触屏幕后产生的一系列相关的事件类型：ACTION_DOWN ACTION_MOVE ACTION_UP

点击事件：ACTION_DOWN ---ACTION_UP

滑动事件：ACTION_DOWN --- ACTION_MOVE n --- ACTION_UP

TouchSlop是一个系统常量，表示被识别为 滑动事件的最小距离，`ViewConfiguration.get(getContext()).getScaledTouchSlop()` 获得，dp为单位

VelocityTracker：计算滑动速度的工具类，输入event，输出速度，有正负表示方向

```kotlin
val tracker = VelocityTracker.obtain()
tracker.addMovement(event)

var interval = 1000// 事件间隔，单位ms
tracker.computeCurrentVelocity(interval)
val v_x = tracker.getXVelocity()
val v_y = tracker.getYVelocity() // 单位 px per interval ms
tracker.clear()
tracker.recycle()

```

GestureDetector: 手势检测工具类，输入event（在View.onTouchEvent接受所有event）输出 手势Listener的通知

```kotlin
// 创建GestureDetector对象，实现 OnGestureListener或OnDoubleTapListener接口里关注的事件
val detector = GestureDetector(this)
detector.setIsLongpressEnable(false) // 解决长按屏幕后无法拖动问题

//View#onTouchEvent中接受event
return detector.onTouchEvent(event)
```

Scroller：用于实现view弹性滑动；将View.scrollTo scrollBy瞬间滑动行为 平滑处理



### View 滑动的实现

View 滑动相关的属性：mScrollX：View左边缘和View内容左边缘 水平方向的距离；mScrollY：View上边缘与View内容上边缘 垂直方向的距离，而且是动态更新的

1. 通过View本身提供的 scrollTo/scrollBy 实现：计算mScrollX和mSrollY属性，View内部调用onScrollChanged后 invalidate
2. 使用动画给View添加平移效果：通过动画，改变translationX 和translationY的值，来平移View而实现，所以不能改变View的位置，移动的也只是影像；适用于交互少的控件
3. 改变View 的LayoutPramas 让View重新布局实现滑动；适用于滑动还有交互的控件

### View的用户事件传递机制

有很多InputDevice 比如mouse dpad keyboard touchscreen 不同的设备会有特别的事件，比如鼠标有HOVER事件

事件处理有很过，在View Class 搜索dispatch，就会有很多事件的处理，这里只关注Touch事件的处理，作为例子分析



View视图 是一个递归定义的树结构，事件流在视图树上的下沉流动 一般是从 rootView 到 叶子结点 或者 中间的事件拦截节点，而如果事件流动到叶子结点都没有被处理，就会冒泡，一路返回到rootView 或者其上层结构比如Window处理

主体流程：

1. 分发事件函数入口：dispatchMotionEvent
2. 首先判断是否拦截 onInterceptMotionEvent；ViewGroup默认不拦截任何事件；拦截针对事件序列的头事件：ACTION_DOWN，若拦截事件头，后续整个事件序列就无须重新进入 拦截 函数判断
3. 拦截：就会自己处理事件 onTouchEvent：View实现组件默认消耗 事件
4. 不拦截：遍历子节点，寻找符合条件的子节点 dispatchMotionEvent：事件位置在控件的区域范围内 组件是否正在动画 

主要就是这三个函数的大流程，配合上一些Listner，以及一些事件掩码 和处理的Flag的 工作的

·其他

1. View的OnTouchListener的onTouch优先级比 onTouchEvent的优先级高，且前者会掩盖后者

   1. ```kotlin
      //noinspection SimplifiableIfStatement
                  ListenerInfo li = mListenerInfo;
                  if (li != null && li.mOnTouchListener != null
                          && (mViewFlags & ENABLED_MASK) == ENABLED
                          && li.mOnTouchListener.onTouch(this, event)) {
                      result = true;
                  }
      
                  if (!result && onTouchEvent(event)) {
                      result = true;
                  }
      ```

2. mFistTouchTarget 是链表，记录响应事件，传递事件的链条的

3. onInterceptTouchEvent 只有ViewGroup的子类才有，因为到View的实现组件的节点，事件捕获过程一定会结束

### 滑动嵌套 滑动冲突

根本策略：根据xxx判断标准，来判断 事件交给哪一个Scroolable组件处理（哪一个Scrollable组件拦截事件）

标准：

1. 嵌套的Scrollable组件滑动方向不同：以方向为判断标准，水平还是垂直来确定给谁
2. 嵌套的Scrollable组件滑动方向相同：优先响应内部还是外部的偏好，一般优先响应内部，当内部滑动到底了，事件交给外部处理
3. 两种情况组合：处理方式也组合

处理：

1. 外部判断处理法：在Nested Scroller的Outter Scroller ViewGroup的onInterceptTouchEvent的 ACTION_MOVE 中判断是否拦截事件，比如ViewPager在deltaX > deltaY的时候 ViewPager拦截并处理事件，否则交给子view处理： 在事件下沉时判断拦截
2. 内部判断处理法：稍微麻烦，Outter Scrollable View不拦截任何事件，内部处理时，当不需要的事件时，不consume事件，让事件冒泡；实现：需要重写Inner View Element的dispatchTouchEvent，在ACTION_DOWN的时候 paren.requestDisallowInterceptTouchEvent(True)，在ACTION_MOVE时判断事件是否交给Outter Scrollable View处理，需要时parent.requestDisallowInterceptTouchEvent(False)，outter scrollable view需要重写onInterceptTouchEvent，保证不拦截ACTION_DOWN事件：在事件冒泡时判断消费

相关函数：

```kotlin
requestParentDisallowInterceptTouchEvent()：让Parent不拦截ACTION_DOWN，使得事件序列能到达当前下层View
onInterceptTouchEvent()
postInvalidOnAnimation()
```

## 第四章 View的工作原理

主要介绍三大流程：Measure Layout Draw

遍历试图实现，所以三个流程入口为：`performTraversals`，会依次调用：`performMeasure performLayout performDraw`

API 设计安排：①measure layout draw 函数的内容多数是 缓存优化，避免EXACTLY尺寸重measure 等类型的优化操作② onXXX 给继承重写VIew的开发者覆盖内容的接口 ③ performXXXX

### 1. Measure：刷新大小信息

MeasureSpec：测量说明书，保存测量中间内容的数据结构

32bit Java int数据类型表示，高2位表示SpecMode 低30位表示数据 SpecSize

SpecMode：UNSPECIFIED，EXACTLY（match_parent =父元素内部size；或者直接填写数值大小 ），AT_MOST（wrap_content <= 父元素内部size）

根据父容器的MeasureSpec + View自身的LayoutParams +父 View的Padding 和 自身的Margin=> View的MeasureSpec：综合配置信息和父容器信息，以便后面计算，将数值刷新存储到View的对应属性字段中

执行流程：①当元素为View时：`measure(widthMeasureSpec,heigthMeasureSpec)`通过measure来测量自身，代码主要是处理MeasureSpec缓存，当size没变时候直接访问缓存，测量计算的主体逻辑在View.onMeasure；②当ViewGroup，负责自身，同事遍历子View的measure，ViewGroup没有重写View的onMeasure，但提供了measureChidren方法，遍历子元素数组，使用measureChild：获得child的LayoutParams的width和height，父ViewGroup的MeasureSpec以及Pading，然后调用View的measure方法，具体的ViewGroup实现类根据自身的布局特性来实现onMeasure方法

注意：

1. 系统measure过程可能会多次执行某些view的measure才能确定对应的大小，所以在onMeasure函数里有时不能拿到准确的size，所以在完全完成measure阶段后的layout阶段的onLayout获得View size相关的属性才是准确的
2. Activity的生命周期与View的流程也不是同步的，在Activity生命周期方法中获得View的尺寸不一定准确：解决方案
   1. onWindowFocusChanged：当activity的窗口获得焦点和失去焦点时会被调用，比较频繁，在此方法里调用`view.getMeasureWidth/Height（）`可以获得准确的宽高信息，但是调用频繁会有较高的性能成本
   2. `view.post(runnable)`：使用post讲一个runnable投递到消息队列的队尾，然后等待Looper执行这个runnable的时候，view是已经初始化好的状态
   3. ViewTreeObserver：onGlobalLayoutListener回调接口就可以保证代码在 layout阶段执行，此时measure已经完成了
   4. `view.measure(widthMeasureSpec, heightMeasureSpec)`：手动调用view.measure来获得size信息；但是这个方法要传入两个 参数，给出两个measurespec时，如果两个MeasureSpec依赖于父容器的大小，就不可行，当给出LayoutParams给出明确的宽高时或者 wrap_content时，才可能计算出这个view的大小



### 2. Layout 流程：刷新位置信息

根据布局约束 和 第一阶段 measure 的信息，计算出位置信息并填充到 view的相应属性字段

流程：ViewGroup位置信息确定后，就会在onLayout中遍历子元素并调用其layout方法，在layout判断layoutmode修改的话调用视图元素的onLayout，以及通知onLayoutListener；View和ViewGroup都没有实现onLayout，具体实现类必须要实现

`onLayout(boolean changed, int left top right bottom)` 遍历子元素child，访问measure信息 相应的Margin 和 约束字段计算出 left top right bottom 调用child的layout，可参考LinearLayout的onLayout代码

注意：

1. getHeight/width 实现是使用 measure阶段的mBottom mTop / mLeft mRight 作差得出，也就是认为 测量宽高就是View的宽高



### 3. Draw流程：刷新屏幕显示内容

流程：

1. 绘制背景 `drawBackground`
2. 绘制content，`onDraw` <== 自定义View绘制时主要作用的地方
3. 绘制children，`dispatch`
4. 绘制fading edges 有必要的话`drawAutofillHighlight`
5. 绘制overlay，如果有的话`mOverlay.getOverlayView().dispatchDraw(canvas)`
6. 绘制装饰`onDrawForeground`
7. 绘制focus效果`drawDefaultFocusHighlight`

### 自定义View

主动看以及实现的布局的源码较好

**继承View并重写onDraw**

1.  稍微修改绘制内容，需要自己处理pading
2. 添加属性资源 让xml可用：①在values创建 自定义属性的xml文件，如attr_xx_view.xml <resources <declare-styleable name="CircleView" <attr name="xxx" formate = "xxx" \>\>\> ②：在CircleView的构造函数中使用从AttributeSet获得对应的TypedArray，再获得对应的属性值`context.obatainStyledAttributes(attrs,R.styleable.CircleView).getColor(R.styleable.CircleView_circle_color,Color.RED)`③ 在布局文件中，使用app xmlns 设置对应的属性
3. 要实现onMeasure， setMeasureDimension设置保存结果

**继承ViewLayout**：重点是layout 以及滑动冲突的解决 对于效果层叠的layout onMeasure也是需要关注的

1. onMeasure：在计算和setMeasureDimension前，必须measureChildren，递归调用子类的测量并获得结果
2. onLayout：遍历子元素，当不是GONE的时候，对被遍历的子View调用其 layout`childView.layout()`
3. 解决滑动冲突，滑动行为等



## 第5章 RemoteView 

RemoteView提供远程更新 操作 View的功能接口

RemoteView：通知栏 小部件都属于RemoteView，这二者界面都在SystemServer进程，需要跨进程措施

1. 通知栏：NotificationManager 来 notify的
2. 桌面小组件：AppWidgetProvider 广播实现的，本质是BroadcastReceiver

RemoteView提供了一系列set方法，但是比起View的是有限的，而且也只支持有限的View类型

### Notification

使用NotificationManager.notify 配合Notification RemoteView PendingIntent来操作

### AppWidgetProvider

1. 提供页面布局文件 /res/layout/
2. 提供小组件配置信息xml /res/xml/appwidget_provider/info.xml <appwidget-provider <initialLayout minHeight minWidth updatePeriodMills 等等参数
3. 完善实现类代码 extends AppWidgetProvider，主要是 onReceive 和 onUpdate函数此外有onEnable onDisable onDelete 通知可用
4. 在AndroidManifest声明Receiver，在meta-data 指定resource为2的配置文件

### PendingIntent

PendingIntent是在将来的某个时刻发生，Intent是立马发生

典型使用场景是 给RemoteView添加点击事件：应为无法为跨进程的View setOnClickListener设置点击事件，所以必须依赖于广播的数据，Intent为载体

支持三种待定意图：启动Activity 启动Service 发送广播

getActivity(Context, requestCode, Intent, Flags) 相当于 context.startActivity(Intent)

getService(Context, requestCode, Intent, Flags) 相当于 contex.startService(Intent)

getBroadcast(Context, requestCode, Intent, Flags) 相当于 context.sendbroadcast(Intent)

requestCode：PendingIntent发送发的请求码，一般来说非0就可以

Flags：PendingIntent的匹配规则：什么情况下PendingIntent是相同的：如果RequestCode 和 被包装的Intent 相同，那么这两个PendingIntent也相同

​	Intent相同的规则：如果两个Intent的ComponentName 和 intent-filter相同，那么这两个Intent就相同；注意 extra的数据不参与匹配过程

> 1. FLAG_ONE_SHOT：当前的PI只能被使用一次，然后就会被自动cancel，如果后续有相同的PI,那么他们的send方法调用会失败；通知栏表现为 同类的通知只能使用一次
> 2. FLAG_NO_CREATE：当前的PI不会主动创建，如果当前PI之前不存在，getActivty、getService getBroadcast就会返回NULL，不常用
> 3. FLAG_CANCEL_CURRENT：如果当前的PI已经存在，那么他们都会被Cancel，然后系统重新创建一个新的PendingIntent；对通知栏来说，被cancel的消息就会无法打开
> 4. FLAG_UPDATE_CURRENT：如果当前PendingIntent已经存在，那么他们都会被更新，即他们的extra会替换成最新的

### RemoteViews的实现机制

RemoteView的manager（如NotificationManager AppWidgetManager）将本地调用的 set的一些列操作，包装为 Action 动作事件流，通过binder，流向远程进程的RemoteView的Service（如NotificationManagerService AppWidgetService），然后用LayoutInflator 布局xml生产对象，再使用performApply解析执行ReflactionAction，反射执行或者简单执行相应的 动作行为



### RemoteViews可以用于跨进程的View更新

B activity构造RemoteViews，将RemoteViews放在Intent里用sentBroadcast的方式 发送给其他进程的A activity的动态注册的BroadcasReceiver，

A在BroadcastReceiver里 从Intent的extra取出RemoteView，调用RemoteView的apply和reapply将获得View对象，然后在B的是视图树上 addView(remoteView) 来达到刷新页面的效果

**RemoteViews的本质实现是：将skia的视图封装的View对象在不同进程都创建多个访问句柄，这多个访问句柄的操作都是同一个 skia视图的操作**

## 第6章 Drawable 可绘制的

用途：作为ImageView的内容 或者 View的背景

系统默认提供的：

### 自定义Drawable

主要是实现draw的override，绘制对应的Canvas，但是自定义的无法在xml中使用

## 第7章 Android动画深入分析

1. View动画：直接对View的显示区域的内容做 图形变换，是一种渐变式动画
2. 帧动画：播放一组图片而来的动画效果
3. 属性动画：通过动态的改变（插值，平滑）View对象的属性值，在vsync信号来的时候，渲染引擎读取View对象的属性值，来将效果绘制出来

### View动画

四种View动画对应四个Animation的子类：TranslateAnimation ScaleAnimation RotateAnimation AlphaAnimation 分别对应四个可配置的xml标签：本质是图形变换应用到 位图矩阵

xml关注的属性：

1. 指定的插值器：`android:interpolator`
2. 是否共享插值器：`android:shareInterpolator` 几个动画是不是共用一个动画速度，效果表现出来就是 效果渐变快慢是不是一致

scale ：分别指定水平和垂直缩放的起始和结束值

translate：分别指定x轴和y轴的平移起始位置和结束位置

rotate：指定轴心的坐标和起始以及结束的角度

alpha：起始以及结束的透明度

用set为root xml组合多个View动画

**自定义View动画**：继承Animation类，在initialize初始化一些资源 配置工作，在applyTransformation进行一些矩阵操作，一般用Camera简化矩阵变换，将Transform的matrix作用效果

**View动画特殊使用场景**：

1. ViewGroup的子元素出场效果：LayoutAnimation 使用 xml配置：\<layoutAnimation 使用delay指定延时 animationOrder指定动画顺序 animation则是指向要引用的View动画xml
2. Activity的转场动画：使用overriderPendingTransition(R.anim.enter_anim, R.anim.exit_anim) ，需要在startActivity 或者 finish 之后才能生效
3. Fragment转场动画：使用FragmentTransaction.setCustomAnimations() 添加切换动画



### 帧动画

使用xml描述 animation-list 为root，内涵多个 item，应用多个drawable

使用简单，容易引起OOM



### 属性动画：渐变修改属性值达到动画效果，可以对任意属性做动画

命名方式为 Animator与View动画的api做区分

建议使用代码操作属性动画：因为代码可以方便的获得当前属性值

**插值器 Interpolator**：对起始值和结尾值 做中间序列的平滑处理

**估值器 Evaluator**：根据属性改变的百分比计算改变后的属性值（不一样属性值的空间不一样，比如透明度和颜色就不一样）

**属性动画监听器**：start end cancel repeat 四种状态的监听

**注意**：属性动画要求动画作用的对象提供该属性的 set 和 get 访问器：get获得属性初值，set刷新属性进行动画

getWidth是view的方法，但是view没有提供setWidth方法，只是其子类TextView和Botton实现了setWidth方法，但是这个width属性对应的是最大宽度，`android:width`,而非布局宽度 `layout_width`

要解决上述问题：将对应属性的 setter和getter暴露出来就好

1. 给对象添加getter setter ，有权限的话
2. 继承这个类，添加setter getter
3. 使用ValueAnimation，监听动画过程，自己实现属性的改变

**原理**：属性动画运行在Looper的线程中，代码最终会调用AnimationHandler的start方法，而这个AnimationHandler是一个Runnable，调用到了run方法，在调用了一些jni之后，会调用ValueAnimator的doAnimationFrame方法，内调用animationFrame，最终会通过反射来set属性的值，如果属性没有初始值，就会调用反射调用get

### 使用动画注意事项

1. 帧动画容易导致OOM问题

2. 属性动画有一类无限循环，必须在activity退出时即使停止，否则会导致activity无法释放导致内存泄露；但是View动画不会导致类似问题（内部处理了，或者本来操作的就skia显示效果，而与View对象基本没有关系）

3. View动画是对View的影像做动画，在View动画完成后，setVisibility无法生效，可以view.clearAnimation清除View动画便可解决

4. 动画过程尽量使用 dp而不是用px，px会导致不同设备效果不一致

5. 动画元素的交互：在3.0后，属性动画位置与事件位置是一致的，VIew动画的事件位置在原位置

6. 硬件加速，使用动画的过程 开启硬件加速，会提高动画的流程性

   > 在application activity windows view 各级别都支持硬件加速
   >
   > 一般在重点View上开启：
   >
   > 1. 在动画开始前和结束后 `View.setLayerType(View.LAYER_TYPE_HARDWARE, null) View.setLayerType(View.LAYER_TYPE_NONE, null)`
   > 2. sdk14以上可以：`myView.animate().translationX(150).withLayer().start()`





## 第8章 Window和WindowManager

1. Window是View的上下文，所有的View相关的操作的具体实现都是此处，具体是WWindowManagerImpl，Impl又具体依赖于WindowManagerGlobal，对于需要通知WindowManagerService的行为通过 基于Binder的WindowSession的来处理实现

2. 相关的数据有：拥有的View数组，remoteView数组，异步删除View的dyingView的数组：就是各种状态的View对象；基于Handler的线程 动画处理，View.post 任务等，一些相关的LayoutParams

3. 在View的相应作用域内，所用的操作在本进程的操作，通过在Window的ViewRootImpl去具体操作完成的，View必须依托于Window才会有作用，不能单独存在；而在系统全局的作用域内，最小的操作单位为Window，而Window的布局等相关管理是WindowManagerService来完成；
4. Activity ActivityManager ActivityManagerService是生命周期，状态机的抽象，是用户主观的一个活动的抽象，这个activity的概念是面向用户而设计的，将用户的互动 映射 到activity的生命周期代码 映射 到线程，任务以及资源上下文；而Window WindowManager WindowManangerService 是视窗的概念，是View的集合，是可视化的外层包装概念； 将Activity与Window 一比一的绑定映射，可以让视图获得对应用户活动状态变换的感知能力，且将用户活动单位化，一个一个的Activity，对应用户脑中一个一个任务活动的概念，在软件工程角度，使得其模块化，适合于多个开发者合作和任务分工：：：：只要是视图的操作必定和Window相关，只要是用户活动概念变化，一定只和Activity相关；；；；然而，**一个Window 各个View的相关操作都是和Context绑定的，所以对应Activity而言，Window是Context上下文的一种视图资源而已**，与activity绑定的window往往被activity视为content，所有一些content命名的api

### Window 的类型

1. 应用Window(1-99)如Activity的Window：作用域是Activity，是activity 显示View对象集合的句柄，PhoneWindow DecorView为root的视图树
2. 子Window(1000-1999) 如Dialog的Window：作用域是父window，与activity的Window使用基本类型，但是必须依赖于父Window
3. 系统Window(2000-2999) 如Toast的Window，系统状态栏：作用域是system，基本上是系统常驻的

系统window需要权限申请



## 第9章 四大组件的工作过程

### Activity

Activity.startActivity - startActivityForResult - Instrumentation.execStartActivity(this, mMainThread.getApplicationThread, mToken, this, intent, requestCode, options) - ActivityManagerNative.getDefult.startActivty --binder aidl-- ActivityManagerService.startActivity(IApplicationThread caller, String callingPackage, Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode, int startFlags, ProfilerInfo info, Bundle options) - startActivityAsUser - ActivityStackSupervisor.startActivityMayWait() - startActivtyLocked - startActivityUncheckedLocked - ActivityStack.resumeTopActivitiesLocked - resumeTopActivityInnerLocked - ActivityStackSupervisor.startSpecificActivityLocked( ActivityRecord r, boolean andResume, boolean checkConfig ) - realStartActivityLocked (使用app.thread.scheduleLaunchActivity) --binder-- ApplicationThread.scheduleLaunchActivity 使用 主线程Handler H来发送H.LAUNCH_ACTIVITY 消息 - 对应线程的处理ActivityThread.handleLaunchActivity() - performLaunchActivity() 完成Activity对象的创建和启动过程以及应用上下文等所有资源的创建 - handleResumeActivity来调用 onResume等生命周期等方法 

简化 Activity - Instrumentation - ActivityManagerNative - ActivityManagerService(全局的包校验 权限校验，AndroidManifest读取，activity搜索和路由转发到对应应用进程等功能) - ActivityStackSupervisor - ActivityStack - 其他应用进程或本应用进程 ApplicationThread具体执行activity操作

简化：三个过程：Activity请求的过程 - AMS响应 校验 路由转发的过程 - ApplicationThread创建自身上下文，执行的创建或生命周期的过程 ====》请求 数据准备路由转发 上下文创建-执行

1. app.thread 类型为IApplicationThread（IBinder），提供了大量Activty启动停止的接口函数，同时作为Binder接口，具体实现IApplicationThread的是ActivityThread的内部类ApplicationThread，同时ApplicationThread继承了ApplicationThreadNative，ApplicationThreadNative内部的ApplicationThreadProxy将RPC操作转发到AMS

2. ActivityThread 的 performLaunchActivity：

   > 1. 从ActivityClientRecord获取待启动的Activity的组件信息 
   > 2. 通过Instrumentation的newActivity使用类加载器创建Activity对象
   > 3. 使用LoadedApk的makeApplication方法尝试创建Application对象
   > 4. 创建ContextImpl并通过Activity的attach方法完成一些重要数据的初始化和关联：Context与activity的关联，activity与Window的关联
   > 5. 调用mInstumentation.callActivityOnCreate 来调用 activity的onCreate，通知业务层 Activity已经成功创建



### Service 的工作过程

创建过程：  ContextWrapper.startService() - mBase/ContextImpl.startService() - startServiceCommon() - ActivityManagerNative.getDefault().startService() --Binder-- ActivityManagerService.startService() - synchronize{ ActiveServices,startServiceLocked() } - startServiceInnerLocked() - bringUpServiceLocked(ServiceRecord , Intent.getFlags(), boolean callerFG, false) - realStartServiceLocked(ServiceRecord r, ProcessRecord app, boolean execInFg) --binder-- app.thread.scheduleCreateService创建对象并调用其onCreate；sendServiceArgsLocked调用service的其他方法如onStartCommand ，都是进程间通信 - ApplicationThread.scheduleCreateService{ 对H发送H.CREATE_SERVICE sendMessage } - H handler handleCreateService() { 类加载器创建Service对象，创建Application并调用其onCreate，创建ContextImpl并通过Service的attach 与之关联，调用Service的onCreate方法并将Service对象存储到ActivityThread的列表中 }

绑定过程：ContextWrapper.bindService() - ContenxtImpl.bindServiceCommon() - 获得ServiceConnection LoadedApk.getServiceDispathcer(){从 mServices(Map of ServiceConnect to ServiceDispatcher)} - ActivityManagerNative.getDefault().bindService() --binder(可能Service组件处在其他进程)-- AMS.bindService() - mService(ActiveServices).bindServiceLocked() - bringUpServiceLocked() - realStartServiceLocked() {使用ApplicationThread来完成Service的创建和生命周期函数调用} - ActiveServices.requestServiceBringUpLocked() - ApplicationThread.scheduleBindService() --Handler - H.BIND_SERVICE - ActivityThread.handleBindService() { 根据Service的token取出Service对象，onBind返回一个Binder } - 通知已经绑定完成ActivityManagerNative.getDefault().publishService() - ActiveServices.publishServiceLocked() {ConnectionRecord.ServiceConnection.conneted(ComponentName, Service的Binder)}



主体逻辑就是：请求 - Binder（ActivityManagerNative.getDefault()）路由来处理不在同一个进程的情况 - Handler(H.MESSAGE_TYPE)分发对应线程处理相关任务 - 完成

1. 关注 ServiceConnection：维护远程Service的映射关系；ServiceDispatcher 维护 Service所在的进程（Binder）的映射关系



### BroadcastReceiver的工作过程

广播的基本使用：

1. 继承BroadcastReceiver并实现其 onReceive方法
2. 在AndroidManifest静态注册 或者 使用ContextWrapper.registerReceiver动态注册
3. 使用sendBroadcast 发送广播

注意：广播可能处在其他进程，所有也会使用到Binder的跨进程通信

### 广播的动态注册过程

ContextWrapper.registerReceiver() - ContextImpl.registerReceiverInternal() - ActivityManagerNative.getDefault().registerReceiver( ApplicationThread, PackageName, IIntentReceiver(ReceiverDispather), String broadCastPermission, userID ) - AMS.registerReceiver() { 将远程的InnerReceiver对象以及IntentFilter对象存储起来，注册就完成了 }



### 广播的发送和接收

send发送Intent时，AMS根据IntentFilter查找出匹配的接收者，并发送给他们处理

广播的类型：普通广播，有序广播，粘性广播

过程：ContextWrapper.sendBroadcast() - ContextImpl.sendBroadcast() - AMN.broadcastIntent() --binder -- AMS.broadcastIntent() - broadcastIntentLocked(){ 将匹配Intent-Filter的接受者添加到 BroadcastQueue（BQ），使用BQ的相关函数发送Intent} - BQ.scheduleBroadcastsLocked() --使用mHandler切换线程 -- BQ.processNextBroadcast(){ 对ParallelBroadcasts遍历deliverToRegisteredReceiverLocked() } - performReceiverLocked(ProcessRecord app, IIntentReceiver) - app.thread.scheduleRegisteredReceiver() - IIntentReceiver.performReceive() - LoadedApk.ReceiverDispather.performReceiver() - Args.sendFinished(AMN) { Args实现了Runnable，使用AMN-AMS跨进程后，交给ActivityThread H来跨线程 调动 BroadcastReceiver.onReceive()}



### ContentProvider 的工作过程

1. ContentProvider 的 onCreate要先于 Application的 onCreate执行 意味着，ContentProvider是没有AMN ContextImpl的上下文环境的
2. 当启动一个应用的进程时，会首先进入 ActivityThread的main入口方法，ActivityThread创建ApplicationThread Binder与AMS建立通信渠道后attach调用 AMS.attachApplication 使用bindApplication 跨进程调用到ActivityThread，完成握手，让后会先加载ContentProvider，然后再makeApplication，再调用Application的onCreate，意味着，在Application的onCreate就能访问ContentProvider



新开进程：

Process::start - main(){1. 准备跨线程的MainLooper 2. 创建ActivityThread并attach 3. AsynTask.init 4. 开启主循环 Looper.loop()}

分析 ActivityThread.attach() :ActivityManager.getService().attachApplication() -> ActivityThread.handleBindApplication(){ 1. 创建ContextImpl Instrumentation 2. 创建Application对象 3. 启动当前进程的ContentProvider并onCreate 4. 调用Application的onCreate}

### 总结

使用XXXRecord 绑定对应组件 ApplicationThread ActivityThread ComponentName PackageName Binder Token ProcessState 之间的映射关系，使用XXXDispatcher 建立 本地组件 与 其他进程的Binder的映射关系，使用Handler H在指定线程分发，使用AMN-AMS在指定进程分发



## Android的消息机制



MessageQueue#enqueueMessage(): 单链表插入操作

MessageQueue#next()：无限循环，如果队列为空，就一致阻塞；当有新消息到来时，会返回这条消息并从单链表中移除

### Looper

构造方法中创建 MessageQueue，让后绑定当前线程

为线程创建looper

```java
Looper.prepare();// Looper.prepareMainLooper()
Handler handler = new Handler();
Looper.loop();

Looper.quit();
//Looper.quitSafty();处理完所有消息后才退出
```

Loop():

死循环，next返回Null时才退出死循环，当quit 或 quitSafty调用时，next才会返回NULL

处理：`msg.target.dispatchMessage(msg)`

Target 是发送这条消息的Handler对象

**Looper.loop()在那一个线程中调用，那么所有Message的处理便在哪一个线程，当我们希望Message的消息处理处理在哪一个线程，就将其提交给绑定了对应Looer的Handler便好**

所以**Handler能跨线程的原理**是：当线程threadA创建时，初始化ThreadALooper将自己存在ThreadLocal中，并loop()死循环，在threadA线程创建mHandler对象时候，会从ThreadLocal中取出当前线程绑定的threadALooper，并建立引用；mHandler在threadX子线程 发送Message，message.target 标识发送的mHandler，将Message enquequeMessage 互斥访问threadALooper的MessageQueue中，当threadALooper的loop() 取出Message，ThreadA 调用 的loop() 调用 msg.target.dispatchMessage()，就达到了在ThreadA中调用mHandler实现的dispatchMessage()的目的 **简而言之：Handler绑定的Looper是在哪一个线程，那么这个Handler发送的消息的处理 dispatchMessage就在Looper所在的线程执行**

若果想在子线程创建的Handler对象的dispatchMessage能在主线中执行的话，必须在Handler对象构建时 传入主线程的Looper ,getMainLooper完成

### Handler

工作为：消息发送和接受

发送：post - sendMessage - sendMessageDelayed - sendMessageAtTime - enqueueMessage，如果要延后执行，也就是异步执行，将会Message.setAsynchronous()

处理：msg.target.dispatchMessage() {检查msg的callback并handleCallback(msg)，即msg.callback.run()，否则使用Handler构造时传入的Callback interface来处理，若没有Callback则使用Handler的handleMessage()函数处理，即我们派生Handler时override的handleMessage()}，Callback interface也是单个函数的接口：handleMessage这个函数。也就是说，我们实现处理可以通过传个Handler一个 handleMessage Callback interface实现，也可以通过子类重载handleMessage

 Handler还有可以指定Looper来初始化

### Message

Message对象由于要经常创建使用，一般不使用构造，而是使用 obtain来从对象池里获得对象，以复用消息对象，用完recycle



### 主线程 MainLooer 和 H handler

是在ActivityThread 的 main函数就启动的，是每个进程创建时，默认主线程入口main函数



## 第11章 Android 线程和线程池

1. Android主线程用于界面相关的消息响应：页面渲染 用户事件等；子线程则用于IO和计算等行为
2. Android对执行上下文有不同的用户接口：AsyncTask IntentService HandlerThread等
3. AsyncTask封装线程池和Handler：主要场景是在子线程获得数据后更新UI
4. IntentService是一个Service，内部使用了HandlerThread，执行玩会自动退出，免除开发者要处理的一些业务无关的准备和善后操作：突出优点是IntentService是一个服务，Android不容易杀死他，而一个线程没有处在有活动的四大组件进程里，就很容易被杀死
5. HandlerThread封装了Handler，可以方便的跨线程通信
6. 线程池：用户代码是提交一个一个的任务实体放在一个执行上下文去执行，往往任务的执行时间是片段，如果有一个零散的任务就创建一个线程，其创建成本就会很高，也会意味着更多的调度，且难以对不同特点的任务经常分类处理，比如IO繁重 计算繁重的线程 零碎的小任务 长时间的任务，都适合不同线程管理方式，线程池通过池化的思想来复用 统一管理 分类处理 来优化线程的使用
7. Android Java开发者基于Jvm，来源于是Java线程池，即Executor来派生特定类型的线程池

### AsyncTask

轻量级的异步任务类，可以在线程池执行后台任务，然后将执行的 进度 最终结果传递给主调线程，用于更新UI；封装了Handler，便于跨进程通信；当AsyncTask不适用与长耗时任务：使用的是线程池模型：

`pulic abstract class AsyncTask<Params, Progress, Result>` Params：传入的参数，Progress：进度，Result：处理的结果

四个核心方法：

`onPreExecute()`：主线程调用，处理准备工作

`doInBackground(Params ...params)`：线程池中执行，params表示输入参数，此方法中使用 `publishProgress()`更新任务进度，会调用`onPregressUpdate(Progress ...valus)`；而且也要返回 最终结果给 `onPostExecute（）` 返回结果

`onProgressUpdate(Progress...values)`：主线程中执行，被 publishProgress触发

`onPostExecute(Result result)`：主线程执行，doInBackground调用结束，返回值时触发



使用：

```java
private class DownLoadFileTask extends AsyncTask<URL, Integer, Long> {
  protected void onPreExecute(){
    showDialog("Start Download");
  }
  
  protected Long doInBackground(URL ...urls){
    publishProgress(Integer num);
    
    return totalSize;
  }
  
  protected void onProgressUpdate(Integer... progress){
    setProgressPercentage(progress[0]); // 更新UI
  }
  
  protected void onPostExecute(Long result){
    showDialog("Downloaded" + result + "bytes");//更新UI
  }
}

protected void download(){
  new DownLoadFileTask().execute(url1, url2, url3);// 同时发起三个异步任务
}
```

限制：

1. AsyncTask类必须在主线程中加载，意味着第一次访问AsyncTask必须在主线程 --- 在ActivityThread的main函数中，AsyncTask.init() 调用了，就满足了上文的条件，线程池必须在主线程调用下创建
2. AsyncTask对象必须在主线程创建
3. execute必须在UI线程调用
4. 不要在程序中直接调用上述的4个函数
5. 一个AsyncTask对象只能执行一次，
6. 在Android 1.6之前，AsyncTask是串行执行，之后改为线程池处理并行任务，Android 3.0 之后为避免并发错误，又改为一个线程来串行执行，但是可以 asynctask.executeOnExecutor() 传入Executor线程池来并行执行；execute会调用executeOnExecutor但传入的是默认的串行线程池

原理：

用FutureTask封装Params，提交给SerialExecutor线程池用于任务的排队，THREAD_POOL_EXECUTOR用于真正的执行任务，sHandler用于在主线程中处理



### HandlerThread

继承了Thread，在run时创建了Thread绑定的Looper，并loop()循环起来

外接需要通过handler的方式通知HandlerThread来执行任务，不同任务的处理需要在handler dispatchMessage函数的支持

主要结束后要 quit 或 quitSafty



### IntentService

继承了Service，并且是抽象类，需要派生子类来明确任务

其优先级比一般的Thread要高很多，且不容易被系统杀死

原理：创建一个HandlerThread，并用对应HandlerThread的looper来创建一个mServiceHandler；通过mServiceHandler发送的消息的处理（mServiceHandler.dispatchMessage()）都将在HandlerThread的Looper.loop() 即HandlerThread的线程中执行

每次启动IntentService，onStartCommand都会调用一次，并处理Intent：处理就是调用onStart，使用Message包装传来的Intent，用mServiceHandler发送，使其在HandlerThread线程中处理，使用的是mServiceHandler的dispatchMessage调用onHandleIntent，当onHandleIntent执行完成后，会调用stopSelf(int startId)来尝试停止服务（不用stopSelf()是为了等所有的消息都处理完才会停止）

主要：

1. 由于Looper是顺序处理消息的，所有IntentService表现出来的也就是顺序处理任务

使用：继承IntentService，实现其onNewIntent 函数，重载onDestroy；使用startService(Intent) 触发任务

```java
public class LocalIntentService extends IntentService{
  override onNewIntent()
}

Intent servive = new Intent(this, LocalIntentService.class);
service.putExtra(...);
startService(service);

```

### Android 中的线程池

线程池优点：

1. 重用线程，避免创建和销毁的开销
2. 有效控制最大并发数，避免大量线程抢占资源而导致阻塞
3. 能对线程进行管理，提供定时执行，间隔repeat执行的能力

来源于Java的Executor接口，真正实现是ThreadPoolExecutor



#### ThreadPoolExecutor

```java
public ThreadPoolExecutor(
int corePoolSize, // 核心线程数，核心线程一般会一直存活，即使处于限制，映射为CPU 核数
int maximunPoolSize, // 线程池的最大线程数，当存活线程数量达到最大值，后面的任务会被阻塞
long keepAliveTime, // 非核心线程存活超时时长，超过会被回收
TimeUnit unit, // 上一个参数的单位，
BlockingQueue<Runnable> workQueue, // 线程池的任务队列，execute提交的任务Runnable对象会存储在此
ThreadFactory threadFactory // 单函数接口 newThread() ,用于创建新线程
)
```



#### 线程池分类：已经指定了部分参数的ThreadPoolExecutor

1. FixedThreadPool 

   newFixedThreadPool()创建，线程数量固定，只有核心线程，空闲不回收，所有线程活动时新任务处于等待，没有超时机制，任务队列没有大小限制，LinkedBlockingQueue\<Runnable\>

   特点：核心线程不回收，所以响应速度快

2. CachedThreadPool

   newCachedThreadPool()创建，线程数量不固定，线程数量范围 0 - Integet.MAX_VALUE，无核心线程，有60s超时回收，SynchronousQueue\<Runnable\>（由于马上创建线程，所以是个空集合）

   特点：没有任务时，也没有线程，几乎不占用系统资源

3. ScheduledThreadPool

   newScheduledThreadPool()创建，核心线程固定，非核心线程没有限制，非核心线程闲置马上回收，DelayedWorkQueue

   主要用于执行定时任务和周期重复任务

4. SingleThreadExecutor

   newSingleThreadExecutor() 创建，一个核心线程，所有任务按顺序执行，不需要有线程同步



## 第12章 图片加载 缓存策略 列表优化

### Bitmap加载

BitmapFactory # decodeFile decodeResource decodeStream decodeByteArray ：分别表示不同的数据源

高效加载：按需加载 使用BitmapFactory.Options来 采样 和 裁剪

inSampleSize：规则 小于1时按等于1处理，inSampleSize * inSampleSize 像素采样为 1 个像素，总像素数为 源像素数除以 inSampleSize 的平方

 操作流程：

1. BitmapFactory.Options inJustDecodeBounds 设置为true，解码图片的摘要信息
2. 从BitmapFactory.Options 读取图片信息 outHeight，outWidth
3. 结合信息 计算出 inSampleSize值
4. 将BitmapFactory.Options inJustDecodeBounds 设为false，解码图片为bitmap

### 缓存策略

缓存算法：LRU least recently used：最近最少使用，当缓存满时，优先淘汰最近最少使用的缓存Item

有LruCache 实现内存缓存，DiskLruCache实现辅存缓存；一般使用两者结合为 二级缓存



**LruCache**

使用support-v4包的LruCache

泛型类，使用LinkedHashMap以强引用的方式存储外接缓存Item，提供了get put方法

使用：用匿名内部类，重新 sizeOf方法：确定Item的大小，重新entryRemove解决复杂的Item缓存移除



**DiskLruCache**

磁盘缓存，不属于Android SDK，需要手动导入

open用于指定缓存的文件目录和和缓存总大小MB 单点最大的缓存个数

缓存添加Editor：edit(key) 返回Editor对象，newOutputStream流对象，commit() 提交写入操作

缓存查找Snapshot：get(key) 返回SnapShot对象，getInputStream流对象

remove() delete()



### 第13章 其他技术

**使用CrashHandler来抓取应用的Crash信息**

1. 实现CrashHandler implements UncaughtExceptionHandler 实现 uncaughtException(Thread thread, Throwable ex)
2. 在Application初始时 onCreate中，调用Thread.setDefaultUncaughtExceptionHandler(mCrashHandler);

**multidex 解决方法数越界**

Android 单 dex文件最大包含 65536个方法，包含AndroidFramework jar 应用代码的所有方法，超过这个数后编译会报错：method ID not in 65536

解决思路：将单个Dex文件拆分为多Dex文件，google提供的 multidex，启动速度会降低：需要加载多个dex文件

**动态加载技术**：不发布应用更新某些模块 插件化

**反编译**：



## 第14章 JNI NDK native编程

JNI：JavaNativeInterface

NDK：Native Development Kit：使用开源C/C++库；so反编译困难，安全；便于平台移植；提高某些执行效率

### JNI

1. 使用native标记函数
2. 编译java文件获得class文件，然后用javah反编译出接口，实现cpp文件
3. 编译
4. 使用System.loadLibray() 加载

### NDK

1. 配置NDK环境

### 第15章 Android性能优化



**布局优化**

减少视图树层级

include标签：加载指定的布局文件，支持；layout_xx属性，以及id属性覆盖指定布局文件的根id

merge标签：一般和include搭配减少布局的层级：直接将merge内的元素include到所在的 布局内

ViewStub

继承了View，非常轻量级，宽高都为0，本身不参与任何布局和绘制，ViewStub的意义在于按需加载所需的布局文件

场景：很多布局文件在正常情况下不会显示，比如网络异常界面，这个时候没必要在整个界面初始化的时候将其加载进来，通过ViewStub做到在使用时再加载，提高程序初始化时的性能；

inflateId 属性指向布局的Id，在代码中 ViewStub.inflate() 或者设置setVisibility时才会使用inflateId将自己替换掉

**绘制优化**

onDraw不要创建局部对象，不要做耗时操作

**内存泄露优化**

1. 静态变量导致的内存泄露：例如 activty中 有static的对象，导致Activity无法被回收
2. 单例模式导致内存泄露：例如Activity作为监听接口实现类并注册监听，而注册管理的 对象是用static实现的单例对象，单例对象没有 注销监听器的操作，导致static 持有Activity对象
3. 属性动画导致内存泄露：Activity 必须要适时的停止 无限循环的 属性动画

**响应速度优化**

核心思想：不在主线程做重操作
