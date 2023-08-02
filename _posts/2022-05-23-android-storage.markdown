---
layout: post
title: "Android存储方式的官方推荐"
published: true
categories: [tech]
tags: [Android, 摘抄]
---

### Android官方推荐最佳实践

Android密钥库系统

KeyChain api and jetpack Sercurity库提供，保护密钥材料免遭未经授权的使用

Android 不仅提供数据隔离机制、支持完整文件系统加密并提供安全通信通道，还提供大量使用加密来保护数据的算法

### 最佳实践

数据保存在设备的基本做法

- 内部存储区
- 外部存储区
- ContentProvider

#### 1. 内部存储区

- 避免对 IPC 文件使用 `MODE_WORLD_WRITEABLE` 或 `MODE_WORLD_READABLE` 模式，因为在这两种模式下，系统不提供针对特定应用限制数据访问的功能，也不会对数据格式进行任何控制
- 敏感数据提供额外保护，使用security库对本地文件进行加密，可保护丢失设备上数据，而无需进行文件系统加密



文件级加密：Android 7.0 及更高版本支持[文件级加密](https://source.android.com/security/encryption/file-based?hl=zh-cn)，采用文件级加密时，可以使用不同的密钥对不同的文件进行加密，也可以对加密文件单独解密。支持文件级加密的设备还可以支持[直接启动](https://developer.android.com/training/articles/direct-boot?hl=zh-cn)。该功能处于启用状态时，已加密设备在启动后将直接进入锁定屏幕，从而可让用户快速使用重要的设备功能，例如无障碍服务和闹钟。

元数据加密：Android 9 引入了对存在硬件支持的[元数据加密](https://source.android.com/security/encryption/metadata?hl=zh-cn)的支持。采用元数据加密时，启动时出现的单个密钥会加密未通过 FBE 进行加密的任何内容（例如目录布局、文件大小、权限和创建/修改时间）。该密钥受到 Keymaster 的保护，而 Keymaster 受到启动时验证功能的保护。

#### 2. 外部存储

在[外部存储设备](https://developer.android.com/guide/topics/data/data-storage?hl=zh-cn#filesExternal)（例如 SD 卡）上创建的文件不受任何读取和写入权限的限制，存储非敏感信息

#### 3. 内容提供者

[内容提供程序](https://developer.android.com/guide/topics/providers/content-providers?hl=zh-cn)提供结构化存储机制，可以将内容限制为仅供您自己的应用访问，也可以将内容导出以供其他应用访问。在创建要导出以供其他应用使用的 `ContentProvider` 时，您可以指定允许读取和写入的单一[权限](https://developer.android.com/guide/topics/manifest/provider-element?hl=zh-cn#prmsn)，也可以针对读取和写入操作分别指定权限。您应仅对需要完成相应任务的应用授予权限。请注意，与其因移除权限而影响现有用户，不如以后提供新功能时再添加权限。

如果您仅使用内容提供程序在您自己的应用之间共享数据，最好将使用的 [`android:protectionLevel`](https://developer.android.com/guide/topics/manifest/permission-element?hl=zh-cn#plevel) 属性设置为 `signature` 保护级别。签名权限不需要用户确认，因此，这种方式不仅能提升用户体验，而且在相关应用使用相同的密钥进行[签名](https://developer.android.com/tools/publishing/app-signing?hl=zh-cn)来访问数据时，还能更好地控制对内容提供程序数据的访问。

内容提供程序还可以通过以下方式提供更细化的访问权限：声明 [`android:grantUriPermissions`](https://developer.android.com/guide/topics/manifest/provider-element?hl=zh-cn#gprmsn) 属性，并使用用来启动组件的 `Intent` 对象中的 `FLAG_GRANT_READ_URI_PERMISSION` 和 `FLAG_GRANT_WRITE_URI_PERMISSION` 标记。使用 `<grant-uri-permission>` 元素还能进一步限制这些权限的范围。

## 使用权限

由于 Android 通过沙盒机制管理各个应用，因此应用必须以明确的方式共享资源和数据。为此，应用会通过声明自己需要的权限来获取基本沙盒未提供的额外功能，包括对相机等设备功能的访问权限。

除了请求权限之外，您的应用还可以使用 [``](https://developer.android.com/guide/topics/manifest/permission-element?hl=zh-cn) 元素来保护对安全性要求较高且会被其他应用访问的 IPC，例如 `ContentProvider`。一般而言，我们建议您尽量使用访问权限控制，而不使用需要用户确认的权限，因为权限管理对用户来说可能比较复杂。例如，对于同一开发者提供的不同应用之间的 IPC 通信，不妨对权限使用 [signature 保护级别](https://developer.android.com/guide/topics/manifest/permission-element?hl=zh-cn#plevel)

### AccountManager(账号管理)

可供多个应用访问的服务应使用 `AccountManager` 进行访问。如果可行，请使用 `AccountManager` 类来调用基于云的服务；此外，请勿将密码存储在设备上。

使用 `AccountManager` 检索 `Account` 后，请先确认 `CREATOR` 再传递凭据，以免无意中将凭据传递给错误的应用。

如果凭据仅供您创建的应用使用，那么您可以使用 `checkSignature()` 验证访问 `AccountManager` 的应用。另外，如果只有一个应用使用该凭据，那么您可以使用 `KeyStore` 存储凭据。



目前加密算法，其实也是基于随机数种子，而随机数种子

使用安全随机数生成器 `SecureRandom` 来初始化 `KeyGenerator` 生成的任意加密密钥。如果使用的密钥不是安全随机数生成器生成的，那么会显著降低算法的强度，而且容易导致离线攻击。



#### EncryptedSharedPreference

可以配置加密算法的SharedPreference，对Key 和 Value进行加密

SharedPreference：使用xml这种半结构化描述语言做序列化的 map数据结构

**使用数据结构的标准思想来理解编程世界的一切**：

线程：是代码指令的数据队列
kotlin 的flow：是惰性的序列，同时可以支持线程的异步能力；序列操作的行为被队列化进行排队可控
加密算法：这个好像不能用数据结构的角度来描述
函数：是指令的集合


### 数据的分类

二进制文件：大于4kb 小于4kb
- 图片
- 录音
- 音频
- 自定义二进制文件

文本文件：有编码的二进制，同质的

- 应用专属存储空间：存储仅供应用使用的文件，可以存储到内部存储卷中的专属目录或外部存储空间中的其他专属目录。使用内部存储空间中的目录保存其他应用不应访问的敏感信息。
- 共享存储：存储您的应用打算与其他应用共享的文件，包括媒体、文档和其他文件。
- 偏好设置：以键值对形式存储私有原始数据。
- 数据库：使用 Room 持久性库将结构化数据存储在专用数据库中。

### 重要概念

Environment：语义 设备的环境
Context：语义 App的上下文 ： Android的版本更新中，正在将外部信息都收敛到这个语义中

以下两者容易混淆
内外概念：App私有数据 全局公开数据
内部外部概念：内存/磁盘概念：对内部存储空间目录，在API29以上，会对这些位置加密，且内部空间比较小，使用前需要验证可用空间

持久性文件 还是 缓存文件

ndroid 系统的版本越新，就越依赖于文件的用途而不是位置来确定应用对特定文件的访问和写入能力。特别是，如果您的应用以 Android 11（API 级别 30）或更高版本为目标平台，WRITE_EXTERNAL_STORAGE 权限完全不会影响应用对存储的访问权限。这种基于用途的存储模型可增强用户隐私保护，因为应用只能访问其在设备文件系统中实际使用的区域


### App 私有外部存储空间
1. 需要验证外部存储器的可用性

Environment.getExternalStorageState() 查询该卷的状态
MEDIA_MOUNTED
MEDIA_MOUNTED_READ_ONLY

#### 持久性文件

>= 4.4 19：无需请求权限就可以访问外部的应用专属目录
<= 9 28: 具有适当的存储权限，就可以访问其他应用的应用专属文件
》= 10 29 默认情况授予对外部存储空间的分区访问（分区存储），启用分区存储，就无法访问属于其他应用的应用专属目录

相关api 访问应用专属的外部存储空间
context.getExternalFilesDir()


#### 缓存文件

context.externalCacheDir()
externalCacheDir.delete()

#### App私有外部存储的媒体文件

context.getExternalFilesDir(Environment.DIRECTORY_PICTURES, albumName)

> 请务必使用 DIRECTORY_PICTURES 等 API 常量提供的目录名称。这些目录名称可确保系统正确处理文件。如果没有适合您文件的预定义子目录名称，您可以改为将 null 传递到 getExternalFilesDir()。这将返回外部存储空间中的应用专属根目录


这个是StorageManager的内容
> getAllocatableBytes() 的返回值可能大于设备上的当前可用空间量。这是因为系统已识别出可以从其他应用的缓存目录中移除的文件,如果有足够的空间保存您的应用数据，请调用 allocateBytes()。否则，您的应用可以请求用户从设备移除一些文件或从设备移除所有缓存文件


### App 私有 内部存储空间访问
Context.filesDir  <== 应用的普通持久文件
context.fileList() 获得所有文件名称的数组（filesDir 内的文件）
context.getDir(dirName,Context.MODE_PRIVATE) 创建目录，但是父目录是 filesDir

缓存文件：卸载时删除或者更早的移除

`File.createTempFile(fileName, null, context.cacheDir)` Context.cacheDir

使用File API 或者 context.deleteFile() 来删除


### 共享 存储空间

媒体内容：MediaStore
文档和文件：
数据集：BlobStoreManager

#### 与媒体库抽象互动：ApplicationContext.ContentResolver.Query
系统会自动扫描外部存储卷，并将自动的存储到对应的集合中
图片 /DCIM
/Movies  /Alarms AudioBook Music Notification 
Download/

MediaStore.Images
MediaStore.Video
MediaStore.Audio 
MediaStore.Downloads
MediaStore.Files 如果开启了分区存储，集合只会出现应用创建的所有文件，如果没有开启，则是所有媒体文件

### 数据库

### Sharedprefer

### MediaStore

支持直接加载缩略图
 ApplicationContext.ContentResolver.loadThumbnail(uri, size, null)

等优化过的API可以搭配使用

注意事项：如果缓存了媒体文件，请定期检查媒体库的更新
当使用直接文件路径随机读取和写入媒体文件时，进程速度可能最多慢一倍


### FileProvider：其他应用访问app内部存储的方式：是一种数据外发


### 参考

https://developer.android.com/training/data-storage


### 其他问题

应用安装在里：内部还是外部存储

为确保应用的性能不受影响，请勿多次打开和关闭同一文件

存储空间管理activity：android:manageSpaceActivity

开启了分区存储的应用，果用户卸载并重新安装您的应用，您必须请求 READ_EXTERNAL_STORAGE 才能访问应用最初创建的文件。此权限请求是必需的，因为系统认为文件归因于以前安装的应用版本，而不是新安装的版本。

### 存储卷的概念
以 Android 10 或更高版本为目标平台的应用可以访问系统为每个外部存储卷分配的唯一名称。此命名系统可帮助您高效地整理内容并将内容编入索引，还可让您控制新媒体文件的存储位置。

以下几种卷尤为有用，需要特别注意：

VOLUME_EXTERNAL 卷用于提供设备上所有共享存储卷的视图。您可以读取此合成卷的内容，但无法修改这些内容。
VOLUME_EXTERNAL_PRIMARY 卷代表设备上的主要共享存储卷。您可以读取和修改此卷的内容。
您可以通过调用 MediaStore.getExternalVolumeNames() 查看其他存储卷

### Public 内存的访问和处理

对于 >= 29 版本，requestLegacyExternalStorage，让可以用old api 去操作，而不是仅仅MediaStoreManager管理

对于 >= 6.0 <= android Q 版本访问 公共文件夹，需要 WRITE_EXTERNAL_STORAGE（会隐含申请 READ_EXTERNAL_STORAGE）权限，才能操作，且可使用 File API 操作文件

而 >= android Q 版本，无视 WRITE_EXTERNAL_STORAGE权限，使用MediaStoreManager API