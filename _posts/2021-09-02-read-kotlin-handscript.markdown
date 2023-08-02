---
layout: post
title: "读书摘要-kotlin编程权威指南"
published: true
categories: [ComputerScience, Book]
tags: [Android]
---

# Kotlin编程权威指南

{:toc }

本文从类型系统，面向对象系统，内存管理系统，执行系统，Backend平台特性，总体语言语法设计特点 等方面入手，着眼于语法层面，会与Java，C/C++ 静态语言以及Python JavaScript动态语言对比 介绍Kotlin编程语言，Kotlin语言设计哲学的第一性问题，为何要设计Kotlin，主要解决了什么问题，适用于什么场景

## 总结：特色Kotlin主义编程

1. 静态类型系统 配合 强大的类型推断 让kotlin代码有动态语言的使用 变量的简洁语法体验
2. 将所有逻辑代码块 都 优化为了表达式，让{ ... }的内部表达式 处理为同 函数作用域，将表达式赋予了代码块的强大能力：极大的方便了代码的表达，而且编写编译器的代码，用函数的作用域，生命周期来实现 代码块，也是quickjs引擎的必要实现方式；用匿名函数实现 lambda表达式实现了 函数与表达式的统一
3. 让编译器的编译行为可控，而且让开发者能在同一套代码上控制编译器的行为 : 比如Notiing数据类型 
4. 各种语法糖，都是为了像 人类语言一样去表达代码，比如 infix中缀函数，最后一个参数为lambda表达式可将{}放在入参列表（）之后，函数只有一行表达式可以省略{} etc.
5. 利用语法设计将可能运行时可能出错提前到编译时解决：对于数据 的 读写性为符号通用性质 可空性附加与类型系统 的处理方案，静态类型推断
6. 优秀的异常处理方案
7. 函数处理为一种 数据类型
8. 与Host的平台有很好的互操性，对Host平台使用同一套抽象方案，比如Any类型对应Java 的Object类，在Javascript中对应object，但是方法完全不同
9. 基于类型系统，添加可控性作为 null 的解决方案

## 第1章 使用IDEA体验Kotlin开发

IntelliJ 使用 kotlinc-jvm 编译Kotlin代码为字节码，运行在JVM平台上，JVM平台本质是一个软件，可以读取 .class 文件并执行

Kotlin REPL read-execute-print-loop

## 第2章 数据类型

类型检查type check 是kotlin一个特色

定义变量：`var VariableName:Int = 5` 分别是 符号声明关键字，符号名，符号绑定的数据类型，符号绑定的值

符号声明关键字：var 或者 val：**语意上不仅仅是符号声明，还是可读性的体现**， 可以表达 是可变符号（Variable）还是 数值（Value），分别对应关键 var 和 val

### 类型推断 类型检查 类型系统

现况：在强类型语言中，可见Java 和 C/C++，会对数据类型的声明明确要求，且是语法正确性检查的一部分，优点是在编译时，就能分析出类型不匹配导致的错误，让运行期的错误尽量提前，缺点是语法拖沓繁杂；在动态语言如Python中，不要求声明数据变量的类型，只用直接声明一个 变量名称的符号，在使用之前，正确的初始化复制便好，语法精炼，符合人的主观行为，缺点是在符号/变量 调用时候，需要运行时绑定符号与类型数据和符号定义搜索，在性能上有成本

解决：Kotlin语言作为2016年推出的语言，企图在语法上有动态语言的优势，在性能上又有静态语言的特点，整合 声明时非必要不提供数据类型的简洁语法 与 编译时类型明确 的工具便是类型推断，倒不如理解为类型传播，对于同一个变量数据，只用在数据源头明确类型，其他相关的衍生数据的类型都可以推断出来，比如一个变量的数据类型是 String，那么String的各种函数操作的结果的类型就是隐式明确的，可以依赖于类型推断系统推断出来，如果使用的IDEA编辑器的话，会用灰白色警告冗余的类型信息

总结起来，**Kotlin使用的是 静态类型系统（Static Type System）配合 类型推断 类型检查 **

### Kotlin内置数据类型

String char Boolean Int Double List Set Map

### 编译时常量：字面量的优化版本，给编译器使用，而不直接用于执行系统，对于执行系统而言，就是字面量

绝对只读，只能在函数之外定义（编译时常量必须在编译时赋值，而函数等都是在运行时才会调用），相当于cpp的 constexpr

`const val MAX_EXPERIENCE:Int = 5000`

### 学会查看Kotlin的字节码

kotlin编译出来的Java字节码 （IDEA action: Show Kotlin Bytecode）

### Kotlin中的Java基本数据类型

Java提供了基本数据类型，如int byte 等，也提供了引用类型 Integer，**Kotlin只提供引用类型**

只要可能，处于高性能的需求，Kotlin编译器会在Java字节码中改用基本数据类型

## 第3章 条件语句

 ### 1. 条件

比较运算符

=== 是 引用比较：两个符号绑定的是不是同一个实例，同一个内存地址，相当于Java的 ==

== 是 结构比较：两个对象在语义上是否相等，相当于Java的equals

其他： < > <= >= != !==

逻辑运算符

&&: 逻辑与：当两者都为 true，才为true

!!: 逻辑或：任一为true，结构为true

!: ；逻辑非：取反

### 2. 条件语句（语句代码块）

```kotlin
if( condition ){}
else if (condition2) {}
else {}
```



### 3. if条件表达式（表达式）

```kotlin
val a = if(){}
	else if(){}
	else if(){}
	else {}

val str = if(true) "string content" else "other content" //分支只有一条语句，可以省略 代码块分割{}
```



### 4. range （表示范围的概念）.. 操作符（提供非常直观的语法，利于DSL）

提供高效的范围 生成 集合的操作，类比MATLAB的m语言的语法，但是能力比较差，不能提供步长，也没有类似的切片

语法规则： start..end   表示集合 [start, end]

辅助： in 操作符 `in 1..5` 判断

```kotlin
if( a in 34..78){ ... }
```



其实现的原理:

`34..78`  对应 `34.rangTo(78)`, 是对应调用的数据类型的 rangTo函数重载

此外还有 downTo

```kotlin
public infix fun Int.downTo(to: Int): IntProgression {
    return IntProgression.fromClosedRange(this, to, -1)// 返回了一个Iterator
}
```

此外还有 util： 1 util 3

```kotlin
public infix fun Int.until(to: Int): IntRange {
    if (to <= Int.MIN_VALUE) return IntRange.EMPTY
    return this .. (to - 1).toInt()
}
```



### 5. when 表达式

```kotlin
val faction = when(argument){
  rule1 -> expr-with-return
  rule2 -> expr-with-return
  else -> expr-with-return
}
```

原理：是函数常量

### 6. String 模板 （提供的基本模板功能）(弱化功能的EL expression language)

$name: 取符号内容

${ ...statement } : 稍复杂语句内容 DSL

 ## 第4章 函数

函数 是语句的集合，**复用代码逻辑，但不复用数据**，所以递归调用函数才会 重复创建函数帧

区分 复用数据 还是 复用逻辑

### 1. 使用函数复用逻辑

函数结构：`private fun functionName(argument1:Type, argument2:Type):ReturnType  { function body }`

函数头：可见性修饰符 函数关键符 符号（函数名） 入参列表（值参列表） 返回值类型

函数体：语句的集合 

### 2. 函数调用：functionName( argument list )

() : 可以理解为 函数调用操作符

### 3. 默认值参：在函数头提供默认值

```kotlin
private fun functionName(argument:Int = 4){
  ...
}
```

### 3.5 具名函数参数：在函数调用传入值参是指定变量名，可不遵循函数头声明值参列表顺序（提高代码灵活性）

```kotlin
private fun functionName(value:Int, name:String, default:Int = 2)

functionName(name = "abv", value = 12, default = 12)
```



### 4. 单表达式函数：当函数只有一条语句省略{}

```kotlin
private fun formatHealthStatus(healthPoint:Int, isBlessed:Boolean) = when(healthPoint){  ... }
```

### 5. Unit 函数，Unit类型

不是所有函数都有返回值，有些函数利用副作用 side effect 来执行任务，比如改变变量的状态，根据输入产生系统输出，日志等

```kotlin
private fun castFireBall(numFireballs:Int) = println("cast $numFireballs fireballs")
// 等价于
private fun castFireBall(numFireballs:Int):Unit = println("cast $numFireballs fireballs")
```

Kotlin 称这种函数为 Unit函数，返回值为 Unit 类型

```kotlin
public object Unit {// Unit类型是一个object对象类型
    override fun toString() = "kotlin.Unit" // toString函数返回值
}
```



### !6. 语言设计：如何表达空？Kotlin的解决方案

编程语言表达空的问题：由于通用的编程语言都引入了类型系统，让 空 的概念在类型系统之上实现是个问题

比如 用void表达 函数不返回任何东西，void关键字流于表面：如果函数不返回东西，就忽略他的类型；并且void无法解释现代语言重要特征： 泛型  

而Unit概念就是解决这个问题：表示返回符号 没有数据内容，但符号绑定有类型信息的函数，就与泛型系统兼容

**符号绑定有 类型数据 内容数值**：如果 空 的概念，应该由两部分组成，类型信息为空  内容数据为空

c中 void修饰变量表示 没有类型，但是有数值内容，void修饰函数表示 函数返回 没有内容数值，也没有类型信息，所以void在c的概念中会有歧义

kotlin中 Unit类型表示 没有数值内容，但是有类型信息，所以在 Kotlin 代码上体现为 一个没有内部数据的单例，只是一个类型而已

而在Java中，void与c中void类似，但是有一个自动装箱操作，void自动装箱为Void类型，代表空类型，如果打印toString，"null"

### !7. Nothing 函数类型：意味着函数不可能成功执行完成

TODO() 函数就是

### 8. Java的文件级函数

kotlin文件里不与 类关联的函数 在 Java里体现为 以文件名为类名的 static 函数

### 9. 反引号中的函数名：避免Java的任何潜在冲突

比如kotlin与运行Java平台有不同的关键字，比如 kotlin中 is 是保留关键字，但在Java中不是：

```java
public static void is(){
  ...
}
```

```kotlin
fun doStaff(){
  `is`() // 在kotlin中调用 java 的 is函数
}
```



## 第5章 匿名函数 与 函数类型

函数 本身于是一种数据

函数体 其实就是函数字面量，val a = { 函数字面量 } 与  val str = "字符串字面量" 没有任何区别

函数体的代码只不过是 函数这种数据类型的 数据信息，是表达式的集合，以及函数上下文，在执行系统里的生命周期 的组合概念 

**在Kotlin中，一个函数的完整信息组成：可见性信息，所绑定的类（拓展），函数头/函数签名，函数体（函数字面量）；函数签名组成：函数名标识符（将函数复制到变量中，函数标识重绑定），函数的返回数据类型，函数的入参列表**



**数学定义中lambda表达式**，意味着lambda应该是表达式，GPL中都用匿名函数 来 实现lambada表达式，函数和表达式的概念需要在编程语言中统一

### 1. 匿名函数（省略了符号绑定的函数）

语法形式：

```kotlin
{ argumentlist -> expressions }
{ expressions } 
```

调用：{...}()

#### 1.1 函数类型 function type

注意 函数类型 不同于 有个叫Function的类型，函数类型是 函数签名的信息，这种函数类型信息存储在 Function类型的数据里，是编译器要封装保存的，执行系统需要的，在Kotlin中 将不同的 函数类型 处理为一种类型，让其将函数赋值给一个符号的语法有效（可理解为 函数类型 是 Function类型的子类，但是也是有冲突）

```kotlin
:(形参列表) -> 返回值类型

// for example
val greeting : ()->String = {
  "Hello world"
}

// for example 

fun greetingFunction():String 
// 函数类型为：()->String
```



#### 1.2 隐式返回：没有return关键字，默认返回最后一个表达式的值

#### 1.3 it 关键字：只有一个参数的匿名函数，可用it替代其变量名

```kotlin
val greetingFunction:(String) -> String = {"hello $it"}
```

#### 1.4 匿名函数多个参数

同样支持函数类型推断

```kotlin
val greetingFunction:(String,Int)->String = {name,times -> "hello $name for $times"}
val greetingFunctin = {name:String,times:Int -> "hello $name for $times"}
```



### 3. 函数lambda参数

同样，明确了函数类型，函数变量名后，按照变量来使用便好，但是这个变量的操作只有 （） 函数调用操作符

简略语法：如果函数的lambda参数位于参数列表最后一个，或者只有一个lambda参数，就可以将lambda表达式放到函数调用的结尾，以便更加直观书面



### 4. 函数内联 inline：对lambda表达式运行时封装的对象的优化

在JVM平台上，定义的lambda表达式会以对象实例的形式存在（以对象的方式，给lambda表达式一个上下文），JVM会为所有同lambda打交道的 变量分配内存，就会产生内存开销，还会带来严重的性能问题，函数内联就是优化的一个措施：用 inline 函数**标记使用lambda的函数**即可

```kotlin
inline fun runSimulation(palyName:String, 
                        greetingFunction:(String,Int)->String){
  greetingFunction("xxx", 2)
}

fun op(){
  runSimulation("liutao"){
    name ,times -> println("hello $name for $times")
  }
}
```

Inline 关键字作用：inline标记后函数runSimulation后，调用runSimulation就不会使用lambda对象实例，哪里需要使用lambda，编译器就会将函数体 复制粘贴到哪里，**主要是避免了 lambda函数作为函数参数传递时，必须要封装为对象实例来传引用**

ps：使用lambda的递归函数无法内联

### 5. 函数引用：将以fun定义的函数（具名函数）转换为一个值参( ::functionName)

凡是使用了lambda表达式的地方，都可以使用 函数引用

:: 取函数引用操作符



### 6. 函数类型 作为返回类型

kotlin的lambda是闭包，可以访问lambda表达式所在函数内的变量，匿名函数引用着所在函数的变量

```kotlin
fun returnLambda():(String)->Unit{
    var a = 0
    a++
    return {
        println("$it"+ ++a)
    }
}

fun main(args:Array<String>){

    for (a in 1..7){
        returnLambda()("liutao")
    }
}
```



### 8. 为什么要使用匿名函数和函数类型：对于只出现一次的函数，匿名函数可以减少模板代码

对比Java8的lambda，需要先声明接口，作为kotlin的函数类型 的补充，让编译系统通过



## 第6章 null安全 与 异常机制

### 1. 可控性：变量值是否可以为null

### 3. 编译时间与运行时间

kotlin是一门编译语言，代码要先编译成机器语言指令，再由编译器的特殊程序执行

编译时捕获的错误为编译时错误，运行时出错为运行时错误，编译时错误要好过运行时错误

### 4. kotlin null解决方案

kotlin在类型系统的基础上，让类型有可控性 性质，编码时就明确 对应的数据是否可以为空

kotlin不允许在可空类型值上调用函数，除非手动接管安全管理

1. ?.      :跳过函数调用
2. ?:      :提供备选值
3. ?.let  :在let函数作用内做null检查和处理
4. !!.      :强制调用
5. If       :if作用域内做null检查和处理，作为不是null的条件分支里，编译器知道变量值不可能为null，就可以直接使用"."调用函数

### 5. 异常机制

throw 可以手动抛出异常

try-catch 可以主动捕获可能的异常：**try语句块内的代码是按照定义顺序执行的，就是没有指令重排序和超标量流水优化机制**

#### 1. 先决条件函数 类似于 assert：kotlin提供的内置检查函数

checkNotNull

require

requireNotNull

error

assert



### 7. null讨论：null值即没有值，是真实世界的常态

是对应类型的所有可能取值的 反集

符号绑定的 数据内容为 不存在，而存在的状态构成了 对应类型的所有可能取值



### 8. 已检查异常 和 未检查异常

kotlin世界，所有异常都是未检查异常，unchecked 异常：对于是否要用 try-catch 包裹可能导致异常的代码，不做强制要求

Java同时支持 已检查异常，编译器强制检查，必须用try-catch 语句，以确保异常都做了防范处理：但是在实际开发场景中，往往容易容易出现，虽然异常被捕获了，但在处理时又直接忽略掉了，即所谓的 吞异常，让代码反而更难以调试

异常也是一种正常的程序返回状态，是不可或缺的运行时信息

### 9. null的Java互操

java代码中应该使用 @NotNull 和 @Nullable 保证kotlin能正确的与Java代码互操

kotlin定义的任何函数都会按照kotlin的可控性规则工作，哪怕是以及转译为JVM上的Java代码



## 第7章 字符串 & 操作

字符串：一串有序的字符，常用 连续内存数组为内存形式

kotlin的字符串都是都是不可变的，无论是 var 或者 val，var可以将符合重新绑定数据，但是原先字符串的内容是不可以更改的

### 1. 字符串截取

```kotlin
val indexOfXXX = StringA.indexOf('\`')
val newString = StringA.subString(0 util indexOfXXX)
// substring:截取并返回一个新的字符串

val (type, name, price) = StringB.split(',')
// splite 返回切割后的List集合


```

### 2.字符串替换

replace 可以使用正则表达式

简单使用

```kotlin
phrase.replace(Regex("aeiou")){
  when(it.value){
    "a" -> "4"
    "e" -> "3"
    "i" -> "2"
    "o" -> "1"
    "u" -> "0"
    else -> it.value
  }
}
```



### 3. 字符串比较

引用相等： === 两个符号是否绑定在同一个堆对象

结构相等（语义上相等）; equals() / ==

### 4. 遍历字符串

`StringA.forEach(::println)`

## 第8章 数

Kotlin的所有数字类型都是有符号的

字符串数值转换：toFloat toDouble toDoubleOrNull toIntOrNull toLong toBigDecimal

Double类型格式化： `"${"%.2f".formate(doubleNum)}"`

Double类型转换为Int：toInt(), 会存在精度丢失，小数部分被抛弃的策略，kotlin.math.roundToInt 则会执行四舍五入的策略

位运算：Integer.toBinaryString(), shl(bitcount), shr(bitcount), inv() inverse 按位取反, xor(number) 按位异或, and(number) 按位与



## 第9章 标准库函数

支持lambda的通用工具类标准函数

### 1. apply 配置函数，在lambda表达式里，每个配置函数都能隐式作用于接受者，返回：当前接受者，方便链式调用

### 2. let it 关键字能引用变量，可用于任何类型的接受者，返回：最后一行表达式的值

### 3. run 作用域行为与apply差不多，返回：最后一行表达式的值

### 4. with 是run的变体，功能和行为都差不多，但是with 第一个参数需要值参，场景少

### 5. also 类似于let ，把接受者作为值参传递给lambda，绑定到it，但返回 接受者对象，方便链式调用

### 6. takeIf 需要提供predicate条件，成立 true，返回接受者对象，不成立 false，返回null ，适用于有条件的调用，很有用

### 7. takeUnless 是takeIf的相反情况，最好不用



## 第10章 List Set

List：列表 有序，允许重复，元素数据类型必须要一样，创建{ listOf mutableListOf toList toMutableList }零索引存放，访问 [index] getOrElse，元素检查 contains(item)/containsAll(list)；add  addAll += -= remove 清空clear，遍历 for（item in list）{operation }利用迭代器函数；list.forEach{ operation } forEachIndexed；随机洗牌 list.shuffled() ; first() last()，只有list支持解构

Set：集合 无序，不允许重复（会自动去重），不会自动做索引排序（无index 无[index]访问）有elementAt 访问；+= -= add addAll remove removeAll(list/set) 删除所有值相同的元素 clear



读取文件 

```kotlin
import java.io.File
var menuList = File(pathString).readText().split("\n")
```

去重后排序

`slist.toSet().toList()`

### 1. 数组类型

kotlin也支持各种Array引用类型，**会编译成Java的基本数据类型**，为了与Java互操作

IntArray DoubleArray ByteArray BooleanArray等等



### 2. 只读与不可变

kotlin的 不可变性 是在类型系统之上的，如果使用 as 做类型转换，那么可变性也会随之更改

```kotlin
var myList:List<Int> = listOf(1,2,3)
(myList as MutableList)[2] = 100
myList
1，2，100
```



## 第11章 Map



map：存放一组元素，默认只读，使用参数化类型告诉编译器的集合类型，支持遍历，键具有唯一性，

mapOf（k to v，k to v） `infix to() 返回一个Pair `；shuffled() 混淆，+= 新加入的同键的Pair，会替换掉旧 value；

读取map值：支持按键索引 [key] getValue(key) getOrElse(key) getOrDefault(key)

`aMap["key"]=value` "=" 添加键值对或者更新value

put putAll 

getOrPut `getOrPut(key){ insertValue }` 存在则返回键值，不存在则插入键值

-= -

clear

### 遍历

```kotlin
aMap.forEach(){ key, value -> operation }
```



## 第12 章 类 定义类

### 1. 可见性与封装性

默认可见性为 公开可见

public default 对外部可见

private 类内部可见

protected 类内部或子类可见

internal 同一模块可见

**kotlin 中，访问可见性与 继承可见性不同**

### 5. 类属性

由 var val 控制属性值为只读还是可变

#### 5.1 属性field

属性还提供简洁的语法：setter getter

kotlin为每一个属性产生一个field，一个getter，一个setter（如果需要，定义为var）；你不能直接定义field，kotlin会封装field，保护其中数据，只暴露给setter和getter使用；只有var 属性才有setter

1. 可以自定义setter getter行为，通过覆盖函数

2. 属性的setter和getter可以独立设置可见性
3. 计算属性，kotlin不会为之生成 field
4. var 和 val定义的属性的区别就是属性值是否可改（setter方法没了）

```kotlin
class Player {
  var name = "default"
  	get() = field.capitalize()
  	set(value) {
      field = value.trim
    }
  
  var name = "default"
  	get() = field.capitalize()
  	private set(value) {
      field = value.trim
    }
  // 这个compute是计算属性，应为每次访问Player().compute 都会调用get() 代码生产，他始终没有初始值或默认值，就不用支持字段来存储值
  var compute
  	get() = (1..6).shuffled().first()
  // 只读属性
  val readOnlu:String
  	get() = "content"
  fun other(){}
}
```

### 7. 包： 同Java的概念，是一个代码文件的逻辑分组

### 8. 防范竞态条件

如果一个类属性即可空又可变，那么引用它之前必须保证它非空

### 9. internal可见性允许同一模块内共享类文件，对开发kotlin库文件很有用



### 第13章 初始化

如何在初始化时加入一些逻辑：主构造函数里，声明时初始化，次构造函数里，初始化代码块里

初始化顺序：主构造函数声明的属性 - 类级别的属性赋值 - init 代码块 - 次构造函数 **类级别属性的初始化与init的顺序由代码位置决定**

构建类实例时，类属性都必须完成初始化，这是规则是kotlin空值安全系统的重要部分，但有变通，见延迟初始化

```kotlin
//主构造函数 如果为类指定了主构造函数，那么所有的实例都要使用主构造函数构造，次构造函数只是多了一个入口，任然要使用 this 调用主构造函数
class Player(_name:String, val health:Int, var color:Color=Color.Green, private isImmortal:Boolean){
  // 属性必须在构造类实例时完成初始化，，必须给属性赋初始值
  var town:String = "Beijing"
  // 如果类属性有复杂的初始化逻辑，应该考虑放到函数或者初始化块里处理
  var desc
  	get() = "name is: $_name"
  
  lateinit var alignment:String
  fun use_alignment(){
    // 校验属性是否初始化
    if(::alignment.isInitialized) println(alignment)
  }
  // 惰性初始化
  val leagl = by lazy { choose_a_leagal() }
  
  
  //init 代码块：设置变量或者值，执行一些有效性检查；不管调用主构造 还是次构造， init都会在类实例构造时执行
	init {
    require(health > 80, {"health value must bigger than 80"})
    town = "Changping"
  }
  
  //次构造函数： 要么直接调用主构造函数，要么简洁调用主构造函数初始化
  constructor(_name:String):this(_name, 100, false){
    town = "HaiDian"
  }
  
  constructor():this("unknow",100,false){
    town = "xxxx"
  }
  
  
  

}
```

Java基本类型int的默认值是0

### 延迟初始化

1. lateinit修饰属性，与编译器交互，告诉编译器不强制要求属性在类实例构造时初始化，允许延迟初始化，**但是在使用属性之前，任然要保证以及被初始化**
2. 属性类型为 可空的数据类型

### 惰性初始化

**kotlin使用代理机制实现惰性初始化**

代理机制使用 by 关键字，kotlin标准库里有内置的 代理可用，lazy就是其中一个，对应的属性首次被引用的时候才会真正执行初始化

Lazy代理实现依赖于SynchronizedLazyImpl，大体逻辑为：用内部Initializer包装lambda函数，用volatile修饰 是否已经初始化的标志位，重写 value 的get()访问器，当访问属性时调用代理SynchronizedLazyImpl对象的内部 属性value的 get()，这个get方法 检查属性没有有初始后 synchronized加锁调用initialier() ，即传入的lambda函数，以此来包装全局，多线程环境下仅仅初始化一次，且是当访问时才初始化

Delegate.oberservable() 代理可以观察属性变化并通知



## 第14章 继承

被继承类 需要 `open` 修饰，包括类，函数，以及属性，对于子类可见但不需要继承的函数或者属性，可见性修饰符需要>=protected，子类中可用super访问超类

子类需要用 override 修饰继承的 实体

继承的语法： `class subclass : superclass_constructor(){}`

继承可以一直继续下去，没有层级的限制

也可以使用 final 关键字 修饰函数或者属性，来关闭对子类的可继承性

### 类型检测 is 关键字

### Kotlin类层次

每个类都有一个共同的超类 Any，所有类都是Any的隐式子类，类似于Java的 java.lang.Object

#### 1. 类型转换 as 关键字/ as操作符

类型转换允许将某个对象看做一个特定的类型的实例

类型转换使用时注意安全，如Int 到 Long这样精度更高的数字类型就是安全的操作

类型转换允许你**尝试**将某个类型的变量转换为**任何**目标类型，对于不安全的类型转换，最好放在 try-catch代码块中

#### 2. 智能类型转换

kotlin编译器在确定 `any is Player`条件检查属实后，就能直接当做Player类实例调用 而无需 显式的 做as类型转换，编译器在内部自动做了类型转换



## 第15章 对象 / 特殊类

### 1. object 关键字 ： 定义一个Singleton的类---kotlin以对象的思路处理单例，而不是类

使用object关键字可以产生一个对象，且是应用存活的整个生命周期

关键字有三种方式：对象声明 object declaration，对象表达式 object expression，伴生对象 companion object

#### 对象声明

```kotlin
object Game{
  init{
    println("global")
  }
}
```

当Game被引用到，Game对象就会被创建，引用对象的操作是 访问对象的属性或函数

#### 对象表达式

有时候，需要一个现有类的变体实例，但是只要使用一次，就无须特定写一个子类继承，而是使用对象表达式，产生一个 扩展类功能的对象

这个对象的生命期与对象绑定的符号一致，如下例中的 instance变量的生命期一致

```kotlin
val instance = object : ParentClass(){
  override fun load() = "extend the class"
}
```

#### 伴生对象：类比Java的static final class 且 Singleton的内部类

场景：想将某个对象的初始化 与 一个类实例捆绑在一起，可以考虑伴生对象

一个类只能有一个伴生对象

分两种情况：①类初始化的时候，伴生对象就会初始化，适合用来存放和类定义有上下文关系的单例数据 ②访问伴生对象的属性或函数时候，触发伴生对象的初始

```kotlin
class Outter {
	...
  companion object {
    fun load() = File(PATH).readBytes()
  }
}

// 可以 Outter.load() 直接访问
```

### 2. 嵌套类：区别于内部类

使用class定义的一个嵌套在另一个类内的命名类

```kotlin
object Outter {
  private class Inner(arg:String){
    ...
  }
}
```

Inner类只和Otter有关，都在内部使用，就可以作为私有内部类嵌套于其中



### 3. 数据类 data class

有 toString equals hashcode copy等函数自动生成

有解构声明：`operator fun component1~N() = varName`

适用场景：作为数据Model

一个类成为数据类的必要条件：

1. 数据类型必须有至少带一个参数的主构造函数
2. 数据类主构造函数的参数必须是 val 或 var
3. 数据类不能使用 abstrace open sealed 和 inner 修饰

### 4. 枚举类：有语义的常量，描述性更强

枚举类可以定义函数

```kotlin
enum class Direction(private val coordinate:Coordinate){
  NORTH(Coordinate(0,-1)),
  EAST(Coordinate(1,0)),
  SOUTH(Coordinate(0,1)),
  WEST(Coordinate(-1,0));
  
  fun updateCoordinate() {
    ...
  }
}
```



### 5. 运算符重载

|操作符 | 函数名|

| 操作符 | 函数名     | 作用                     |
| ------ | ---------- | ------------------------ |
| +      | plus       | 一个对象添加到另一个对象 |
| +=     | plusAssign |                          |
| ==     | equals     |                          |
| >      | compareTo  |                          |
| []     | get        |                          |
| ..     | rangTo     |                          |
| in     | Contains   |                          |



如果重写equals ，也应该一并重写hashcode：JVM平台，equals内会调用hashcode

hashcode能唯一表示某个类的实例，可以帮助提高代码执行效率，使用map时，能提升以键取值得效率



### 密封类/ 代数数据类型/sealed class：表示一组子类型的闭集

```kotlin
sealed class StudentStatus{
  object NotEnrolled : StudentStatus()
  class Active(val courseID:String):StudentStatus()
  object Graduated : StudentStatus()
}
```



## 第16章 接口 抽象类

接口 interface：定义一组公共属性和行为 适用：类之间想要共享属性和方法，又无法继承时，作为接口协议

抽象类 abstract class：

### interface （like - a）

属性不初始化，函数没有函数体，有函数体的就为默认实现，默认实现可以减少模板代码

接口的函数和属性默认是 open的

```kotlin
interface Fightable{
  var healthPint:Int
  fun attack(opponent:Fightable):Int
  fun withDraw(){
    println("with draw !!!")
  }
}

class Player(...) : Fightable {
  override var healthPoint:Int = 12
  override fun attack(opponent:Fightable):Int {...}
}
```

### 抽象类

```kotlin
abstract class Monster(val name:String
                      val description:String
                      override var healthPoints:Int) : Fightable{
 override fun attack(opponent:Fightable):Int {...} 
}
```

接口里不能定义构造函数，抽象类有构造函数

一个类可以继承多个接口，但是只能有一个抽象类，单继承



## 第17章 泛型 Generic

泛型系统允许函数和其他类型接受当前无法预知的类，大大提高了代码的复用率

泛型：支持操作的类型的集合

语法：<T，R> 作为类型占位符

```kotlin
//类支持泛型
class LootBox<T>(item : T){
  //函数支持泛型
	fun fetch(): T? { ... }
	// 多泛型参数
  fun <R> fetch( function:(T)->R ):T?{
		...
  }
  
}

```

### 泛型约束

泛型是支持操作的类型的集合，那么约束泛型便是：在这个集合里，以继承关系为条件，对所有可能的类型的集合进行约束，得到其子集，限制类型的范围

```kotlin
class LootBox<T:Loot>(item:T) {...}
```



### vararg 关键字：接受多个值参

### in out

当涉及到泛型数据的操作的时候，就会有两个 泛型数据类型 的约束关系产生

A .consume(   B. product( )   )   B 产生的只能是B的子类，A能 消费/使用 的只能是A的父类

泛型参数可以扮演两种角色：生产者（返回值是泛型），消费者（输入值是泛型）

out 表明：泛型将扮演可读而不可写的 生产者角色，返回值为 T，协变 ，< T的，T的子类

in 表明：泛型将扮演可写而不可读的 消费者角色，形参列表为 T，逆变，> T 的，T的超类

### reified 关键字： 运行时保留 类型信息

泛型一般会擦除类型信息，如果想知道某个泛型参数具体是什么类型，reified关键字能帮助检查泛型参数的类型，而不用反射

## 第 18 章 拓展（被Kotlin编译器特殊处理的函数/属性）

拓展可以在不直接修改类定义的情况下添加新功能：比如无法接触某个类，没有使用open等因素导致无法继承

拓展专用文件的命名约定通常为 Ext结尾

一个private拓展只能在定义的文件内可见

拓展这种语言特性，本质上还是在利用面向对象编程范式

### 定义拓展函数

语法：指定接受者类型的 普通函数定义，但是可以访问 this super等引用

```kotlin

fun String.easyPrint() = println(this)
```

### 定义泛型拓展函数

泛型拓展函数不仅可以支持任何类型的接受者，还可以保留接受者的类型信息

泛型拓展函数在kotlin标准库随处可见

```kotlin
fun <T> T.easyPrint():T{
	println(this)
  return this
}

public inline fun <T,R> T.let(block:(T)->R){
	return block(this)
}
```

### 拓展属性

语法：添加类接受者类型的 属性定义

### 可空类拓展：在可空类型上定义拓展，就可以直接在拓展函数内解决可能出现的空值问题

infix 适用于有单个参数的拓展和函数，让函数调用 可以以 中缀表达式编写，比如 `0 util 100` 比如 `"key" to 1`

### 拓展的原理

Kotlin拓展是个静态方法，其拓展对象就是一个值参，**但是kotlin编译器会搜索对应的符号调用**

拓展的Java字节码

```Java
public static final String ExpandFunction(@NotNull String $receiver, int amount){......}
public static final String ExpandFunctionWithGeneric(@NotNull Object $receiver, int amount){......}
```

在Python中，我们也是往往会定义一个 self，表示自己的指针，而调用的时候，Python解释器会对类内函数自动传入self，与此处kotlin编译器对拓展的特殊处理是一样的

### 别名

```kotlin
import com.xxx.extensions.random as randomizer
```

### 标准库的拓展

包含拓展的标准库文件通常都是以类名加s后缀来命名的，重度使用拓展来定义核心API，让标准库即保持轻量，还能提供很多功能，而且为kotlin 运行在不同的平台提供一致的拓展行为

### 带接受者的函数字面量：利于DSL的编写

拓展 本质就是一个带接受者的函数，叫self 也罢，叫this也罢，叫receiver也罢

```kotlin
public inline fun <T> T.apply(block: T.() -> Unit):T {
  block()
  return this
}
```

Block 不仅是个 lambda表达式，还是一个泛型拓展

### 第19章 函数式编程 lambda 演算

三大类函数：变换 过滤 合并

函数式编程的设计理念：**不可变数据的副本在链上的函数间传递**

原始数据不会改变

kotlin能支持函数编程范式，主要原因是它支持高阶函数

### 变换

变换函数会遍历集合内容，使用变换器函数变换每一个元素

### 过滤

接受一个 predicate函数，用他按给定条件检查接受者集合里的元素给出 true或false判定，true则通过过滤，false则忽略掉

### 合并

合并函数能将不同的集合合并为一个集合



### 序列：惰性集合

及早集合 vs 惰性集合

及早集合：List Map Set 都是，所有数据都已经准备好，并允许你访问

惰性集合：性能表现优异，只产生需要的，集合元素按需产生

迭代器函数：只要序列有新值就被调用一下的函数

序列：Sequence：不索引排序，不记录元素数目

#### 生产序列：generateSequence

```kotlin
generateSequence(0) {it+1}
	.onEach{ println("Count:$it")}
// 给定一个种子，以及generate生产算子（lambda函数），就会不停的产生返回值
val oneThousandPrimes = generateSequence(3) { value -> value +1 }
	.filter { it.isPrime() }
	.take(1000)
```

#### 转换为Sequence：List Set 都可以 asSequence() 转换为Sequence

### 评估代码性能：计时函数

measureNanoTime( lambda )

measureTimeInMills( lambda )

### 5. Arrow.kt 错误处理方案

Haskell中，有个类型Maybe，是个可选值，数据值 或者 错误

Arrow.kt 库能享受到Haskll中的高级函数式编程特性，有个Either类型，对应Maybe，无需try-catch，也能妥善处理某个会出错的操作



## 第20章 Kotlin与Java 互操作

与Java无缝操作，支持Jvm平台，是kotlin最重要的特性，才能有 Kotlin first in Android，才有kotlin的发展

### 2. 互操作性 与 可控性 可控性注解

@Nullable

@NotNull

### 3. 类型映射

kotlin的基本数据类型，对应Java的原始数据类型

Int.javaClass  是Java的支持类名

### 4. setter与getter 互操

Java的setter getter的private字段，kotlin中可以直接作为属性访问

### 5. 类之外 

@file:JvmName("Hello") 注解kotlin文件， 指定编译类名

@JvmOverloads 注解Kotlin方法：解决Java不支持默认参数，使用方法重载匹配Kotlin的犯法

@JvmField注解类属性：暴露kotlin的字段给Java的调用者，让Java避免使用getter方法； Kotlin和Java管理类变量方式不同：Java使用字段和setter getter，kotlin使用属性 和 back field，导致java可以直接读写字段，而Kotlin必须借助访问器，尽管语法上似乎没有什么差别

注解 能用来以静态方式提供伴生对象里定义的值，让Java代码也可以像kotlin代码一样访问 伴生对象的字段

@JvmStatic 类似于 @JvmField，允许Java直接调用 伴生对象的函数

### 6. 异常

@Throws(IOException::class) 注解函数体有 throw 未检查异常，而函数头又不像Java有 throw 声明 的Kotlin函数 ，让Java调用时，能以检查异常的方式调用kotlin函数

### 7. Java的 函数类型

Java里使用 FunctionN这样的类型来处理 Kotlin中的函数类型的变量数据

## 第21章 kotlinx.coroutines

协程不是kotlin编译器内置功能，async await 不是关键字，甚至不在标准库里，只是出现在官方库中

设计文稿：https://github.com/Kotlin/KEEP/blob/master/proposals/coroutines.md

官方文档：https://kotlinlang.org/docs/coroutines-guide.html#additional-references

A *coroutine* is an instance of suspendable computation. It is conceptually similar to a thread, in the sense that it takes a block of code to run that works concurrently with the rest of the code. However, a coroutine is not bound to any particular thread. It may suspend its execution in one thread and resume in another one

协程理解为轻量级的线程

协程提供异步不阻塞的行为又不失代码可读性

```kotlin
fun main() = runBlocking { // this: CoroutineScope
    launch { // launch a new coroutine and continue
        delay(1000L) // non-blocking delay for 1 second (default time unit is ms)
        doWorld()
    }
    println("Hello") // main coroutine continues while a previous one is delayed
}

suspend fun doWorld() {
    delay(1000L)
    println("World!")
}
```



launch：coroutine builder，启动一个新协程

delay：一个挂起函数，不会阻塞本线程

runBlocking：一个CoroutineScope，是连接协程和普通代码的连接点，launch只能在scope里调用

coroutiune遵循 Structured Concurrency 原则：意味着协程启动只能在明确界定了协程生命周期的CoroutineScope中

结构化并发保证你发起的许多协程不会丢失也不会泄露

Suspend 函数可以在协程内调用，函数内也可以调用其他 挂起函数（suspend函数）

### Scope Builder/structured concurrency

`coroutineScope{ suspend fun }`

CoroutineScope is responsible for the structure and parent-child relationships between different coroutines. We always start new coroutines inside a scope. *Coroutine context* stores additional technical information used to run a given coroutine, like the coroutine custom name, or the dispatcher specifying the threads the coroutine should be scheduled on.类似于进程之于线程，scope是coroutine的分组，上下文

All these functions take a lambda with receiver as an argument, and the implicit receiver type is the `CoroutineScope`

`runBlocking{  }`是个特例，会阻塞所在线程

```kotlin
// scope会传入一个receiver
fun main() = runBlocking { /* this: CoroutineScope */
    launch { /* ... */ }
    // the same as:    
    this.launch { /* ... */ }
}
```



We can say that the nested coroutine (started by `launch` in this example) is a child of the outer coroutine (started by `runBlocking`). This**"parent-child" relationship** works through scopes: the child coroutine is started from the scope corresponding to the parent coroutine.

It's also possible to start a new coroutine from the global scope using `GlobalScope.async` or `GlobalScope.launch`. This will create a top-level **"independent" coroutine.**

The mechanism providing the structure of the coroutines is called **"structured concurrency"**. Let's see what benefits structured concurrency has over global scopes:

- The scope is generally responsible for child coroutines, and their lifetime is attached to the lifetime of the scope.
- The scope can automatically cancel child coroutines if something goes wrong or if a user simply changes their mind and decides to revoke the operation.
- The scope automatically waits for completion of all the child coroutines. Therefore, if the scope corresponds to a coroutine, then the parent coroutine does not complete until all the coroutines launched in its scope are complete.

### Coroutine Builder

launch 返回一个Job对象 作为协程的句柄，可以用来显式的等待协程结束

```kotlin
val job = launch { // launch a new coroutine and keep a reference to its Job
    delay(1000L)
    println("World!")
}
println("Hello")
job.join() // wait until child coroutine completes
println("Done") 
```

aysnc starts a new coroutine and returns a `Deferred` object. `Deferred` represents a concept known by other names such as `Future` or `Promise`: it stores a computation, but it *defers* the moment you get the final result; it *promises* the result sometime in the *future*.推迟 的概念，并保证结果将来会在



The main difference between `async` and `launch` is that `launch` is used for starting a computation that isn't expected to return a specific result. `launch` returns `Job`, which represents the coroutine. It is possible to wait until it completes by calling `Job.join()`. `Deferred` is a generic type which extends `Job`. An `async` call can return a `Deferred<Int>` or `Deferred<CustomType>` depending on what the lambda returns (the last expression inside the lambda is the result). launch不期待结果，async期待结果

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val deferreds: List<Deferred<Int>> = (1..3).map {
        async {
            delay(1000L * it)
            println("Loading $it")
            it
        }
    }
    val sum = deferreds.awaitAll().sum()
    println("$sum")
}
```

Deferred.await() 和 Deferred.awaitAll() 返回 async的未来结果，和Java Future基本一致

### Coroutines ARE light-weight﻿

每个协程 抽象为一个 Job对象，运行在线程池里，将有限线程的上下文 虚拟化为了不阻塞，没有切换线程上下文成本的协程上下文

以挂起suspend取代阻塞block，以协程Job取代线程上下文Thread

协程是suspendable computation

当协程挂起后，会暂停计算/运行，并且存到memory里

### channel：协程间数据通信

Channels are communication primitives that allow us to pass data between different coroutines.

`Channel` is represented via three different interfaces: `SendChannel`, `ReceiveChannel`, and `Channel` which extends the first two

Several types of channels are defined in the library. They differ in how many elements they can internally store, and whether the `send` call can suspend or not. For all channel types, the `receive` call behaves in the same manner: it receives an element if the channel is not empty, and otherwise suspends.

1. An unlimited channel is the closest analog to queue: producers can send elements to this channel, and it will grow infinitely
2. A buffered channel's size is constrained by the specified number. Producers can send elements to this channel until the size limit is reached
3. The “Rendezvous” channel is a channel without a buffer; it's the same as creating a buffered channel with zero size. One of the functions (`send` or `receive`) always gets suspended until the other is called. If the `send` function is called and there's no suspended `receive` call ready to process the element, then `send` suspends. Similarly, if the `receive` function is called and the channel is empty - or in other words, there's no suspended `send` call ready to send the element - the `receive` call suspends. The "rendezvous" name ("a meeting at an agreed time and place") refers to the fact that `send` and `receive` should "meet on time".
4. A new element sent to the conflated channel will overwrite the previously sent element, so the receiver will always get only the latest element. The `send` call will never suspend.

```kotlin
val rendezvousChannel = Channel<String>()
val bufferedChannel = Channel<String>(10)
val conflatedChannel = Channel<String>(CONFLATED)
val unlimitedChannel = Channel<String>(UNLIMITED)
```



### CoroutineDispatcher

CoroutineDispatcher指定协程的目标线程

`async(Dispatcher.Default){ .... }`，如果不显式指定，async会使用默认使用 outer scope的，最佳实践是不指定Dispatcher

`Dispatchers.Default` represents a shared pool of threads on JVM. This pool provides a means for parallel execution. It consists of as many threads as there are CPU cores available, but still it has two threads if there's only one core. 

Dispatcher.MAIN 指代主线程，在Android里Activity里，代表主线程也即UI Thread

```kotlin
launch(Dispatchers.Default) {
    val users = loadContributorsConcurrent(service, req)
    withContext(Dispatchers.Main) {
        updateResults(users, startTime)
    }
}
```

可以使用withContext指定suspend 函数所在的Dispatcher，在加载完数据后，更新UI使用withContext(Dispathcer.Main) 非常实用，不用新开一个协程 然后 join 那么复杂

### Flow：日后详细总结各种使用

```kotlin
fun simple(): Flow<Int> = flow { // flow builder
    for (i in 1..3) {
        delay(100) // pretend we are doing something useful here
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    // Launch a concurrent coroutine to check if the main thread is blocked
    launch {
        for (k in 1..3) {
            println("I'm not blocked $k")
            delay(100)
        }
    }
    // Collect the flow
    simple().collect { value -> println(value) } 
}


/////////////
(1..3).asFlow() // a flow of requests
    .transform { request ->
        emit("Making request $request") 
        emit(performRequest(request)) 
    }
    .collect { response -> println(response) }
```

**flow builder**

The `flow { ... }` builder from the previous examples is the most basic one. There are other builders for easier declaration of flows:

- [flowOf](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow-of.html) builder that defines a flow emitting a fixed set of values.
- Various collections and sequences can be converted to flows using `.asFlow()` extension functions.