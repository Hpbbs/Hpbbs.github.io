---
layout: post
title: Posix pthread基本使用
published: true
categories: [Unix,框架阅读]
---

**无论在理论上线程的概念如何定义，在 编程 上的体现，只是POSIX thead api的 同步工具 和线程 id， 可以说 *mutex/cond/signal（同步工具） + pthread_id（Naming工具/区分的具柄）= Thread***

只要支持了POSIX thread api，那么就可以说，是支持多线程编程模型的

**我们可以以系统 thread 的能力为基础，虚拟化出我们的 虚拟线程 vThread 库 应运而生**

其次，接口api对上层保证的调度服务需要保证

### barrier

栅栏：顾名思义，当所有线程都抵达到 同一个栅栏，才会给所有的线程放行
所以，这是一个 多线程之间 同步的工具
类似于计算机操作系统课程的 pv 操作工具

init：创建 barrier，并指明拦截的线程数量 （只有一个线程创建一次）
wait：等待状态（每个线程调用一次）
destroy：放行，所有阻塞的线程进入线程调度 （只有一个线程调用一次）

### mutex

互斥量：用于对 竞争区 进行互斥访问的工具，达成一临界区

init：初始化一个 互斥量

lock：对代码块加锁（其他线程不可进入，直接阻塞）
unlock：对代码快解锁（释放竞争区，被当前mutex锁住的线程将被唤醒）
trylock：试图加锁，如果得到锁失败也不会被阻塞，可以做别的事情，或者 自旋 等待（自旋可以降低系统阻塞线程和唤醒的开销，对于 竞争区cpu占用 很小时，使用trylock自旋是个不错的选择）

destroy：销毁 互斥量

### cond

condition 条件变量，用于与 mutex 配合使用
cond + mutex = 操作系统课程 管程

cond 会维护被mutex阻塞线程的列表，在条件成立的时候，从阻塞线程列表选取（根据不同的策略选取）线程 唤醒 获得锁后 进入 临界区

wait：阻塞并 进入 cond 排队 （操作系统理论课程的 P 操作）
signal：释放锁，唤醒cond的其他一个阻塞线程 （操作系统理论课程的 V 操作）
broadcast：释放锁，唤醒cond的其他所有阻塞线程，与signal相比，broadcast 对于所有阻塞线程 公平的
（但是先来排队的，应该先唤醒，如果你认为如此，那么signal才是意义上的“公平”，毕竟“公平”不同标准下，解释不同）

```
The pthread_cond_broadcast() function shall unblock all threads currently blocked on the specified condition variable cond.

The pthread_cond_signal() function shall unblock at least one of the threads that are blocked on the specified condition variable cond (if any threads are blocked on cond).
```

### rwlock

当 mutex 锁住 临界资源时候，只能选择 进入或不进入
但是，考虑一个问题，如果我们 一个写线程，多个读线程的时候，每一个读线程之间的行为完全不会影响到另外一个读线程---- 我们希望对于读线程而言，是可以并行操作

我们可以使用读写锁（读写两种行为分离的锁）来降低锁的粒度，提高并行度（一个线程写的时候，对资源是独占的，但是支持多个线程同时 读操作）

lock :- rdlock wrlock

init: 初始化

rd/wrlock：加锁
tryrd/wrlock： try接口
timerd/wrlock：定时持有锁
unlock：释放锁

destroy: 销毁锁

setnamp_np: 设定锁的名字

--- 

以上为基本的同步工具，主要为锁，
很显然，我们似乎可以使用 mutex 模拟出其他锁的行为，但是直接提供更加方便


### signal

kill： 杀死线程
sigmask：获得或设置 信号的掩码（mask 屏蔽不关心的信号）

### Thread Local storage（TLS）

对线程共享的内存空间，划分出每个线程的“private” 数据
（pthread是使用 kv api 来做管理的）

- pthread_getspecific() (Get Thread Local Storage Value by Key) retrieves the thread local storage value associated with the key。      
- pthread_getspecific() may be called from a data destructor.

- pthread_key_create() (Create Thread Local Storage Key) creates a thread local storage key for the process and associates the destructor function with that key.
- pthread_key_delete() (Delete Thread Local Storage Key) deletes a process-wide thread local storage key.
- pthread_setspecific() (Set Thread Local Storage by Key) sets the thread local storage value associated with a key.

### thread manipulate

重要的管理属性：（api：作为 attr object）   
detachstate：JOINABLE DETACHED   
schedparm：优先级 以及 调度策略   
contentionscope：PTHREAD_SCOPE_SYSTEM   
inheritessched：继承优先级   
schedpolicy：调度策略   

api：

create：创建线程

join：wait for detach thread terminate, 返回返回状态

detach: 标记线程为 detached ，当线程结束的时候，线程资源会被系统自动释放，without the need for another thread to join with the terminated thread

once：performs one time initialization based on a specific once_control variable---保证多个线程执行到此只会执行一次

self：获得当前线程标识

exit: terminal calling thread

### sem semophore

信号量，不在pthread.h中，同时进程IPC的方式，加入用了POSIX标准

是一个计数器，用于为多个进程对共享数据对象的访问：
使用流程：
1. 测试控制该资源的信号量
2. 如果信号量值为正，可进程可以使用这个 该资源，而且 计数器 会下降，标识进程消耗了一个资源
3. 如果信号量值为0，那么该进程进入休眠状态（不是阻塞啊），直到信号量值大于0，进程被唤醒，回到第一步
4. 如果进程不使用该资源了，信号量增加1，如果有休眠进程等待此信号量，唤醒他们（是他们，而不是选择一个唤醒）

常用信号量为 二元信号量 ，表明控制单个资源，（多个资源可能会选用其他的IPC方式）

信号量的测试和减1是 原子操作

sem_init: 初始化一个无名信号量，传入的参数 pshare 表明这个信号量是在 同一进程中线程间共享 还是在 进程之间共享

sem_destroy:

sem_post:

sem_wait: