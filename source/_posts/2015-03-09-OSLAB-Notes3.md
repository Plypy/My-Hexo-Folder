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

~~按我揣测来看，老师的意思是让我们在内核态下使用proc来读出各种各样的信息。依我愚见，这是不能完成的，因为内核是proc的提供者，而非使用者，内核态下连文件系统的概念都还没有（尚为源码，还未实现），怎么去读取。而且就算有方法读取，但是你作为提供者，为什么还要费工夫再以使用者的身份调用自己的API，多此一举。所以我把基于系统调用的和proc的分开成两个做了。~~

### UPDATE
---
其实肯定是有方法可以做的，毕竟一切皆可实现，只不过是漂亮不漂亮，符不符合正常逻辑的问题。我跟指导老师谈了一下这个，老师告诉我编译的时候是没有文件系统，可运行的时候就有了，然后读文件的方法用vfs就可以。详见[vfs_read()](http://lxr.free-electrons.com/source/fs/read_write.c#L432)，当然还存在一些其他的函数，更底层并且更不安全。总的来说吧，合理的逻辑就是，内核提供文件以及proc等，然后各种用户态的程序再去访问它们。虽然我们可以皆由hack调用其他的函数去读取文件，但这本质上是脏的，是不符合设计哲学的，不过毕竟是实验，听老师的……有兴趣的同学可以去hack一下，我还是保留我这个方案。

另外老师要求要直接输出到屏幕上，以下的代码调用的`printk`生成的输出都需要通过`dmesg`去访问，老师说不符合要求……
printk其实是有记录级别的，就是常见的那种log level，这些level的宏定义在`linux/kernel.h`中。warning, error, info啊之类的，可以看一下百度百科，另外关于printk的教程在[此](http://www.xml.com/ldd/chapter/book/ch04.html)。

这些记录级别其实就是一个数字，越小的越严重，在Linux运行的时候他的console有一个console log level可以通过`cat /proc/sys/kernel/printk`来查看，第一个数字既是。一般来说默认的应该是4（warning），那么只有小于4的可以被输出到console中。

可以通过`printk(KERN_DEBUG "str")`这样来明确具体输出的级别，当不声明级别的时候一般默认为KERN_WARNING(4)。为了保证输出到console，可以采用最高级别KERN_EMERG。

但是如果已经编译了内核了，再修改再编译就太蛋疼了，可以通过`dmesg -n x`来将其修改，我们使用5就可以显示结果了。

另外如果你在图形界面下的终端去执行的话，仍然会看不到dmesg的结果，需要切换到text console(tty1~tty6),可以通过Ctrl+Alt+Fx切换到ttyx，切换到tty7即可回到图形界面。tty会要求你登录，依次输入用户名，密码即可，接下来就跟操作终端一样了。

我编译的内核的tty给挂掉了，显示的是一个空黑屏，AskUbuntu上的[这个帖子](http://askubuntu.com/questions/162535/why-does-switching-to-the-tty-give-me-a-blank-screen)提供了解决方案，遇到同样问题的可以参考一下。


**注意老师要求的是使用SYSCALL， 可以忽略proc那部分**


---
### UPDATE END


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

基本上就是为自己的模块提供上init和exit函数，然后再用`module_init`,`module_exit`去注册即可，另外这里`MODULE_LICENSE`是一个声明许可证的宏，用GPL就行了。再添加一个Makefile，这是实验书上的。(注意Makefile是使用TAB字符进行缩进的)
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
基本上就是为我们的虚拟文件提供了open服务。观察代码，我们可以发现，虚拟文件系统这个名字描述地非常准确，实际上这个文件在物理上并不存在（即不存在于磁盘中），每当用户请求打开文件的时候，内核才会动态生成其内容。
代码如下：
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

