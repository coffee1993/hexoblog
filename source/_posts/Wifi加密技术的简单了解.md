title: Wifi加密技术的简单了解
date: 2016-03-18 20:49:10
tags:
- wifi
categories:
- Android
---


> 学java的时候，了解过[对称加密技术和非对称加密技术]()，


# WEP (wired Equevlent privacy 有线等效加密)
名字由来
  - WEP加密技术是要提供相同于有线局域网相当的机密性，而以此命名
WEP 安全技术源于名为RC4(Rivest Chiper) 的串流加密技术，使用CRC-32验和，属于一种对称加密技术，到现在为止，这种加密技术并不安全，所以称为WEP为最快能破解的加密技术，常见的有128位WEP加密和64位WEP加密，64位的更弱。
<!-- more -->

  - 加密过程：

  ![加密算法过程图](http://7xrw2w.com1.z0.glb.clouddn.com/WEP%E7%A7%98%E9%92%A5%E5%8A%A0%E5%AF%86%E5%9B%BE.jpg)

  标准的64比特的WEP是用40bit的秘钥+24bit的IV(initailization vector)初向量 及KSA = IV + PASSWORD，PGRA是用RC4生产出的密钥流序列PGRA = RC4(KSA),PGRA再同明文DATA+CRC-32完整性校验值进行异或运算,的到Encrypted data加密数据，最后又再次用到IV，将IV + Encryted data 一起发出去


  - 接收方解密过程

  ![接收方解密](http://7xrw2w.com1.z0.glb.clouddn.com/WEP%E7%A7%98%E9%92%A5%E6%8E%A5%E6%94%B6%E7%AB%AF%E8%A7%A3%E5%AF%86%E5%9B%BE.jpg)

  CIPHERTEXT 为密文。它采用与加密相同的办法产生解密密钥序列，再将密文与之XOR 得到明文，将明文按照CRC32 算法计算得到完整性校验值CRC-32′，如果加密密钥与解密密钥相同，且CRC-32′= CRC-32，则接收端就得到了原始明文数据，否则解密失败

## WEP的不足
 - WEP的破解理论在01年被S.Fluhrer， I.Martin 和A.Shamir 证明：利用已知的初始矢量IV 和第一个字节密钥
流输出，并结合RC4 密钥方案的特点，攻击者通过计算就可以确定WEP密钥。
 - CRC-32算法存在缺陷,802.11b 允许IV重复使用，CRC算法并不加密，攻击者可以利用CRC构造自己的加密数据，加密数据就被篡改
 - 因为其自身密码体制的缺陷，攻击者只要收集足够的数据包，就可以分析并还原密码。

## 为什么保留了WEP加密技术？

WEP 加密技术弱却为什么保留下来了呢？因为网卡等操作系统原因，为了满足一下早期无线用户的需要，旧设备只有WEP加密技术才能无线网络连接成功，所以保留了加密技术。


# WPA/WAP2 加密技术

## WPA/WPA2加密技术相对于WEP的安全性如何提高
  - 增加了身份验证机制：Pre-shardKey,EAP(一种[协议架构](http://coffee1993.github.io/2016/03/22/%E6%B5%85%E8%B0%88IEEE802.1X_EAP%E6%9E%B6%E6%9E%84/)，并非是一种算法，常见的有TLS，PEAP,LEAP,MD5),Pre-shardKey 是家庭验证级别及我们常见的wifi密码，不需要服务器，而EAP是工业级别，需要服务器验证，如TLS。
  - 用TKIP 和 AES 代替了RC4串流加密技术
  - 用MIC 和 CCMP 代替了CRC-32 循环冗余码的检验

## WPA和WPA2的不同
  - WPA使用TKIP + MIC 的方式进行数据传输加密和数据完整性编码验证，而WPA2 使用 AES + CCMP进行了数据传输加密算法和数据完整性编码校验。
  - WPA = 802.1X + EPA/Pre-shardKey + TKIP + MIC (工业级别 EAP/家庭级别Pre-shardKey)
  - WPA2 = 802.1X + EAP + AES + CMPP (工业级别 EPA /家庭级别 Pre-shardKey)

## WPA-PSK四次握手
  -
