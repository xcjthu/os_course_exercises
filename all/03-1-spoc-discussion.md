# lec5: SPOC思考题

##**提前准备**
（请在上课前完成）

- 完成lec５的视频学习和提交对应的在线练习
- git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises in github repos。这样可以在本机上完成课堂练习。
- 理解连续内存动态分配算法的实现（主要自学和网上查找）

NOTICE
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 思考题
---

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次

1.操作系统中存储管理的目标是什么？

为每一个进程分配、释放内存空间。


### 5.2 地址空间和地址生成
1.描述编译、汇编、链接和加载的过程是什么？

编译将高级编程语言转换成汇编指令；

汇编将汇编指令转成二进制指令，其中的符号转换为编号；

链接将多个模块及函数库排列成线性序列，并将其中的一些跨模块调用的符号转换成编号地址；

加载将程序进行重定位，将所有表示地址的编号转换成实际地址。



2.动态链接如何使用？尝试在Linux平台上使用LD_DEBUG查看一个C语言Hello world的启动流程。  (optional)

通过命令

```asm
LD_DEBUG=reloc ./a.out
```

得到下列输出

```assembly
     35343:
     35343:	relocation processing: /lib/x86_64-linux-gnu/libc.so.6 (lazy)
     35343:
     35343:	relocation processing: /lib/x86_64-linux-gnu/libgcc_s.so.1 (lazy)
     35343:
     35343:	relocation processing: /lib/x86_64-linux-gnu/libm.so.6 (lazy)
     35343:
     35343:	relocation processing: /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (lazy)
     35343:
     35343:	relocation processing: ./a.out (lazy)
     35343:
     35343:	relocation processing: /lib64/ld-linux-x86-64.so.2
     35343:
     35343:	calling init: /lib/x86_64-linux-gnu/libc.so.6
     35343:
     35343:
     35343:	calling init: /lib/x86_64-linux-gnu/libgcc_s.so.1
     35343:
     35343:
     35343:	calling init: /lib/x86_64-linux-gnu/libm.so.6
     35343:
     35343:
     35343:	calling init: /usr/lib/x86_64-linux-gnu/libstdc++.so.6
     35343:
     35343:
     35343:	initialize program: ./a.out
     35343:
     35343:
     35343:	transferring control: ./a.out
     35343:
hello world!
     35343:
     35343:	calling fini: ./a.out [0]
     35343:
     35343:
     35343:	calling fini: /usr/lib/x86_64-linux-gnu/libstdc++.so.6 [0]
     35343:
     35343:
     35343:	calling fini: /lib/x86_64-linux-gnu/libm.so.6 [0]
     35343:
     35343:
     35343:	calling fini: /lib/x86_64-linux-gnu/libgcc_s.so.1 [0]
     35343:
```

可以看到，在运行程序之前，先链接、重定向了几个动态链接库



### 5.3 连续内存分配
1.什么是内碎片、外碎片？

内碎片是指，分配给进程内部，没办法利用的部分。（分配给进程的内存大小，大于其申请的内存大小）

外碎片是指，两块被分配的内存空间之间的未分配的部分。



2.最先匹配会越用越慢吗？请说明理由（可来源于猜想或具体的实验）？

会，因为最先匹配总是分配最先找到的大小足够的内存空间，在前面将大部分内存空间使用过了之后，每次需要新的内存空间时，搜索的时间就会变长。



3.最差匹配的外碎片会比最优适配算法少吗？请说明理由（可来源于猜想或具体的实验）？

会，最优匹配将会产生大量的较小的外碎片，最差匹配小的外碎片将会少一些。



4.理解0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm算法中分区释放后的合并处理过程？ (optional)

最优匹配：分区释放后，需要查找与被释放分区地址临近的空闲分区，进行合并，并将其重新按照由小到大的顺序进行重排序；

最先匹配：同最优匹配，找地址临近的空闲分区进行合并，但是由于在维护数据结构时本身就是按照地址排序，所以不需要进行重排序；

最差匹配：同最优匹配，合并临近空闲分区，并重排序；

buddy system：需要判断临近分区是否可以合并，然后需要进行大小、地址的二维重排序。




### 5.4 碎片整理
1.对换和紧凑都是碎片整理技术，它们的主要区别是什么？为什么在早期的操作系统中采用对换技术？  

对换主要是将一个进程存储在内存中的内容，在进程处于等待状态时换到外存中存储。

紧凑则是通过移动进程在内存中的位置，让碎片集中起来，获得更大的可用内存空间。

早期内存资源十分有限，因此需要利用外存的存储空间，来让机器正常运行。



2.一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

操作系统，将内存中进程的数据，存储到外存中，再将正在外存中的进程数据加载到内存中。



### 5.5 伙伴系统

1.伙伴系统的空闲块如何组织？

伙伴系统的空闲块按照空闲块大小和地址进行二维排序。



2.伙伴系统的内存分配流程？伙伴系统的内存回收流程？

分配流程：伙伴系统首先找到最小的可用的空闲内存块，如果空闲块大小大于申请大小的两倍，则将空闲块对半分。

回收流程：考察相邻的空闲块，如果和相邻的空闲块相加后大小为2的整数次幂，且最低的地址是空闲块大小的整数倍，则合并，否则不合并。



## 课堂实践

观察最先匹配、最佳匹配和最差匹配这几种动态分区分配算法的工作过程，并选择一个例子进行分析分析整个工作过程中的分配和释放操作对维护数据结构的影响和原因。

  * [算法演示脚本的使用说明](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep3-malloc.md)
  * [算法演示脚本](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep3-malloc.py)

例如：
```
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p BEST -n 5 -c
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p FIRST -n 5 -c
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p WORST -n 5 -c
```

### 扩展思考题 (optional)

1. 请参考xv6（umalloc.c），ucore lab2代码，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在Linux应用程序/库层面，用C、C++或python来实现malloc/free，给出你的设计思路，并给出可以在Linux上运行的malloc/free实现和测试用例。


2. 阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。
