---
layout: post
title:  "Rust 安装，卸载，nightly/stable版本切换（全局或工作空间），提高下载速度"
date:   2020-01-26
categories: [ComputerScience]
tags: [Rust]
published: true
---

***版权所有，转载前注明原址***
***时间：2020年1月26日20:10:44***   
***created by：Hpbbs***

* toc
{:toc}

### 1.1 安装
**1.1.1 下载Rust的安装器**
Rust推荐使用rustup程序来管理rust编译器等基本工具，可以官网现在得到[Rustup 安装包地址](https://www.rust-lang.org/zh-CN/tools/install)   

![安装截图](https://img-blog.csdnimg.cn/20200126174849569.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)

> *rustup是安装和管理 **Rust 构建版本**的工具。* rustup 用于管理不同平台下的 Rust 构建版本并使其互相兼容， 支持安装由 Beta 和 Nightly 频道发布的版本，并支持其他用于交叉编译的编译版本
*cargo是rust的**包管理器和构建系统工具***。它将常用命令集于一身，无需引入其它命令。

>rustup程序是rust的安装程序，也是他的版本管理程序，类似于Python的Anaconda发行版的conda工具，非常方便使用管理。cargo是rust的构建工具，暂不介绍，需要明白的是：rustup是管理语言自身的，cargo是管理第三方拓展的。

**1.1.2 运行安装器配置安装**
此处以stable版本为例
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200126175116852.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
![安装配置](https://img-blog.csdnimg.cn/2020012617572323.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
Ok，这就安装成功了，这就是使用了安装器的简洁之处，当然我们需要检测是否安装成功

**测试是否安装成功**
运行命令行：<kbd>win</kbd>+<kbd>R</kbd>，输入cmd
输入
> rustc --version
> 或者 rustc -V

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200126180108571.png)
若如图出现版本号，便安装成功，如若出现问题，需检测环境变量，如下：

**配置 PATH 环境变量**
在 Rust 开发环境中，所有工具都安装在 ~/.cargo/bin 目录中，您可以在这里找到包括 rustc、cargo 和 rustup 在内的 Rust 工具链。

Rust 开发者通常会将该目录加入 PATH环境变量中。在安装过程中，rustup 会尝试配置 PATH。 由于不同平台、命令行 Shell 之间存在差异，rustup 中也可能存在 Bug，因此在终端重启或用户重新登录之前，rustup 对 PATH 的修改可能不会生效，甚至完全无效。

如果安装后在终端尝试执行 rustc --version 失败，那么，以上内容就是最可能的原因。

**安装指定版本**
运行命令行：<kbd>win</kbd>+<kbd>R</kbd>，输入cmd
输入
```powershell
rustup install 版本号
```

### 1.2 卸载
由于使用安装器 ，卸载Rust环境很简单：
运行命令行：<kbd>win</kbd>+<kbd>R</kbd>，输入cmd
输入
```powershell
rustup self uninstall
```
### 1.3 更新

运行命令行：<kbd>win</kbd>+<kbd>R</kbd>，输入cmd
输入
```powershell
# 更新到目前最新版本
rustup update
# 更新到指定版本号
rustup update _version_number_xxx
```

不指定版本号命令会对toolchain的所有版本（stable，nightly，beta）的rust更新，如果只安装了其中一个，则只会更新一个。下图是有nightly和stable环境的情形
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200126190441466.png)


### 1.4 Rust镜像源切换
研究Rust时，由于众所周知的原因，发现拉取 crates.io 仓库代码实在太慢，cargo安装下载更新慢的简直让人无法容忍，很多次超时导致引用库没法编译，就把它的更新下载源给替换了，这里可以切到国内镜像(中科大镜像源)，配置如下：

找到当前用户目录下 .cargo/ 的.cargo 文件夹，进入.cargo 当前目录，在当前目下创建 config 文件

打开 config 文件，编写以下内容：

>[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index"

如果所处的环境中允许使用 git 协议，可以把上述地址改为：
registry = "git://mirrors.ustc.edu.cn/crates.io-index"

可以自行搜索其他镜像源

### 1.5 Rust nightly/beta/stable版本切换
#### 1.5.1 查看已安装版本
运行命令行：<kbd>win</kbd>+<kbd>R</kbd>，输入cmd
输入
```powershell
rustup toolchain list
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200126190712114.png)
或者，进入<code>$Home/.rustup/toolchain</code>查看已安装的版本

#### 1.5.2切换到特定版本
运行命令行：<kbd>win</kbd>+<kbd>R</kbd>，输入cmd
输入
```powershell
rustup default stable/nightly/beta
```

**切换全局Rust环境**：defalut
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200126192931198.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)
<code>rustup default version</code>   
若version为nightly，则转为nightly
若version指定stable，则切换为stable

若计算机中为安装nightly或stable，执行命令是将自动下载对应版本

**切换工作目录的Rust环境**：override
在对应的工作目录打开命令行，如下输入命令：
<code>rustup override set version</code>
version:stable/nightly/beta
```powershell
PS D:\workspace\rustwp> rustc -V
rustc 1.40.0 (73528e339 2019-12-16)
PS D:\workspace\rustwp> rustup override set nightly
info: using existing install for 'nightly-x86_64-pc-windows-msvc'
info: override toolchain for 'D:\workspace\rustwp' set to 'nightly-x86_64-pc-windows-msvc'

  nightly-x86_64-pc-windows-msvc unchanged - rustc 1.42.0-nightly (6d3f4e0aa 2020-01-25)

PS D:\workspace\rustwp> rustc -V
rustc 1.42.0-nightly (6d3f4e0aa 2020-01-25)
PS D:\workspace\rustwp>

```
**切换工作目录的rust环境为全局默认环境**：
<code>rustup override unset</code>

```powershell
PS D:\workspace\rustwp> rustup override unset 
info: override toolchain for 'D:\workspace\rustwp' removed
```

### 欢迎关注技术复兴
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200116215717390.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Rvd2VyT3M=,size_16,color_FFFFFF,t_70)

**toolchain介绍**
需要明白 <code>toolchain</code>在rust中的含义：当做版本来认知，方便管理

可以在命令行输入<code>rustup help toolchain</code>来查看和理解toolchain是什么和如何使用

```
C:\Users\18755>rustup help toolchain

rustup-toolchain
Modify or query the installed toolchains

USAGE:
    rustup toolchain <SUBCOMMAND>

FLAGS:
    -h, --help    Prints help information

SUBCOMMANDS:
    list         List installed toolchains
    install      Install or update a given toolchain
    uninstall    Uninstall a toolchain
    link         Create a custom toolchain by symlinking to a directory
    help         Prints this message or the help of the given subcommand(s)

DISCUSSION:
    Many `rustup` commands deal with *toolchains*, a single
    installation of the Rust compiler. `rustup` supports multiple
    types of toolchains. The most basic track the official release
    channels: 'stable', 'beta' and 'nightly'; but `rustup` can also
    install toolchains from the official archives, for alternate host
    platforms, and from local builds.

    Standard release channel toolchain names have the following form:

        <channel>[-<date>][-<host>]

        <channel>       = stable|beta|nightly|<version>
        <date>          = YYYY-MM-DD
        <host>          = <target-triple>

    'channel' is either a named release channel or an explicit version
    number, such as '1.8.0'. Channel names can be optionally appended
    with an archive date, as in 'nightly-2017-05-09', in which case
    the toolchain is downloaded from the archive for that date.

    The host may be specified as a target triple. This is most useful
    for installing a 32-bit compiler on a 64-bit platform, or for
    installing the [MSVC-based toolchain] on Windows. For example:

        $ rustup toolchain install stable-x86_64-pc-windows-msvc

    For convenience, omitted elements of the target triple will be
    inferred, so the above could be written:

        $ rustup toolchain install stable-msvc

    The `rustup default` command may be used to both install and set
    the desired toolchain as default in a single command:

        $ rustup default stable-msvc

    rustup can also manage symlinked local toolchain builds, which are
    often used for developing Rust itself. For more information see
    `rustup toolchain help link`.

```


**override介绍**

```powershell
PS D:\workspace\rustwp> rustup help override
rustup.exe-override 
Modify directory toolchain overrides

USAGE:
    rustup.exe override <SUBCOMMAND>

FLAGS:
    -h, --help    Prints help information

SUBCOMMANDS:
    list     List directory toolchain overrides
    set      Set the override toolchain for a directory
    unset    Remove the override toolchain for a directory
    help     Prints this message or the help of the given subcommand(s)

DISCUSSION:
    Overrides configure rustup to use a specific toolchain when
    running in a specific directory.

    Directories can be assigned their own Rust toolchain with `rustup
    override`. When a directory has an override then any time `rustc`
    or `cargo` is run inside that directory, or one of its child
    directories, the override toolchain will be invoked.

    To pin to a specific nightly:

        $ rustup override set nightly-2014-12-18

    Or a specific stable release:

        $ rustup override set 1.0.0

    To see the active toolchain use `rustup show`. To remove the
    override and use the default toolchain again, `rustup override
    unset`.
```
