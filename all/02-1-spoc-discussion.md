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
-  x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？<br>
答：<br>
BIOS完成硬件初始化和自检后，会根据CMOS中设置的启动顺序启动相应的设备，这里假定按顺序系统要启动硬盘。<br>
但此时，文件系统并没有建立，BIOS也不知道硬盘里存放的是什么，所以BIOS是无法直接启动操作系统。<br>
另外一个硬盘可以有多个分区，每个分区都有可能包括一个不同的操作系统，BIOS也无从判断应该从哪个分区启动，<br>
所以对待硬盘，所有的BIOS都是读取硬盘的0磁头、0柱面、1扇区的内容，然后把控制权交给这里面的MBR。<br>
MBR由两个部分组成：即主引导记录MBR和硬盘分区表DPT。在总共512字节的主引导分区里其中MBR占446个字节（偏移0--偏移1BDH)，<br>
一般是一段引导程序，其主要是用来在系统硬件自检完后引导具有激活标志的分区上的操作系统。<br>
DPT占64个字节（偏移1BEH--偏移1FDH),一般可放4个16字节的分区信息表。<br>
最后两个字节“55，AA”（偏移1FEH，偏移1FFH)是分区的结束标志.<br>

- 比较UEFI和BIOS的区别。<br>
答：<br>
UEFI，全称Unified Extensible Firmware Interface，即“统一的可扩展固件接口”, 是适用于电脑的标准固件接口，<br>
旨在代替BIOS（基本输入/输出系统）。此标准由UEFI联盟中的140多个技术公司共同创建，其中包括微软公司。<br>
UEFI旨在提高软件互操作性和解决BIOS的局限性。<br>
UEFI启动对比BIOS启动的优势有三点：<br>
1. 安全性更强：UEFI启动需要一个独立的分区，它将系统启动文件和操作系统本身隔离，可以更好的保护系统的启动。<br>
2. 启动配置更灵活：EFI启动和GRUB启动类似，在启动的时候可以调用EFIShell，在此可以加载指定硬件驱动，选择启动文件。比如默认启动失败，在EFIShell加载U盘上的启动文件继续启动系统。 <br>
3. 支持容量更大:传统的BIOS启动由于MBR的限制，默认是无法引导超过2TB以上的硬盘的。随着硬盘价格的不断走低，2TB以上的硬盘会逐渐普及，因此UEFI启动也是今后主流的启动方式。<br>

- 理解rcore中的Berkeley BootLoader (BBL)的功能。<br>
答：<br>

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？   0X55AA<br>
- x86中在UEFI中的可信启动有什么作用？   通过启动前的数字签名检查来保证启动介质的安全性。<br>
- RV中BBL的启动过程大致包括哪些内容？    

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？<br>
答：<br>
中断：外部意外的响应；<br>
异常：指令执行意外的响应；<br>
系统调用：系统调用指令的响应。<br>

-  中断、异常和系统调用的处理流程有什么异同？<br>
相同点：都会进入异常服务例程，切换为内核态。<br>
不同点：<br>
      1.源头不同，中断源是外部设备，异常和系统调用源是应用程序；<br> 
      2.响应方式不同，中断是异步的，异常是同步的，系统调用异步和同步都可以。<br>
      3.处理机制不同，中断对用户程序是透明的，异常会重新执行用户指令或杀死用户进程，系统调用一般是用户程序调用的<br>

- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？<br>
答：<br>
static int (*syscalls[])(uint32_t arg[]) = {<br>
    [SYS_exit]              sys_exit,<br>
    [SYS_fork]              sys_fork,<br>
    [SYS_wait]              sys_wait,<br>
    [SYS_exec]              sys_exec,<br>
    [SYS_yield]             sys_yield,<br>
    [SYS_kill]              sys_kill,<br>
    [SYS_getpid]            sys_getpid,<br>
    [SYS_putc]              sys_putc,<br>
    [SYS_pgdir]             sys_pgdir,<br>
    [SYS_gettime]           sys_gettime,<br>
    [SYS_lab6_set_priority] sys_lab6_set_priority,<br>
    [SYS_sleep]             sys_sleep,<br>
    [SYS_open]              sys_open,<br>
    [SYS_close]             sys_close,<br>
    [SYS_read]              sys_read,<br>
    [SYS_write]             sys_write,<br>
    [SYS_seek]              sys_seek,<br>
    [SYS_fstat]             sys_fstat,<br>
    [SYS_fsync]             sys_fsync,<br>
    [SYS_getcwd]            sys_getcwd,<br>
    [SYS_getdirentry]       sys_getdirentry,<br>
    [SYS_dup]               sys_dup,<br>
};<br>

分类有如下四种：<br>
进程管理：包括 fork/exit/wait/exec/yield/kill/getpid/sleep<br>
文件操作：包括 open/close/read/write/seek/fstat/fsync/getcwd/getdirentry/dup<br>
内存管理：pgdir命令<br>
外设输出：putc命令<br>

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。

 
## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？<br>
答：<br>
汇编指令的区别<br>
      系统调用：使用INT和IRET指令<br>
      函数调用：使用CALL和RET指令<br>
安全性的区别<br>
      系统调用有堆栈和特权级的转换过程，函数调用没有这样的过程，系统调用相对更为安全<br>
性能的区别<br>
      时间角度：系统调用比函数调用要做更多和特权级切换的工作，所以需要更多的时间开销<br>
      空间角度：在一些情况下，如果函数调用采用静态编译，往往需要大量的空间开销，此时系统调用更具有优势<br>

- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？<br>

- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？<br>

## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。
