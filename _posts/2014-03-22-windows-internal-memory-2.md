---
layout: post
title: "Windows internal: Memory 2"
description: "virtual memory address convert to physical address"
category: 
tags: [windows]
---
{% include JB/setup %}

#   地址转译

##  x86虚拟地址转译
*   CR3寄存器，页目录物理基址
*   标准非PAE，两级页表： 页目录索引(10位)、页表索引(10位)、字节索引(12位)
    *   PDE： 每个PDE表有1K个表项，每项4字节
    *   PDE： 每个PTE表有1K个表项，每项4字节
*   PAE，三级页表:  页目录指针索引(2位), 页目录索引(9位)、页表索引(9位)、字节索引(12位)
    *   PDE： 每个PDE表有512个表项，每项8字节
    *   PDE： 每个PTE表有512个表项，每项8字节

###  在非PAE模式下的地址转译
*   !vtop PFN VirtualAddress， 最快捷的方式

	首先获取PFN， 有两种方式， 从!process中查看DirBase, 或用r cr3查看, PFN就是cr3中的值右移12位
        0: kd> r cr3
        cr3=00185000

        0: kd> !vtop 185 845ecf68 
        X86VtoP: Virt 845ecf68, pagedir 185000
        X86VtoP: PDE 185844 - 001c4063
        X86VtoP: PTE 1c47b0 - 045ec121
        X86VtoP: Mapped phys 45ecf68
        Virtual address 845ecf68 translates to physical address 45ecf68.
        
        // 比较虚拟地址内容及物理地址内容，完全一样，所以是正确映射的
        0: kd> !dd 0x45ecf68
        # 45ecf68 fefc45c7 6affffff 56006a01 046a59e8
        # 45ecf78 c0efe800 08c2ffe6 ec458b00 8be44589

        0: kd> dd 845ecf68 
        845ecf68  fefc45c7 6affffff 56006a01 046a59e8
        845ecf78  c0efe800 08c2ffe6 ec458b00 8be44589
        
*   !pte, 直接获得PTE内容，然后计算物理地址 

        0: kd> !pte 845ecf68 
                         VA 845ecf68
        PDE at C0300844         PTE at C02117B0
        contains 001C4063       contains 045EC121
        pfn 1c4   ---DA--KWEV   pfn 45ec  -G--A--KREV

        //The number 0x45ec is the page frame number (PFN) of this PTE
        physical address = 0x45ec*0x10000(4k, 页大小) + 0xf68(页偏移) = 0x45ecf68
        
*   manual convert

        0: kd> .formats 845ecf68 
        Evaluate expression:
          Binary:  10000100 01011110 11001111 01101000

        Page directory index = 0y1000010001 = 0x211
        Page table index = 0y0111101100 = 0x1ec
        Byte index = 0y111101101000 = 0xf68

        // 查看PTE内容, 两种方式
        1.  计算PTE的虚拟地址 
        PTE virsual address   =   
		PTE_BASE              (On a non-PAE system, this is 0xC0000000)
                + (page directory index) * PAGE_SIZE    (0x1000, 4K)
                + (page table index) * sizeof(MMPTE)    (This is 4 bytes on non-PAE x86 systems)
              =   0xc0000000
                + 0x211 * 0x1000
                + 0x1ec * 4
              =   0xC02117B0
        0: kd> dd C02117B0 l1
            c02117b0  045ec121
            pnf = 0x45e

        2.  计算PTE的物理地址
        0: kd> r cr3
        cr3=00185000

        PDE physical address = 0x185 * 0x1000 + 0x211 * 4 = 0x185844
            0: kd> !dd 0x185844 L1
            #  185844 001c4063  PDE content: pfn -> 0x1c4
        PTE physical address = 0x1c4 * 0x100 + 0x1ec * 4 = 0x1c47b0
            0: kd> !dd 0x1c47b0 L1
            #  1c47b0 045ec121  PTE content: pfn -> 0x45e
        
        // 计算物理地址
        physical address = pnf * 0x1000 + byte index = 0x45e000 + 0xf68 = 0x45ef68
        
###  在PAE模式下的地址转译
*   !vtop   失败，这个方法不好用
*   !pte

        1: kd> r cr4
            cr4=000406f9    bit 5:1  PAE开启
    
        1: kd> !pte 84e13a68 
                            VA 84e13a68
        PDE at C0602138            PTE at C0427098
        contains 00000000039C1863  contains 000000007D413963
        pfn 39c1      ---DA--KWEV  pfn 7d413     -G-DA--KWEV

        //The number 7d413 is the page frame number (PFN) of this PTE
        physical address = 0x7d413*0x1000(4k, 页大小) + 0xa68(页偏移) = 0x7d413f68

        1: kd> dd 84e13a68 
        84e13a68  00d00003 00000000 84ebc3f8 00000000
        1: kd> !dd 7d413a68
        #7d413a68 00d00003 00000000 84ebc3f8 00000000
        转译成功

*   manual convert

        1: kd> .formats 84e13a68 
            Binary:  10000100 11100001 00111010 01101000
        Page directory pointer index = 0y10 = 0x2
        Page directory index = 0y000100111 = 0x27
        Page table index = 0y000010011 = 0x13
        Byte index = 0y101001101000 = 0xa68

        计算PTE的虚拟地址 
        PTE virsual address   =   
		PTE_BASE 	(this is 0xC0000000)
                + (page directory pointer index) * (PAGE_SIZE * PAGE_ENTRY_NUM)     (0x1000, 0x200)(页大小为4k，一个页目录中页表项占8字节，共有512个页表项)
                + (page directory index) * PAGE_SIZE    (0x1000, 4K)
                + (page table index) * sizeof(MMPTE)    (This is 8 bytes on PAE x86 systems)
              =   0xc0000000
                + 0x2 * 0x1000 * 0x200
                + 0x211 * 0x1000
                + 0x13 * 8
              =   0xC027098

        1: kd> dd C0427098 l1
            c0427098  7d413963
            pnf = 0x7d413
        pyhsical address = pnf * 0x100 + Byte index = 0x7d413a68


##  x64虚拟地址转译
使用四级页表方案, 手动比较麻烦， !vtop不靠谱，最好用的还是!pte

    1: kd> !PTE fffff803`5b2be43c
                                           VA fffff8035b2be43c
    PXE at FFFFF6FB7DBEDF80    PPE at FFFFF6FB7DBF0068    PDE at FFFFF6FB7E00D6C8    PTE at FFFFF6FC01AD95F0
    contains 0000000000384063  contains 0000000000345063  contains 000000000034D063  contains 00000000020BE121
    pfn 384       ---DA--KWEV  pfn 345       ---DA--KWEV  pfn 34d       ---DA--KWEV  pfn 20be      -G--A--KREV

    只关心PTE的pfn即可， pnf = 0x20be
    pyhsical address = pnf * 0x1000 + 0x43c = 0x20be43c
    
    1: kd> !dd  0x20be43c
    # 20be43c f8b60f44 c89f8b48 0f000000 0b7477ba

    1: kd> dd fffff803`5b2be43c
    fffff803`5b2be43c  f8b60f44 c89f8b48 0f000000 0b7477ba
    可以看到虚拟地址内容与物理地址完全相同，是映射关系

====================================================================

以上看到，最好的地址转译方式是用!pte， 只关心PTE的pnf即可

#### 使用到的windbg命令

* 	!pte [virtual address]  查看PDE和PTE内容
*	d*用来查看虚拟地址下的内容，!d*用来查看物理地址下的内容
*	!vtop windbg提供的地址转译命令,但是好像只有在x86且non-PAE模式不才好用
*	.formats  以不同的进制输出数据
*	r [cr*]  显示特定寄存器中的值


