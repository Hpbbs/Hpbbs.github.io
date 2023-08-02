---
layout: post
title: "Android Dependency Injection: Dagger 基本使用"
published: true
categories: [ComputerScience]
tags: [Android,Framework]
---

### DI 简单介绍

依赖是什么：依赖是 数据对象与数据对象之间的主从关系，而且依赖项是必须的

依赖注入是什么：外界主动向数据对象提供依赖项的过程，“外界 主动 提供” --》 “注入”，而不用数据对象主动的去获取或者创建

- setter 可以注入 《--- 字段注入
- 函数参数可以传入，比如构造器 〈 --- 构造器注入
- 反射填充 《 -- 反射注入

依赖注入给予控制反转的原则：[控制反转](https://en.wikipedia.org/wiki/Inversion_of_control)



自动依赖注入是什么？

- 解决手动依赖注入的繁杂，以及大量的样板代码
- 在无法传入依赖项前构造依赖项，（例如，当使用延迟初始化或将对象作用域限定为应用流时），则需要编写并**维护管理内存中依赖项生命周期的自定义容器（或依赖关系图）**



有一些库通过自动执行创建和提供依赖项的过程解决此问题。它们归为两类：

- 基于反射的解决方案，可在运行时连接依赖项。
- 静态解决方案，可生成在编译时连接依赖项的代码。



**Dagger** 为您创建和管理依赖关系图，从而便于您在应用中使用 DI。它提供了完全静态和编译时依赖项，解决了基于反射的解决方案（如 [Guice](https://en.wikipedia.org/wiki/Google_Guice)）的诸多开发和性能问题。



###  三、Hilt：基本容器和基本使用

https://developer.android.com/training/dependency-injection/hilt-android

Hilt 通过为项目中的每个 Android 类提供容器并自动为您管理其生命周期，定义了一种在应用中执行 DI 的标准方法 ---- 看来Hilt的目标可不是那么简单

Hilt在Dagger的基础上构建而成



1. 添加gradle依赖



- @HiltAndroidApp：必须一个Application级别的根容器：@HiltAndroidApp 注释在重写的Application中
  - Hilt会生成代码，包括一个应用的基类（逆序包路径加上GeneratedInjector的路径），充当应用级别容器，会附加到Application对象的生命周期

在提供了应用级组件后，可以为 @AndroidEntryPoint 提供依赖项

- @AndroidEntryPoint：会为项目的每个Android类生成一个单独的Hilt组件，这些组件可以从他们的父类接受依赖项，需要访问依赖项（注入），可以使用 @Inject

> 由 Hilt 注入的字段不能为私有字段。尝试使用 Hilt 注入私有字段会导致编译错误



- @Inject 绑定：提供 “将哪一个类型的实例 作为哪一个类型的字段（依赖项）“的完整信息

  - 可以用在构造器上
  - 可以用在非private字段上 

- @Module Hilt 模块，它会告知 “Hilt 如何提供某些类型的实例” 的信息，您必须使用 `@InstallIn` 为 Hilt 模块添加注释，以告知 Hilt 每个模块将用在或安装在哪个 Android 类中；

  - 即@InstallIn提供的信息是@Inject类似的信息

- @Binds 修饰 接口中的抽象函数，提供定位信息

  - 函数返回类型会告知 Hilt 函数提供哪个接口的实例（还是依赖于类型系统，获得类型信息）
  - 函数参数会告知 Hilt 要提供哪种实现。

- @Provides 注入实例（如果某个类不归您所有（因为它来自外部库，如 [Retrofit](https://square.github.io/retrofit/)、[`OkHttpClient`](https://square.github.io/okhttp/) 或 [Room 数据库](https://developer.android.com/topic/libraries/architecture/room)等类），或者必须使用[构建器模式](https://en.wikipedia.org/wiki/Builder_pattern)创建实例，也无法通过构造函数注入。）

- 如果有多种注入依赖项候选，需要使用 限定符

  > ```kotlin
  > @Qualifier
  > @Retention(AnnotationRetention.BINARY)
  > annotation class OtherInterceptorOkHttpClient
  > ```



好用的地方来了！！！

Hilt 提供了一些预定义的限定符。例如，由于您可能需要来自应用或 Activity 的 `Context` 类，因此 Hilt 提供了 `@ApplicationContext` 和 `@ActivityContext` 限定符



Hilt 目前支持以下 Android 类：也就是不同的Scope

- `Application`（通过使用 `@HiltAndroidApp`）
- `Activity`
- `Fragment`
- `View`
- `Service`
- `BroadcastReceiver`
- 如果您使用 `@AndroidEntryPoint` 为某个 Android 类添加注释，则还必须为依赖于该类的 Android 类添加注释。例如，如果您为某个 Fragment 添加注释，则还必须为使用该 Fragment 的所有 Activity 添加注释。
- Hilt 注入的类可以有同样使用注入的其他基类。如果这些类是抽象类，则它们不需要 `@AndroidEntryPoint` 注释。: 抽象类可以直接继承拓展

> **注意**：在 Hilt 对 Android 类的支持方面还要注意以下几点：
>
> - Hilt 仅支持扩展 [`ComponentActivity`](https://developer.android.com/reference/kotlin/androidx/activity/ComponentActivity) 的 Activity，如 [`AppCompatActivity`](https://developer.android.com/reference/kotlin/androidx/appcompat/app/AppCompatActivity)。
> - Hilt 仅支持扩展 `androidx.Fragment` 的 Fragment。
> - Hilt 不支持保留的 Fragment。



#### 到底什么是为Android类生成的组件？

对于您可以从中执行字段注入的每个 Android 类，都有一个关联的 Hilt 组件，您可以在 `@InstallIn` 注释中引用该组件。每个 Hilt 组件负责将其绑定注入相应的 Android 类

| Hilt 组件                   | 注入器面向的对象                           |
| :-------------------------- | :----------------------------------------- |
| `ApplicationComponent`      | `Application`                              |
| `ActivityRetainedComponent` | `ViewModel`                                |
| `ActivityComponent`         | `Activity`                                 |
| `FragmentComponent`         | `Fragment`                                 |
| `ViewComponent`             | `View`                                     |
| `ViewWithFragmentComponent` | 带有 `@WithFragmentBindings` 注释的 `View` |
| `ServiceComponent`          | `Service`                                  |

| 生成的组件                  | 创建时机                       | 销毁时机                  |
| :-------------------------- | :----------------------------- | :------------------------ |
| `ApplicationComponent`      | `Application#onCreate()`       | `Application#onDestroy()` |
| `ActivityRetainedComponent` | `Activity#onCreate()`          | `Activity#onDestroy()`    |
| `ActivityComponent`         | `Activity#onCreate()`          | `Activity#onDestroy()`    |
| `FragmentComponent`         | `Fragment#onAttach()` ！！！！ | `Fragment#onDestroy()`    |
| `ViewComponent`             | `View#super()`                 | 视图销毁时                |
| `ViewWithFragmentComponent` | `View#super()`                 | 视图销毁时                |
| `ServiceComponent`          | `Service#onCreate()`           | `Service#onDestroy()`     |

**默认情况下，Hilt 中的所有绑定都未限定作用域。这意味着，每当应用请求绑定时，Hilt 都会创建所需类型的一个新实例**

不过，Hilt 也允许将绑定的作用域限定为特定组件。Hilt 只为绑定作用域限定到的组件的每个实例创建一次限定作用域的绑定，对该绑定的所有请求共享同一实例

| Android 类                                 | 生成的组件                  | 作用域                   |
| :----------------------------------------- | :-------------------------- | :----------------------- |
| `Application`                              | `ApplicationComponent`      | `@Singleton`             |
| `View Model`                               | `ActivityRetainedComponent` | `@ActivityRetainedScope` |
| `Activity`                                 | `ActivityComponent`         | `@ActivityScoped`        |
| `Fragment`                                 | `FragmentComponent`         | `@FragmentScoped`        |
| `View`                                     | `ViewComponent`             | `@ViewScoped`            |
| 带有 `@WithFragmentBindings` 注释的 `View` | `ViewWithFragmentComponent` | `@ViewScoped`            |
| `Service`                                  | `ServiceComponent`          | `@ServiceScoped`         |

确保在生命周期内提供同一Instance

如何将绑定的作用域限定为 Hilt 模块中的某个组件。绑定的作用域必须与其安装到的组件的作用域一致



- @EntryPoint : 入口点是由 Hilt 管理的代码与并非由 Hilt 管理的代码之间的边界。它是代码首次进入 Hilt 所管理对象的图的位置。入口点允许 Hilt 使用它并不管理的代码提供依赖关系图中的依赖项。如需访问入口点，请使用来自 `EntryPointAccessors` 的适当静态方法。参数应该是组件实例或充当组件持有者的 `@AndroidEntryPoint` 对象。确保您以参数形式传递的组件和 `EntryPointAccessors` 静态方法都与 `@EntryPoint` 接口上的 `@InstallIn` 注释中的 Android 类匹配



#### Dagger 于 Hilt：

Hilt就是Dagger的预定义和预处理

关于 Dagger，Hilt 的目标如下：

- 简化 Android 应用的 Dagger 相关基础架构。
- 创建一组标准的组件和作用域，以简化设置、提高可读性以及在应用之间共享代码。
- 提供一种简单的方法来为各种构建类型（如测试、调试或发布）配置不同的绑定。

因此在 Android 应用中使用 Dagger 要求您编写大量的样板。Hilt 可减少在 Android 应用中使用 Dagger 所涉及的样板代码。Hilt 会自动生成并提供以下各项：

- **用于将 Android 框架类与 Dagger 集成的组件** - 您不必手动创建。
- **作用域注释** - 与 Hilt 自动生成的组件一起使用。
- **预定义的绑定** - 表示 Android 类，如 `Application` 或 `Activity`。
- **预定义的限定符** - 表示 `@ApplicationContext` 和 `@ActivityContext`。



#### 四、 xxx

**依赖项注入会为您的应用提供以下优势：**

- 重用类以及分离依赖项：更容易换掉依赖项的实现。由于控制反转，代码重用得以改进，并且类不再控制其依赖项的创建方式，而是支持任何配置。
- 易于重构：依赖项成为 API Surface 的可验证部分，因此可以在创建对象时或编译时进行检查，而不是作为实现详情隐藏。
- 易于测试：类不管理其依赖项，因此在测试时，您可以传入不同的实现以测试所有不同用例。

### 依赖注入的替代方案

服务定位器设计模式还改进了类与具体依赖项的分离。您可以创建一个名为服务定位器的类，该类创建和存储依赖项，然后按需提供这些依赖项



与依赖项注入相比：

- 服务定位器所需的依赖项集合使得代码更难测试，因为所有测试都必须与同一全局服务定位器进行交互。
- 依赖项在类实现中编码，而不是在 API Surface 中编码。因此，很难从外部了解类需要什么。所以，更改 `Car` 或服务定位器中可用的依赖项可能会导致引用失败，从而导致运行时或测试失败。
- 如果您想将作用域限定为除了整个应用的生命周期之外的任何区间，就会更难管理对象的生命周期。

#### 基本原理

#### 重点概念，核心逻辑

#### 实现上的重点技术，使用上的重点概念



## Android 最佳实践

- 编译时生成代码，而非运行时反射，比起反射有性能优势
- 管理依赖项
- 



### 思考

依赖注入主要解决的问题是：

- 围绕的问题核心：C供给A给B使用：A - B - C 这种依赖的问题；A 怎么知道给这个B的问题；C如何管理A的问题；注入后，A的管理权变更后的问题

- 概念上对紧密关联的两个对象内存的关系 的描述；也就是 二元递归关系的管理
- 同时暴露出：其实对于A 这个对象的管理，就在于确定scope，而这个人scope本身也是GC关注的问题



目标解决问题的沿途发展：

- 工厂设计模式
  - 类和外部都被考虑在内，是一种主动和被动的组合 
  - 更加关注“注入”这个动作的细节，什么时候注入，使用什么注入，根据当前目标对象的需要 上下文情况来明确：“使用什么如何组装”这个细节问题
- 服务定位器设计模式
  - 使用服务定位器模式，类可以控制并请求注入对象，是内部主动
- 依赖注入
  - 使用依赖项注入，应用可以控制并主动注入所需对象，是外界主动
  - 更加关注依赖关系，关注依赖项的整个流程，而不关注“如何注入的，细节”，目标是消除模板代码

这几种解决方案的优劣？



其实viewModel本身就是一种依赖注入！！！ 但是其scope很明确，所以直接存储在对应 FragmentManager 的 viewModelStore 或者Activity的viewModelStore里面 -------------- 其核心就是：主动的切换对应依赖项 数据对象 的 可访问性 以影响 JVM内存的生命期

viewModel 本身就是一种逻辑容器



**最基本的使用流程：生产消费**

 提供对象对另一个对象使用，但是要负责创建的对象的管理

所以分为三小步：生产，管理，消费

提供的信息是：构造什么类型的对象，注入到哪里，并切这个对象存放在哪里，什么时候删除？

```mermaid
graph LR;

Provide --> Component
Component --> EntryPoint
```





### Dagger 官网

- Dagger is a fully static , compile time code generation for Java Kotlin and Android（特性明确）
- Aims address many of the development and performance issues that have plagued reflection-based solutions（定位精准）

- Maintain by Google（大厂背书）



replacement of **FactoryFactory** [dependency injection](http://en.wikipedia.org/wiki/Dependency_injection) design pattern without the burden of writing the boilerplate 证明了我之前理解的正确性 

使用标准 javax.inject annotaion JSR330， 每个类都很容易测试：注入一个Fake Object，是DI的一个非常重要的使用场景

##### 设计上实现上相关的问题

- 在实现上，是代码生成，也就是编译原理的部分内容

- 优秀的代码生成两个领域，值得思考

- 在对象管理里上：作用域管理，类似GC
- 提供涉及到Builder模式，以及Lazy Wrapper模式



##### Dagger 比起基本JSR330的能力

But `@Inject` doesn’t work everywhere:

- Interfaces can’t be constructed. 接口
- Third-party classes can’t be annotated. 第三方类
- Configurable objects must be configured! 被配置的，限制了注入入口

##### Building the Graph



## JSR330 javax.inject

Named

Inject

Provider

Qualifier

Scope

Singleton



```java
@Target({ METHOD, CONSTRUCTOR, FIELD })
@Retention(RUNTIME)
@Documented
public @interface Inject {}

@Target(ANNOTATION_TYPE)
@Retention(RUNTIME)
@Documented
public @interface Qualifier {}

@Qualifier
@Documented
@Retention(RUNTIME)
public @interface Named {

    /** The name. */
    String value() default "";
}

public interface Provider<T> {

    /**
     * Provides a fully-constructed and injected instance of {@code T}.
     *
     * @throws RuntimeException if the injector encounters an error while
     *  providing an instance. For example, if an injectable member on
     *  {@code T} throws an exception, the injector may wrap the exception
     *  and throw it to the caller of {@code get()}. Callers should not try
     *  to handle such exceptions as the behavior may vary across injector
     *  implementations and even different configurations of the same injector.
     */
    T get();
}

//Dagger 有很多基础Provider的子类
  
@Target(ANNOTATION_TYPE)
@Retention(RUNTIME)
@Documented
public @interface Scope {}

@Scope
@Documented
@Retention(RUNTIME)
public @interface Singleton {}
```



总的而言就是：Class Provider 提供，Scope限定同等类型，Qualifier Name业务语义限定，Inject注入



文章推荐

[1] https://proandroiddev.com/deep-dive-into-dagger-generated-code-part-1-58f3cb9563de
