---
layout: post
title:  "EventBus框架结构"
date:   2020-02-11
categories: Android
tags: [Android,框架学习]
---
{: toc }
# EventBus
## 0. 基本介绍
EventBus is an open-source library for Android and Java using the publisher/subscriber pattern for loose coupling. EventBus enables central communication to decoupled classes with just a few lines of code – simplifying the code, removing dependencies, and speeding up app development.

EventBus是事件总线的实现框架，首先介绍一下事件总线

**总线**：类比DBus，CBus，总线是 流的统一管道路径
**事件总线**：流动 事件消息 的 统一管道  

## 1. 使用场景
为简化并且更加高质量地在 Activity，Fragment，Thread 和Service等之间的通信，同时解决组件之间高耦合的同时任然继续高效的通行，事件总线的设计出现了。Android常见框架如EventBus，otto
## 2. 基本使用
### 2.1 使用之前：名词概念介绍
##### **三要素**
1. Event：事件，可以是任意类型的对象
2. Subscriber：事件订阅者，事件流可达的对象，我们需要处理对应的事件；在EventBus3.0之前：事件处理的方法只限于onEventXXX；之后：事件处理方法名可任意，但要添加@Subscribe注解
3. Publisher：事件发布者；可在任意线程的任意位置发送事件：调用Event的post方法

##### **四线程模型（ThreadMode）**
* POSTING(Default)：事件源所在线程就是事件处理函数所在线程
* MAIN：事件处理在主线程处理（显然要注意主线程的操作时间限制，推荐只UI更新操作）
* BACKGROUND：事件处理在非主线程；若事件源在主线程，则新建线程，若事件源在非主线程，时间处理位于事件源同线程；事件处理函数不在主线程 当然 无法进行UI操作
* ASYNC：异步模式；事件处理函数必定与事件源所在线程不同，以实现异步操作；事件处理函数所在线程不能为UI线程
### 2.2 基本使用（5个步骤）
0. Gradle 添加依赖
	```java
	compile 'org.greenrobot:eventbus:3.0.0'
	```

1. 自定义事件类 对应 三要素的 Event
	```java
	public class MessageEvent{
	...
	}
	```
2. 所在线程订阅事件 预 注册(对应三要素 Publisher)
	```java
	EventBus.getDefault().register(this);
	```
3. 发送事件
	```java
	EventBus.getDefault().post(messageEvent);
	```
4. 处理事件（对应三要素 Subscriber）

	```java
	@Subscribe (threadMode = ThreadMode.MAIN)
	public void randomName(MessageEvent messageEvent){
	// do something
	}
	```
5. 别忘了取消事件订阅
	```java
	EventBus.getDefault().unregister(this);
	```
### 2.3 特别介绍
**EventBus的粘性事件**
定义：发送事件之后，再注册事件订阅，订阅者依然能收到之前发送的粘性事件
猜想实现：维护了一个粘性事件的队列，保存了所有粘性事件

**使用**
五个步骤没有什么不同，改变的是：
发送事件：<code>EventBus.getDefault().postSticky(event)</code>  
处理函数注解：<code>@Subscribe(threadMode = ThreadMode.POSTING, sticky=true)</code>  
## 3. 痛点问题解决
## 4. 其他框架对比
## 5. 设计模式解析
## 6. 静态代码文件结构
### 基本源码阅读
### 6.1 获得总线实例对象 **EventBus.getDefault()**：
```java
/** Convenience singleton for apps using a process-wide EventBus instance. */
//使用双重检测 实现的 单例模式
    public static EventBus getDefault() {
        EventBus instance = defaultInstance;
        if (instance == null) {
            synchronized (EventBus.class) {
                instance = EventBus.defaultInstance;
                if (instance == null) {
                    instance = EventBus.defaultInstance = new EventBus();//跳到默认构造函数
                }
            }
        }
        return instance;
    }
	// 默认构造函数
	public EventBus() {
        this(DEFAULT_BUILDER);// 使用了 EventBusBuilder 的构造函数
    }
	// EventBusBuilder 的构造函数
	// 这里使用了建造者模式 实现的 EventBus 装配
    EventBus(EventBusBuilder builder) {
        logger = builder.getLogger();
        subscriptionsByEventType = new HashMap<>();
        typesBySubscriber = new HashMap<>();
        // 如我们猜想，这里使用ConcurrentHashMap 来保存粘性事件
        stickyEvents = new ConcurrentHashMap<>();
        mainThreadSupport = builder.getMainThreadSupport();
        mainThreadPoster = mainThreadSupport != null ? mainThreadSupport.createPoster(this) : null;
        backgroundPoster = new BackgroundPoster(this);
        asyncPoster = new AsyncPoster(this);
        indexCount = builder.subscriberInfoIndexes != null ? builder.subscriberInfoIndexes.size() : 0;
        subscriberMethodFinder = new SubscriberMethodFinder(builder.subscriberInfoIndexes,
                builder.strictMethodVerification, builder.ignoreGeneratedIndex);
        logSubscriberExceptions = builder.logSubscriberExceptions;
        logNoSubscriberMessages = builder.logNoSubscriberMessages;
        sendSubscriberExceptionEvent = builder.sendSubscriberExceptionEvent;
        sendNoSubscriberEvent = builder.sendNoSubscriberEvent;
        throwSubscriberException = builder.throwSubscriberException;
        eventInheritance = builder.eventInheritance;
        // 可以看出来 使用了Java的ExecutorService线程池
        executorService = builder.executorService;
    }
```

### 6.2  订阅者注册**EventBus.getDefault().register()**

```java
/**
     * Registers the given subscriber to receive events. Subscribers must call {@link #unregister(Object)} once they
     * are no longer interested in receiving events.
     * <p/>
     * Subscribers have event handling methods that must be annotated by {@link Subscribe}.
     * The {@link Subscribe} annotation also allows configuration like {@link
     * ThreadMode} and priority.
     */
    public void register(Object subscriber) {
        Class<?> subscriberClass = subscriber.getClass();
        // 1. 查找需要订阅方法
        // 通过@Subscribe注解收集的订阅方法，通过反射 Method，包装为SubscriberMethod对象,并且保存属性，如下
        // SubscriberMethod(Method method, Class<?> eventType, ThreadMode threadMode, int priority, boolean sticky) 
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
        // 2. 对方法进行订阅
        // 包装 subsriber 和 subscriberMethod 对象为：Subscription对象
        // 如此便通过 类组合 的方式，建立了事件处理方法 与 subscriber类的关系（subscriber所在线程的关系）
        // 将Subscription 工具 evnetType 存入 SubscriptionsByEventType(Map集合) Map<Class<?>, CopyOnWriteArrayList<Subscription>>
        // 如此便通过map的方式 建立了Event类与相应的订阅方法反射的联系
        synchronized (this) {
            for (SubscriberMethod subscriberMethod : subscriberMethods) {
                subscribe(subscriber, subscriberMethod);
            }
        }
    }
```
### 6.3 发送事件 EventBus.getDefault().post(event)
```java
/** Posts the given event to the event bus. */
    public void post(Object event) {
    	// currentPostingThreadState 是 线程无关的 threadlocal 数据结构
    	// PostingThreadState:保存着当前线程 的 事件队列 和线程信息状态
    	/**
    	 private final ThreadLocal<PostingThreadState> currentPostingThreadState = new ThreadLocal<PostingThreadState>() {
        @Override
        protected PostingThreadState initialValue() {
            return new PostingThreadState();
        }
    };
    	*/
        PostingThreadState postingState = currentPostingThreadState.get();
        // 取得当前线程的 事件队列
        List<Object> eventQueue = postingState.eventQueue;
        eventQueue.add(event);
		// 修改状态后 对事件队列 全部postSingeEvent
        if (!postingState.isPosting) {
            postingState.isMainThread = isMainThread();
            postingState.isPosting = true;
            if (postingState.canceled) {
                throw new EventBusException("Internal error. Abort state was not reset");
            }
            try {
                while (!eventQueue.isEmpty()) {
                    postSingleEvent(eventQueue.remove(0), postingState);
                }
            } finally {
                postingState.isPosting = false;
                postingState.isMainThread = false;
            }
        }
    }
	// 投递单条 Event 调用 postSingleEventForEventType
	private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
        Class<?> eventClass = event.getClass();
        boolean subscriptionFound = false;
        if (eventInheritance) {
            List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
            int countTypes = eventTypes.size();
            for (int h = 0; h < countTypes; h++) {
                Class<?> clazz = eventTypes.get(h);
                subscriptionFound |= postSingleEventForEventType(event, postingState, clazz);
            }
        } else {
            subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
        }
        if (!subscriptionFound) {
            if (logNoSubscriberMessages) {
                logger.log(Level.FINE, "No subscribers registered for event " + eventClass);
            }
            if (sendNoSubscriberEvent && eventClass != NoSubscriberEvent.class &&
                    eventClass != SubscriberExceptionEvent.class) {
                post(new NoSubscriberEvent(this, event));
            }
        }
    }
    // 根据 Evnet类型查找 绑定的订阅者Subscription 
    // 内部调用了 postToSubscription，对每一个订阅者发送事件
	private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) {
        CopyOnWriteArrayList<Subscription> subscriptions;
        synchronized (this) {
            subscriptions = subscriptionsByEventType.get(eventClass);
        }
        if (subscriptions != null && !subscriptions.isEmpty()) {
            for (Subscription subscription : subscriptions) {
                postingState.event = event;
                postingState.subscription = subscription;
                boolean aborted;
                try {
                    postToSubscription(subscription, event, postingState.isMainThread);
                    aborted = postingState.canceled;
                } finally {
                    postingState.event = null;
                    postingState.subscription = null;
                    postingState.canceled = false;
                }
                if (aborted) {
                    break;
                }
            }
            return true;
        }
        return false;
    }
	// 根据线程模型枚举的值 分情况 加入主线程队列，加入poster队列，加入当前线程队列
    private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
        switch (subscription.subscriberMethod.threadMode) {
            case POSTING:
                invokeSubscriber(subscription, event);
                break;
            case MAIN:
                if (isMainThread) {
                    invokeSubscriber(subscription, event);
                } else {
                    mainThreadPoster.enqueue(subscription, event);
                }
                break;
            case MAIN_ORDERED:
                if (mainThreadPoster != null) {
                    mainThreadPoster.enqueue(subscription, event);
                } else {
                    // temporary: technically not correct as poster not decoupled from subscriber
                    invokeSubscriber(subscription, event);
                }
                break;
            case BACKGROUND:
                if (isMainThread) {
                    backgroundPoster.enqueue(subscription, event);
                } else {
                    invokeSubscriber(subscription, event);
                }
                break;
            case ASYNC:
                asyncPoster.enqueue(subscription, event);
                break;
            default:
                throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
        }
    }

```

## 7. 动态代码执行逻辑流
## 8. 反思思考、
### 8.1 post event为何要四层调用
决定于 事件源 与 订阅者的 路径确定
1. post（）：将事件添加到 本线程 待投 **队列** 
2. postSingleEvent（）：对待投队列的每个事件 ，根据已有Subscription **集合**，判断是否有订阅者
3.  postSingleEventForEventType（）：根据 Evnet类型查找 **绑定**的订阅者Subscriptions
4. postToSubscription（）：根据订阅者**绑定**ThreadMode判断 与Event 投递线程 类型

映射关系为：

1. event 与当前线程队列的关系
2. 当前线程队列与 订阅者有无的关系
3.  event 与 订阅者集合的关系
4. event 与 订阅者 其他线程的关系

## 9. 开发使用常见问题
## 10. 常见素材
1. [EventBus 官方网站](https://greenrobot.org/eventbus/)
2. [EventBus GitHub地址](https://github.com/greenrobot/EventBus)
3. [知乎主题帖]()
4. 问题1
5. 问题2
6. 优秀博文1
7. 优秀博文2