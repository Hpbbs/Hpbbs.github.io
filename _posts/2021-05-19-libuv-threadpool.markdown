---
layout: post
title: "Libuv线程池核心代码笔记"
published: true
categories: [ComputerScience]
---

Libuv 线程池

### 源码流程

全局配置

```c
#define MAX_THREADPOOL_SIZE 1024

static uv_once_t once = UV_ONCE_INIT;
static uv_cond_t cond;
static uv_mutex_t mutex;
static unsigned int idle_threads;
static unsigned int slow_io_work_running;
static unsigned int nthreads;
static uv_thread_t* threads;
static uv_thread_t default_threads[4];
static QUEUE exit_message;
static QUEUE wq;
static QUEUE run_slow_work_message;
static QUEUE slow_io_pending_wq;
```
**Changed in version 1.30.0: the maximum UV_THREADPOOL_SIZE allowed was increased from 128 to 1024.**

初始化线程
基本流程：

1. threadpool 全局资源初始化
基本是整个线程池的 文件域 内部变量的初始化
包括：条件变量初始化 mutex初始化 默认数量线程的创建（借助semphomore给create和wait加锁）
wait queue，slow io pending queue，run slow的队列初始化

2. threadpool的默认线程创建

运行的函数是 worker

3. 将所有worker暂停

```c
static void init_threads(void) {
  unsigned int i;
  const char* val;
  uv_sem_t sem;

  nthreads = ARRAY_SIZE(default_threads);
  val = getenv("UV_THREADPOOL_SIZE");
  if (val != NULL)
    nthreads = atoi(val);
  if (nthreads == 0)
    nthreads = 1;
  if (nthreads > MAX_THREADPOOL_SIZE)
    nthreads = MAX_THREADPOOL_SIZE;

  threads = default_threads;
  if (nthreads > ARRAY_SIZE(default_threads)) {
    threads = uv__malloc(nthreads * sizeof(threads[0]));
    if (threads == NULL) {
      nthreads = ARRAY_SIZE(default_threads);
      threads = default_threads;
    }
  }

  if (uv_cond_init(&cond))
    abort();

  if (uv_mutex_init(&mutex))
    abort();

  QUEUE_INIT(&wq);
  QUEUE_INIT(&slow_io_pending_wq);
  QUEUE_INIT(&run_slow_work_message);

  if (uv_sem_init(&sem, 0))
    abort();

  for (i = 0; i < nthreads; i++)
    if (uv_thread_create(threads + i, worker, &sem))
      abort();

  for (i = 0; i < nthreads; i++)
    uv_sem_wait(&sem);

  uv_sem_destroy(&sem);
}
```

再来看worker线程执行了啥

大概流程：

1. 初始化工作队列
2. 对全局mutex加锁 排队进入 for 循环
3. 如果只有 slow io 任务，当前线程就会 idle
4. 取队列头处理（不同的队列头）
5. 取得队列item的 uv_work_t（work回调和不同的loop） 调用work 回调

注意：work里面有很多 mutex cond 的条件变量和互斥锁的操作
对工作队列的 mutex 锁，cond 是与mutex相关的
每个 uv_work_t 的loop也会有wq_mutex的loop执行操作锁

考虑三者关联：

- thread_pool 的内部的工作队列
- 线程池的每个线程：知识线程号，没有什么本质的作用，（可以通过线程号获得对应的线程操作具柄？）
- 每个loop与对应的线程的 执行关系
- 每个loop与每个 work 的bind关系，但是 loop 运行又是提到 对应的work thread 运行的

```c
uv_mutex_lock(&mutex);
  for (;;) {
      while (QUEUE_EMPTY(&wq) ||
        idle thread +=1
      q = QUEUE_HEAD(&wq);
```

提交整体
```c
static void post(QUEUE* q, enum uv__work_kind kind) {
```

### 整体设计结构

### 有何优点 缺点 何 潜在缺陷

### 有无场景化的限制


### 引用

- [libuv threadpool 官网]( http://docs.libuv.org/en/v1.x/threadpool.html?highlight=threadpool )
- 优秀博客