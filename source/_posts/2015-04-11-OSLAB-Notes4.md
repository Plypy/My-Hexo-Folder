title: OSLAB Notes4
date: 2015-04-11 00:05:52
tags:
- Linux Kernel Compiling
categories:
- LerningNotes
---

这个实验的要求是利用linux的字符设备（char devices）创建一个类似管道（pipe）的媒介以供进程间进行通信。

我主要参考了Linux Devices Drivers， Third Edition（[LDD3](https://lwn.net/Kernel/LDD3/)）这本书，有关字符设备的内容在第三章以及第六章，另外该书的源码在github上有，[here](https://github.com/csl/lld3-example/)。愿意深入研究的同学可以去看一下。（**注意：LDD3针对的是2.6，如果使用的是3.x版本需要修改一些地方，我的Ubuntu是3.13**）


完成这个实验，主要需要两方面的知识，一是Linux的字符设备的相关函数，二是如何利用信号量来进行同步。省事起见，我的代码很多细节都没有考虑，完全是为了达到实验效果而写:)


##字符设备
Linux将所有的外设都包装为文件来进行处理，这样能极大方便用户态的程序，使用现成的文件操作就可以与外设进行交互。为了包装成文件，需要提供相应的一些操作，如文件的打开，关闭，读写等。在内核中定义了这样的一个结构file_operations，通过其成员可以为一个文件提供各种操作，如其`read`成员负责着文件的读取，具体的可以参考LDD3 ch03。若为了完成本次实验的效果，只需要使用`read`和`write`就好。

读操作函数形式为，`ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);`
写操作函数形式为，`ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);`
这里第一个参数为在内核中文件的指针；第二个参数为用户态程序提供的用来交互的buffer，我们向其中读写；第三个参数为用户态希望读写的长度；第四个则是偏移量。

在定义了我们的操作后，需要将其与设备关联起来，并且在内核中注册。设备有major number和minor number两个标号，major区分着设备的类型，而由于同一类型的设备可能有多种，需要使用minor来进行区分。这里我们不管minor，只实现一个就好。注册字符设备可以使用两种方法，LDD3上推荐使用的新方法比较麻烦，需要申请、注册、各种初始化，不表。我们使用老方法。

注册`int register_chrdev(unsigned int major, const char *name, struct file_operations *fops);`

注销`int unregister_chrdev(unsigned int major, const char *name);`

注册时可以直接硬编码一个major，但是这样可能会出现冲突等问题。我们可以令major为0，`register_chrdev`会为我们返回注册到的号，使用printk将其输出即可。注意，注册后并不会在文件系统中生成文件，需要另外编码，或者在用户态中使用`mknod`。简单起见，我们使用后者。


##同步
因为多个进程要同时操作一个文件，这会带来竞争问题。我们可以使用信号量以及睡眠/唤醒机制来控制文件的同步。这里具体可以参考LDD3 ch06。

####信号量
信号量semaphore，其定义在`<linux/semaphore.h>`内。我们只需要以下的几种操作：

初始化
```c
struct semaphore sem; 
sema_init(&sem,1);	//将sem初始化为1，即一个mutex
```
P操作 `down_interruptible(&sem)`,V操作`up(&sem)`。（down_interruptible，故名思议，允许在函数执行时发生中断，不解释细节,下同）

####睡眠/唤醒
当某资源不可用时，我们可以通过令进程进入睡眠态来阻塞进程，而后将其唤醒，这样能使得效率高一些。

唤醒的时候存在这样一个问题，我们需要知道去哪找那些睡着了的进程，也就是说需要存储下来睡眠态的进程。内核提供了`wait_queue_head_t`这样的一种数据结构用以存储睡眠的进程。其初始化方法为`init_waitqueue_head(&que)`。

当我们希望一个进程睡眠时，可以使用`wait_event_interruptible(que, condition)`来将其放入`que`中以备将来唤醒。这里的condition可以是任意的表达式，其作用相当于循环中的入口条件，开始时当condition不满足时进程会进入睡眠，当其被唤醒后会再次检查condition若仍不满足会继续睡眠。这里就很迷惑了，函数是按值传递的，condition怎么还能这样用，还可以检测它变动的值？其实看源码的话会发现，`wait_event_interruptible`是一个宏函数，它会被展开成相应的条件循环逻辑。

换行时使用`wake_up_interruptible(&que)`，其会将`que`中的所有使用`wait_event_interruptible`放入的进程唤醒。

##制作管道
有了以上的预备知识后，也就能开始搞我们的程序了(buggy)。

为了尽量简单，我们将存储的buffer，以及等待队列等数据结构都只做一份全局的，因为我们只需要一个设备。注册模块的时候完成各种初始化以及字符设备的注册，并将注册到的major号输出出来以备使用。

具体数据结构如下,
```
struct plypy_pipe {
    wait_queue_head_t inq, outq;       /* read and write queues */
    char buffer[MAXN], *end;           /* static buffer */
    char *wp;                          /* where the data ends */
    struct semaphore sem;              /* mutual exclusion semaphore */
};
```
`inq`,`outq`分别用来存储读/写的进程。`buffer`数组用来存储数据，`end`是一个辅助的变量用来标记buffer的末尾。`wp`用来标记buffer数据的末尾，可以用来判断`buffer`是否为空。`sem`则为一个信号量。

简单起见，我们的读写逻辑是这样的。`buffer`中只存储一次写的数据，不支持连续写，不支持连续读。即只有在`buffer`为空的时候，才可以再写入下一个数据；只有在`buffer`中有数据的时候，才能读取数据，并且每次读取完毕后将其设为空。可以看出我们的管道只支持‘写读写读写读……’这样的操作序列，并且每次数据的传输都是从某一个写进程传向某一个读进程，并非广播。

在读写数据时，涉及到一次数据从内核到用户的传输，需要使用`copy_to_user`，`copy_from_user`两个函数来完成。

读写的流程也比较简单，不再赘述，直接看源码吧，如下：
```c
/**
 * Create a virtual char devices
 **/

#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/fs.h>
#include <linux/semaphore.h>
#include <linux/types.h>
#include <linux/wait.h>
#include <linux/cdev.h>
#include <linux/sched.h>
#include <asm/uaccess.h>

#define MAXN 1024
#define PLYPY_DEV_NAME "plypy_chrdev"


/* static int plypy_dev_open(struct inode *, struct file *filp); */
static ssize_t plypy_dev_read(struct file *, char *, size_t, loff_t *);
static ssize_t plypy_dev_write(struct file *, const char *, size_t, loff_t *);
/* static int plypy_dev_release(struct inode *, struct file *filp); */

struct file_operations fops =
{
    /* .open = plypy_dev_open, */
    /* .release = plypy_dev_release, */
    .read = plypy_dev_read,
    .write = plypy_dev_write
};

int Major;
struct plypy_pipe {
    wait_queue_head_t inq, outq;       /* read and write queues */
    char buffer[MAXN], *end;           /* static buffer */
    char *wp;                          /* where the data ends */
    struct semaphore sem;              /* mutual exclusion semaphore */
};

static struct plypy_pipe plypy_pipe;
static struct plypy_pipe *dev = &plypy_pipe;

static ssize_t plypy_dev_read(struct file *filp, char __user *buf, size_t count,
                              loff_t *offset)
{
    if (down_interruptible(&dev->sem))
        return -ERESTARTSYS;

    /* There may be multiple readers, so the use of loop is necessary */
    while (dev->buffer == dev->wp) { /* nothing to read, wait for inputs */
        up(&dev->sem);

        if (wait_event_interruptible(dev->inq, (dev->buffer != dev->wp)))
            return -ERESTARTSYS;
        /* Loop and reacquire the lock */
        if (down_interruptible(&dev->sem))
            return -ERESTARTSYS;
    }

    /* read data */
    count = min(count, (size_t)(dev->wp - dev->buffer));
    if (copy_to_user(buf, dev->buffer, count)) {
        /* error happened */
        up(&dev->sem);
        return -EFAULT;
    }
    dev->wp = dev->buffer;
    up(&dev->sem);

    wake_up_interruptible(&dev->outq);
    return count;
}


static ssize_t plypy_dev_write(struct file *filp, const char __user *buf,
                               size_t count, loff_t *offset)
{
    if (down_interruptible(&dev->sem))
        return -ERESTARTSYS;

    while (dev->buffer != dev->wp) { /* the old data haven't been retrieved */
        up(&dev->sem);
        if (wait_event_interruptible(dev->outq, (dev->buffer == dev->wp)))
            return -ERESTARTSYS;
        /* P and loop again */
        if (down_interruptible(&dev->sem))
            return -ERESTARTSYS;
    }

    count = min(count, (size_t)( dev->end - dev->buffer ));
    if (copy_from_user(dev->buffer, buf, count)) {
        /* error happened */
        up(&dev->sem);
        return -EFAULT;
    }
    dev->wp += count;
    up(&dev->sem);
    wake_up_interruptible(&dev->inq);

    return count;
}

static int plypy_init(void)
{
    plypy_pipe.end = dev->buffer+MAXN;
    plypy_pipe.wp = dev->buffer;
    init_waitqueue_head(&dev->inq);
    init_waitqueue_head(&dev->outq);
    sema_init(&dev->sem, 1);

    Major = register_chrdev(0, PLYPY_DEV_NAME, &fops);
    if (Major < 0) {
        return Major;
    }
    printk(KERN_INFO "The %s is assigned major number %d",
           PLYPY_DEV_NAME, Major);
    printk(KERN_INFO "Use 'mknod /dev/%s c %d 0' to create a file",
           PLYPY_DEV_NAME, Major);
    return 0;
}

static void plypy_exit(void)
{
    unregister_chrdev(Major, PLYPY_DEV_NAME);
    printk(KERN_INFO "The %s is destroyed", PLYPY_DEV_NAME);
}

module_init(plypy_init);
module_exit(plypy_exit);

MODULE_LICENSE("GPL");

```

##编译&测试
我使用的是如下的Makefile进行的测试
```
source := plypy
cdevname := plypy_chrdev
major := $(shell awk -v mod='$(cdevname)' '$$2==mod{print $$1}' /proc/devices)

ifneq ($(KERNELRELEASE),)
	obj-m:=$(source).o
else
	KERNELDIR:=/lib/modules/$(shell uname -r)/build
	PWD:=$(shell pwd)
endif
build:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules

install:
	insmod $(source).ko
	mknod /dev/$(cdevname) c $(major) 0

remove:
	rmmod $(source)
	rm /dev/$(cdevname)

clean:
	rm modules.order Module.symvers *.ko *.o
```

`source`这里是你的源文件的名字（无后缀），`cdevname`是注册字符设备时使用的名字，需要通过它在`/proc/devices`里找刚刚我们的设备注册到的major。

在root下依次执行如下命令，编译安装模块并创建字符设备文件。
```
#make build
#make install
```
接下来可以用`cat`和`echo`来测试，开启一个终端执行`#cat /dev/plypy_chrdev`，在另一个终端下不断用`echo`写入数据，如下：
```
#echo 20 > /dev/plypy_chrdev
#echo 30 > /dev/plypy_chrdev
```
可以看到每次写入后，均会在`cat`中出现。

若要编程测试的话也比较简单，无非就是一端`read`，一端`write`。

读程序
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>           /* O_RDWR */
#include <unistd.h>             /* read/write */
#include <fcntl.h>              /* open */
#define MAXN 128

char buffer[MAXN];
int main(void)
{
    int fd = open("/dev/plypy_chrdev", O_RDWR);
    while (1) {
        printf("Read something?");
        memset(buffer, 0, sizeof(buffer));
        while (getchar() != '\n') /* eat it all */
            continue;
        read(fd, buffer, MAXN-1);
        puts(buffer);
    }
    return 0;
}
```

写程序
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>           /* O_RDWR */
#include <unistd.h>             /* read/write */
#include <fcntl.h>              /* open */
#define MAXN 128

char buffer[MAXN];
int main(void)
{
    int fd = open("/dev/plypy_chrdev", O_RDWR);
    while (1) {
        printf("Write something:\n");
        memset(buffer, 0, sizeof(buffer));
        gets(buffer);
        write(fd, buffer, strlen(buffer)+1);
    }
    return 0;
}
```
编译后在root下执行即可。

##End
就这样
