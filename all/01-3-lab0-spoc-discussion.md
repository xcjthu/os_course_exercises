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

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

  在进程管理上，由于操作系统需要在cpu上进行进程的切换，因此在硬件上需要支持中断操作；虚存管理要求硬件进行地质映射，因此需要MMU、TLB等硬件；对于文件系统，由于文件是存储在硬盘上的，因此硬件需要提供这样的存储介质。对应需要的特权指令有软件中断相关的；设置地址转换方式，以及类似于写TLB、设置页表等指令；执行设备I/O等指令。

  

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

  实模式和保护模式的主要区别在于内存是否收到保护，实模式下，16位寻址空间；保护模式下，32位寻址空间，有很好的机制保证操作系统和各个进程的安全。

  物理地址是指物理介质（例如内存）中直接对应的地址；线性地址是指通过了段模式映射之后的地址，是逻辑地址和物理地址中间层的地址；逻辑地址是指进程直接使用的地址。

  
- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？

  

  

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```c
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

​	“:” 后面的数字表示各个域在结构体中占的位数。



- 对于如下的代码段，

```c
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

```c
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

unsigned 只有32位，即只关心 gd_off_15_0和gd_ss域即可，因此执行上述指令后，intr的值为0x00020003



### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

   ```assembly
   .text
   .globl switch_to
   switch_to:                      # switch_to(from, to)
   
       # save from's registers
       movl 4(%esp), %eax          # eax points to from
       popl 0(%eax)                # save eip !popl
       movl %esp, 4(%eax)
       movl %ebx, 8(%eax)
       movl %ecx, 12(%eax)
       movl %edx, 16(%eax)
       movl %esi, 20(%eax)
       movl %edi, 24(%eax)
       movl %ebp, 28(%eax)
   
       # restore to's registers
       movl 4(%esp), %eax          # not 8(%esp): popped return address already
                                   # eax now points to to
       movl 28(%eax), %ebp
       movl 24(%eax), %edi
       movl 20(%eax), %esi
       movl 16(%eax), %edx
       movl 12(%eax), %ecx
       movl 8(%eax), %ebx
       movl 4(%eax), %esp
   
       pushl 0(%eax)               # push eip
   
       ret
   ```

   这是一段用来切换进程的汇编代码，这里首先将所有的几个寄存器保存在栈上一段连续的地址，再从指定的一块内存地址上恢复各个寄存器的值，这样就完成了进程间状态的切换。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

   

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

宏定义可以很好的减少代码重复，让一些常量的含义更加清晰。

#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
