---
layout: post
title: "The Linux Kernel Mode Programming---Part1"
tagline: "The Linux Kernel Mode Programming     Part1"
description: ""
tags: [Linux]
---
{% include JB/setup %}

##  什么是Kernel module？
内核模块是能够在需要的时候动态的加载或卸载的代码块，他们扩展了内核的功能而不需要重启系统。例如， 驱动程序就是一种类型的内核模块，允许内核访问连接到系统的硬件设备。如果没有内核，我们就需要重新编译内核并把新的功能加到内核映像中去，这会使得内核越来越大，且要求每次添加新功能时都要重新编译并重新启动系统。


##  模块是怎样加载到内核的？
查看已经加载内核模块

    $ lsmod

例： 需要加载模块msdos.ko到内核，而msdos.ko信赖于fat.ko

### 手动加载内核模块, 使用命令insmod

    $ sudo insmod /lib/modules/2.6.11/kernel/fs/fat/fat.ko
    $ sudo insmod /lib/modules/2.6.11/kernel/fs/msdos/msdos.ko


### 智能方式加载内核模块， modprobe

    $ modprobe msdos

可见，modprobe只需提供内核的名称就可以加载内核，而insmod要提供路径名及模块依赖关系。modprobe是如何工作的呢？
*   首先， 如果内核需要有一个模块的功能但该模块还没有加载到内核中，则内核模块守护进程（服务）*kmod*调用**modprobe**来加载这个模块, 并提供该模块的名字msdos
*   然后**modprobe**去查看_/etc/modprobe.conf_看是否该模块是个别名。下一步，**modprobe**查看_/lib/modules/version/modules.dep_, 查看模块的依赖，该文件是由*depmod -a*创建的包含了模块的依赖关系。
*   最后，**modprobe**使用**insmod**先加载fat.ko，并提供完整路径名，然后再加载dos.ko

### The Simplest Module “Hello， World”

    /* 
    *   hello-1.c - The simplest kernel module
    */
    #include <linux/module.h>
    #include <linux/kernel.h>

    int init_module(void)
    {
        printk(KERN_INFO "Hello world 1\n");
        return 0;   //non 0 means failed, module can't be loaded
    }

    void cleanup_module(void)
    {
        printk(KERN_INFO "Goodby world 1.\n");
    }

内核模块至少有两个函数
+   start func:     init_module(), 当模块被加载到内核时被调用
+   end func:       cleanup_module(), 当模块从内核中卸载时被调用 
+   从内核2.3.13之后，这两个名字是可以改变的，可以指定任意的名字

每个模块都要包含*linux/module.h*, 当要用到printk()时要包含*linux/kernel.h*

### printk
**printk**是内核的log机制，目的是记录信息或是给出一些警告。每个printk()都要指明优先级，共有8个优先级每个都有macro对应，默认的是DEFAULT_MESSAGE_LOGLEVEL。
当*syslogd*和*klogd*在运行时，log信息被保存在_/var/log/messages_中。

### 编译内核模块
Makefile
    
    obj-m += hello-1.o
    all:
        make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
    clean:
        make -C /lib/modules/$(shell uanme -r)/build M=$(PWD) clean

then run make in terminal
    
    $ make
    make -C /lib/modules/3.8.0-29-generic/build M=/home/persnail/workspace/git/repos/LKMPG/HelloWorld1 modules
    make[1]: Entering directory `/usr/src/linux-headers-3.8.0-29-generic'
      CC [M]  /home/persnail/workspace/git/repos/LKMPG/HelloWorld1/hello-1.o
      Building modules, stage 2.
      MODPOST 1 modules
      CC      /home/persnail/workspace/git/repos/LKMPG/HelloWorld1/hello-1.mod.o
      LD [M]  /home/persnail/workspace/git/repos/LKMPG/HelloWorld1/hello-1.ko
    make[1]: Leaving directory `/usr/src/linux-headers-3.8.0-29-generic'

**hello-1.ko**就是编译完成的内核模块，查看它的信息
    
    $ modinfo hello-1.ko
    filename:       hello-1.ko
    srcversion:     140276773A3090F6F33891F
    depends:        
    vermagic:       3.8.0-29-generic SMP mod_unload modversions

加载内核模块

    $ sudo insmod ./hello-1.ko

查看是否已经加载
    
    $ cat /proc/modules | grep hello
    hello_1 12426 0 - Live 0x0000000000000000 (POF)

卸载内核模块

    $ sudo rmmod hello-1.ko

查看log信息

    $ less /var/log/syslog 
    Oct 25 00:58:48 persnail-Vostro-2420 kernel: [ 7423.160253] Hello world 1.
    Oct 25 01:00:41 persnail-Vostro-2420 kernel: [ 7536.613425] Goodbye world 1.
