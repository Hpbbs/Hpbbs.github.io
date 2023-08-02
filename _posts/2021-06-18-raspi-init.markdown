---
layout: post
title: 树莓派初始化-无屏幕仅路由器和板子
author: Hpbbs
categories: [硬件,树莓派]
---

### 官网下载 raspi-imager

### 打开，选择对应版本

下属教程只适用于 raspberry pi os 的各个版本os
不适合 ubuntu android 等

### touch wpa_supplicant.conf

```conf
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
ssid="Zion_net/Xiaomi_999E"
psk="lhbshwqw123/wifi.71201"
priority=1
} 
```

设置默认连接的wifi以及优先级， network可以多个，代表自动连接多个wifi热点

### touch ssh

创建ssh 文件于启动盘，告诉rasppi开机时候打开 ssh 登录

### ssh pi@ip-address -p raspberry

在局域网内通过ssh协议登录终端

### sudo raspi-config

1. 修改密码
2. 修改主机名
3. 打开相应的开关

### 传输文件

scp

```shell

scp -r pi@192.168.31.182:/path-to /local/path

scp A B
A --> B
```

### 更改repo

`sudo vim /etc/apt/sources.list`
```
deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi
deb-src http://mirrors.ustc.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi
```
`sudo vim /etc/apt/sources.list.d/raspi.list`
```
deb http://mirrors.ustc.edu.cn/archive.raspberrypi.org/debian/ buster main
deb-src http://mirrors.ustc.edu.cn/archive.raspberrypi.org/debian/ buster main
```

```
sudo apt-get update
sudo apt-get upgrade
```

### install LAMP

同centos，使用yum包管理器