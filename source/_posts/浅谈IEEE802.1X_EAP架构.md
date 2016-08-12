title: 浅谈 IEEE 802.1X EAP 架构
date: 2016-03-22 19:46:35
tags:
categories:
- 网络协议
---

## 一线三点基本角色关系
supplicant 申请者 ---- Authenticator 验证者 ---- Authentication Server 验证服务器
<!-- more -->
## IEEE 802.1X :

802.1X，802.1X，802.1X 此处的X需要是大写，标准的应该是大写，但是由于802.11协议簇的出现和学习，很多人都把这个大写X给忽略了，所以拿出来说三遍提醒自己。

802.1X是指IEEE制定的用户接入网络的认证标准，全称是“基于端口的网络**接入控制**”，802.1X属于802.1协议组，连接到WLAN或LAN就需要这种认证机制。

协议运行在数据链路层EAP协议RADIUS协议，也就是运行在以太网，WLAN网络之前。


![](http://7xrw2w.com1.z0.glb.clouddn.com/802.1X_wired_protocols.png)

EAP数据首先被封装在EAPOL帧中，传输于申请者（Supplicant）和验证者（Authenticator）之间。随后又封装在RADIUS或Diameter，传输于验证者和验证服务器（Authentication server）之间。

## suplicant 申请者
简单理解就是客户端设备，我们的电脑，手机等网络设备。

在无线网络中这个设备需要具备的要求：安装有无线网卡，并安装有支持特定EAP method 的supplicant的程序，[了解wpa_supplicant](http://coffee1993.github.io/2016/03/18/%E6%98%8E%E7%99%BDwpa-supplicant/)。

## Authenticator 验证者

在以太网中，Authenticator就是一台以太网交换机，无线网络中，它就是一台AP(Access point 无线接入点)。

## Authenticator 验证服务器
运行着支持RADIUS 和 EAP 协议的服务器主机，服务器维护者申请者的相关账号记录，它也保有一份自己的身份证明。
