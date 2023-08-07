---
layout: post
title: "UncaughtExceptionHandler "
published: false
categories: [ComputerScience]
tags: [Java]
mermaid: true
---



# UncaughtExceptionHandler

类图静态关系

```mermaid
classDiagram
class ThreadGroup{
	-ThreadGroup parent
}
class ThreadDead{
	<<is a kind of Tag: Who tag it?>>
}
class Thread{
	-UncaughtExceptionHandler defaultUncaughtExceptionHandler
	-UncaughtExceptionHandler uncaughtExceptionHandler
}
class Thread_UncaughtExceptionHandler{
	<<interface>>
	uncaughtException(Thread t,Throwable e)
}

class Error
ThreadDead --|> Error
```

处理责任链：From ThreadGroup#uncaughtException() comment

```mermaid
graph LR;

JVM --> Thread
Thread -- handle by --> UncaughtExceptionHandler
Thread --if UncaughtExceptionHandler is null--> ThreadGroup
ThreadGroup --choice one:Pass to parent threadgroup --> ParentThreadGroup
ThreadGroup --choice two:Handle by Thread default --> Thread.DefaultUncaughtExceptionHandler
ThreadGroup --choice three:check ThreadDead --> PrintStack


```

```java
public void uncaughtException(Thread thread, Throwable e){
  // do the basic logger
  ActivityManager.getService().handleApplicationCrash()
  if(e instanceof ThreadDead){
    // UI mention the user: UI can show when application dead
    Process.killProcess(Process.myPid());
    System.exit(10);
  }
}
```

Android 平台的特殊处理：提供了一个默认的UncaughtHandler 注入 Thread.sDefaultUncaughtExceptionHandler ，主要为打印系统日志，拉起系统提示对话框，杀死App，

UncaughtHandler代码也显示了Crash的原因：

当主线程有未捕获的异常，然后交给Android的UncaughtHandler处理，默认就是Kill process



Android的基本处理

1. 保存Android默认的UncaughtHandler，如果要使用的话
2. 填充自己的Thread#UncaughtThreadHandler
3. 检查当然线程是否是主线程，如果是主线程，重启对应的栈顶的Activity，并让MainLooper.loop()（参考下文）起来，来保证正常运行

让MainLooper 继续循环，post一个Runnable，在runnable里面 while(true)死循环try-catch调用Looper.loop()，就能保证主线程异常一定能被catch到而不至于走到Android 杀死进程的UncaughtHandler
