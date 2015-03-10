title: OSLAB Notes3
date: 2015-03-09 21:15:38
tags:
- Linux Kernel Compiling
categories:
- LerningNotes
---

这学期的操作系统实验基本上算是上学期操作系统课的一个延续，实验的内容上面也是。。。多有重复。。。

第一个任务就是添加系统调用，编译内核。。。上一次的笔记[OSLAB Adding a system call to Linux kernel](/2014/11/03/OSLAB-Adding-a-system-call-to-Linux-kernel/).
以下是老师的要求

> 为Linux内核设计添加一个系统调用，将系统的相关信息（CPU型号、操作系统的版本号、系统中的进程等类似于Windows的任务管理器的信息）以文字形式列表显示于屏幕，并编写用户程序予以验证
> 对于proc文件系统的相关说明，读取proc文件系统的相关信息，可考虑相应的proc编程实验

关于proc，可以参考[wikipedia](http://en.wikipedia.org/wiki/Procfs)和`man proc`。简要来说就是内核通过一个虚拟的文件系统，向用户空间的程序提供的一个信息交换的渠道。比如说你可以用 `cat /proc/version`读出你的操作系统的相关信息，实际上各种工具如`uname, ps`所做的事情就是读取proc文件并进行解析。

按我揣测来看，老师的意思是让我们在内核态下使用proc来读出各种各样的信息。依我愚见，这是不能完成的，因为内核是proc的提供者，而非使用者，内核态下连文件系统的概念都还没有（尚为源码，还未实现），怎么去读取。而且就算有方法读取，但是你作为提供者，为什么还要费工夫再以使用者的身份调用自己的API，多此一举。所以我把基于系统调用的和proc的分开成两个做了。

我只实现了显示内核版本，数个进程名与PID的功能。关于内核版本的查询方式，可参照`/proc/version`使用`utsname()->release`，[源码](http://lxr.free-electrons.com/source/fs/proc/version.c)。
遍历所有进程可以采用如下代码（代码我是手敲的没编译，可能存在错误，下同）
```c
#include <linux/sched.h>
struct task_struct *task;
for_each_process(task)
{
    printk("%s [%d]\n", task->comm, task->pid);
}
```

### 动态模块加载
再具体实现的时候，可以先用Linux的动态模块来测试，这样就不需要说整整编译一次源码了,可以参考实验书。下面是一个最简单的例子。
```c
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>

static int plypy_init(void)
{
    printk("Loading Ply_py's module\n");
    return 0;
}

static void plypy_exit(void)
{
    printk("Dropping Ply_py's module\n");
}

module_init(plypy_init);
module_exit(plypy_exit);
MODULE_LICENSE("GPL");
```

基本上就是为自己的模块提供上init和exit函数，然后再用`module_init`,`module_exit`去注册即可，另外这里`MODULE_LICENSE`是一个声明许可证的宏，用GPL就行了。再添加一个Makefile，这是实验书上的。(Also, please have in mind that Makefile uses tab for indentation)
```
ifneq ($(KERNELRELEASE),)
    obj-m:=plypy_mod.o
else
    KERNELDIR:=/lib/modules/$(shell uname -r)/build
    PWD:=$(shell pwd)
modules:
    $(MAKE) -C $(KERNELDIR) M=$(PWD) modules
endif
```
基本上这个Makefile就是切换了一下目录，然后使用了当前正在运行的内核编译模块的Makefile。然后`# make`，（一般#前缀表示root用户，$表示普通用户）。接下来正常的话会生成一堆文件，其中有一个`plypy_mod.ko`，是我们用来加载的模块。
使用`# insmod plypy_mod.ko`来加载，`# rmmod plypy_mod.ko`来卸载。
同时借助`dmesg`可以观察到相应的信息。

然后可以先将，之前读取内核版本以及进程的逻辑置于我们模块的init函数中做一个测试。

### PROC_FS
由于proc提供的是一个虚拟的文件系统，所以我们需要将我们的信息包装成一个文件的形式，为其提供`open,read`等操作相对应的服务，参照`fs/proc/version.c`。
基本上就是为我们的虚拟文件提供了open服务，代码如下：
```c
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/sched.h>
#include <linux/utsname.h>
#include <linux/proc_fs.h>
#include <linux/seq_file.h>
#include <linux/fs.h>

struct proc_dir_entry *entry;

static int plypy_proc_show(struct seq_file *m, void *v)
{
    struct task_struct *task;
    int i = 0;
    seq_printf(m, "Kernel version: %s\n", utsname()->release);
    seq_printf(m, "Processes, to name a few:\n");
    for_each_process(task)
    {
        if (i++ > 9) break; // show only 10 processes at most
        seq_printf(m, "%s [%d]\n", task->comm, task->pid);
    }
    return 0;
}

static int plypy_proc_open(struct inode *inode, struct file *file)
{
    return single_open(file, plypy_proc_show, NULL);
}

static const struct file_operations plypy_proc_fops = {
    .open       = plypy_proc_open,
    .read       = seq_read,
    .llseek     = seq_lseek,
    .release    = single_release,
};

static int plypy_init(void)
{
    printk("Loading Ply_py's module\n");
    entry = proc_create("plypy", 0, NULL, &plypy_proc_fops);
    return 0;
}

static void plypy_exit(void)
{
    proc_remove(entry);
    printk("Dropping Ply_py's module\n");
}
module_init(plypy_init);
module_exit(plypy_exit);
MODULE_LICENSE("GPL");
```
通过insmod加载了后，可以通过`cat /proc/plypy`观察结果。

### SYSCALL
这个可以参照之前的那篇[OSLAB Adding a system call to Linux kernel](/2014/11/03/OSLAB-Adding-a-system-call-to-Linux-kernel/)，只用把具体的函数逻辑改一改就行，如下：
```
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/sched.h>
#include <linux/utsname.h>
#include <linux/kernel.h>

asmlinkage long sys_plypy_hello(void)
{
    struct task_struct *task;
        int i = 0;
        printk("Kernel version: %s\n", utsname()->release);
        printk("Processes, to name a few:\n");
        for_each_process(task)
        {
                if (i++ > 9) break; // show only 10 processes at most
                printk("%s [%d]\n", task->comm, task->pid);
        }
    return 0;
}
```

就这样。
