title: "OSLAB Adding a system call to Linux kernel"
date: 2014-11-03 19:47:22
tags: 
- Linux Kernel Compiling
- VirtualBox
- Shithole
categories:
- LerningNotes
---
操作系统实验记录---编译Linux内核添加系统调用
---

学过好多东西。。。但往往一定时间后自己都会忘掉。。。想去学就还得再重复一次之前的过程。还是把自己学习的过程记录下来吧，方便以后查阅。

我渣交计算机大三的课程是相当的充实啊，海量的专业课与实验，再加上其他的一些事情，最近几天变身“真学狗”。。。

这次操作系统实验是给Linux内核添加一个系统调用，然后重编内核。我的环境如下：
Windows下VirtualBox 4.6+Ubuntu12.04(64bit)+Linux3.16

### 虚拟机&Ubuntu
编译Linux内核当然首先需要一个Linux的发行版了，我用的是小白福音Ubuntu。实验指导是让在虚拟机下编译内核的，但听夏赢家说可以直接在实际的系统下搞这件事情，这样最后不过只是给系统添加了一个启动时的选项而已，不会影响原来的内核。

于是我欢心雀跃的跑到我Ubuntu下面开始编译。。。为了追求速度，用了多线程编译。没曾想，电脑太渣，只听风扇飞转，过一会机器就黑了。。。大概是CPU过热保护断电了吧，呃，Linux的桌面版对于个人用户来说还是有些渣啊。思考再三后我决定还是在虚拟机下搞这件事情，因为看到SO上有不少人说搞内核这个东西可能会"Messing up your production machine"

正题开始，首先我们需要VirtualBox，推荐使用较新的4.x版本，有不少方便的功能，以及一个Ubuntu的映像文件，自己去下载吧。(**UPDATE** 蟹老板告诉我vmware可以轻松ctrl+c/v。。。，想试的同学可以去搞一下)

好了后新建一个虚拟机，选择对应版本的Ubuntu，这个一定要跟自己准备安装的Ubuntu的版本对上。然后再几个选项，注意硬盘容量这里要选大一点。。。否则会不够用，我被坑了两次，大概30G就够用了。还有这里需要注意一点，Windows下VirtualBox会默认将磁盘文件存储在C盘下，这个路径想改的话可以在Settings-General里改掉。**并且**把内存设置得大一些，1G就够了，否则会悲剧，下面会讲到。

然后我们再来配置一下这个虚拟机，我改了这几个
+ General-->Shared Clipboard：共享虚拟机和主机器的黏贴板
+ System-->Acceleration：硬件加速，能快一点，但似乎有些机器需要在BIOS上先启用硬件加速功能
+ System-->Processor: 这里选择和你机器一样的CPU数目，效率会高一些。
+ Network：我使用了最为简单的NAT，不需要配置什么的

*下面这些部分都是关于配置VirtualBox的虚拟机与主系统进行文件拷贝的*
+ Shared Folders：虚拟机和主机间共享的文件夹，可以方便的用来传输文件。在Machine Folders里添加一个你想要向虚拟机里共享的文件夹吧，然后把auto mount，permanent勾上。

接下来启动虚拟机，按照提示选择之前准备好的Ubuntu镜像，安装就是了。安装完毕后进入Ubuntu，还不能直接就开始干活。在上方菜单栏里找Devices-->Insert Guest Additions CD image。这个是VirtualBox的一个增强插件，不安装的话无法使用共享文件夹等功能。点击后，虚拟机会加载这个镜像，然后弹出窗口，选Run就是了。关于Guest Additions 具体参考官方文档 https://www.virtualbox.org/manual/ch04.html

这是在 `/media`下面会加载我们之前共享的文档，Ctrl+Alt+T呼叫出终端，执行
    
    cd /media
    ls -l

可以看到这个目录下有一个sf_开头的文件夹这个就是我们共享的文件夹。我的是`sf_OSLAB`。但是此时如果我们访问的话是会显示`Permission denied`，因为应当注意到这个文件夹是属于`vboxsf`这个组的，我的用户名为plypy,执行

    sudo adduser plypy vboxsf

然后注销再进入系统就可以搞定了

### 向Linux内核添加Hello world syscall

呼出终端，建立一个文件夹用于此次实验

    mkdir OSLAB

---

**Update:**本来用的是助教ftp上的3.13但是悲剧了，编译安装后没办法启动于是我就去下载了 3.16，最近网速蛮快的，我就直接在虚拟机下下载了压缩包

    wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.16.tar.xz

**More Update**操**的，3.16也挂了。。。不过还好3.16显示了Kernel Panic的信息，"not syncing out of memory and no killable processes"关掉虚拟机，把内存增加设置到1G就解决问题了。想必之前3.13没成功也是这个问题，但是3.13当时没有显示Kernel Panic信息。我也不清楚,还是推荐大家用3.16或者其他的稳定版本吧。

---

把东西都放进那个共享的文件夹后，在Linux下执行命令将东西都拷贝到之前建立的目录下吧

    cp /media/sf_OSLAB/* OSLAB/ -r

我将Linux的压缩包放在了里面，所以进去解压

    cd OSLAB
    tar -xvf linux-3.16.tar.xz

接下来进入解压后的目录

    cd linux-3.16

`arch/x86/syscalls/syscall_64.tbl`这个文件存储的是64位所有syscall的表，进去添加一个。如果你安装的是32位的Ubuntu的话修改`syscall_32.tbl`就好。

    cd arch/x86/syscalls

这张表的结构是

    <number> <abi> <name> <entry point>

+ number， 是对应的syscall的编号。
+ abi,文档说是The ABI, or application binary interface, to use. Either 64, x32, or common for both。
+ name，是名字
+ entry point是你定义的函数的名字。按照惯例应该为 sys_function_name

我添加了一条

    317 common  plypy_hello     sys_plypy_hello

关于如何编辑这个文件，可以使用

    vi syscall_64.tbl

这使用的是系统自带的vim，这是一个蛮不错的编辑器。想学习的话运行一下`vimtutor`。或者简单一点用`gedit`也可以，

    gedit syscall_64.tbl

接下来找`include/linux/syscalls.h`，将我们的syscall声明添加进该头文件，仿照其他的声明，写

    asmlinkage long sys_plypy_hello(void);

因为我们将添加的是一个无参数的syscall所以声明成这样，注意在C语言下声明不带参量的函数需要使用void关键字，如
    
    return_value foo(void);

另外这里的`asmlinkage`是用来告诉GCC不要将这个函数的参量存入寄存器而是栈中,详见Google。还有syscall需要返回一个long。

接下来去实现自己的syscall，建立`kernel/plypy_hello.c`,如下

~~~c
#include <linux/kernel.h>
asmlinkage long sys_plypy_hello(void)
{
    printk("Ply_py says, Hello World!\n");
    return 0;
}
~~~

这里调用了printk，其是printf的兄弟函数，作用是向kernel的日志文件写信息。

由于我们添加了新的源文件，为了将其链接进来。修改`kernel/Makefile`,将`plypy_hello.o`添加至obj-y的那个表中,完了是这样的。

    obj-y     = fork.o exec_domain.o panic.o \
                cpu.o exit.o itimer.o time.o softirq.o resource.o \
                sysctl.o sysctl_binary.o capability.o ptrace.o timer.o user.o \
                signal.o sys.o kmod.o workqueue.o pid.o task_work.o \
                extable.o params.o posix-timers.o \
                kthread.o sys_ni.o posix-cpu-timers.o \
                hrtimer.o nsproxy.o \
                notifier.o ksysfs.o cred.o reboot.o \
                async.o range.o groups.o smpboot.o plypy_hello.o

至此我们就可以编译内核啦。

### 内核编译

接下来返回所有源文件的根目录，开始编译

首先把编译需要的东西下载了

    sudo apt-get install build-essential libncurses5-dev

配置

    make menuconfig

基本采取默认配置即可，可以在General-->Local version处修改一下，方便区分自己搞的内核。 Save后,开始编译链接，之前开启了多个处理器的选项，所以可以使用多线程编译，-j(n)选项是使用n线程编译，差不多几个核就几线程吧。

    make -j2

编译是一个非常漫长的过程。。。至少对于我的渣电脑来说

然后编译各个模块

    make modules -j2

安装模块及内核

    sudo make modules_install
    sudo make install

### 测试

到这里就把内核安装好了，接下来重启一下，在启动的时候应该能看到Grub的启动菜单，选择之前编译好的linux3.16plypyhello就好，使用如下代码

```c
#include <sys/syscall.h>

#define SYS_PLYPY_HELLO 317

int main(void)
{
    syscall(SYS_PLYPY_HELLO);
    return 0;
}
```
然后编译运行

    gcc testsyscall.c -o tester
    ./tester

此时kernel的日志文件里应该多了一句"Ply_py says, Hello World!",查看一下

    dmesg

就这样吧。

### 错误处理

**编译出错**，这个错误只可能发生在之前修改的几个文件当中。我数次把kernel写成了kernal。。。

修复完错误后需要重新编译，但得先清除上一次编译遗留的东西，由于我们的配置非常少所以不妨直接全部清除 `make distclean`,然后再重复编译的过程就可以了。具体可以参考帮助文档`make help`

**删除内核文件**
有时会悲剧地编译通过后无法启动。。。把多余的内核放在有限的虚拟机空间里也不太好，可以去`/lib/modules/`,`/boot/`下删除掉之前生成的东西

    sudo rm -rf *plypyhello/ 

然后更新一下Grub
    
    sudo update-grub

### 总结

真tm吃力。。。

