title: Android源代码基于goldfish-android-goldfish-3.4内核 硬件抽象层探究 HAL
date: 2016-04-07 19:06:33
tags:
categories:
- Android
---

## 目的

似乎作为软件开发人员来说，一旦涉及硬件，不管是从心里还是从实际操作上都总有一层厚厚的隔阂，但是要认识Android清晰层次化的认识Android系统，也得咬牙去接触，我终究在等待这么一天：其实它没有想象的那么难，它层次相当分明，有助于们整体上去认识Android系统，更便于我们对源代码的分析。

## 探究的位置：
<!-- more -->
>学Android 的同学都看过下面一个图：

![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_Android.jpg)

在上图中，Application层就是一个个应用程序。Framework提供一个java的运行环境以及对功能实现的封装，Runtime/ART是一个java虚拟机，通过它编译java。从Libraries那些名字也可以看出来，这里有很多高端大气库，它是功能实现区，多媒体编解码，浏览器渲染，数据库实现，and so on。Kernel部分负责交互硬件。


>再看看站在 HAL的角度 重视这个架构：

![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_HAL.png)

内核空间中可以有特权操作硬件设备，而用户空间不行，用户空间包含了上层的一些应用层的功能模块，以及面向用户的Application。这样的分层结构实际上在保护移动设备厂商。
Hardware Abstract Layer (HAl) 硬件抽象层运行在用户空间中。



---

## 开发Android硬件驱动程序

### 实现内核驱动模块

目录结构
   - ~/Android/kernel/goldfish-android-goldfish-3.4
      - drivers
        - freg
          - freg.h
          - freg.c
          - Kconfig
          - Makefile
关于命名 freg:
按照老罗的最高指示：我开发的是一个虚拟的字符硬件设备驱动程序，手里没有任何单片机或者寄存器，当然是开发虚拟的了...实体的玩意儿我也搞不了呀，这个字符硬件虚拟设备只有一个大小为4个字节的可读可写寄存器，叫做fake register,驱动程序命令为freg。

在其目录下创建上述四个文件：
```
  coffee@ubuntu:~/Android/kernel/goldfish-android-goldfish-3.4/drivers/freg$ ls
  freg.c  freg.h  Kconfig  Makefile
```

freg.h

```c++
  #ifndef _FAKE_REG_H_  
  #define _FAKE_REG_H_

  #include <linux/cdev.h>
  #include <linux/semaphore.h>

  //描述虚拟硬件设备在 文件系统中的名称
  #define FREG_DEVICE_NODE_NAME  "freg"
  #define FREG_DEVICE_FILE_NAME  "freg"
  #define FREG_DEVICE_PROC_NAME  "freg"
  #define FREG_DEVICE_CLASS_NAME "freg"

  //描述虚拟硬件设备
  struct fake_reg_dev {
  int val;                //虚拟寄存器
  struct semaphore sem;   //信号量 实现寄存器的同步访问
  struct cdev dev;        //标准 Linux 字符设备结构体变量， 标志该设备类型 是一个 字符设备
  };

  #endif

```


freg.c 为驱动程序的实现文件，该程序主要实现了 向用户空间提供了三个接口来访问该虚拟设备中的寄存器val
  - 接口1 proc文件系统接口
  - 接口2 设备文件系统接口
  - 接口3 devfs文件系统接口

freg.c



```
  #include <linux/init.h>
  #include <linux/module.h>
  #include <linux/types.h>
  #include <linux/fs.h>
  #include <linux/proc_fs.h>
  #include <linux/device.h>
  #include <asm/uaccess.h>

  #include "freg.h"

  //主设备号 和 从设备号
  static int freg_major = 0;
  static int freg_minor = 0;

  static struct class* freg_class = NULL;
  static struct fake_reg_dev* freg_dev = NULL;



  //传统的设备文件操作方法
  static int freg_open(struct inode* inode, struct file* filp);
  static int freg_release(struct inode* inode, struct file* filp);
  static ssize_t freg_read(struct file* filp, char __user *buf, size_t count, loff_t* f_pos);
  static ssize_t freg_write(struct file* filp, const char __user *buf, size_t count, loff_t* f_pos);


  // 传统的设备文件操作方法表
  static struct file_operations freg_fops = {
          .owner = THIS_MODULE,
          .open = freg_open,
          .release = freg_release,
          .read = freg_read,
          .write = freg_write,
  };

  // devfs 文件系统的设备属性操作方法
  static ssize_t freg_val_show(struct device* dev, struct device_attribute* attr,  char* buf);
  static ssize_t freg_val_store(struct device* dev, struct device_attribute* attr, const char* buf, size_t count);

  // devfs 文件系统的设备属性
  static DEVICE_ATTR(val, S_IRUGO | S_IWUSR, freg_val_show, freg_val_store);

  // 传统的设备文件操作方法实现： 打开 设备
  static int freg_open(struct inode* inode, struct file* filp) {
  	struct fake_reg_dev* dev;

  	dev = container_of(inode->i_cdev, struct fake_reg_dev, dev);
  	filp->private_data = dev;

  	return 0;
  }


  // 传统的设备文件操作方法实现： 释放
  static int freg_release(struct inode* inode, struct file* filp) {
  	return 0;
  }


  // 传统的设备文件操作方法实现： 读取 val寄存器的值
  static ssize_t freg_read(struct file* filp, char __user *buf, size_t count, loff_t* f_pos) {
  	ssize_t err = 0;
  	struct fake_reg_dev* dev = filp->private_data;

  	if(down_interruptible(&(dev->sem))) {
  		return -ERESTARTSYS;
  	}

  	if(count < sizeof(dev->val)) {
  		goto out;
  	}

  	if(copy_to_user(buf, &(dev->val), sizeof(dev->val))) {
  		err = -EFAULT;
  		goto out;
  	}

  	err = sizeof(dev->val);

  out:
  	up(&(dev->sem));
  	return err;
  }


  // 传统的设备文件操作方法实现： 写 val寄存器
  static ssize_t freg_write(struct file* filp, const char __user *buf, size_t count, loff_t* f_pos) {
  	struct fake_reg_dev* dev = filp->private_data;
  	ssize_t err = 0;

  	if(down_interruptible(&(dev->sem))) {
                  return -ERESTARTSYS;
          }

          if(count != sizeof(dev->val)) {
                  goto out;
          }

  	if(copy_from_user(&(dev->val), buf, count)) {
  		err = -EFAULT;
  		goto out;
  	}

  	err = sizeof(dev->val);

  out:
  	up(&(dev->sem));
  	return err;
  }


  // 传统的设备文件操作方法 - 内部使用： 读出 val -- buf缓冲器
  static ssize_t __freg_get_val(struct fake_reg_dev* dev, char* buf) {
  	int val = 0;

  	if(down_interruptible(&(dev->sem))) {
                  return -ERESTARTSYS;
          }

          val = dev->val;
          up(&(dev->sem));

          return snprintf(buf, PAGE_SIZE, "%d\n", val);
  }

  // 传统的设备文件操作方法 - 内部使用： 写入  buf缓冲器  -- val
  static ssize_t __freg_set_val(struct fake_reg_dev* dev, const char* buf, size_t count) {
  	int val = 0;

          val = simple_strtol(buf, NULL, 10);

          if(down_interruptible(&(dev->sem))) {
                  return -ERESTARTSYS;
          }

          dev->val = val;
          up(&(dev->sem));

  	return count;
  }


  // devfs 文件系统的设备属性操作方法实现  读出 设备属性val值
  static ssize_t freg_val_show(struct device* dev, struct device_attribute* attr, char* buf) {
  	struct fake_reg_dev* hdev = (struct fake_reg_dev*)dev_get_drvdata(dev);

          return __freg_get_val(hdev, buf);
  }
  // devfs 文件系统的设备属性操作方法实现  写入 设备属性val值
  static ssize_t freg_val_store(struct device* dev, struct device_attribute* attr, const char* buf, size_t count) {
  	 struct fake_reg_dev* hdev = (struct fake_reg_dev*)dev_get_drvdata(dev);

          return __freg_set_val(hdev, buf, count);
  }

  // devfs 文件系统的设备属性操作方法实现 --内部使用  读出 寄存器 val值 -- 缓冲区
  static ssize_t freg_proc_read(char* page, char** start, off_t off, int count, int* eof, void* data) {
  	if(off > 0) {
  		*eof = 1;
  		return 0;
  	}

  	return __freg_get_val(freg_dev, page);
  }

  // devfs 文件系统的设备属性操作方法实现  --内部使用 写入 缓冲区 -- 寄存器 val值
  static ssize_t freg_proc_write(struct file* filp, const char __user *buff, unsigned long len, void* data) {
  	int err = 0;
  	char* page = NULL;

  	if(len > PAGE_SIZE) {
  		printk(KERN_ALERT"The buff is too large: %lu.\n", len);
  		return -EFAULT;
  	}

  	page = (char*)__get_free_page(GFP_KERNEL);
  	if(!page) {
                  printk(KERN_ALERT"Failed to alloc page.\n");
  		return -ENOMEM;
  	}

  	if(copy_from_user(page, buff, len)) {
  		printk(KERN_ALERT"Failed to copy buff from user.\n");
                  err = -EFAULT;
  		goto out;
  	}

  	err = __freg_set_val(freg_dev, page, len);

  out:
  	free_page((unsigned long)page);
  	return err;
  }


  //创建 /proc/freg文件
  static void freg_create_proc(void) {
  	struct proc_dir_entry* entry;

  	entry = create_proc_entry(FREG_DEVICE_PROC_NAME, 0, NULL);
  	if(entry) {
  //		entry->owner = THIS_MODULE;  // error in here
  		entry->read_proc = freg_proc_read;
  		entry->write_proc = freg_proc_write;
  	}
  }
  //删除 /proc/freg文件
  static void freg_remove_proc(void) {
  	remove_proc_entry(FREG_DEVICE_PROC_NAME, NULL);
  }

  //初始化设备
  static int  __freg_setup_dev(struct fake_reg_dev* dev) {
  	int err;
  	dev_t devno = MKDEV(freg_major, freg_minor);

  	memset(dev, 0, sizeof(struct fake_reg_dev));

  	cdev_init(&(dev->dev), &freg_fops);
  	dev->dev.owner = THIS_MODULE;
  	dev->dev.ops = &freg_fops;

  	err = cdev_add(&(dev->dev),devno, 1);
  	if(err) {
  		return err;
  	}

      sema_init(&(dev->sem),1);
  	dev->val = 0;

  	return 0;
  }

  //模块加载方法
  static int __init freg_init(void) {
  	int err = -1;
  	dev_t dev = 0;
  	struct device* temp = NULL;

  	printk(KERN_ALERT"Initializing freg device.\n");

  	err = alloc_chrdev_region(&dev, 0, 1, FREG_DEVICE_NODE_NAME);
  	if(err < 0) {
  		printk(KERN_ALERT"Failed to alloc char dev region.\n");
  		goto fail;
  	}

  	freg_major = MAJOR(dev);
  	freg_minor = MINOR(dev);

  	freg_dev = kmalloc(sizeof(struct fake_reg_dev), GFP_KERNEL);
  	if(!freg_dev) {
  		err = -ENOMEM;
  		printk(KERN_ALERT"Failed to alloc freg device.\n");
  		goto unregister;
  	}

  	err = __freg_setup_dev(freg_dev);
  	if(err) {
  		printk(KERN_ALERT"Failed to setup freg device: %d.\n", err);
  		goto cleanup;
  	}

  	freg_class = class_create(THIS_MODULE, FREG_DEVICE_CLASS_NAME);
  	if(IS_ERR(freg_class)) {
  		err = PTR_ERR(freg_class);
  		printk(KERN_ALERT"Failed to create freg device class.\n");
  		goto destroy_cdev;
  	}

  	temp = device_create(freg_class, NULL, dev, "%s", FREG_DEVICE_FILE_NAME);
  	if(IS_ERR(temp)) {
  		err = PTR_ERR(temp);
  		printk(KERN_ALERT"Failed to create freg device.\n");
  		goto destroy_class;
  	}

  	err = device_create_file(temp, &dev_attr_val);
  	if(err < 0) {
  		printk(KERN_ALERT"Failed to create attribute val of freg device.\n");
                  goto destroy_device;
  	}

  	dev_set_drvdata(temp, freg_dev);

  	freg_create_proc();

  	printk(KERN_ALERT"Succedded to initialize freg device.\n");

  	return 0;

  destroy_device:
  	device_destroy(freg_class, dev);
  destroy_class:
  	class_destroy(freg_class);
  destroy_cdev:
  	cdev_del(&(freg_dev->dev));
  cleanup:
  	kfree(freg_dev);
  unregister:
  	unregister_chrdev_region(MKDEV(freg_major, freg_minor), 1);
  fail:
  	return err;
  }


  //模块卸载方法
  static void __exit freg_exit(void) {
  	dev_t devno = MKDEV(freg_major, freg_minor);

  	printk(KERN_ALERT"Destroy freg device.\n");

  	freg_remove_proc();

  	if(freg_class) {
  		device_destroy(freg_class, MKDEV(freg_major, freg_minor));
  		class_destroy(freg_class);
  	}

  	if(freg_dev) {
  		cdev_del(&(freg_dev->dev));
  		kfree(freg_dev);
  	}

  	unregister_chrdev_region(devno, 1);
  }

  MODULE_LICENSE("GPL");
  MODULE_DESCRIPTION("Fake Register Driver");

  module_init(freg_init);
  module_exit(freg_exit);


```

Kconfig

在编译freg之前，我们需要通过 make menuconfig 命令来设置这些选项，指定驱动程序freg的编译方式。
下面的配置文件有三种方式来编译：
  - 建立在内核中
  - 编译成内核模块
  - 不编译到内核中


```bash
  config FREG
  tristate "Fake Register Driver"
  default n
  help
  This is the freg driver for android system.

```

Makefile

```bash
  obj-$(CONFIG_FREG) += freg.o
```


### 修改Kconfig文件

我们需要修改内核的根Kconfig文件，让直行make menuconfig 的时候，让编译系统找到驱动程序freg的Kconfig文件
一般来说，各个CPU体系架构目录下的Kconifg文件都会通过source "drivers/Kconfig" 命令将drivers 目录下的Kconfig文件包含进去。

打开drivers/Kcofig文件 添加 source "drivers/freg/Kconfig"

![Kconfig](http://7xrw2w.com1.z0.glb.clouddn.com/Android_Kconfig_freg.png)


### 修改内核Makefile文件

不仅是Kconfig需要让编译器找到，新增的驱动的Makefile也要让编译器找到，所以需要修改内核的Makefile文件。
drivers/Makefile下添加
```bash
  obj-$(CONFIG_FREG) += freg/
```

### 编译内核驱动程序模块

设置编译选项 make menuconfig

  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_MakeMenuconfig.png)

  ![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_%E7%BC%96%E8%AF%91%E9%80%89%E9%A1%B9.png)
执行 make menuconfig 命令来配置编译方式：
Fake Register Driver 选项
  - ❤ 号 表示建立在内核中
  - M 表示 编译成内核模块
exit后 执行 make
error1
```bash
  In function 'freg_create_proc':
  error: 'struct proc_dir_entry' has no member named 'owner'
```
原因：

由错误信息可以看出struct proc_dir_entry结构体中没有找到owner的成员。
看到引用的proc_fs.h头文件，发现里面的struct proc_dir_entry结构体中，是有owner成员的，Linux API 里说在2.6版本，struct proc_dir_entry:owner就已经被移除了
我也尝试注释掉owner使用的代码，跟踪到 freg.c 的create文件：
![](http://7xrw2w.com1.z0.glb.clouddn.com/Android_freg_error.png)
error2 :
```bash
  implicit declaration of function 'init_MUTEX' [-Werror=implicit-function-declaration]
```

原因：
在新版本的linux内核中，init_mutex已经被废除了，新版本使用sema_init函数。查了一下早期版本的定义：
新内核中的定义：
```
    /*
     * linux/include/asm-arm/semaphore.h
     */
    #ifndef __ASM_ARM_SEMAPHORE_H
    #define __ASM_ARM_SEMAPHORE_H

    #include <linux/linkage.h>
    #include <linux/spinlock.h>
    #include <linux/wait.h>
    #include <linux/rwsem.h>

    #include <asm/atomic.h>
    #include <asm/locks.h>

    struct semaphore {
        atomic_t count;
        int sleepers;
        wait_queue_head_t wait;
    };

    #define __SEMAPHORE_INIT(name, cnt)             
    {                               
        .count  = ATOMIC_INIT(cnt),             
        .wait   = __WAIT_QUEUE_HEAD_INITIALIZER((name).wait),   
    }

    #define __DECLARE_SEMAPHORE_GENERIC(name,count)
        struct semaphore name = __SEMAPHORE_INIT(name,count)

    #define DECLARE_MUTEX(name)     __DECLARE_SEMAPHORE_GENERIC(name,1)
    #define DECLARE_MUTEX_LOCKED(name)  __DECLARE_SEMAPHORE_GENERIC(name,0)
    static inline void sema_init(struct semaphore *sem, int val)
    {
        atomic_set(&sem->count, val);
        sem->sleepers = 0;
        init_waitqueue_head(&sem->wait);
    }

    static inline void init_MUTEX(struct semaphore *sem)
    {
        sema_init(sem, 1);
    }
    .....
    ....

```

老内核中的定义
```
    static inline void sema_init(struct semaphore *sem, int val)
     {
            static struct lock_class_key __key;
            *sem = (struct semaphore) __SEMAPHORE_INITIALIZER(*sem, val);
          lockdep_init_map(&sem->lock.dep_map, "semaphore->lock", &__key, 0);
     }

    #define init_MUTEX(sem)         sema_init(sem, 1)//在最新的版本中没有这两个宏定义了
    #define init_MUTEX_LOCKED(sem)  sema_init(sem, 0)

    extern void down(struct semaphore *sem);
    extern int __must_check down_interruptible(struct semaphore *sem);
    extern int __must_check down_killable(struct semaphore *sem);
    extern int __must_check down_trylock(struct semaphore *sem);
    extern int __must_check down_timeout(struct semaphore *sem, long jiffies);
    extern void up(struct semaphore *sem);

```

解决办法：绕过函数 init_MUTEX ,在freg.c中直接调用：sema_init(),不过要添加一个整形参数1
```
      semaphore.h：

      static inline void init_MUTEX(struct semaphore *sem)
      {
        sema_init(sem, 1);
      }
```

freg.c 修改 init_MUTEX()调用
```
    //初始化设备
    static int  __freg_setup_dev(struct fake_reg_dev* dev) {
        int err;
        dev_t devno = MKDEV(freg_major, freg_minor);
        memset(dev, 0, sizeof(struct fake_reg_dev));
        cdev_init(&(dev->dev), &freg_fops);
        dev->dev.owner = THIS_MODULE;
        dev->dev.ops = &freg_fops;

        err = cdev_add(&(dev->dev),devno, 1);
        if(err) {
            return err;
        }   

        sema_init(&(dev->sem),1); //modify init_MUTEX()
        dev->val = 0;

        return 0;
    }
```
编译成功：

![成功截图](http://7xrw2w.com1.z0.glb.clouddn.com/Android_freg_success.png)

内核镜像文件zImage 保存在arch/arm/boot目录下，可以用它来启动Android模拟器

### 验证内核驱动程序模块

>分别验证 proc文件系统接口, devfs 文件系统， dev文件系统来访问虚拟寄存器

首先打开Android模拟器：

```
  emulator -kernel /kerner/goldfish-android-goldfish-3.4/arch/arm/boot/zImage &
  adb shell
```

#### proc文件系统接口访问虚拟寄存器验证

```
  cd dev //验证驱动程序freg是否成功注册虚拟硬件设备到设备文件系统中
  ls freg
  freg //有显示说明注册成功

  root@generic://proc # cat freg                                                 
  0
  root@generic://proc # echo '5' > freg  

  root@generic://proc # cat freg                                                 
  5
  //成功显示
```

#### devfs 文件系统接口访问虚拟寄存器验证

```
  root@generic://sys/class/freg/freg # cat val                                   
  5
  root@generic://sys/class/freg/freg # echo '0' > val                            
  root@generic://sys/class/freg/freg # cat val                                   
  0
  //成功显示
```


#### 开发C程序来验证dev文件系统访问虚拟寄存器的正确性


通过c程序来对寄存器进行读写操作：
编写源文件：
 - Android/external
    - freg  
      - freg.c  
      - Android.mk

freg.c:

```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>

    #define FREG_DEVICE_NAME "/dev/freg"

    int main(int argc, char** argv)
    {
     int fd = -1;
     int val = 0;

     fd = open(FREG_DEVICE_NAME, O_RDWR);
     if(fd == -1)
     {   
         printf("Failed to open device %s.\n", FREG_DEVICE_NAME);
         return -1;
     }   

     printf("Read original value:\n");
     read(fd, &val, sizeof(val));
     printf("%d.\n\n", val);

     val = 5;
     printf("Write value %d to %s.\n\n", val, FREG_DEVICE_NAME);
         write(fd, &val, sizeof(val));


     printf("Read the value again:\n");
         read(fd, &val, sizeof(val));
         printf("%d.\n\n", val);

     close(fd);

     return 0;
    }

 ```

 freg.c 源文件的编译脚本文件:Android.mk
 ```bash
 LOCAL_PATH := $(call my-dir)
 include $(CLEAR_VARS)
 LOCAL_MODULE_TAGS := optional
 LOCAL_MODULE := freg
 LOCAL_SRC_FILES := $(call all-subdir-c-files)
 include $(BUILD_EXECUTABLE)
 ```

对于C程序来说，include后面的参数为BUILD_EXECUTABLE 可执行应用模块程序。编译结果在：
out/target/product/generic/system/bin中
编译：
 ```bash
  source ./build/envsetup.sh
  mmm ./external/freg/
  make snod
```
进入Android系统执行
```bash
  emulator -kerner /Kernel/goldfish-android-goldfish-3.4/arch/arm/boot/zImage &
  adb shell
  cd system/bin/
  ./freg
```
当我们看到了输出为5 ，说明C程序可以执行，虚拟寄存器正常使用。
---



## 开发Android硬件抽象层模块

## 开发Android硬件访问服务

## 开发Android应用程序来使用硬件访问服务




> 声明： 相关内容及源代码参考老罗的《Android系统源代码情景分析》感谢老罗！
