# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。
 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”
 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。
 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？
 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”
 - 安装好ucore实验环境，能够编译运行lab8的answer
 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```
 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。
 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
 - 在piazza上就lec3学习中不理解问题进行提问。

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
-  请描述在“计算机组成原理课”上，同学们做的MIPS CPU是从按复位键开始到可以接收按键输入之间的启动过程。

MIPS CPU按复位键之后，首先初始化CPU状态寄存器、其他寄存器、MMU/TLB、CP0寄存器，BIOS从flash中加载OS，跳转到OS入口处

-  x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

从引导扇区读入的是加载程序。因为磁盘的引导扇区只有512字节，不足以存储操作系统的内核映像。因此由加载程序将操作系统内核映像读入空闲空间。

- 比较UEFI和BIOS的区别。

UEFI是面向不同硬件平台、不同操作系统的统一接口，BIOS是固定的硬件平台上的程序，一般固化在计算机主板上。UEFI是C语言写成，BIOS是对应硬件平台的汇编语言写成。UEFI可以看成是BIOS的升级版（？）

- 理解rcore中的Berkeley BootLoader (BBL)的功能。

BBL在被BIOS唤醒后加载Linux内核，并跳转到Linux内核开始处执行。



## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？

55AA

- x86中在UEFI中的可信启动有什么作用？

减少安全风险，确认将要启动的代码是可信、完整并安全的

- RV中BBL的启动过程大致包括哪些内容？

BBL是一个supervisor execution environment。它首先用一个硬件线程筛选设备信息，把Linux可以处理的留下；然后设置中断异常处理，进入supervisor mode，并使supervisor mode能读写内存；设置机器模式的trap handler，然后转入supervisor mode，最后跳到linux的起始地址。

（学习自https://www.sifive.com/blog/all-aboard-part-6-booting-a-risc-v-linux-kernel）

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？

中断：外设与系统进行交互；异常：程序运行中出现异常情况；系统调用：提供接口给用户程序，但同时没有安全问题。

-  中断、异常和系统调用的处理流程有什么异同？

不同：中断是异步的，异常是同步的，系统调用可能是同步或异步的；中断的处理是持续、透明的，异常处理是终止或重新执行，系统调用是等待或持续的。

- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。

 
## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？

系统调用是调用系统内可信的函数，函数调用是调用用户的函数；系统调用需要完成用户态到内核态的切换，并在切换之前做安全检查

- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？
- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。