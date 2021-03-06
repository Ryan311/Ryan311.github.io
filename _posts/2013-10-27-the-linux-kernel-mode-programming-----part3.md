---
layout: post
title: "The Linux Kernel Mode Programming --- Part3"
tagline: "The Linux Kernel Mode Programming     Part3"
description: ""
tags: [Linux]
---
{% include JB/setup %}

##  Modules *VS* Programs

###  Begin and End
应用程序开始于一个main()函数，执行一些指令后就结束。
模块的工作方式略有不同。一个模块开始于init_module()或是由宏module_init()指定的初始化函数。这个函数是模块的入口函数，它告诉内核该模块提供了哪些功能并使内核在需要的时候调用模块的功能函数。该入口函数执行完成时就退出，内核只有在需要的时候才会调用模块提供的代码。
所有的模块在从内核卸载时调用cleanup_module或宏module_exit()指定的退出函数。它会反注册掉入口函数提供到内核的功能函数

### Functions available to modules
模块不能调用标准I/O库，只能调用由内核提供的函数，内核的导出函数可以查看/proc/kallsyms

系统调用运行在内核空间，由内核提供。标准库函数运行在用户空间，提供比系统调用更为方便的接口, 当完成一项具体功能时，标准库中的函数会调用相对应的系统调用。
可以用*strace*来查看程序中调用了哪些系统调用。

### User Space VS Kernel Space
在Windows中，称为User Mode和Kernel Mode, 意义相同。
CPU能够运行在不同的模式中，在Intel IA32体系结构中有４种模式，ring 0、１、２、３。Unix和Windows都使用了其中的两个ring，ring0(supervisor mode)和ring4(the lowest ring)

### Name Space
当写内核代码时，即使最小的模块也会链接到整个内核。如果一个模块导出的变量与另一个模块导出的变量相同，就会引起命令冲突。这种问题的解决方法是将你的变量都定义成静态的并且所有的符号都使用良好定义的前缀。内核使用的前缀都是小写的。如果你不想将所有符号都定义为静态的，另一种选择是声明一个符号表，并在内核中注册它。

### Code Space
应用程序使用虚拟地址空间，由内存管理器分配并管理,将虚拟地址映射到实际的物理地址。任何两个程序使用的地址空间是相互独立的。

内核也有自己的内存地址空间。因为模块可以动态的加载或卸载到内核中，它拥有与内核相同的地址空间，而不是有独享的空间。所以如果你的模块发生段错误，整个内核就会发生段错误，这是非常危险的。

### Device Drivers
设备驱动程序是一种类型的模块，向内核提供访问硬件的功能。在Unix系统中，每个硬件设备由/dev中的一个文件表示。

每个硬件设备由一个主设备号和次设备号来表示。主设备号表示哪种驱动可以用于访问该硬件，每个Driver都被赋于一个唯一的主设备号。拥有相同主设备号的设备文件（设备）由相同的驱动程序来管理。
设备驱动程序通过次设备号来区分由它控制的不同的设备。
设备可以分成三类: 字符设备、块设备、网络设备
当系统安装时，设备文件由*mknod*命令来创建，创建新的字符设备文件
    
    mknod /dev/coffee c 12 2

新的字符设备文件的主／次设备号为12, 2。

内核只关心设备的主设备号，它告诉内核要使用哪种设备驱动程序来操作该设备。内核不会使用次设备号。驱动程序中才会用到它，用于定位具体的设备。
