
# Lab01
### 环境 Ubuntu 16.04.4 LTS 虚拟机
### 工具qemu, 
## 实验步骤：

### 1.制作根文件系统rootfs.img
先用root身份登录,然后执行命令

~~~
git clone https://github.com/mengning/menu.git
cd menu
gcc -Wall -pthread -o init linktable.c menu.c test.c -m32 -static
cd ../rootfs 
cp ../menu/init ./ 
find . | cpio -o -Hnewc |gzip -9 > ../rootfs.img 
~~~
可能会提示缺少某些库，可以选择下载multilib库，或者按照提示下载


### 2.下载并编译内核
下载的linux内核版本为linux-4.13。解压后使用 make menuconfig 配置内核，将kernel hacking中的compiler设置中将compile the kernel with debug info选中,然后make 编译即可

make menuconfig 命令需要ncurses库，而且要命令行终端宽度大于80字符，注意不要让终端太小。

### 3.启动追踪内核
将rootfs.img移动到内核文件夹，然后安装qemu
在终端内输入命令:

qemu -kernel arch/x86/boot/bzImage -initrd rootfs.img -s -S

qemu启动并且冻结在开始状态
在另一个终端中进入gdb
输入命令：

file vmlinu
break start_kernel
target remote:1234

此时gdb就连接上了qemu，可以从gdb控制qemu的暂停和继续

## 重要事件
说明：我在下面提到的很多函数名只适用于4.13版本
###  init_task的创建
Init_task是系统的第一个进程，也是所有其他进程的祖先。它是在start_kernel运行之前，在平台相关的代码中被创建的。

它是一个task_struct结构体（其中包含超过100项内容），由系统静态定义，而不是通过fork或kernel_thread创建。当然在init_task出现时系统中其实严格来说还没有进程这一概念，从最开始的汇编代码到start_kernel函数都包含在它的内容之中。init_task 由宏INIT_TASK 创建及初始化，扼要地说，定义了它在内存中的位置，它的stack ,thread_info联合体，它的一些状态信息，等等。
### start_kernel的运行
start_kernel是平台无关的内核代码的入口（但事实上代码还会无数次跳回arch目录下执行）这是一个非常庞大的函数，它的主要工作是完成内核的初始化以及生成第一个用户态进程init，比如说初始化死锁检测器，早期控制组群cgroups_init_early（用于限制进程对资源的使用），设立cpu区域(boot_cpu_init())，初始化内存管理系统(mm_init())，初始化虚拟文件系统(vmalloc_init())，初始化调度器(sched_init())，初始化页表，初始化中断向量表，初始化系统时间等等等等等等……

start_kernel还包括一些除错机制的建立，如为栈设置金丝雀值，初始化检查死锁的机制，初始化管理对象生命周期的机制等等。

start_kernel在最后调用rest_init函数

### init进程与kernel_thread进程的创建
在rest_init 函数中调用两次kernel_thread产生进程kernel_init及kernel_thread。

![rest_init 函数两次调用kernel_thread]()

kernel_init进程产生之后，系统才有了占有独立资源的进程概念。kernel_init完成各项配置之后，会通过调用do_execve()函数变成用户态进程init. 而init_task最终会演化成idle进程。具体说来，首先调用init_idle_bootup_task将当前进程（也就是init_task）优先级变成idle类。
![调用init_idle_bootup_task]()

再调用schedule()函数调度一下，当前进程就切换到了init进程。

![调用init_idle_bootup_task]()

此后调用do_idle()，init_task进程就彻底变成了功能为循环检查进程队列的idle进程。

![调用init_idle_bootup_task]()

pid = 1 的 init 进程会继续更高阶的初始化，如初始化 driver, 打开 console, 最后根据不同的系统配置， 执行对应的初始化脚本 ，或者是任一个预设的 init 程序，

![调用init_idle_bootup_task]()

完成整个系统的初始化。其中包括产生shell界面，用于和用户交互。

pid = 2 的 kthreadd 是一个内核线程，它是所有其他内核线程的父线程，负责处理其他内核线程创建请求。它的任务就是管理和调度其他内核线程kernel_thread,。它循环执行一个kthread的函数，该函数的作用是检查kthread_create_list全局链表，链表不为空时就把第一项退出列表并调用kernel_thread()创建线程。因此所有的内核线程都是直接或者间接的以kthreadd为父进程



