---
layout: post
title:  "读书笔记-深入理解JVM-周志明"
date:   2020-02-11
categories: JVM
tags: [读书笔记, Android, JVM]
---

# 第1章 了解Java生态

JVM的几个特性：解释执行，即时编译，提前编译；热点代码探测，

适用场景区分：嵌入式 桌面 服务端，启动时间 预热 垃圾清理时延 可管理的CPU内存的系统资源的规模 内存占用

# Part Two 自动内存管理

## 第2章 Java内存区域

概念模型：代表虚拟机的统一外观，当时各个具体的虚拟机的实现不一定按照概念模型定义来设计，而是提供等价的实现

### 2.2 运行时数据区域 Java 虚拟机内存模型

由于Java虚拟机是 栈式VM

线程共享的： 方法区Method Area + 堆Heap

线程隔离的：虚拟机栈 VM Stack 本地方法栈 NativeMethodStack 程序计数器PCRegister

#### PC

线程当前执行的字节码的 下一条字节码指令；Java方法则为字节码指令的地址，Native方法则为Undefine

#### Java虚拟机栈

线程私有，与线程生命期相同，每个方法执行时，都会创建一个栈帧：存储局部变量表，操作数，动态连接 方法出口等信息

**局部变量表**：编译期可知的 数据基本类型 引用类型 returnAddress类型(字节码指令地址)；使用slot为最小物理内存单位，变量槽为一个 字 大小；运行时不会改变其大小

#### 本地方法栈：为虚拟机使用的本地方法服务

#### Java Heap

所有线程共享的区域；几乎所有对象实例和数组都在此分配内存；值类型的支持可能会让对象实例就在栈上分配；

Heap是垃圾收集器管理的区域：一般基于分代收集理论设计（基于对象生命期特点来分类）

Java Heap 内存物理上可不连续，但逻辑上必须连续

#### 方法区 Method Area

线程共享

存储：虚拟机加载的：类型信息 常量 静态变量 即时编译器编译后的代码缓存等

JDK7 将字符串常量池 静态变量 移到了Java堆中，JDK8 使用了 本地内存 实现的元空间 meta-space

回收目标主要是：常量池的回收 和 类型的卸载

**运行时常量池**：属于方法区的一部分，包含编译时常量（主要对应class常量池）和 运行时常量（运行时添加的常量）



#### 直接内存 Direct Memory

不是虚拟机运行时数据区的一部分，也不是虚拟机规范定义的内存区域

将 引用对象 与 本地方法直接分配的堆外内存 映射，避免了Java堆与Native堆中来回复制数据的成本

不受Java堆大小的限制



### 2.3 虚拟机视角的对象

#### 2.3.1 对象的创建

**对象的创建过程**：遇到new指令；定位常量池中对应的 类的符号引用；检查对应类是否被加载 解析 和初始化，没有则执行类的加载过程；为新对象在Java堆中分配内存(大小由类加载解析的信息确定)；对对象进行必要的设置（对象头）：属于哪一个类 如何找到类元数据信息 对象哈希编码(调用hashCode才会计算) GC分代信息；对象的初始化操作（init操作）。**简而言之：准备类信息 内存 对象头 然后初始化**

需要考虑的问题：

内存分配，找到可用的内存，与内存是否规整有关，与堆区采用的垃圾收集算法有关，Serial ParNew 等带压缩整理过程的收集器时，内存规整，使用指针碰撞分配内存；如果使用CMS这种基于清除的算法收集器时，内存不规整，需要用内存链表管理

内存分配的是Java堆区，是线程共享的区域，便要考虑内存分配时的数据竞争问题，保证对象内存分配的原子性：采用的是CAS组合失败重试方式保证操作的原子性；或者将Java堆预先分配一小部分给线程私有，对应于ThreadLocalAllocationBuffer：本地线程分配缓冲



#### 2.3.2 对象的内存布局

三部分：对象头 对象实例数据 对齐填充

**对象头**：

1. 对象自身的运行时信息：哈希编码 GC年龄 代锁状态标志 线程持有的锁 偏向线程ID 偏向时间戳 Mark Word；Unit 根据不同运行状态复用内存P51
2. 类型指针：指向类型元数据，数组的化存有长度

**对象体**：字段内容：继承的或自定义的，存储顺序受到虚拟机分配策略影响：longs/double ints shorts/chars bytes/booleans oops(ordinary object pointers)：相同字宽的放在一起，帮助内存对齐

**填充**：Hotspot 自动内存管理系统要求对象的起始地址为 8 字节的整数倍，填充用于对齐

#### 2.3.3 对象的访问方式

**为了方便访问，使用 栈 上的reference数据来 操作堆上的具体数据对象**

reference主流为两种方式：句柄 或 直接指针

句柄：在Java堆上分配 句柄池，reference -> 句柄池Item -> 对象内存数据；对象移动常见的虚拟机中，当对象移动时，reference指向的是稳定的句柄

直接指针：reference -> 对象内存数据；不需要多一次间接访问的开销



## 第3章 垃圾收集器 与内存分配策略

垃圾收集器的历史远比Java古老，Lisp是第一个尝试垃圾收集的语言

- 那些内存需要回收
- 什么时候回收
- 如何回收

当垃圾收集成为系统达到高并发量的瓶颈时，就必须对自动化的垃圾收集和内存分配实施必要的监控和调节

程序计数器，虚拟机栈 本地方法栈，每个栈帧的大小在编译期就能确定，内存随作用域 生命期而确定内存的申请和释放，行为确定

Java堆和方法区有着显著的不确定性，在对象不手动管理时，运行时，不同动态数据决定的条件分支所创建的对象不可以预知，是随着动态的上下文而不同的，Just like the life， we cant predicate

判断对象需要被回收：GC roots的可达性分析，不可达为不可能再访问的对象，需要被回收

#### 3.2 对象存活判断

1. 引用计数算法，原理简单，判定效率高，但是有很多特殊情况要处理：比如对象循环引用处理的问题
2. 可达性分析算法：多数jvm标准实现的虚拟机采用的方式



##### 可达性分析

1. （根节点枚举，列举GC roots对象）找到坑定能确定不回收的对象，或者对象生命周期 状态转变虚拟机可感知了解的



## Part Three 虚拟机执行子系统

### 第6章 类文件结构

采用类似c的结构体 来描述内容（实现或许就是直接定义结构体）主要有两种表述：1. 无符号数，u1 u2 u3 u4 分别对应 1 2 3 4 字节的无符号数，来表示对应的slot槽宽，以及 内存内容 2. 表：就是类似于c结构体那样，对于内存布局的描述，但是基本单位是 无符号数的槽

对于数量确定的 不同表项：出现的先后规则严格定义

对于数量不一定 表项：起始计数器，加上连续的计数器个严格定义内部结构的表组成



### 第7章 虚拟机类加载机制

**非编译时连接**：与编译时连接的语言不同，Java语言里，类型的加载 连接和初始化过程都是在运行时进行期间完成的，让Java的提前编译产生额外的困难，让类加载时产生额外的性能开销，优点是：提供了极高的拓展性和灵活性，Java可以动态扩展的能力的语言特性就是依赖于运行期动态加载和动态连接的这个特点

Applet Jsp OSGi（服务动态注册 ）技术都是依赖于Java语言运行期类加载机制



**生命周期 七个过程**

一个类型从 被加载到虚拟机内存开始，到卸载出内存为止，他的整个生命周期会经历7个过程：加载Loading 验证Verification 准备Preparation 解析Resolution 初始化Initialization 使用Using and 卸载Unloading

验证 准备 解析 为 连接过程

解析阶段要进行 类似于重定位的符号绑定过程，在某些情况下可以在初始化后再开始，为Java语言支持运行时绑定特性（动态绑定 晚绑定）



#### 7.2 类加载时机

类初始化：有且仅有：6中情况，初始化前必 加载 连接之后

称为对一个类型进行**主动引用**（不会触发类型初始化的引用称为被动应用）

1. new getstatic putstatic invokestatic 字节码调用，没有初始化则初始化
   1. new 关键字实例化对象
   2. 读取或设置类型的静态字段 (除外：被final修饰，编译时使用常量传播机制优化到常量池的静态字段) 《== 原文这句不能理解，即使优化到了class文件常量池，但也是要加载这个class文件，内存才会有存在这个 字段的访问引用啊，表示不能理解，或者说是 calss文件存在多个类，加载了calss文件，但是没有加载对应类型？
   3. 调用类型的静态方法时候
2. 使用 java.lang.reflect 包对类进行反射调用的时候，若类型没有初始化则初始化
3. 当虚拟机启动时，需要初始化主类
4. 初始化类时，其父类还没有初始化，则先将父类初始化
5. 使用JDK7新加入的动态语言支持时：如果 java.lang.invoke.MethodHandle 实例最后解析的结果为REF_getStatic REF_putStatic REF_invokeStatic REF_newInvokeSpecial 四种类型的方法句柄时，并且这个方法句柄对应的类没有进行过初始化时，则先触发其初始化
6. 当一个接口定义了JDK8新加入的默认方法(default 修饰的方法)时，如果这个接口的实现类发生了初始化，那该接口要在其之前初始化

HotspotVM -XX:+TraceClassLoading 参数可观察到操作是否导致子类的加载

**被动引用**：

1. 子类引用父类的静态字段，不会导致子类的初始化

2. 数组定义来引用类不会触发类的初始化

   ```java
   SuperClass[] sca = new SuperClass[10];
   // 会初始化 [xxx.SuperClass 即自动生产的数组类；但不会初始化 xxx.SuperClass
   ```

3. 常量（static final）在编译阶段会存入调用类的常量池中，本质上没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化

4. 接口和类真正的区别是当类要初始化时，要求其所有父类都已经初始化，但一个接口初始化时，不要求其全部父接口初始化，只有在使用到了父接口时才会初始化

#### 7.3 类加载的过程

- **加载 Loading**：1. 通过类全限定名获取此类的二进制字节流（不要求文件：比如网路数据，计算所得 动态代理的类） 2. 将字节流所代表的静态存储结构转化为方法区的运行时数据结构 3. **在内存中生产一个代表这个类的 java.lang.Class对象（作为类文件的操作句柄），作为方法区这个类的各种数据的访问入口** 4. 开发人员可控性最强的阶段，根据想法实现自己的类加载器控制字节流的实现方式（重写类加载器的findClass 或loadClass），赋予应用程序获取运行代码的动态性

数组类本身不通过类加载器创建，有Java 虚拟机直接在内存中动态构建出来，数组类的可访问性与元素类型的可访问性一致

一个类型必须与类加载器一起确定唯一性

**加载阶段与连接阶段的部分动作是交叉执行的**：流水线提升性能，也方便连接时使用加载阶段的信息，加载阶段使用连接过程中的结果支持继续的加载过程

- **验证阶段**：确保class文件的字节流中包含信息符合 Java虚拟机规范 的全部约束要求，包装安全的保障；1. 文件格式验证 保证数据流能正确的解析并存储于方法区内 2. 元数据验证：对字节码描述的信息进行语义分析，保证符合 Java语言规范 的全部约束 3. 字节码验证：确定语义是合法并且是符合逻辑的，比如尽可能多的将验证的内容提前到编译时，在Code属性表新增一个 StackMapTable，描述栈帧，代码块的一些状态，类型信息，将类型推导转变为类型检查 4. 符号引用验证



- **准备阶段**：正式为类中定义的变量（static的静态变量）分配内存并设置初始值；概念上来说，内存是在方法区，当实现上只用实现这个 方法区 这个语义概念，比如之前是放在永久代，JDK7后随着Class对象一起放在Java 堆区

变量是类变量而不是实例变量，初始值“通常是变量字段类型的初始值”，而不是代码赋予的字面量的值，比如Java中基本类型都是 0 值，boolean就是false(0)，reference就是 null

如果是static final，就会生成ConstantValue属性，在准备阶段直接赋值，字面量的值



- **解析**：符号引用解析，符号绑定：Java虚拟机将常量池内的符号引用替换为直接引用的过程；**注意，区别于将符号与运行时实例内存绑定的过程**

Java虚拟机规范 不约束解析发生的具体时间，只约束在访问指定之前，这个符号对应的目标必须确定；involeDynamic是个例外，对应的引用是 动态调用点限定符 Dynamically-Computed Call Site Specifier，动态指等程序实际运行到这条执行时，解析动作才能开始执行 相对的，其余的解析过程都是“静态”的，不以运行时数据信息为参考，还没执行代码前就解析完成

符号引用：在Class文件中以CONSTANT_Class_info CONSTANT_Fieldref_info CONSTANT_Methodref_info 等类型的常量出现：Symbolic Reference：就是符号与目标的对应关系，与虚拟机的内存布局无关，是一种概念上的对应绑定关系

直接引用：Direct Reference：与虚拟机的内存布局有关的，必须能在内存上访问到目标，比如指向目标的指针，比如内存某个位置的偏移量，比如简洁定位到目标的句柄；如果有了直接引用，那么说明目标是可访问的，必须以及存在于内存上

解析动作主要针对 类或接口 字段 类方法 接口方法 方法类型 方法句柄和调用点限定符这7类引用进行，在常量池有8种常量类型与之对应



- **初始化**：初始化是类加载的最后一个阶段，前面阶段，除了加载阶段可以自定制类加载外，其余的阶段行为都是Java虚拟机主导并执行的，直到初始化阶段，Java虚拟机才开始执行Java类中的用户代码，将可控权交给应用程序

  在准备阶段，已经对字段值进行了 0 初始化，初始化阶段，进行用户行为的初始化

  初始化阶段执行的是 clinit() 方法的初始化行为：

  1. clinit方法：编译器自动收集类中的所有变量的赋值动作，static{} 代码块行为，合并产生的；**编译器的收集顺序由语句在源文件 出现的顺序决定，静态代码块中只能访问到定义在静态语句块之前的变量，定义在静态代码块后的变量，代码块中可以访问，但是不可以赋值**
  2. clinit方法不同于类的构造函数，不需要显示的调用父类的 clinit，虚拟机会确保 子类的 clinit执行前，父类的clinit以及执行完毕：意味着，第一个执行的clinit 必定是 java.lang.Object类的clinit函数
  3. 由于父类 clinit 优先于 子类的 clinit，也即父类的变量 和 static{} 要优先于 子类的
  4. clinit 对于 类或接口 不是必须的，没有变量赋值 静态代码块，就可以不生产 clinit
  5. 接口中不能使用静态代码块，但有变量赋值操作，接口中也会有 clinit 函数生成，但接口不会 执行父类接口的 clinit，实现类也不会 执行接口的clinit，只有当访问到接口的字段时，才会初始化对应的接口执行clinit
  6. 虚拟机需要保证多线程环境下，一个类的clinit被正确的加锁同步



#### 7.4 类加载器

类加载器：由类本身和类加载器一起确定类的唯一性，类加载器不同的话，会影响 Class.equals() isAssignableFrom() isInstance() intanceof 的结果

**双亲委派模型**：

过程优先尝试使用 parent.loadClass，不得则自身 findClass

Java中已存在的结构有三层加载器：BootstrapClassLoader（\lib -Xbootclasspath添加的目录 无法被Java程序直接引用，使用null，则是默认根加载器）；ExtensionClassLoader （\lib\ext java.ext.dirs系统变量指定的目录 允许将通用性类库放在ext目录拓展Java SE 的能力 JDK9之后被模块的天然拓展能力取代）；ApplicationClassLoader（getSystemClassLoader()获取，负责加载 用户类路径classpath上的所有类库，如果没有自定义类加载器，则通常是这个加载器）；UserDefinitionClassLoader（用户自定义的类加载器）：：：使用组合方式，自下而上的双亲委托 加载类

三层结构是Java推荐的最佳实现，好处：1. Java中的类随着他的加载器一起具备了优先级的层级关系 2. 对保证Java的稳定运作很重要

双亲委派模型：有基础类型一致性的效果，越基础的类由越上层的类加载器加载

缺点是：对于动态性支持不足，OSGi就将双亲委派模型的树结构改为了网状结构，让bundle内的类的import语句委派给 Export类的bundle的类加载器加载，对于Dynamic import的Bundle，委派给Bundle对应的加载器加载



#### 7.5 Java 模块化 JPMS：Java Platform Module System

模块化目标：可配置的封装隔离机制

信息有：依赖  可读性 服务

模块化优势：

1. 解决JDK9之前只使用类路径来查找依赖的问题，由于模块有显式的依赖声明，可在运行前就检查依赖的完备性，从而避免很大一部分由类型依赖而引发的问题
2. 解决类路径上跨Jar文件的public类型的可访问问题，模块提供了更加精细的访问控制，也就是更加紧致的 命名空间，public要指明模块

**模块的兼容**：提出ModulePath与传统的ClassPath兼容，路径决定是传统Jar包对待，还是以模块对待

兼容的处理规则：

	1. 传统Jar文件路径：默认自动打包到一个匿名模块
 	2. 模块路径方法：匿名模块所有内容对具名模块是不可见的，因为无法显式声明一个匿名的依赖，即传统Jar路径的Jar对模块不可见
 	3. Jar文件在模块路径的访问规则：把传统Jar，不包含模块定义的Jar文件方式到模块路径中，以模块对待，就会变成一个自动模块（Automatic Module）

`java --list-modules` 查看所有模块

**模块化的最大驱动力**：支持多版本模块共存，运行时部署，替换；而不仅仅是提供一个受管理的命名空间

#### 7.5.2 模块化下的类加载器

保证兼容性，没有完全修改类加载器架构和双亲委派模型

修改：

1. 拓展类加载器 被 平台加载器 替换：整个JDK都基于模块化进行构建，rt.jar tool.jar 被拆分为MOD文件，拓展加载器的目的为了拓展JDK而存在，\lib\ext 目录就没有了存在意义；而且，新版JDK也取消了\jre 目录，因为随时可以组合构建出程序运行所需的JRE来，比如可以直接使用jlink，将java.base模块中的类型打包一个“JRE”；
2. 启动类加载器 平台加载器 应用程序加载器 都不在派生于URLClassLoader而是派生于BuildinClassLoader，BuildinClassLoader实现了模块化架构下类从模块中加载的逻辑，模块内资源访问以及与传统Jar包兼容

JDK9之后的三层类加载器与双亲委派模型：

BootClassLoader也是Java类，应用代码可引用了，但是保持习惯依然使用Null时默认使用启动类加载器

当PlatformClassLoader受到类加载的请求，在委派给父加载器前，会判断类型所属的模块，如果可以找到，则优先派发到对应模块的类加载器

JMPS明确规范指定了三个类加载器指定各自负责的模块



## 第8章 虚拟机字节码 执行引擎

执行字节码，根据VM的目标，可以选择 解释执行，编译执行，混合的方式实现；

外观上来看：任何虚拟机的输入输出都是一致的



### 8.2 运行时栈帧结构

StackFrame 是运行时支持 虚拟机进行方法调用和执行的 数据结构，“线程”

StackFrame 包括 局部变量表 操作数栈 动态连接 方法返回地址 和 一些额外的辅助信息，在class文件都能找到概念上对应的静态对照物

StackFrame 需要的局部变量表大小 最大栈深度，在编译时就能完全分析确定的，存在 方法表的Code属性中：一个栈帧所需要的内存在编译时就可确定，只决定于程序源码和具体的虚拟机实现的栈内存布局形式

一个线程的函数调用栈的所有栈帧都是执行状态，但只有栈定的栈帧是运行的，称为 Current Stack Frame，对应方法为 当前方法 Current Method

#### 局部变量表 Local Variables Table

一组变量值的存储空间，用于存放 方法参数和方法内部定义的局部变量，Code属性表中的 max_locals 数据项确定该方法所需的变量表的最大容量，容量的单位是变量槽 Variable Slot，变量槽可根据硬件决定，一般为32bit

虚拟机使用 索引定位，索引是变量槽的位置 偏移量，0~max_locals

对象实例的引用应具有的能力：1. 根据引用直接或间接的查找到对象的在Java堆内存的位置 2. 直接或间接的访问到对象类型在方法区中类型信息(Class对象) C++没开RTTI就没有这种能力 <= 能支持第二项，为何Java的泛型就是类型擦除的伪泛型呢？

调用方法时，虚拟机使用局部变量表来完成参数值到参数列表的传递；如果是实例方法，局部变量表索引0会传入对应对象的引用（this指针），其余参数按照参数表顺序排列；参数表分配完毕后，再根据方法体内定义的变量顺序和作用域分配其余的变量槽；局部变量槽是可重用的，当对应的 引用变量 离开作用域后，但是没有对 该引用变量所在位置的变量槽重用，而此方法没有结束，该引用变量是依然存在的，会影响 其引用的对象的回收

注意：局部变量表内存分配后不会有0初始化，也即如果局部变量声明了必须初始化才能使用，否则可能会有奇怪的值

#### 操作数栈 Operand Stack

栈，最大深度为Code属性的max_stacks指定

方法执行过程中，会有根据字节码指令往操作数栈中读写数据

不同栈帧的数据应该是独立的，但可以想象，在现实场景中，操作数栈的一部分是下一个函数的 参数，可直接作为下一函数输入参数，即操作数栈与下一个函数的局部变量表有重合的部分，以优化内存适合和参数复制

#### 动态连接

每个栈帧都包含 一个 指向运行时常量池中该 栈帧所属方法的引用，为了支持方法调用过程的动态连接 （标记栈帧所属的方法，由此可在运行时得知 方法所在的类 类的父类 接口，得知继承的上下文信息，以便多态的确定吗？）

#### 方法返回地址

方法的退出：只有两种：字节码指令退出和异常未处理退出

当前栈帧出栈：回复上层方法的局部变量表和操作数栈，将返回值压入调用者的操作数栈，调整PC值指向方法调用的后一条指令位置（取决于对应虚拟机的具体实现，概念上来说是这些过程）

#### 附加信息

Java虚拟机规范 允许虚拟机的实现 增加一些规范里没有的信息到栈帧之中，比如调试，性能等信息

### 8.3 方法调用

方法调用的任务：确定被调用方法的版本，尚未涉及运行时方法的字节码指令



解析和分派两小节关注的点不同，解析和分派不是互斥的，只是有相关性

#### 8.3.1 解析（关注的是在编译期还是运行期）

解析 Resolution：编译期可知，在类加载的解析阶段，将 符号引用转化为 确定的 直接引用

符合编译期可知，运行期不可变的 方法主要有 静态方法 和 私有方法两大类，前者与类型关联，后者与在外部不可被访问，决定了在编译期就可以确定下来



调用方法的指令：

- invokeStatic：对应静态方法调用
- invokeSpecial：对应实例构造器 init 方法，私有方法，父类中的方法
- invokeVirtual：对应虚方法调用
- invokeInterface：对应接口方法调用，**运行时**确定一个该接口的对象
- invokeDynamic：**运行时**动态解析出调用点限定符所引用的方法，再执行对应的方法

1，2 是在类加载解析阶段就能唯一确定方法

1，2，3，4  分派逻辑在Java内部固化，5是用户设定的引导方法来确定

1，2 支持的方法，以及被 invokeVirtual调用的带final的方法，五种称为 非虚方法 Non-Virtual Method，**静态解析 StatucResolution** ；其他称为虚方法，**动态解析 DynamicLinking**

#### 8.3.2 分派 多态（关注的是确定方法版本的搜索匹配行为）

外观类型（Static Type 或 Apparent Type）：变量声明的类型，类型转换后的类型；编译时就能确定

运行类型（Runtime Type 或 Actual Type）：数据的具体实际类型（用哪一个类构造器构造）；由于运行时数据和分支语句，随机数的存在，运行期才能确定

**静态分派**：所有通过外观类型来决定方法执行版本的分派动作；典型比如 方法重载；发生在编译阶段，因此确定静态分派的动作不是虚拟机执行的，而是编译器执行的；但是方法重载版本确定不是唯一的，因为**字面量有类型不确定性**，比如 'a' 可以匹配char int long Charater Serializable char... args Object等不同形参的重载

**动态分派**：根据数据的 实际类型 确定方法；与重写（override）密切关联；invokeVirtual 指令调用：确定数据的实际类型-在继承结构递归查找方法-没有则AbstractMethiodException；**invokeVirtual是方法的指令，当然 字段就没有多态性**

**单分派与多分派**：

单 与 多是基于宗量的个数决定的

宗量：方法的接受者 和 方法的参数 统称为方法的宗量

目前：Java 是静态多分派（外观类型会多次影响分派），动态单分派（动态分派：只关注方法的接受者类型这一个宗量）；后期可能会改变，支持动态多分派

Java目前发展没有直接变为动态语言的迹象，而是通过内置动态语言执行引擎，加强与其他虚拟机上动态语言交互能力的方式来间接满足动态性的需求

动态分派的实现：虚方法表和接口方法表来实现，方法表中存着各个方法的实际入口地址



### 8.4 动态语言支持 invokedynamic指令

动态语言 关键特征：类型检查的主体过程是在运行期而不是编译期进行的

动态语言没有 符号类型，没有变量的外观类型，只有数据类型

所以可以理解为：静态语言 通过编码时指定 符号的类型 这条信息，来进行一些类型检查服务，尽量多的让异常在运行前检查出来可行，有利于稳定性，利于大规模项目的少错误；而动态语言 不需要开发人员给定 符号的类型 这条消息，也就不会做一些类型检查服务，但是有极大的灵活性，提升开发效率。 《= 从这个角度来看，静态语言应该是在动态语言的基础上添加 符号的类型 的这条信息 升级出来的

invokedynamic指令和 java.lang.invoke 在JDK7出现的技术背景：语言层面无法高效解决多态性

**动态确定目标方法的机制：方法句柄 Method Handle**

之前方法参数只能是 类型数据，不能是函数，或者说没有 函数的类型

现在给出了 MethodType 语义是方法类型

**MethodHandle 与反射 Reflection的区别**

所处的级别不一样，反射操作的是 Java的Class对象，面向的是字节码；MethodHandle操作的是字节码，面向的是虚拟机

1. MethodHandlers.lookup() 提供了findStatic findVirtual findSpecial 对应于invokestatic，invokeVirtual invokeinterface， invokespecial 指令，是模拟字节码层次的方法调用；反射是在模拟Java代码层次的方法调用
2. 反射的Method对象包含方法的签名，描述符以及方法属性表中的各种属性的Java端表示；MethodHandle只包含执行该方法的信息

invokedynamic 是通过CallSite搜索执行一个方法，通过函数类型 在数据上搜索函数方法并执行；重点是其分派逻辑由程序员决定而不是虚拟机
