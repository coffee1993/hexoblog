title: 编译Android4.4源代码及内核源代码的过程记录
date: 2016-03-26 14:32:11
tags:
categories:
- Android
---
## 目的：

> 1.记录编译过程，理解相关的一些细枝末节的东西
> 2.为学习源代码做准备

<!-- more -->

## 操作系统准备

### 安装ubuntu12.04 VMWare 或者纯ubuntu系统或者双系统
>本来装了一个ubuntu12.04但只分配了磁盘20个G，虽然android源代码只有几个G,但是编译出来的SDK有40多个G,分配硬盘根本不够用。所以一定要留有足够的空间，所以重新安装了ubuntu，狠心分配了70G固态，真是下了血本。

>目前公司配了一个256G的固态笔记本thinkpad T450，不用都觉得浪费

### 安装ubuntu纯系统：

U盘启动盘制作工具：
建议使用：
[Universal-USB-Installer-1.9.5.2.exe](http://pan.baidu.com/share/link?shareid=3222351390&uk=1511009177&fid=615110612452976)

ubuntu镜像地址就自己google了，很多很好的镜像源。

### VMware ubuntu 12.04 配置截图：### Android build 的构成
- 主要由make文件、shell脚本、Python脚本构成，主要是make文件。
![](http://7xrw2w.com1.z0.glb.clouddn.com/ubuntu%E7%B3%BB%E7%BB%9F%E9%85%8D%E7%BD%AE.png)


### 安装软件源

经室友百度大神[刘凯](http://www.king-liu.net/)推荐，选择阿里云的源比较快，这里贴个连接
- [阿里云源 http://mirrors.aliyun.com/ ](http://mirrors.aliyun.com/)
- 进入后选择ubuntu help
- 根据自己ubuntu版本选择
- 编辑/etc/apt/sources.list文件(需要使用sudo), 在文件最前面添加以下条目(操作前请做好相应备份)

## 设置windows ubuntu共享文件夹

设置windows ubuntu共享文件夹需要安装VMWare Tools，安装新版的VMWare 已经把VMWare Tools 安装好了。
接下来对虚拟机设置的选项里启动共享文件夹启动，并在windows下创建。ubuntu需要重启。

/mnt/hgfs/ubuntu虚拟机共享文件夹
截图：
![VMWare](http://7xrw2w.com1.z0.glb.clouddn.com/Android_VMWare.png)

## git工具安装
sudo apt-get install git-core gunpg (个人数据安全，用GPG保护个人隐私数据)
sudo apt-get install git; (后面编译要用到,这个也很重要)


## 简单接触Android Build系统

### Android build 系统所解决的问题
简单的说就是用来编译Android的，但是编译Android是一个相当复杂的工程，要实现在不同平台下编译，编译的时候还要面向不同的硬件设备，还要提供面向各个厂商的定制扩展。Android源代码包含很多模块，模块的统一管理也是一个难点。



## 源代码下载及准备

### repo 下载
源列表:(需要翻墙)
[Android版本列表](https://source.android.com/source/build-numbers.html#source-code-tags-and-builds)

google推荐 https://source.android.com/source/downloading.html

选择google推荐的镜像源下载，网络需要翻越

[清华大学镜像源](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)

mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo

如果提示无法连接到 gerrit.googlesource.com，可以编辑 ~/bin/repo，把 REPO_URL 一行替换成下面的：
REPO_URL = 'https://gerrit-google.tuna.tsinghua.edu.cn/git-repo' 我是替换后才成功

然后初始化：
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-4.0.1_r1

输入git 用户名 及邮箱
出现 curl: (22) The requested URL returned error: 404 Not Found Server does not provide clone.bundle; ignoring.
无视即可。

祝你成功！

完成后开启并发同步源码树
repo sync -j4

### 本地下载链接：
[百度网盘下载](http://pan.baidu.com/s/1cFPu6i)
### 解压Android4.4源代码
我用图形化界面直接解压了源代码7z包。
[linux解压大全](http://www.cnblogs.com/eoiioe/archive/2008/09/20/1294681.html)
但是添加了创建的存取源代码文件夹的权限： chmod a+x ./Android

## java JDK

### 安装jdk
>jdk-6u35-linux-i586.bin



这是一个32位的jdk，ubuntu是64位的，所以我安装的时候遇到了问题，需要加载64位的jdk
最终选择的下载版本：
![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_jdk.png)
Oracle Java 1.6
[官网下载地址](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase6-419409.html)
根据自己操作系统版本下载，下载的时候要小心，有个rpm包格式，ubuntu本是不支持rpm安装的，需要把rpm包装换成deb格式
所以下载的时候要选择.bin文件
注意不要用OpenJDK. 这是个坑, 官方文档虽然有写
cd /usr/local
sudo mkdir java
将jdk bin包放入local/java下
修改权限所有用户都可执行
chmod a+x jdk-6u45-linux-x64.bin
安装
./jdk-6u45-linux-x64.bin

### 配置java环境变量

gedit ~/.bashrc

```bash
# java path
export JAVA_HOME=/usr/local/java/jdk1.6.0_45
export JRE_HOME=/usr/local/java/jdk1.6.0_45/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$JAVA_HOME:$PATH

```

由于/etc/profile 与 ~/.bash_profile 都是再取得login shell 的时候才回去读取配置文件，所以将设置写入上述文件后，需要注销再登录后生效。
否则就要利用source命令或者小数点 . 将配置文件的内容读入进目前的shell中。

source ~/.bashrc
java -version
![](http://7xrw2w.com1.z0.glb.clouddn.com/AndroidjavaVersion.png)


## 那些ubuntu编译环境需要使用到的包~  们!


```bash
sudo apt-get install git git-core gnupg flex bison gperf build-essential p7zip-full curl zlib1g-dev libc6-dev
sudo apt-get install lib32ncurses5-dev x11proto-core-dev libx11-dev
sudo apt-get install lib32z-dev libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown
sudo apt-get install libxml2-utils xsltproc gcc-multilib lib32readline5-dev
```
以下是编译Blink所需要的安装包，大致一样，如果编译android遇到什么问题再参考
git-core  
gnupg 个人数据安全 用GPG(简称)保护个人隐私数据
flex bison  编译相关
gperf hash 函数生成器 给定一堆字符串生成hash函数和hash表 以C 或者C++的形式
build-essential 实际上是一个meta package 提供编译程序必须的软件包列表信息 有了它才知道头文件在哪里，库函数在哪里
p7zip-full   包含 7z ，支持 7z、LZMA2、XZ、ZIP、CAB、GZIP、BZIP2、ARJ、TAR、CPIO、RPM、ISO 和 DEB 格式。
curl 利用URL语法在命令行下传输文件工具 支持多种协议http Ftp TELNET
zlib1g-dev 用到libz.so 需要用到这个包  用到libz.so才发现需要安装这个包。
libc6-dev 和glibc是同一个东西 glibc 是Linux 系统中最底层的api 其他所有库都要依赖此库
lib32ncurses5-dev
ia32-lib
x11proto-core-dev
libx11-dev
**lib32readline5-dev**
lib32z-dev
libgl1-mesa-dev
g++-multilib
gcc-multilib
mingw32
tofrodos
python-markdown 用python 实现的markdown
libxml2-utils
xsltproc C语言编写的扩展样式表语言XSLT引擎 通过XSL层叠样式表把XML转换成HTML或者PDF XHTML...

**加粗的包没有安装成功，原因：**

libreadline5-dev is no longer included in Ubuntu 12.04.
You may wish to try with libreadline-dev instead.

### 源代码下载链接：

  [百度网盘下载](http://pan.baidu.com/s/1cFPu6i)

### 解压Android4.4源代码

我用图形化界面直接解压了源代码7z包。
但是添加了创建的存取源代码文件夹的权限： chmod a+x ./Android


source ./build/envsetup.sh
make

Success
用时1分钟不到，让我觉得有点儿惶恐
![success](http://7xrw2w.com1.z0.glb.clouddn.com/AndroidSuccess.png)
赶紧再编译一次，肯定有问题。


### 错误
- 错误1
  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Androiderror1.png)
  重新make了一下
  遇到第一个错误 broken pipe 管道错误
  解决方式： sudo apt-get install bison
  这个包安装过，但是还是单独安装了一下

- 错误2
  /bin/bash: flex: command not found
  make: *** [out/host/linux-x86/obj/EXECUTABLES/aidl_intermediates/aidl_language_l.cpp] Error 127

  重新安装 flex
- 错误3
  /bin/bash: xmllint: command not found
  sudo apt-get  install libxml2-utils


### 编译了19:30 - 23:42
  成功截图:
  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_success.png)


### 启动模拟器
  模拟器位于：/Android/out/host/linux-x86/bin/中
  ![]()


  为了方便，可以把模拟器目录配置到环境变量中PATH 中
  ```
    $ source build/envsetup.sh
    $ lunch sdk-eng
    $ emulator
  ```
  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_%E6%A8%A1%E6%8B%9F%E5%99%A8.png)
  模拟器的启动需要四个文件，分别是
  - zImage Linux 内核镜像文件
  - system.img  Android 系统镜像文件
  - userdate.img Android 系统镜像文件
  - ramdisk.img Android 系统镜像文件

  直接运行emulator 默认使用的是zImage文件位于 out/host/linux-x86/bin 下

  ![]()


## 打包成SDK

>因为有些设备有特定的功能，需要SDK提供API，所以需要定制自己的SDK

```bash
  $ source build/envsetup.sh
  $ lunch sdk-eng
  $ make sdk
```


### 错误

- 1

```bash
  sdk/eclipse/scripts/create_all_symlinks.sh: line 285: cd: tools/base: No such file or directory
  make: *** [out/host/linux-x86/obj/EXECUTABLES/monitor_intermediates/monitor] Error 1
```

  sdk/eclipse/scripts/create_all_symlinks.sh: 中找不到tools/base。
  注释掉需要tools/base的地方
  create_all_symlinks:
  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_makesdkmodify.png)

  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_modify2.png)


  ./sdk/build/tools.atree：
  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_modify3.png)

- 2
```
Error: ## Unknown source 'testutils' to copy in 'sdk/eclipse/plugins/com.android.ide.eclipse.tests'
make: *** [out/host/linux-x86/obj/EXECUTABLES/monitor_intermediates/monitor] Error 1
```
  sdk/eclipse/scripts/create_all_symlinks.sh 未知source testutils
  注释掉 TEST
  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_modify4.png)

### sdk 编译完成
  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_sdk_success.png)
  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_sdk%E4%BD%8D%E7%BD%AE.png)
### 备份
  Android大小：
  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_Android%E5%A4%A7%E5%B0%8F.png)

  备份SDK

```bash
  coffee@ubuntu:~/Android/out/host/linux-x86/sdk/android-sdk_eng.coffee_linux-x86.zip
  coffee@ubuntu:~/Android/out/host/linux-x86/sdk/android-sdk_eng.coffee_linux-x86
```


>Android 源代码工程默认不包含它所使用的Linux内核源代码，如果需要运行定制的内核需要编译。


## 下载 编译 Android 内核源代码过程记录

### goldfish 下载

  - 首先需要到google官网上下载内核代码，下载链接
  - [android-goldfish-3.4官方下载](https://android.googlesource.com/kernel/goldfish/+/android-goldfish-3.4)
  - 我在win10下下载，并通过ubuntu共享文件夹放到ubuntu下
  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_goldfish.png)

  - 将文件解压后放入源代码根目录下新建的Kernel文件夹内，并进行编译前的准备：
  ```bash
    tar -zxvf android-goldfish-3.4.tar.gz
  ```
### 修改 Makefile

  - Android 模拟器所使用的CPU的体系结构是arm的，因此我们需要将Makefile文件中的 ARCH 变量设置成 arm  

  - 由于我们是在PC上的模拟器上编译内核，因此还需要Makefile文件中指定交叉编译工具，即修改CROSS_COMPILE变量的值

  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_Makefile.png)

  - Android源代码目录为我们准备了适用于编译Android模拟器内核文件的交叉编译工具，它位于Android源代码目录下的prebuilt/linux-x86 的tools中
  - CROSS_COMPILE 设置为为arm-eabi- 表示 所使用的交叉编译工具以arm-eabi作为前缀。

### 编译

```bash
  $ make goldfish_armv7_defconfig
```
  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_make%20goldfish_armv7_defconfig.png)

### 错误
  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_error1.png)

  - 解决方式：

  ```bash
    coffee@ubuntu:~/Android/kernel/goldfish-android-goldfish-3.4$ export PATH=$PATH:~/Android/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6/bin
  ```


### 再次make 编译结果成功：

  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_kernelsuccess.png)

  - 编译好的zImage 位于arch/arm/boot/目录下

### 启动模拟器

```bash
  $ source build/envsetup.sh
  $ lunch sdk-eng
  $ emulator -kernel ./kernel/goldfish-android-goldfish-3.4/arch/arm/boot/zImage &
```
  - 这个 & 代表后台启动，这样就可以用adb shell调试模拟器了。

```bash
  coffee@ubuntu:~/Android$ adb shell
  cd /proc
  root@generic:/proc # cat version
  Linux version 3.4.67 (coffee@ubuntu) (gcc version 4.6.x-google 20120106 (prerelease) (GCC) ) #1 PREEMPT Fri Apr 1 00:39:02 PDT 2016
```
  - 通过后面两行可以看出：
  - 内核镜像文件的版本为Linux version 3.4.67
  - 编译的主机名为：coffee@ubuntu
  - 编译工具为gcc version 4.6.x-google 20120106 (prerelease) (GCC)
  - 编译日期：#1 PREEMPT Fri Apr 1 00:39:02 PDT 2016


### 默认内核 和 自己编译的内核的区别

- 启动默认内核，查看内核信息
  emulator &
  adb shell:
```bash
  root@generic:/proc # cat version
  Linux version 3.4.0-gd853d22 (nnk@nnk.mtv.corp.google.com) (gcc version 4.6.x-google 20120106 (prerelease) (GCC) ) #1 PREEMPT Tue Jul 9 17:46:46 PDT 2013
```

- 启动自己编译的内核，查看内核信息

```bash
  coffee@ubuntu:~/Android$ adb shell
  cd /proc
  root@generic:/proc # cat version
  Linux version 3.4.67 (coffee@ubuntu) (gcc version 4.6.x-google 20120106 (prerelease) (GCC) ) #1 PREEMPT Fri Apr 1 00:39:02 PDT 2016
```



## 后记

  前前后后共耗时两天呆图书馆, 这只是为了深入学习做的第一步准备，可以通过应用程序来验证系统的实现原理或者执行逻辑。可以通过具体的应用程序实例来分析Android系统的源代码。
