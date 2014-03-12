---
layout: post
title: "The Linux Kernel Mode Programming --- Part2"
tagline: "The Linux Kernel Mode Programming     Part2"
description: ""
tags: [Linux]
---
{% include JB/setup %}

##  宏

在Linux2.4之后，引入了如下两个宏，在模块中就没有必要使用init_module()和cleanup_module()，而是可以自定义实现这两个功能的函数, part1中的模块可以改写为

    #include <linux/module.h>   //Needed for all modules
    #include <linux/kernel.h>   //Needed for KERN_INFO
    #include <linux/init.h>     //Needed for the macros

    static int __init hello_2_init(void)
    {
        printk(KERN_INFO "Hello, world 2\n");
        return 0;
    }

    static void __exit hello_2_exit(void)
    {
        printk(KERN_INFO "Goodbye, world 2\n");
    }

    module_init(hello_2_init);
    module_exit(hello_2_exit);

*    __init表示模块加载到内核时，首先执行hello_2_init()函数，执行完后就可以释放掉该函数所占的内存区，它在之后不会再用到。
*    __exit表示模块加载到内核后，只有当该模块从内核中卸载掉时才执行函数hello_2_exit()
*   module_init()   声明初始化函数init function
*   module_exit()   声明清理函数cleanup function
*   MODULE_LICENSE("GPL")    声明代码遵循的GPL协议
*   MODULE_AUTHOR()         该模块是谁写的
*   MODULE_DESCRIPTION()    该模块的简单功能描述
*   more in linux/init.h

##  向模块传递参数
模块在加载的时候可以带有命令行参数，但和传统的程序中的argc/argv不同。

为了能向模块传递参数，模块要导出变量变量的名字以便加载模块的程序(insmod)可以看到, 要用到宏module_param()（defined in linux/moduleparam.h)

    /* 
     * module_param(var, int, 0000)
     *  The first param is the parameters name
     *  The second param is it's data type
     *  The final argument is the permissions bits, 
     *  for exposing parameters in sysfs (if non-zero) at a later stage.
     */
    int myint = 3;
    module_param(myint, int 0);

    /*
     * module_param_array(arr, type, num, perm);
     *  The first param is the parameter's (in this case the array's) name
     *  The second param is the data type of the elements of the array
     *  The third argument is a pointer to the variable that will store the number 
     *  of elements of the array initialized by the user at module loading time
     *  The fourth argument is the permission bits
     */
    int myinitarray[2];
    module_param_array(myintarray, int , NULL, 0);

在加载模块时就可以使用如下命令

    $ sudo insmod mymodule.ko myint=5





