title: 单独编译打包Android应用程序模块
date: 2016-04-06 20:54:39
tags:
categories: Android
---
> 在Android源代码工程目录下执行make命令，经过漫长的等待之后，就可以得到Android系统镜像system.img了。当我们开发了一个Android应用程序，肯定是不需要再次漫长的make的，通过记录单独编译和打包Android应用程序模块，加深对其理解和记忆

## 导入单独编译模块的命令：mmm

```bash
  coffee@ubuntu:~/Android$ source ./build/envsetup.sh
```
  - 默认情况下，mmm命令是不可以用的，我们需要在当前的终端中导入mmm命令，执行上述脚本，可以获得许多有用的工具。

  <!-- more -->

## mmm 执行编译
  - mmm -help 可以看出其本身可以带很多参数，遇到什么再看那个参数干嘛的吧，一般来说mmm 工程项目路径就行了
  
```bash
  coffee@ubuntu:~/Android$ mmm ./packages/experimental/animation/
  ...m

  ...

  Processing target/product/generic/obj/APPS/animation_intermediates/package.apk
  Done!
  Install: out/target/product/generic/system/app/animation.odex
  Install: out/target/product/generic/system/app/animation.apk
  make: Leaving directory /home/coffee/Android'

```
  - 编译打印的信息告诉了我们apk的路径。Android系统自带的App都放在这具目录下。另外，Android系统的一些可执行文件，例如C编译的可执行文件，放在out/target/product/generic/system/bin目录下，动态链接库文件放在out/target/product/generic/system/lib目录下，out/target/product/generic/system/lib/hw目录存放的是硬件抽象层（HAL）接口文件。


## 重新打包Android 系统镜像文件

  - 译好模块后，还要重新打包一下system.img文件，这样我们把system.img运行在模拟器上时，就可以看到我们的程序了。
  
```bash

  coffee@ubuntu:~/Android$ make snod
```

## 后台启动模拟器 查看应用程序

```bash
  $ source build/envsetup.sh
  $ lunch sdk-eng
  $ emulator -kernel ./kernel/goldfish-android-goldfish-3.4/arch/arm/boot/zImage &
```
  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_anim.png)
