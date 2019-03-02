# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？<br>
答：<br>
进程的切换需要硬件支持时钟中断；虚存管理需要地址映射机制，从而需要MMU等硬件；<br>
对于文件系统，需要硬件有稳定的存储介质来保证操作系统的持久性。 <br>
对应的，应当提供中断使能，触发软中断等中断相关的，设置内存寻址模式，设置页表等内存管理相关的，执行I/O操作等文件系统相关的特权指令。


- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？<br>
答：<br>
实模式与保护模式的根本区别是进程内存受保护与否。实模式将整个物理内存看成分段的区域，程序代码和数据位于不同区域，<br>
系统程序和用户程序没有区别对待，而且每一个指针都是指向“实在”的物理地址。这样一来，用户程序的一个指针如果指向了<br>
系统程序区域或其他用户程序区域，并改变了值，那么对于这个被修改的系统程序或用户程序，其后果就很可能是灾难性的。<br>
为了克服这种低劣的内存管理方式，处理器厂商开发出保护模式。这样，物理内存地址不能直接被程序访问，<br>
程序内部的地址（虚拟地址）要由操作系统转化为物理地址去访问，程序对此一无所知。<br>
实模式只能访问地址在1M以下的内存称为常规内存，我们把地址在1M 以上的内存称为扩展内存。<br>
在保护模式下，全部32条地址线有效，可寻址高达4G字节的物理地址空间;<br>
实模式：  内存寻址方式是 段式寻址=段地址 * 16 + 段内偏移地址<br>
         可寻址任意地址，所有指令都相当于工作在特权级<br>
         dos工作在实模式下<br>
保护模式：内存寻址方式为 支持内存分页和虚拟内存<br>
         支持多任务。可以依靠硬件用一条指令即可实现任务切换，不同任务工作在不同优先级下，操作系统工作在最高优先级0上，<br>
         应用程序则运行在较低的优先级上。从实模式到保护模式需要建立GDT,IDT等数据表，然后通过修改控制寄存器CR0的控制位（0）来实现<br>
         Windows工作在保护模式上。<br>
物理地址：是处理器提交到总线上用于访问计算机系统中的内存和外设的最终地址。一个计算机系统中只有一个物理地址<br>
线性地址：线性地址是逻辑地址到物理地址变换之间的中间层，处理器通过段（Segment）机制控制下形成的地址空间<br>
逻辑地址：在有地址变换功能的计算机中，访问指令给出的地址叫逻辑地址<br>

- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？<br>
答：<br>
Machine Mode：机器模式，简称M Mode。<br>
Supervisor Mode：监督模式，简称S Mode。<br>
User Mode：用户模式，简称U Mode。<br>
RISC-V架构定义M Mode为必选模式，另外两种为可选模式。通过不同的模式组合可以实现不同的系统。<br>
RISC-V架构也支持几种不同的存储器地址管理机制，包括对于物理地址和虚拟地址的管理机制，<br>
使得RISC-V架构能够支持从简单的嵌入式系统（直接操作物理地址）到复杂的操作系统（直接操作虚拟地址）的各种系统。<br>

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？<br>
答：<br>
“:”后的数字表示每一个域在结构体中所占的位数，用于表示这个成员变量占多少位(bit)。<br>
intr的值为 0X20003<br>

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
