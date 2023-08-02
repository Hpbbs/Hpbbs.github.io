---
layout: post
title:  "Python数据科学 之 NumPy使用字典式讲解"
date:   2020-01-17
categories: [ComputerScience]
tags: [数据科学,Python,Numpy]
published: true
---

***未经许可不得转载***
***时间：2020年1月17日17:33:15***   
***created by：Hpbbs***
***使用JupyterNotebook编辑***

* toc
{:toc}

### 0x00 前言（NumPy 的设计理念）

对于一个技术的使用或许记背几个常用的函数和基本用法便好，但要理解一个技术和工具，设计理念便是其内核之一，就是文章诗词的风骨，嚼起来才有味道，本部分介绍一些NumPy的优秀设计理念，当然，这些不是Numpy所独有，在介绍时可以思考你学过的别的技术或优秀工具框架同质



#### 0.1 Python的基本数据类型 vs Numpy提供的array数据类型

Python的对象：

- 高级数据对象：integers（整型数据），floating point（浮点数）
- 容器：list（列表：插入和追加），dict（字典：快速查找）

对比C语言等大多数语言，想一想缺少了什么？
<p style="color:red">没有数组</p>

NumPy提供：

- 多维数组
- 接近硬件（高效）
- 为科学计算而设计（便捷）
- 面向数组计算

<p style="color:red">提取一下关键词：数组，接近硬件运行很快的数组，多维数组</p>

 容易得出，Python没有提供基础的连续内存空间的数据类型，只有更高层次的基于抽象模型的列表和字典两种抽象数据类型，对于使用编程语言的用户确实方便了，需要时，也可以用list来模拟一下数组类型进行操作，但是一到需要性能要求很高的计算，Python就显得能力不足了，而这，恰恰就是NumPy所提供的（从此以后，Python的数据类型便可以覆盖所有基础类型），这也是为什么多数数据处理多且对性能要求高的模块软件或工具都会依赖Numpy，比如二维绘图的matplotlib，比如pandas，可以说，无论科学计算领域还是其他领域，NumPy和字符串数据类型一样重要，甚至有过之而无不及

总之，Python缺少的能力，NumPy完美的完成了，还超额完成了

当然我提到了，NumPy比list等容器要快吧，那么快多少呢？口说无凭，来试验一下

#### 0.2 NumPy的实现速度（底层用编译语言实现）


```python
L = range(1000)
%timeit [i**2 for i in L]
```

    323 µs ± 4.96 µs per loop 
    (mean ± std. dev. of 7 runs, 1000 loops each)
    


```python
import numpy as np
a = np.arange(1000)
%timeit a**2
```
	1.48 µs ± 7.63 ns per loop 
	(mean ± std. dev. of 7 runs, 1000000 loops each)
    

<p style="color:red">numpy约是list的218倍（323/1.48）！！！！！</p>

你现在应该不会动摇学习NumPy基本使用的想法了   
我也相信希望你看完本篇文章会对NumPy有相当程度的理解

**你是否会想知道怎么会快这么多？**

从解释的脚本语言特点与编译语言的特性，从NumPy的实现与优化，从list和NumPy分别在哪一个计算机层次上的实现等都有它的叙述，终归于一点，NumPy的官方文档也是这么概要：NumPy是Python脚本语言调用接近硬件的C等编译语言实现的。暂时能有概念上的大致理解便好。

#### 0.3 NumPy的内部设计（arrays对象，统一操作接口）

NumPy内部结构比较复杂，为了用户方便操作，它提供了arrays对象的统一操作接口，对于整个NumPy库的各种操作使用arrays对象的统一操作接口对外提供，在Python编程时，这个arrays的使用和其他的str，list，dict对象保持一致，简约统一

这种设计在很多优秀的工具库都能看到，比如马上要介绍的Matplotlib库的pyplot对象也是担当了面向用户的统一操作接口功能，对于调用者，看不到内部的为追求功能多和优化后的复杂，在设计自己的库的时候，也应根据辛苦，融入这种软件工程的设计原则

Ok，目前就介绍这些，下面开始NumPy入门

#### 0.4 Numpy数据对象的操作

NumPy的基本设计，致力于打造符合大家对于数组常用的操作标准，有标准，就方便他人迁移和快速上手，比如Java，C或其他语言来的，能快速根据原来语言的认知，快速高效的使用。主体会在数组的运算一小节具体介绍。

### 0x01 NumPy库入门

在此部分，我们首先介绍数组的类型的一些基本概念，这些概念都是标准化的，在NumPy中有，在其他语言的数组数据结构一样有，可以说属于数据结构的理论内容   
接着，会介绍数组的创建方法，访问方法以及基本运算

#### 1.0 数组定义

代码说话，他长这样



```python
import numpy as np
arr = np.linspace(1,100,30)
print(arr)
```

    [  1.           4.4137931    7.82758621  11.24137931  14.65517241
      18.06896552  21.48275862  24.89655172  28.31034483  31.72413793
      35.13793103  38.55172414  41.96551724  45.37931034  48.79310345
      52.20689655  55.62068966  59.03448276  62.44827586  65.86206897
      69.27586207  72.68965517  76.10344828  79.51724138  82.93103448
      86.34482759  89.75862069  93.17241379  96.5862069  100.        ]
    

#### 1.1 数据的维度和数组（Dimension，D）

**概念自定**
什么叫做维？

符合概念的叫做模型，就像是类和他的对象
我先介绍几个不同维的模型，先有一个模糊的印象
- 物理里的长，宽，高
- 数学里的点，线，面
- 人体感知可以有视觉，听觉，嗅觉
- 西瓜有 色泽，根蒂，敲声
（你把关系数据的基本概念，比如属性，也是可以类比的，）

一个数据表达一个含义，
一组数据表达一个或多个含义

什么叫维度？

维度（Dimension），又称为维数，是数学中独立参数的数目。
从广义上讲：维度是事物“有联系”的抽象概念的数量，“有联系”的抽象概念指的是由多个抽象概念联系而成的抽象概念，和任何一个组成它的抽象概念都有联系，组成它的抽象概念的个数就是它变化的维度。

什么叫数组？
一组数，但是考虑到实现的问题，他又会有不同的限制，

1. 数组长度定义好后就不能改了
2. 数组的元素必须是同类型
3. 数组的内存模型也必须是线性的


**概念对比**

维VS属性：
在很多不同的领域，这两者几乎可以替换，在周志华人工智能的西瓜书里，
西瓜的色泽，根蒂，敲声就是三个不同的属性，我们可以用三个维的数据组织形式来表达这三个属性的数据集，同样，我们也可以用关系型数据库来存储这三个属性和属性值（当然，关系数据关注的是关系范式，属性是表示数据的概念而已）。

数组维VS线性代数：
在线性代数中，我们有不同的生成子空间的概念，
而子空间，就有0维，1维，2维等不同的子空间，其次，向量的表达不就是数组的形式嘛，怎么就能表达不同的维了呢？是不是感觉和之前所说的以及一直对维的理解有很大矛盾呢？在次不予以说明，能理清这两个区别和共性是辨别你是否理解维和属性的概念的标志。

**通俗表达**
上面的解释是教学型文章的败笔，一个概念没有说清楚呢，倒是来了一堆的新概念、

那我来个通俗的说法吧：
一个数据叫0维数组：
如12，-23，一个对象

一群数，按照线性排列叫1维数组:
排个队：[12,34,44,35,12,34,45]

一群数，组成一维数组后,跑到另外一个数组去当元素，继续线性排队，就是2维数组啦，就是说一位数组的元素是一个数组：
[[],[],[],[],[],[]]
填上数据：
[[12,23,34,56][23,45,43,56],[12,34,23,54],[121,34,56,23]]

三维数组等高维数组，就可以这样递归的定义下去,如，三维数组，就是一维数组的元素是一维数组，这个二等数组的元素又是一维数组，这个三等数组的元素还是一维数...，哦不，是0维数组了，或者就是一个数


#### 1.2 ndarray对象（大门）

NumPy是一个开源的Python科学计算基础库，包含：
- 一个强大的N维数组对象 ndarray
- 广播功能函数
- 整合C/C++/Fortran代码的工具
- 线性代数、傅里叶变换、随机数生成等功能


> NumPy 的引用：
import NumPy as np
这里取个别名叫np是通用标准，全世界都这么干，你能这么应用取别名，就和国际接轨了


ndarray是一个多维数组对象，由两部分构成：
- 实际的数据
- 描述这些数据的元数据（数据维度、数据类型等）

ndarray数组一般要求所有元素类型相同（同质），数组下标从0开始

>ndarry在程序中的别名是 array，这才和Python的str list等统一表述嘛

轴(axis): 保存数据的维度；秩(rank)：轴的数量

ndarray对象的属性：

|属性|说明|
|:--|:--|
|.ndim|dimension,维数，即轴的数量，秩|
|.shape|ndarray对象的尺度，形状，矩阵的n行m列|
|.size|ndarray对象的大小，元素的个数，n\*m|
|.dtype|ndarray对象的元素类型|
|.itemsize|ndarray的每个元素的大小，以字节为单位|


```python
a = np.array([[12,23,34,56],[23,45,43,56],[12,34,23,54],[121,34,56,23]])
print(a)
print('a.ndim:{}'.format(a.ndim))
print('a.shape:{}'.format(a.shape))
print('a.size:{}'.format(a.size))
print('a.dtype:{}'.format(a.dtype))
print('a.itemsize:{}'.format(a.itemsize))
```

    [[ 12  23  34  56]
     [ 23  45  43  56]
     [ 12  34  23  54]
     [121  34  56  23]]
    a.ndim:2
    a.shape:(4, 4)
    a.size:16
    a.dtype:int32
    a.itemsize:4
    

**ndarray的元素类型**

|数据类型|说明|
|:--|:--|
|bool| 布尔类型， True或False|
|整数||
|intc |与C语言中的int类型一致，一般是int32或int64|
|intp |用于索引的整数，与C语言中ssize_t一致， int32或int64|
|int8 |字节长度的整数，取值： [‐128, 127]|
|int16| 16位长度的整数，取值： [‐32768, 32767]|
|int32 |32位长度的整数，取值： [‐231, 231‐1]|
|int64 |64位长度的整数，取值： [‐263, 263‐1]|
|各种字节长度的无符号数||
|uint8 |8位无符号整数，取值： [0, 255]|
|uint16 |16位无符号整数，取值： [0, 65535]|
|uint32 |32位无符号整数，取值： [0, 232‐1]|
|uint64 |32位无符号整数，取值： [0, 264‐1]|
|IEEE754标准浮点数||
|float16| 16位半精度浮点数： 1位符号位， 5位指数(符号)尾数\*1， 10位尾数|
|float32 |32位半精度浮点数： 1位符号位， 8位指数， 23位尾数|
|float64 |64位半精度浮点数： 1位符号位， 11位指数， 52位尾数|
|complex64| 复数类型，实部和虚部都是32位浮点数|
|complex128| 复数类型，实部和虚部都是64位浮点数|

对比：Python语法仅支持整数、浮点数和复数3种类型
精细化数据管理好处：

- 科学计算涉及数据较多，对存储和性能都有较高要求
- 对元素类型精细定义，有助于NumPy合理使用存储空间并优化性能
- 对元素类型精细定义，有助于程序员对程序规模有合理评估

**特点**：数据占用内存大小很精准，适合各种场景数据要求，如此精致，可见一斑

#### 1.3 ndarray数组的创建方法
1. 从Python中的列表、元组等类型创建指定数据的ndarray数组

```python
np.array(object, dtype=None,\
 copy=True, order='K',\
  subok=False, ndim=0)->narray

x = np.array(list/tuple)
x = np.array(list/tuple, dtpyt=np.float32)

```

2. 使用NumPy中函数创建ndarray数组，如： arange, ones, zeros等，创建特征数组

|函数|说明|
|:--|:--|
|np.arange(n)| 类似range()函数，返回ndarray类型，元素从0到n‐1|
|np.ones(shape)| 根据shape生成一个全1数组， shape是元组类型|
|np.zeros(shape)| 根据shape生成一个全0数组， shape是元组类型|
|np.full(shape,val)| 根据shape生成一个数组，每个元素值都是val|
|np.eye(n)| 创建一个正方的n\*n单位矩阵，对角线为1，其余为0|
|从已知形状的数组arr创造相同形状的特征数组||
|np.ones_like(arr)| 根据数组a的形状生成一个全1数组|
|np.zeros_like(arr)| 根据数组a的形状生成一个全0数组|
|np.full_like(arr,val)| 根据数组a的形状生成一个数组，每个元素值都是val|
|||
|np.linspace()| 根据起止数据等间距地填充数据，形成数组|
|np.concatenate() |将两个或多个数组合并成一个新的数组|


```python
import numpy as np
np.arange(8)
```




    array([0, 1, 2, 3, 4, 5, 6, 7])




```python
import numpy as np
np.ones((3,6))
```




    array([[1., 1., 1., 1., 1., 1.],
           [1., 1., 1., 1., 1., 1.],
           [1., 1., 1., 1., 1., 1.]])




```python
import numpy as np
np.zeros((2,3,4))
```




    array([[[0., 0., 0., 0.],
            [0., 0., 0., 0.],
            [0., 0., 0., 0.]],
    
           [[0., 0., 0., 0.],
            [0., 0., 0., 0.],
            [0., 0., 0., 0.]]])




```python
import numpy as np
np.full((2,3,4),3.88)
```




    array([[[3.88, 3.88, 3.88, 3.88],
            [3.88, 3.88, 3.88, 3.88],
            [3.88, 3.88, 3.88, 3.88]],
    
           [[3.88, 3.88, 3.88, 3.88],
            [3.88, 3.88, 3.88, 3.88],
            [3.88, 3.88, 3.88, 3.88]]])




```python
import numpy as np
np.eye(5)
```




    array([[1., 0., 0., 0., 0.],
           [0., 1., 0., 0., 0.],
           [0., 0., 1., 0., 0.],
           [0., 0., 0., 1., 0.],
           [0., 0., 0., 0., 1.]])



#### 1.3 ndarray数组的变换

数组的变换主要指的是对已存在的数组进行维度和数据类型的变换

- 维度变换


|方法|说明|
|:--|:--|
|***维度变换***||
|.reshape(shape)| 不改变数组元素，返回一个shape形状的数组，***原数组不变***|
|.resize(shape) |与.reshape()功能一致，但***修改***原数组|
|.swapaxes(ax1,ax2)| 将数组n个维度中两个维度进行调换|
|.flatten()| 对数组进行降维，返回折叠后的一维数组，***原数组不变***|
|***元素数据类型变换***||
|arr.astype(new_type)|创建新的数组,以指定的数据类型|
|***ndarry数据类型变换***||
|arr.tolist()|将NumPy的array类型的数组装换为Python的list类型|

#### 1.4 数组的切片和索引

索引：给出数组元素的“位置”获得元素的值，取得单个元素
切片：给出数组元素的“区域”获得整体数组元素子集的操作，取得多个元素（新的“子”数组）

**一维数组的索引与切片**：同Python的list类似

```python

a=np.array([1,2,3,4,5,6,7,8,9])

a[2]
# 3
a[1:7:2]
# startIndex:endIndex:stepLength，从startIndex到endIndex前闭后开区间以stepLength为步长的数组，支持负数的反向索引
#array([2,4,6])

```
**高维数组索引与切片**：

可理解为对每一维数据的切片索引后的复合，用‘,’划分




```python
import numpy as np

a = np.arange(36)
a = a.reshape((3,3,4))
print('a')
print(a)
print('a[2,1,-3]')
print(a[2,1,-3])
print('a[:,1:3,:]')
print(a[:,1:3,:])
print('a[:,:,::2]')
print(a[:,:,::2])

# 可以当做坐标来理解
```

    a
    [[[ 0  1  2  3]
      [ 4  5  6  7]
      [ 8  9 10 11]]
    
     [[12 13 14 15]
      [16 17 18 19]
      [20 21 22 23]]
    
     [[24 25 26 27]
      [28 29 30 31]
      [32 33 34 35]]]
    a[2,1,-3]
    29
    a[:,1:3,:]
    [[[ 4  5  6  7]
      [ 8  9 10 11]]
    
     [[16 17 18 19]
      [20 21 22 23]]
    
     [[28 29 30 31]
      [32 33 34 35]]]
    a[:,:,::2]
    [[[ 0  2]
      [ 4  6]
      [ 8 10]]
    
     [[12 14]
      [16 18]
      [20 22]]
    
     [[24 26]
      [28 30]
      [32 34]]]
    

**高级操作索引**

- a\[a<0\]=0
#对数组中小于0的元素置0

#### 1.4 数组的运算

1. 数组与标量的运算
    
    对所有元素的运算



```python
import numpy as np

a = np.linspace(1,49,24,endpoint=False).reshape((2,3,4))
print(a)
a = a/a.mean()
print(a)

# 每一个元素除以数据的平均值
```

    [[[ 1.  3.  5.  7.]
      [ 9. 11. 13. 15.]
      [17. 19. 21. 23.]]
    
     [[25. 27. 29. 31.]
      [33. 35. 37. 39.]
      [41. 43. 45. 47.]]]
    [[[0.04166667 0.125      0.20833333 0.29166667]
      [0.375      0.45833333 0.54166667 0.625     ]
      [0.70833333 0.79166667 0.875      0.95833333]]
    
     [[1.04166667 1.125      1.20833333 1.29166667]
      [1.375      1.45833333 1.54166667 1.625     ]
      [1.70833333 1.79166667 1.875      1.95833333]]]
    

2. NumPy 一元运算函数

fun(x)/x.fun()

|函数 |说明|
|:--|:--|
|np.abs(x) |np.fabs(x) 计算数组各元素的绝对值|
|np.sqrt(x)| 计算数组各元素的平方根|
|np.square(x)| 计算数组各元素的平方|
|np.log(x) np.log10(x) np.log2(x)| 计算数组各元素的自然对数、 10底对数和2底对数|
|np.ceil(x) np.floor(x)| 计算数组各元素的ceiling值 或 floor值|
|np.rint(x)| 计算数组各元素的四舍五入值|
|np.modf(x) |将数组各元素的小数和整数部分以两个独立数组形式返回|
|np.cos(x) np.cosh(x) np.sin(x) np.sinh(x) np.tan(x) np.tanh(x)|计算数组各元素的普通型和双曲型三角函数|
|np.exp(x)| 计算数组各元素的指数值|
|np.sign(x)| 计算数组各元素的符号值， 1(+), 0, ‐1(‐)|

3. NumPy 二元运算函数

fun(x,y)/x.fun(y)

|函数 |说明|
|:--|:--|
|+ ‐ \* \/ \*\*| 两个数组各元素进行对应运算(操作符重载函数)|
|np.maximum(x,y) np.fmax() np.minimum(x,y) np.fmin() |元素级的最大值/最小值计算|
|np.mod(x,y) |元素级的模运算|
|np.copysign(x,y)| 将数组y中各元素值的符号赋值给数组x对应元素|
|> < >= <= == != |算术比较，产生布尔型数组|

**两数组不同质（形状不同）**
此时，两数组运算使用broadcasting机制
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200120210640490.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
### 0x02 数组数据的存取

#### 2.0 区分一维数组和多维数组

数组的一个特点是：在物理内存中存储在连续的内存空间

||||||||||
|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
|0x00|0x01|0x02|0x03|0x04|0x05|0x06|0x07|
|12|34|45|65|43|32|23|45|

数据本身是没有数组形状的信息的，他只是每个元素的值存在每个内存单元中

所以，我们在使用数组时，我们需要存储数组的形状信息
我们可以将一个数组对象数据分为数据头和数据体，数据头存储数组对象的形状，而数据体即内存中的元素值

于是，我们可以通过使用函数a.resize(shape)，reshape等函数来改变数据头中的关于形状的信息，当我们使用数组，比如索引数组元素时，配合数据头，来找到对应的数据

理解数组的维数是如何实现的，我们再来谈论数组数据的存储：

#### 2.1 CSV文件（1-D或2-D数组存取）

CSV文件：Comma-Separated Value（逗号分隔值） 文件
比如：

城市,环比,同比,定基
北京,101.5,120.7,121.4
上海,101.2,127.3,127.8
广州,101.3,119.4,120.0
深圳,102.0,140.9,145.5
沈阳,100.1,101.4,101.6

>可以看出，只有逗号分隔，格式简单，易于理解，同时，限制了CSV只能存取一维数组和二维数组

**存**：

np.savetxt(frame, array, fmt='%.18e', delimiter=None)

- frame : 文件、字符串或产生器，可以是.gz或.bz2的压缩文件
- array : 存入文件的数组
- fmt : 写入文件的格式，例如： %d %.2f %.18e
- delimiter : 分割字符串，默认是任何空格


**取**：
np.loadtxt(frame, dtype=np.float, delimiter=None， unpack=False)

- frame : 文件、字符串或产生器，可以是.gz或.bz2的压缩文件
- dtype : 数据类型，可选
- delimiter : 分割字符串，默认是任何空格
- unpack : 如果True，读入属性将分别写入不同变量



#### 2.2 二进制文件（可适用高维数组）

**存**

tofile(frame, sep='', format='%s')
- frame : 文件、字符串
- sep : 数据分割字符串，如果是空串，写入文件为二进制
- format : 写入数据的格式

**取**

fromfile(frame, dtype=float, count=‐1, sep='')

- frame : 文件、字符串
- dtype : 读取的数据类型
- count : 读入元素个数， ‐1表示读入整个文件
- sep : 数据分割字符串，如果是空串，写入文件为二进制

#### 2.3 NumPy文件（可适用高维数组）

**存**
np.save(fname, array) 或 np.savez(fname, array)

- fname : 文件名，以.npy为扩展名，压缩扩展名为.npz
- array : 数组变量

**取**

np.load(fname)
- fname : 文件名，以.npy为扩展名，压缩扩展名为.npz

### 0x03 NumPy的函数
#### 3.1 随机函数子库 numpy.random库

|函数 |说明|
|:--|:--|
|np.random.rand(d0,d1,..,dn) |根据d0‐dn创建随机数数组，浮点\[0,1)，均匀分布|
|np.random.randn(d0,d1,..,dn)| 根据d0‐dn创建随机数数组，标准正态分布|
|np.random.randint(low[,high,shape]) |根据shape创建随机整数或整数数组，范围是[low, high)|
|np.random.seed(s) |随机数种子， s是给定的种子值|
|shuffle(a)| 根据数组a的第1轴进行随排列，改变数组x|
|permutation(a)| 根据数组a的第1轴产生一个新的乱序数组，不改变数组x|
|choice(a[,size,replace,p])| 从一维数组a中以概率p抽取元素，形成size形状新数组，replace表示是否可以重用元素，默认为False|
|uniform(low,high,size) |产生具有均匀分布的数组,low起始值,high结束值,size形状|
|normal(loc,scale,size) |产生具有正态分布的数组,loc均值,scale标准差,size形状|
|poisson(lam,size) |产生具有泊松分布的数组,lam随机事件发生率,size形状|




#### 3.1 统计函数


|函数 |说明|
|:--|:--|
|sum(a, axis=None)| 根据给定轴axis计算数组a相关元素之和， axis整数或元组|
|mean(a, axis=None)| 根据给定轴axis计算数组a相关元素的期望， axis整数或元组|
|average(a,axis=None,weights=None)| 根据给定轴axis计算数组a相关元素的加权平均值|
|std(a, axis=None)| 根据给定轴axis计算数组a相关元素的标准差|
|var(a, axis=None) |根据给定轴axis计算数组a相关元素的方差|
|min(a) max(a) |计算数组a中元素的最小值、最大值|
|argmin(a) argmax(a) |计算数组a中元素最小值、最大值的降一维后下标|
|unravel_index(index, shape) |根据shape将一维下标index转换成多维下标|
|ptp(a)| 计算数组a中元素最大值与最小值的差|
|median(a) |计算数组a中元素的中位数（中值）|



#### 3.2 梯度函数

|函数|说明|
|:--|:--|
|np.gradient(f)| 计算数组f中元素的梯度，当f为多维时，返回每个维度梯度|

### 0x04 副本（Copies）和视图（View）

在前面的一些基本函数介绍，有些函数返回一个新的数组（称为copies），但有些少量函数则不会

**view**：切片操作会产生一个view

- view只是一种访问数组的方式
- 切片不会重新在新的内存创建数组
- 当修改视图时，本质是对原数组的修改

> 可以使用copy()函数对view创建副本
newArr = arr\[:,:,2\].copy()
此时对newArr的元素值修改不会影响到arr

当我们的数据量稍微有点大，或我们需要对局部数据修改，或整体数据的局部以不同的组织方式出现对我们有价值的情况下，我们常常会引入view机制  
回想一下关系数据库，我们是否经常在模型的基础上创建view，来实现访问控制或数据接口统一等目的


**copy**：会新申请内存空间，存储数据，此时，对新数组操作便不会影响到旧数组，在旧数组的基础上添加条件而新创建的数组称之为旧数组的copy


**One More Thing**：

- copy和view的概念很简单，但是稍不注意，就会出现你不期望的奇怪情况，比如你要操作的数组数据被修改而自己不知道（往往因为你在别的地方操作了该数组的view，但自己却没有关注），这在不熟悉某些操作函数的返回值是view还是copy时容易发生
- copy和view是深拷贝和浅拷贝概念的衍生
- 好在多数view由切片产生，多数数组的函数操作返回值都是copy

### 0x05 基本的数据可视化

此处只简单介绍matplotlib模块pyploy库的plot()函数



```python
import matplotlib.pyplot as plt
import numpy as np
#准备 变量x的数据，数组
x = np.linspace(0,100,20000)
#准备 变量y的数据，数组
y = np.cos(x)
#使用ploy()可视化
plt.plot(x,y)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200120210202987.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)


### 0x06 高阶操作

#### 2-D array 和 线性代数

numpy.linalg子库实现了基本的线性代数功能，比如解线性系统，SVD分解等，由于numpy的实现并不一定高效，所以，我们更多使用scipy模块的linalg子库进行操作，此处不详细介绍，可参考(scipy.linalg)[https://docs.scipy.org/doc/scipy/reference/linalg.html#module-scipy.linalg]

#### 多项式操作

在numpy.polynomial子库提供了多项式的相关操作


```python
import numpy as np
import matplotlib.pyplot as plt

# 多项式
p = np.polynomial.Polynomial([-1,2,3])
print(p(0))
print(p.roots())
print(p.degree())

#契比雪夫多项式
x=np.linspace(-1,1,2000)
y=np.cos(x)+ 0.3*np.random.rand(2000)
p=np.polynomial.Chebyshev.fit(x,y,90)
t=np.linspace(-1,1,200)
plt.plot(x,y,'r')
plt.plot(t,p(t),'k-',lw=3)
```

    -1.0
    [-1.          0.33333333]
    2
    
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200120210230944.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)

### 0x06 后记

NumPy为Python提供了ndarray数据类型，弥补了没有数组类型只能使用list来模拟的窘境，更加提供了适合科学计算的高效函数，是学习Python做计算科学的最基础，务必重视

本人能力有限，难免有不完备和错误的地方，欢迎指正。

欢迎关注微信公众号：技术复兴
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200116215717390.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)