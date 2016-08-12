title: 明白wpa_supplicant
date: 2016-03-18 20:50:42
tags:
categories:
- Android

---

> 谈谈WPA

wpa_supplicant的前缀是wpa,在wifi加密技术一文中谈到了WPA加密技术,[简单介绍wifi加密技术](http://coffee1993.github.io/2016/03/18/Wifi%E5%8A%A0%E5%AF%86%E6%8A%80%E6%9C%AF%E7%9A%84%E7%AE%80%E5%8D%95%E4%BA%86%E8%A7%A3/)，从名词上简单推测这个wpa可能跟wifi加密技术相关

<!-- more -->

> wpa_supplicant 历史简介

单词Supplicant的本意为客户端，请求者。 wpa_supplicant 本是一个开源项目，支持IEEE 802.11i无线接口所定义的WPE, WPA和WPA2安全加密协议，作为一个WPA的应用层认证客户端程序用于桌面操作系统，嵌入式系统，或者便携式电脑，unix,Linux,windows,Mac OS X操作系统等，主要负责完成认证及登录工作。后来被google修改后加入android移动平台。


> 具体工作

- 1.wpa_supplicant的具体工作内容是通过Socket与下层的驱动交互，与上层的用户交互，用户可以通过socket发送命令给wpa_supplicant调动驱动来对wifi芯片进行操作
- 2.对IEEE 802.11i无线接口所定义的WPE, WPA和WPA2安全加密协议的支持

> wpa_supplicant 的解剖

wpa_supplicant 是一个独立运行的[守护进程](www),其核心是一个消息循环，处理WPA状态机，控制命令，驱动事件，配置信息。
编译后的wpa_supplicant 可分为
- wpa_supplicant
  - 后台运行
- wpa_cli
  - 搜索，设置，连接网络
- 代码结构
  - 核心功能
  - 通用辅助功能
  - 加密功能
  - TLS库
  - 配置
  - 控制接口
  - WPA提供者
  - EAP点
  - EAPOL提供
  - 窗口端口和测试程序
具体各个模块，需要的时候继续再分析
附上一张经典的图，介绍和图来自于 http://w1.fi/wpa_supplicant/devel/

![wap_supplicant](http://w1.fi/wpa_supplicant/devel/_wpa_supplicant.png)


> 被goole武装后在Android上的应用

wap_supplicant被 goole修改后，用于了android wifi 模块来配置wifi,它是一个安全中间件,为无线网卡提供统一的安全机制。

Android wifi控制的三大模块

- 最上层: wpa_cli 命令行 + jFrame java图形化界面, 通过unix的本地socket 和 wap_supplicant daemon服务通信,发送命令并接收结果。


- 中间层: wpa_supplicant daemon 服务，起到了一个中间上传下达的作用，对网卡发送字符串命令控制wifi驱动，并将返回状态上传给用户。网卡厂商都设计了一个通用的接口给wpa_supplicant调用

- 底层:  wifi网卡驱动

wpa_supplicant 也是android wifi开发主要依赖的一个模块，主要作用：
- 1.读取配置文件
- 2.初始化配置参数，驱动函数
- 3.让驱动scan当前所有的BSSID(AP mac地址)
- 4.检查扫描的从参数是否和用户设置的相同
- 5.权限验证并连接

android 的wifi驱动模块

> wpa_supplicant 的使用例子

wpa_supplicant　在各种操作系统使用网上都有实际的教程，下面介绍一些链接可供参考
- 1.在linux中配置无线网卡
