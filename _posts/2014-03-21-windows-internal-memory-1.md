---
layout: post
title: "Windows internal: Memory 1"
description: "windows memory management"
category: 
tags: [windows]
---
{% include JB/setup %}

## Keywords

*   PTE: Page Table Entry  系统页表项
    利用系统页表项，可以动态地映射系统页面。每个虚拟地址都跟一个系统空间中的结构联系在一起，
该结构被称为PTE，其中包含了该虚拟地址所对应的物理地址。

*   PDE: Page directory entries 页目录项
    每个PDE是4字节长，页目录描述了该进程所有可能的页表的状态和位置

*   PAE: Physical Address Extension  物理地址扩展
    Intel x86 Pentium Pro处理器引入的一种内在映射模式，通过适当的芯片集，PAE模式可以使得当前的
Intel x86处理器上可以访问多达64GB的物理内存，x64处理器上可以访问多达1024GB的物理内存。

*   PFN: Page frame number 页面帧编号


##   Windows内存管理器

它是执行体的一部分，位于ntoskrnl.exe中

###  主要任务

*   将一个进程的虚拟地址空间转译或者映射到物理内存中
*   当内存过度提交时，将内存中的有些内容转移到磁盘上；并且在以后需要这些内容时，再将它们读回到物理内存中

### 核心服务

*   内存映射文件
*   写时复制
*   为应用程序使用大的、稀疏的地址空间提供支持
*   PAE

###   共享内存与写时复制

*   例如两个进程使用了同样的DLL， 只需要将引用该dll的代码页面加载到物理内存中一次，然后所有映射了些dll的进程之间共享这些页面
*   每个进程仍然维护它的私有内存区域，在其中存储私有数据，但是程序指令和非修改的数据页面可以被共享。其中可执行映像中代码页面是以“仅可执行”的方式映射，可写的页面被映射为“写时复制”

###   写时复制

*   定时复制页面保护机制是一种优化，内存管理器利用它可以节约内存
*   写时复制是“延时计算”，使得只有当绝对需要的时候才执行一个昂贵的操作，如果该操作不需要的话，就不会浪费时间
*   Unix系统中的fork函数就是很好的例子
*   应用： 两进程共享dll， 调试器中实现断点

###   页面(默认4K, 0x1000)

*   虚拟地址空间被分成以页面为单位
*   页面是硬件级别上的最小保护单位, 也是内存管理器操作的最小单位, 更小单位的分配操作要用堆管理器

###   堆管理器， heap manager

*   负责管理大内存区域中的内存分配，这些大的内存区域是通过一些页面粒度的内存分配函数来保留的。堆管理器的分配粒度相对较小：32位系统上为8字节，64位为16字节
*   Windows API有专门的堆函数Heap*， 如HeapCreate,它从内存管理器中得到一块大的内存区,用于堆分配.
*   每个进程至少有一个堆（默认的进程堆，可以通过GetProcessHeap来获得）， 也可以调用HeapCreate显示的创建额外的私有堆
*   堆管理器是一个两层结构：可选的前端堆层和核心堆层
*   核心堆层：处理基本功能，包括段内部的内存块的管理、段的管理、堆的扩展策略、提交内存和解除提交，以及大块的管理
*   可选的前端堆有两种类型：预读列表(look-aside list)和低碎片堆(LFH, Low Fragmentation Heap)
    *   预读列表
    *   LFH
*   堆调试工具： pageheap


##   系统堆, 也称为系统内存池

*   进程中的内存分配主要由堆管理器来分配,但内核组件的分配有单独的内存池
*   系统初始化时，内存管理器创建了两种类型的动态大小的内存池，内核模块组件从这两种内存池中分配系统内存
    *   non-paged 非换页池, paged 换页池
    *   API： ExAllocatePoolWithTagExAlloc,  ExFreePoolWithTag
*   Windows也提供了一种快速的内存分配机制为内核模块使用：Look-Aside List
    *   一般内存池分配操作可以是不同大小的，而预读列表只包含固定大小的内存块
    *   Look-Aside List分配速度更快，而一般内存池分配更灵活
    *   API： ExInitializeNPagedLookasideList， ExInitializePagedLookasideList
    *   查看系统的预读列表: !lookaside

###  用于扩展进程的内存使用空间的方法

*   windows32中每个进程默认最大能使用2GB虚拟地址空间(可能设为3GB)
*   AWE(address windowing extensions), 由windows提供的一组API实现
*   PAE(physical address extension)

###   Windows上查看内存信息的工具

*   任务管理器中的Performance
*   Pmon.exe
*   Pstate.exe
*   Poolmon
*   内核调试器中!vm, !memusage, !poolused
