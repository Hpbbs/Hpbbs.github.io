---
layout: post
title: "Linux Native(iOS,Android) Backent Web 组件的一致性"
published: true
categories: [构想]
tags: [tech, 灵感]
---


---

目的

1. 解构Client 和 Server 这两个被默认接受的概念，几乎固化在思维里，当作经验规律前置来处理了，不受概念的僵化
2. 对比 Client 和 Server 两种场景，两种上下文的相同和不同！
3. 了解相关的历史演变，以及Client与Server这两个固化的概念是如何诞生出来的
4. 站在实践的角度：对Client和Server两个 Domain 的相同映射的技术概念
   1. 如单体服务
   2. 如消息队列，服务注册和服务发现

---

### Android 与 Backend端对比简单对比

**横向对比**

总整体看来，无论Native端，还是Backend，其内容和架构都是一样的

但这两者 独立发展这么长时间，有不同的思考，又由于其本质相同，这种思考对另一方有很好的参考价值

**纵向对比**

在计算机系统的不同组织结构组件：中断，数据总线，锁，虚拟内存，处理器等 抽象模型

在微服务对服务进行拆分的时候，所面临，要解决的问题实质是一样，如是，也出现了：

> - Configuration Management
> - Service Discovery
> - Circuit Breakers
> - intelligent routing
> - micro-proxy
> - control-bus
> - one-time tokens
> - Global locks
> - leader-ship election
> - distribute sessions
> - cluster state

这两个处于不同层次的，解决同一个问题模型的 方案和提出的点，也是可以互相参考，借鉴

### 问题模型？

**所以：如何将 这个问题模型 明确下来？这是一个什么问题模型？**

1. 信息抽象机制问题？计算机系统是：输入 - 存储 - 处理 - 存储 - 输出；或者是一个 I/O 模型？
（这个抽象层次太高了，没有参考价值）

2. 分布式 数据处理问题？
3. 拆分，复用，解耦，自动化，分布合作，虚拟化，

这是个什么问题模型？？？？？？？

SpringBoot微服务架构，就等同于 Android应用层 Activity 组件架构，相关的对应的组件 标示着 对应的架构能力需求，需求在不同端可能会有侧重点的不同，但是需要本身几乎是一致的

当然，也应该明显的对比出不同点

安装 ---- 部署

**从数据流动角度**：从不同的数据库，数据双向的流动：从这个角度来看，本质也就是一个 在异构平台的 数据管道，而所谓 客户端，服务端之间的接口不过是：分离的管道衔接器

**从资源的角度而言** ：分散的客户端将用户创建的资源搬运到集中服务器端的 数据管理中，从服务端拉下资源，展示给用户

**从数据处理角度**：人工智能，数据分析等结果，本身只是对 数据的一个 中间处理，完全可以抽象为一个 类似于渲染管线的一个 部分，包裹为一个组件，SaaS的思维逻辑

如何用Saas的逻辑理解Android的 Activity设计？

**从数据特点角度**

Client 端侧的数据特点：能够捕捉的所有的用户个人的“局部”信息

Server 端侧的数据特点：拥有所有用户的一个全局信息

（但是Server就必须要有所有项目全局的信息吗？client就不能联合其他的client，获得一个更大的局部信息嘛，对应于P2P网络，分离的Server）

### 组件能力梳理

总结表格：

| 架构能力                                 | Android/单设备系统平台服务                                   | SpringBoot/分布式服务                                        | 对比结论                                            |
| ---------------------------------------- | ------------------------------------------------------------ | :----------------------------------------------------------- | --------------------------------------------------- |
| UI服务                                   | Activity的生命期，上下文，和UI托管的能力--- 渲染：格式化的填充FrameBuffer颜色值数据数据 | Html抽象的渲染能力---渲染：格式化的填充字符串数据            | UI上Frontend便重                                    |
| CPU执行管理能力                          | Service组件                                                  | Linux进程，没有更上一层的抽象                                | Backend几乎就只是暴露系统提供的能力                 |
| 数据存储服务能力                         | ContentProvider/ContentResolver<br />非规范化框架：Room，mmkv这一些 | JPA协议的ORM框架以及其他No-Sql数据模型：这个是Server端的重点 | 数据上，Backend偏重，数据操作更加丰富               |
| 消息通知能力                             | 1. Broadcast / BroadcastReceiver<br />2. Intent 设计（URI）  | JMS，等一些消息队列（是可以带数据的，但是也不能带大量数据）  | 基本一致                                            |
| 大数据搬运服务（连续搬运连续内存的能力） | 依赖于Socket，Binder的文件系统接口 --- 虚拟硬件              | 文件系统接口/Hadoop等文件系统接口/Socket                     | 总线媒介不同，连接的作用域大小有变化                |
| 服务路由：全局的服务发现和服务注册的能力 | ActivityManagerService 的组件服务注册能力<br />Android将服务路由与页面路由整合了 | 各种框架：Nacos，ZooKeeper等                                 | 基本一致，只有作用域区别                            |
| **功能能力**                             |                                                              |                                                              |                                                     |
| 页面路由                                 | 自建的Navigator，其他三方框架也有，没有标准化                | URL的页面资源设计，Tomcat提供                                | 可见，基于URI的资源，将页面处理为资源的设计更加优越 |
| 运行时容器                               | ART vm with Zygote Runtime Framework                         | JVM with tomcat embeded，（VM隔离级别：Docker，kube）        | 基本一致                                            |



| 开发规范上 | Android               | Backend/SpringBoot |
| ---------- | --------------------- | ------------------ |
| 模块化开发 | 基于Gradle 等构建工具 | 基于Mvn Gradle     |
|            |                       |                    |
|            |                       |                    |

Android中的 四大组件：

- UI服务，生命期服务，提供屏幕资源
- Service，CPU服务，提供CPU资源
- ContentProvider，数据存储服务，提供数据资源和数据能力
- BroadcaReceiver，消息服务
- ActivityManagerService，服务发现和服务注册能力



Android OS Framework层，其重点似乎是：如何将整个设备的硬件资源，如何很好的切分为“Application”这个概念，并在系统层面，将“Application”这个概念当成一个服务，功能，组件（服务化，组件化）：Android的基本处理方式是：以进程为Application的 资源和执行上下文的自然划分，和基于用户的硬件资源管理和基于角色的安全控制（同Unix）

由于虚拟化技术的成熟，是否可以依赖虚拟化的技术，让手机这个Device拥有更高一级的虚拟层级上下文

SpringBoot Framework层，其重点似乎是：打造一个“数据管道“的概念？ 这个的理解上你还是有欠缺，暂时性的理解：

### 构想

1. 能否提供一个框架，可以将 Android Framework的服务型组件，也能直接注册到Nacos类似的服务中心，将单个设备的服务，直接暴露到更加广阔的作用域
