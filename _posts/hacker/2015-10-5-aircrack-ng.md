---
layout: post
category: hacker
title: aircrack-ng在Ubuntu下破解wifi密码
tagline: by 赵宏阳
tags: [wifi 破解]
---

<!--more-->

# 1、获取aircrack-ng #

可以使用`apt-get install aircrack-ng`安装，也可以去[官网](
http://www.aircrack-ng.org/doku.php?id=downloads#linux_packages)找到ubuntu的下载地址，可以下载deb包安装，也可以[下载源码](http://ubuntu2.cica.es/ubuntu/ubuntu/pool/universe/a/aircrack-ng/aircrack-ng_1.0.orig.tar.gz)，这里演示通过源码编译和安装的过程：运行`make`,可能提示`set but not used [-Werror=unused-but-set-variable]`这个错误，修改根目录下的`common.mak`中的`CFLAGS          ?= -g -W -Werror -Wall -O3`，把`-Werror`去掉。（原理是`-Werror`表示把所有的告警信息转化为错误信息，并在告警发生时终止编译过程。）即可正确编译，然后`make install`安装。安装完成后运行`aircrack-ng`，如果有提示信息，则说明安装正确。

# 2、熟悉破解过程 #

Aircrack-ng(大写的A)是一个工具包，里面包含其他工具：

- aircrack-ng破解，只要airodump-ng收集到足够数量的数据包，aircrack-ng就可以自动检测数据包并判断是否可以破解。
- airmon-ng处理网卡工作模式。
- airodump-ng用于捕获802.11数据报文，以便于aircrack-ng破解。
- aireplay-ng发包，干扰。在进行WEP及WPA-PSK密码恢复时，可以根据需要创建特殊的无线网络数据报文及流量。

下面的步骤是：

1. 抓取cap文件
2. 破解cap文件得到密码。

为了给大家一个感性的认识首先给大家一个抓好的[cap文件](http://download.aircrack-ng.org/ptw.cap)，官方对这个文件的解释是：`This is a 64 bit WEP key file suitable for the PTW method`，运行`aircrack-ng ptw.cap`，几秒后就可以看到密码`1F:1F:1F:1F:1F`。

下面演示如何抓取到我们自己的包。

# 3、抓包 #

## 3.1环境准备 ##

`iwconfig`能否看到无线网卡。

## airmon-ng ##

启动无线网卡进入Monitor模式：`airmon-ng start wlan0`

停止监听：`airmon-ng stop mon0`

## airodump-ng ##

`airodump-ng mon0`

`airodump-ng --ivs –w packs -c 11 mon0`

## aireplay-ng 对目标AP使用ArpRequest注入攻击。##

### WEP模式下使用 ###

若连接着该无线路由器/AP的无线客户端正在进行大流量的交互，比如使用迅雷、电骡进行大文件下载等，则可以依靠单纯的抓包就可以破解出WEP密码。但是这样的等待有时候过于漫长，于是就采用了一种称之为“ARP Request”的方式来读取ARP请求报文，并伪造报文再次重发出去，以便刺激AP产生更多的数据包，从而加快破解过程，这种方法就称之为ArpRequest注入攻击。具体输入命令：`aireplay-ng -3 -b AP的mac -h 连接AP的客户端的mac mon0`

### WPA模式下使用 ###

进行Deauth攻击加速破解过程。和破解WEP时不同，这里为了获得破解所需的WPA-PSK握手验证的整个完整数据包，无线黑客们将会发送一种称之为“Deauth”的数据包来将已经连接至无线路由器的合法无线客户端强制断开，此时，客户端就会自动重新连接无线路由器，黑客们也就有机会捕获到包含WPA-PSK握手验证的完整数据包了。此处具体输入命令：`aireplay-ng -0 1 –a AP的mac -c 客户端的mac wlan0 `

## macchanger 更改Mac地址##

