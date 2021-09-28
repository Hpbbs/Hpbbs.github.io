---
layout: post
title: NFC 基础知识
author: Hpbbs
categories: [硬件,Android]
tags: [NFC,硬件,驱动,Android]
---

* toc
{: toc }

## Chapter One NFC简介 RFID简介

### NFC分类

根据是否有源可区分为 passive tag（被动式），active tag（主动式）

频率高低：Low Frequency（LF）：穿透能力好，感应距离短，读取速度慢，100~500kHz
High Frequency（HF）：读取速度略快，感应距离略长， 10~15MHz
Ultra High Frequency/Microwave,UHFV，850~950MHz以及 2.45GHz，以13.56MHz为主，感应距离最长，速度最快，穿透能力最差


### NFC Forum 的使命是推进NFC技术的应用

### NFC主要技术指标：
工作频率：13.56MHz
通讯距离：<10cm
与咸鱼的非接触式智能卡片兼容
数据传输率：106 212 424 kbit/s
已纳入如下技术标准：ISO，ECMA，ETSI

### NFC 与RFID 对比
NFC是给予RFID理论提出来的
相同点：通过频谱中无限频率的电磁感应耦合方式传递信息的；都工作在13.56<MHz频带
不同点：NFC将 非接触读卡器 接触卡 和点对点 功能整合进一块单芯片 ，而RFID必须由阅读器 和 标签组成。目前的NFC手机内置NFC芯片，组成RFID模块的一部分

RFID只能实现信息的读取以及预判，而NFC技术强调的则是信息交互；NFC范围比较小，NFC是一种近距离的私密通信方式，安全性更高
RFID：更多的被应用在生产，物流，跟踪，资产管理上，
NFC：门禁，公交，手机支付等领域发挥作用

近距离无线通信：Bluetooth，WiFi，ZigBee，IrDA（红外），UWB（超宽带），NFC，GPRS（全球定位系统），EMTS（增强型数据速率GSM演进技术）

### NFC三种工作模式
读写模式（读卡器模式）：非接触式阅读器功能：可以读写外部非接触式卡片信息，
P2P模式：NFC终端可以与其他NFC设备进行数据交互，实现下载音乐，交换图片等
卡模拟模式（支付模式）：NFC终端模拟为一张非接触式卡片与读卡器进行交互，用来读取人身上带的行用卡，积分卡等

# Chapter Two NFC技术标准，协议基础

NFC技术标准主要包含四层：Mode Switch, RF Layer ISO层，NFC Protocol, Application
RF Layer(射频层)：ＮＦＣ通信距离大概１０ｃｍ左右
NDEF: NFC数据交换格式：标准的数据传递协议
RTD: Record  type Definition:NFEF数据格式中定义的数据类型
Card Emulation: NFC设备模拟为卡片的标准，尚未成熟


### 2.4 NDEF协议

**NDEF 封装了NFC标签的多种细节信息，因此应用不用关心是在与何种标签通信**

NDEF　是一种轻量级的紧凑的二进制格式，可带有URL,vCard,和NFC定义的各种数据类型
NDEF交换的信息以一系列的记录Record组成
每条记录包含一条有效载荷
记录内容可以是URL，MIME媒体文件或者NFC自定义的数据类型

![NDED 数据组成](https://img-blog.csdnimg.cn/20200518155129749.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518155223266.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518155359236.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518155411569.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518160047976.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518160058411.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518160220519.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
### 2.5 RTD
- 文本RTD
- URI RTD
- Smart Poster RTD：将URL，SMS，等信息综合到了Tag中。
- 通用控制 RTD
- 签名 RTD 


#### RTD_TEXT
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518161355560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)

#### RTD_URI
- 描述从NFC兼容的Tag中取得一个URI，或者在两个NFC设备之间传输URI数据，同时提供另外一种在另一个NF元素（如海报）里存储URI的方法

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518161526875.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020051816175954.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
#### RTD——SmartPoster
sp是将URL sms等信息综合到了一个Tag中，内容其实就是RTD_URI RTD_TEXT 等记录的组合

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518162001323.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)


### 2.6 LLCP logical link control protocol
逻辑控制层协议 为两个NFC设备之间的上层信息单元传输提供了 逻辑物理过程，这组过程对 数据链路服务的用户展现了一个统一的 链路抽象，类比计算机网络的内容就可以，几乎完全一样

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020051816225737.png)

- SSAP: SourceServiceAccessPoint 源服务访问点
- DSAP:DestinationServiceAccessPoint 目标服务访问点

SSAP和DSAP在LLCPDU的头部定义；LLCP支持不超过15个未确认的信息单元

### ２.7 协议汇总
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518162914418.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518162925223.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)


#  Chapter Three NFC开发的Android基础
- gen 文件夹：generated 索引文件目录，建立项目时系统自动生成的，比如 R.java
- assert 资源文件目录，与res的最大使用上的区别是：res目录会为资源生成索引使用的ID，但asset不会。
- res 资源文件夹： 比如drawable，xml文件
- proguard.cfg 代码混淆等手段防止反编译


** Intent-Filter**
NFC开发中的标签调用机制大量的使用了Intent-filter，


# Chapter Four NFC Android API 应用开发

### 4.1 Android中的NFC API
- android.nfc 包：主要提供了NFC设备对NFC tag读写NDEF消息的操作函数，以及两个NFC设备之间的数据交换的函数（一个NFC设备 一个NFC tag），主要有6个类
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518165132774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518165319250.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)

- android.nfc.tech 包：主要提供对不同的tag的读写操作类
- ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518170436508.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)

#### 4.1.4 NFC API 使用第一步：申请NFC权限
- \<user-permission android:name="android,permission.NFC">
- 确定满足相应的最小SDK版本号支持，比如：api9+ api16+
- 只针对应用市场的有NFC功能的用户：
		\<use-feature android:name="android.hardware.nfc" android:required="true">

#### 4.1.4 NFC API 使用第二步：NfcAdapter 的监听通知，NFC 支持检测
用来定义一个Intent，使系统在检测到NFC Tag时通知定义的Activity，并提供用来注册 forground tag消息发送的方法等。
也可以使用NfcAdapter的可用性检测用户的手机是否支持NFC功能以及NFC是否开启。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518171514596.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518171552626.png)
- 大多数Android设备只有一个NFC Adapter，而NFCManager是用来管理Android设备中指出的所有NFC adapter，因此，可以直接使用getDefaultAdapter获取系统支持的Adapter
- isEnable：NFC功能
```java
startActivity(new Intent(Settings.ACTION_NFC_SETTINGS));
//显示NFC的设置页面
```
- isNdefPushEnable() :判断设备的Beam功能是否enable，如果

```java
startActivity(new Intent(Settings.ACTION_NFCSHARING_SETTINGS));
//跳转到 Beam功能的设置页面
```

### NFC 标签调度系统
#### 1. NFC前台调度系统
- 前台调度机制允许 Activity 拦截 Intent 对象，并且声明该Activity的优先级别的高
- 在一些涉及需要在前台呈现的页面中直接获取或者推送NFC信息时的场景很方便

开发：


```java
// 1. 在Activity的 onCreate 方法中添加
// 创建一个PI对象，以便Android系统能在扫描到NFC标签时，用它来封装NFC标签的详细信息
PendingIntent pi = PendingIntent.getActivity(this, 0, new Intent(this, getClass()).addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP),0);

// 声明想要截取处理的Intent 对象过滤器，前台调度系统会在扫描到NFC标签时，用声明的Intent过滤器来检查接受到的Intent对象

IntentFilter ndef = new IntentFilter(NfcAdapter.ACTION_NDEF_DISCOVERED);
try{
	// 处理所有的MIME数据包，你应该明确定义一种你需要的类型
	ndef.addDataType("*/*");
}catch(MalformedMimeTypeException e){
	throw  new RuntimeException("fail",e);
}

intentFiltersArray = new IntentFilter[]{ndef,};

// 建立一个应用程序希望处理的NFC标签技术数组，调用 Object.class.getName()方法来获取想要支持的技术的类
techListsArray = new String[][]{new String[]{NfcF.class.getName()}};

// 重写下列Activity生命周期的回调方法
mAdapter.disableForegroundDispatch(this);// Pause
mAdapter.enableForegroundDispatch(this, pendingIntent, intentFilterArray, techListArray);// Resume
Tag tagFromIntent = intent.getParcelableExtra(NfcAdapter.EXTRA_Tag);// onNewIntent

```

#### 2. NFC标签调度系统（NFC Tag Dispatch System）
通过预先定义好的Tag或NDEF消息来启动应用程序的机制：当扫描到一个NFC Tag 时候，如果Intent中注册了对应的应用程序，那么在处理该Tag时候就会启动该应用
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518175829787.png)
Android系统中的工作流程为：
- 解析NFC标签并搞清楚标签中标识数据负载的MIME类型或者URI
- 把MIME或者URI数据封装到一个Intent中
- 基于Intent来启动Activity。

##### 2.1 NFC 标签映射
1. 了解NFC标签的解析
	1. 读取第一条NdefRecord，来判断如何解析整个NDEF消息
	2. 使用TNF和类型字段，尝试把MIME或URI数据类型映射到NDEF中，将荷载信息包装到Intent对象中
	3. 其他异常处理
2. 检测到NDEF消息时，标签调度系统做了哪一些工作
	1. 完成了NFC标签的信息封装和Intent创建后，将Intent发送给感兴趣的应用程序
	2. 如果有多个应用程序可以处理该Intent对象时，弹出Activity选择器

标签调度系统定义了三种Intent对象：

1. ACTION_NDEF_DISCOVERED： 用于启动包含NDEF负载和已知类型的标签的Activity，最高优先级的Intent，并且标签调度系统在其他任何Intent之前
2. ACTION_TECH_DISCOVERED：如果没有注册处理上一个类型Intent的Activity，那么标签调度系统会尝试使用这种类型的Intent来启动应用程序。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518195225785.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
3. ACTION_TAG_DISCOVERED：最低优先级

#### NFC Intent 过滤器
1. 通常使用 ACTION_NDEF_DISCOVERED类型的Intent
2. 在没有过滤 ACTION_NDEF_DISCOVERED类型的Intent的应用程序 或者 数据负载不是NDEF的数据时，才会从 _NDEF_ 退化为 _TECH_类型的Intent

##### 1. ACTION_NDEF_DISCOVERED：要过滤此种Intent，需要将需要过滤的数据一起声明
```xml
<activity>
	<intent-filter>
		<action android:name="android.nfc.action.NDEF_DISCOVERED"/>
		<category android:name="android.intent.category.DEFAULT"/>
		<data android:scheme="http"
			android:host="cnblogs.com/suno/"
			android:pathPrefix="" 
			/>
	</intent-filter>

</activity>
```
```java
// 创建方式一
NdefRecord rtdURI = NdefRecord.createUri("http://www.cnblogs.com/suno/");
// 创建方式二
NdefRecord.createUri(Uri uri);
// 创建方式三
new NdefRecord(NdefRecord.TNF_WELL_KNOWN, NdefRecord.RTD_URI, new byte[0], payload);
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518204706850.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)

#### ACTION_TECH_DISCOVERED
如果 Activity要过滤ACTION_TECH_DISCOVERED 类型的Intent，就必须要创建一个XML资源文件，该文件在tech-list集合中指定Activity所支持的技术的一个子集，那么Activity被认为是匹配的，通过getTechList()方法来获得标签所支持的技术集合

比如：如果扫描到的标签支持 MifareClassic，NdefFormatable， NfcA，那么为了跟他们匹配，tech-list集合就必须要包含所有的这三种技术，或者指定其中的一种或两种

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518205701106.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518205722182.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)

#### ACTION_TAG_DISCOVERED：使用简单

```xml
<activity>
	<intent-filter>
		<action android:name="android.nfc.action.TAG_DISCOVERED"/>
	</intent-filter>
</activity>
```

### Android Application Record
#### AAR简述

- 在api14 时引入
- 场景：防止其他应用对相同的Intent过滤并潜在的处理用户部署的特定的NFC标签，就要用到AAR
- 原理：AAR提供了标签信息与应用程序之间信息传递的唯一对应性：通过应用程序的包名来保证实现的：AAR将应用程序的包名嵌入到NDEF消息封装中，系统会针对AAR来搜索这个NDEF消息，如果找到AAR，就会基于AAR内部记录的包名来启动应用程序。
- 如果手机上没有安装与NFC标签中的Intent过滤器的程序，手机不会任何反应，有AAR的话就会跳转到应用市场来下载对应的应用程序


#### AAR VS Intent 区别
1. 所在处理的级别：AAR 只能根据记录定义到包名，也就是 应用程序；Intent可以配合IntentFilter找到对应的Activity
2. 量的对应关系：处理Intent的Activity可以有多个，但是 AAR只能是一个应用
3. AAR 的优点是：在用户手机没有安装对应TAG发送的NDEF消息处理程序时，可以跳转到应用市场进行下载
4. 可使用的api版本：AAR 在api14+，Intent在 api9+


#### AAR使用
```java
String packageName = "com.campuselife.nfc";
NdefMessage msg = new NdefMessage(
	new NdefRecord[]{record1,record2,record3,...,NdefRecord.createApplicationRecord(packageName)}
	);
// 使用包名创建一个NDEF消息，最后一个是AAR 记录数据包

// 官方要求：不要将AAR放在NDEF消息的第一条记录（除非是唯一一条记录），因为
// Android系统会检查NdefMessage的第一条记录来判断NFC 标签的MIME类型或URI，用于封装数据Intent包
```

使用createApplicationRecord 方法时 ，注意：
 
- 在enableForegroundDispatch 中，createApplicationRecord 不会对其造成影响
- 需要在api14+系统


### NFC 基础知识到此完结 