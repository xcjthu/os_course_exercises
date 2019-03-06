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
- x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

  BIOS从磁盘读入的第一个扇区内容是磁盘的主引导扇区代码。

  不直接读入操作系统内核映像的原因有：需要通过磁盘上的主引导记录确定要加载哪一个操作系统；对于不同机器有不同的文件系统，BIOS不知道硬盘中存放的是什么，因此无法直接读入操作系统内核映像。

  

- 比较UEFI和BIOS的区别。

  UEFI是一种个人电脑系统规格，用来定义操作系统与系统固件之间的软件界面，作为BIOS的替代方案。

  UEFI启动对比BIOS启动有几个优势：1、安全性更强，UEFI启动需要一个独立的分区，这样将系统启动文件和操作系统隔离，可以更好的保护系统的启动；2、启动配置灵活；3、支持容量更大，BIOS受到MBR格式限制，无法引导大小超过2TB的硬盘。

  

- 理解rcore中的Berkeley BootLoader (BBL)的功能。

  配置m态的中断和一些外部设备，加载内核并切换到s态，提供sbi接口。

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？

  0x55AA

  

- x86中在UEFI中的可信启动有什么作用？

  可以在加载操作系统时，通过数字签名进行验证来保证启动介质的安全性。

  

- RV中BBL的启动过程大致包括哪些内容？

  配置外设、配置PMP、配置中断处理、加载内核、在s态运行内核

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？

  系统调用：应用程序主动向操作系统发出的服务请求

  异常：非法指令或者其他原因导致当前指令执行失败后的处理请求

  中断：来自硬件设备的处理请求

  

- 中断、异常和系统调用的处理流程有什么异同？

  中断：由外部设备触发，异步处理，对用户程序是透明的

  异常：由应用程序的错误产生，同步处理，一般会杀死或重新执行意想不到的错误指令

  系统调用：系统调用由应用程序的主动请求产生，有可能是同步或者异步。

  

- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？

  如下图所示，系统调用有以下这些，大致可以分成：

  进程管理、文件操作、内存管理、外设输出

  ```c
  static int (*syscalls[])(uint32_t arg[]) = {
      [SYS_exit]              sys_exit,
      [SYS_fork]              sys_fork,
      [SYS_wait]              sys_wait,
      [SYS_exec]              sys_exec,
      [SYS_yield]             sys_yield,
      [SYS_kill]              sys_kill,
      [SYS_getpid]            sys_getpid,
      [SYS_putc]              sys_putc,
      [SYS_pgdir]             sys_pgdir,
      [SYS_gettime]           sys_gettime,
      [SYS_lab6_set_priority] sys_lab6_set_priority,
      [SYS_sleep]             sys_sleep,
      [SYS_open]              sys_open,
      [SYS_close]             sys_close,
      [SYS_read]              sys_read,
      [SYS_write]             sys_write,
      [SYS_seek]              sys_seek,
      [SYS_fstat]             sys_fstat,
      [SYS_fsync]             sys_fsync,
      [SYS_getcwd]            sys_getcwd,
      [SYS_getdirentry]       sys_getdirentry,
      [SYS_dup]               sys_dup,
  };
  ```

  

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。


## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？

  系统调用是由操作系统提供给应用程序的服务，从实现的指令上来看，系统调用需要使用INT和IRET指令，函数调用需要使用CALL和RET指令。系统调用时，有堆栈的切换和特权级的切换；函数调用时没有堆栈和特权级的切换。

  

- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？

  函数调用的堆栈操作是把返回地址压栈

  系统调用的堆栈操作有硬件实现的部分

- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？

  系统调用的堆栈操作是软件实现的，epc等寄存器的保存。函数调用则是把返回地址写入寄存器。

  


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。
