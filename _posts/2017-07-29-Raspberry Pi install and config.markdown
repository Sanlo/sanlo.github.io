---
layout: post
title:  "Raspberry Pi 安装与配置"
categories: IoT
tags: Raspberry 安装 
excerpt: 虽然现在Raspberry Pi的安装随着系统的优化越来越简单，但是还是会遇到各种未知的问题。本文仅就我遇到的问题做个总结笔记。 
---

* content
{:toc}

### 0.0 序
本文主要讲解`Raspberry Pi 3 Model B`的安装与配置，配置完成后，可以通过远程SSH和VNC进行连接。

&emsp;&emsp;**最小采购清单；**

&emsp;&emsp;**安装需要的软件；**

&emsp;&emsp;**安装系统至SD卡；**

&emsp;&emsp;**安装之后的系统配置；**

### 1. 最小采购清单

&emsp;&emsp; Raspberry Pi作为最小的个人电脑，由于只有一块主板，所以有些配件是临时还是必需的。
> Raspberry Pi 3 Model B主板

> SD卡（最好8Gb以上），我选用的是SAMSUNG EVO 32 GB Class 10

> 散热片，风扇

> 亚克力外壳

> 带有USB接口的鼠标，键盘

> 显示器，最好带有HDMI接口

> 网线，HDMI线，如果显示器没有HDMI接口，还需要有VGA转HDMI线

&emsp;&emsp; 除了主板，SD卡，散热片，风扇与外壳是正常工作时要使用的之外，其他均是在第一次安装配置Raspberry Pi要用的。SD卡之所以选用SAMSUNG EVO 32 GB Class 10，是由于Windows 10 IoT对于用于安装的SD卡有严格限制，而这一款正好是其推荐的，如果以后想玩Win10 IoT就省得再买了。由于Raspberry Pi 3已经配置的板载的蓝牙和Wifi，后面可以通过Wifi远程进行SSH和VNC连接。如果需要从外网连接家中的Raspberry Pi，可以使用[RealVNC](https://www.realvnc.com/en/download/viewer/)和[Dataplicity](https://www.dataplicity.com)在任何环境中连接。

### 2. 安装需要的软件

[Raspbian](https://www.raspberrypi.org/downloads/)：除了安装完整版的Raspbian，也可以安装简化版的NOOBS。

[SD Memory Card Formatter](https://www.sdcard.org/downloads/index.html)：SD卡官方格式化软件，用于格式化购买的SD卡

[Etcher](https://etcher.io/)： 树莓派官方推荐的开源系统烧录软件，一个最主要的有点是烧录结束之后会进行系统验证，这是常用的Win 32 Disk image所没有的功能。

[Putty](http://www.putty.org/)：用于PC端远程登录Raspi，当然需要在同一网段下。如果想要在外网访问内网里的Raspberry Pi，则需要使用RealVNC或Dataplicity。

### 3. 安装系统至SD卡

### 4. 安装之后的系统配置

