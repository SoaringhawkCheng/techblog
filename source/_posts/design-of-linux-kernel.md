---
title: 「Linux内核设计与实现」学习笔记[DOING]
catalog: true
date: 2020-02-27 22:17:27
subtitle:
header-img:
tags:
- os
categories:
- 工程
---
> 书籍豆瓣链接：[《Linux内核设计与实现》](https://book.douban.com/subject/6097773/)
> 
> 相关资料整理：[读书笔记](https://blog.csdn.net/weixin_33682790/article/details/85601590?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.nonecase)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

## Linux内核简介
### 内核结构
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/os-architecture.png?raw=true)

GNU C库，提供了链接内核的系统调用接口，还提供了在用户空间应用程序和内核之间进行转换的机制

linux内核分三层：

1. 系统调用接口
2. 独立于体系结构的内核代码
3. 依赖于体系结构的代码

内核子系统：

1. 进程管理
2. 内存管理
3. 虚拟文件系统
4. 网络栈
5. 设备驱动器

### 用户空间和内核空间
#### 地址空间划分
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/address-space.png?raw=true)

内核地址空间存放的是内核代码和数据

用户空间存放的是用户程序的代码和数据

Linux使用两级保护机制: 0级供内核使用，3级供用户使用

#### 地址映射模型
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/address-mapping.png?raw=true)

进程代码中的地址是逻辑地址，经过全局描述符表GDT映射为线性地址，经过TLB和PTE映射为物理地址

#### 进程执行状态

1. 用户态 进程执行用户程序代码

CPU在特权级别最低(3级)的用户代码中运行。当进程被中断程序中断时，中断处理程序使用当前进程的内核栈，进程可以象征性地称为处于进程的内核态

2. 内核态 进程执行系统调用陷入内核代码中执行

CPU在特权等级最高(0级)的内核代码中执行。当进程处于内核态时，执行的内核代码会使用当前进程的内核栈。每个进程都有自己的内核栈

### 上下文
#### CPU状态

1. 运行于用户空间，执行用户线程
2. 运行于内核空间，处理进程上下文(系统调用，陷阱)
3. 运行于内核空间，处于中断上下文(同步中断&异步中断)

#### 进程上下文
用户进程需要传递给内核的参数，以及内核需要保存的一整套变量和寄存器的值和当时的环境。进程的上下文可以分为三个部分：

1. 用户级上下文 代码，数据，用户堆栈以及共享存储区
2. 寄存器上下文 通用寄存器，程序寄存器IP，处理器状态寄存器EFLAGS，栈指针ESP
3. 系统级上下文 进程控制块task_struct，内存管理信息(mm_struct, vm_area_struct, pgd, pte)，内核栈

#### 中断上下文 
硬件通过触发信号，导致内核调用中断处理程序，进入内核空间。这个过程中，硬件的 一些变量和参数也要传递给内核，内核通过这些参数进行中断处理。

可以看作硬件传递过来的参数以及内核需要保存的一些其他环境(当前)

### 特性
#### 单内核微内核

1. 单内核

将内核整体上作为单独的大过程实现，运行在一个单独的地址空间

2. 微内核

划分为多个独立的过程，运行在各自的地址空间上，使用IPC通信

#### Linux内核特性

1. 支持动态加载内核模块
2. 支持对称多处理(SMP)，各处理器共享内存子系统以及总线结构
3. 可以抢占
4. 不区分线程进程

## 内核开发的准备
开发内核代码与开发用户代码的差异

1. 无标准C库
2. 必须使用GNU C

	内联函数 将性能要求高，代码长度短的函数定义为内联
	
	内联汇编 使用C语言和汇编混编，偏底层或执行时间严格，一般使用汇编
	
3. 没有内存保护，不分页
4. 内核栈小而固定，2页(64位2*16k)
5. 同步和并发

	内核支持异步中断，SMP和抢占，容易产生竞争条件

## 进程管理
### 进程结构
#### 进程任务队列

task list 任务队列 双向循环链表 节点类型是task struct

task struct 进程描述符 包含进程信息

#### 进程描述符
每个进程的thread info结构在它内核栈的尾端分配，结构中的task域存放的是指向task struct的指针

task struct包含的进程信息：打开的文件，进程地址空间，挂起的信号，进程的状态

task struct成员：

1. parent指针 指向父进程task struct
2. state 进程状态

#### 描述符存放

内核使用唯一的值pid标志每个进程

内核中大部分处理进程的代码都是直接通过task struct进行

通过current宏查找当前正在运行进程的task struct

#### 进程状态
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/process-state.png?raw=true)

#### 进程家族树
所有进程都是PID=1的init进程的后代

### 进程创建
分为两步：fork和exec

#### fork
fork拷贝当前进程创建子进程，或者共享地址空间，使用写时拷贝

子进程和父进程的区别在于，PID PPID和某些资源统计量(挂起的信号)

fork的实际开销：复制父进程页表，给子进程创建进程描述符

linux通过系统调用clone实现fork，clone(SIGCHLD, 0)

fork步骤：

1. 创建内核栈，thread info和task struct，值和当前进程完全相同
2. 清空或设置task struct中的一些值，比如state
3. 更新flags
3. 分配pid
4. 拷贝或共享：打开的文件信息，文件系统信息，信号处理函数，进程地址空间，命名空间等

#### vfork
vfork不拷贝页表项，子进程先运行，clone(CLONE_VFORK|CLONE_VM|SIGCHLD, 0)

#### exec
读取可执行文件并将其载入地址空间开始运行

### 线程在Linux中的实现
线程仅仅被视为一个与其他进程共享某个资源的进程

#### 创建线程
clone(CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND, 0)

#### 内核线程
内核线程没有独立的地址空间，它们只能在内核空间运行

### 进程终结
进程析构发生在显示或者隐式调用exit时

析构的重要动作：

1. 所有与进程相关联的资源被释放
2. 向父进程发送信号，给子进程找养父，进程状态设为EXIT_ZOMBIE状态
3. schedule切换到新的进程
4. 仍占用的内存：内核栈，thread info和task struct，目的是给父进程提供信息
5. 父进程检索到子进程信息后，剩余内存被释放

#### 孤儿进程
父进程在子进程前退出，给子进程在当前进程组找一个线程作为父亲，或者是init

init进程会调用wait，清除所有相关的僵死进程

## 进程调度
进程调度程序，可看作在可运行进程之间，分配有限的处理器时间资源的内核子系统

### 多任务
#### 抢占式多任务
由调度程序决定什么时候停止一个进程运行，以便其他进程得到执行机会

**抢占** 调度程序的强制挂起动作

#### 非抢占式多任务
除非进程自己主动停止运行，否则它会一直执行

**让步** 进程主动挂起自己

### 调度策略
#### IO密集和CPU密集型进程
**IO密集** 进程大部分时间提交IO请求或等待IO请求，IO指任何类型的可阻塞资源

**CPU密集** 进程大部分时间在执行代码

调度策略要兼顾：进程响应速度，最大系统利用率

Linux为保证交互式应用和桌面系统性能，更倾向于优先调度IO密集进程

#### 进程优先级
两种不同的优先级范围：

1. nice值 -20~19 优先级越来越低
2. 实时优先级 0~99 优先级越来越高

#### 时间片
预先设置好的，进程被抢占之前能够运行的时间

### 调度算法
Linux内核有两个调度类：CFS和实时调度类

#### 公平调度
CFS不分配时间片给进程

CFS将nice值作为进程获得处理器运行比的权重

基于所有可运行进程的计算进程运行多久

允许每个进程运行一段时间，循环轮转，选择运行最少的进程调度

```
下面举个直观的例子来说明：
假设系统中只有3个进程ProcessA(NI=+10)，ProcessB(NI=0)，ProcessC(NI=-10)，NI表示进程的nice值，时间片=10ms
1) 调度前，把进程优先级按一定的权重映射成时间片(这里假设优先级高一级相当于多5msCPU时间)。
    假设ProcessA分配了一个时间片10ms，那么ProcessB的优先级比ProcessA高10(nice值越小优先级越高)，ProcessB应该分配10*5+10=60ms，以此类推，ProcessC分配20*5+10=110ms
2) 开始调度时，优先调度分配CPU时间多的进程。由于ProcessA(10ms),ProcessB(60ms),ProcessC(110ms)。显然先调度ProcessC
3) 10ms(一个时间片)后，再次调度时，ProcessA(10ms),ProcessB(60ms),ProcessC(100ms)。ProcessC刚运行了10ms，所以变成100ms。此时仍然先调度ProcessC
4) 再调度4次后(4个时间片)，ProcessA(10ms),ProcessB(60ms),ProcessC(60ms)。此时ProcessB和ProcessC的CPU时间一样，这时得看ProcessB和ProcessC谁在CPU运行队列的前面，假设ProcessB在前面，则调度ProcessB
5) 10ms(一个时间片)后，ProcessA(10ms),ProcessB(50ms),ProcessC(60ms)。再次调度ProcessC
6) ProcessB和ProcessC交替运行，直至ProcessA(10ms),ProcessB(10ms),ProcessC(10ms)。
    这时得看ProcessA，ProcessB，ProcessC谁在CPU运行队列的前面就先调度谁。这里假设调度ProcessA
7) 10ms(一个时间片)后，ProcessA(时间片用完后退出),ProcessB(10ms),ProcessC(10ms)。
8) 再过2个时间片，ProcessB和ProcessC也运行完退出。

这个例子很简单，主要是为了说明调度的原理，实际的调度算法虽然不会这么简单，但是基本的实现原理也是类似的：
1）确定每个进程能占用多少CPU时间(这里确定CPU时间的算法有很多，根据不同的需求会不一样)
2）占用CPU时间多的先运行
3）运行完后，扣除运行进程的CPU时间，再回到 1）
```

#### 实时调度
linux提供两种实时调度策略：SCHED_FIFO和SCHED_RR，比普通非实时的调度策略SCHED_NORMAL具有更高优先级

SCHED_FIFO 不使用时间片，处于可执行会一直执行，直到阻塞或显示释放

SCHED_RR 带时间片的SCHED_FIFO

### 调度实现
#### 时间记账

CFS没有时间片的概念，需要维护进程运行的时间记账sched entity，嵌入在task struct中

sched entity中的vruntime，存放进程的虚拟运行时间，vruntime并不是实际的运行时间，它是实际运行时间进行加权运算后的结果

#### 算法步骤

1. 计算每个进程的vruntime，通过update_curr()函数更新进程的vruntime。
2. 选择具有最小vruntime的进程投入运行。
3. 进程运行完后，更新进程的vruntime，转入步骤2)

#### 进程管理
可运行队列 使用红黑树，存放可执行进程，迅速找到最小的vruntime值的进程

等待队列 简单链表，存放休眠(被阻塞)进程

### 抢占和上下文切换
**上下文切换**，即从一个可执行进程切换到另一个可执行进程，每个进程包含一个need resched标志

#### 用户抢占
内核即将返回用户空间，如果need resched被设置，会导致schedule被调用，此时会发生用户抢占

用户抢占在以下情况产生：

1. 系统调用返回用户空间
2. 从中断处理程序返回用户空间

#### 内核抢占
在一个内核态运行的进程，可能在执行内核函数期间被另一个进程取代

内核抢占的满足条件：

1. 没持有锁(preempt_count)
2. 内核代码可重入(因为内核支持SMP)

内核抢占在以下情况发生：

1. 中断处理正在执行，且返回内核空间之前，隐式调用schedule
2. 内核代码再一次具有抢占性，如解锁等
3. 内核任务显示调用schedule
4. 内核任务阻塞，导致需要调用schedule

## 系统调用
系统调用使用软中断实现：通过引发一个异常来促使系统切换到内核态执行系统调用处理程序

### 意义
系统调用的存在，有以下重要的意义:

1）用户程序通过系统调用来使用硬件，而不用关心具体的硬件设备，这样大大简化了用户程序的开发。

	比如：用户程序通过write()系统调用就可以将数据写入文件，而不必关心文件是在磁盘上还是软盘上，或者其他存储上。

2）系统调用使得用户程序有更好的可移植性。

    只要操作系统提供的系统调用接口相同，用户程序就可在不用修改的情况下，从一个系统迁移到另一个操作系统。

3）系统调用使得内核能更好的管理用户程序，增强了系统的稳定性。

    因为系统调用是内核实现的，内核通过系统调用来控制开放什么功能及什么权限给用户程序。

    这样可以避免用户程序不正确的使用硬件设备，从而破坏了其他程序。

4）系统调用有效的分离了用户程序和内核的开发。

    用户程序只需关心系统调用API，通过这些API来开发自己的应用，不用关心API的具体实现。

    内核则只要关心系统调用API的实现，而不必管它们是被如何调用的。

## 内核数据结构

## 中断和中断处理

## 中断下半部

## 内核同步介绍

## 内核同步方法

## 定时器和时间管理

## 内存管理
### 内存管理结构
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/uma.png?raw=true)

现代计算机的内存组织形式包括UMA和NUMA，其中UMA中所有cpu访问内存的速度都一样快，而在NUMA系统中，每个cpu访问不同内存的速度并不一样。通常，NUMA计算机每个cpu都有自己的本地内存，各个cpu不仅可以访问自己的本地内存，也可以访问其它cpu的内存，但是访问本地内存的速度最快，访问其它cpu内存的速度依其与本cpu的距离增加而依次减慢。

#### 内存管理单位
Linux对内存的管理划分成三个层次，分别是Node、Zone、Page。对这三个层次简介如下：

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/mem-tree.png?raw=true)

层次|说明|
---|---|
Node(节点)|CPU被划分成多个节点，每个节点都有自己的一块内存，可以参考NUMA架构有关节点的介绍|
Zone(区)|每一个Node（节点）中的内存被划分成多个管理区域（Zone），用于表示不同范围的内存|
Page(页)|每一个管理区又进一步被划分为多个页面，页面是内存管理中最基础的分配单位|

#### 内核地址空间
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/kernel-space-physical.png?raw=true)
内核把内核地址空间划分为不同的区：

| 区  | 描述  | 物理内存 |
|------------ |------------|--------|
| ZONE_DMA     | DMA使用的页   | <16MB    |
| ZONE_NORMAL  | 正常可寻址的页 | 16~896MB |
| ZONE_HIGHMEM | 动态映射的页  | >896MB   |

#### 内核内存映射
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/kernel-space-virtual.png?raw=true)

* 低端内存

包含ZONE_DMA和ZONE_NORMAL，

物理地址 = 逻辑地址 – 0xC0000000

* 高端内存

高端内存包含ZONE_HIGHMEM，在x86体系结构中，这个区的内存不能映射到内核地址空间上，也就是没有逻辑地址

### 获取高端内存
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/high-mem.png?raw=true)
#### 临时映射(FIXADDR_START~FIXADDR_TOP) 

内核在FIXADDR_START到FIXADDR_TOP之间保留了一些线性空间用于特殊需求，称为固定映射空间

在这个空间中，有一部分用于高端内存的临时映射，这块空间由如下特点：

1. 每个CPU占用一块空间
2. 每个CPU占用的空间，又分为多个小空间，每个小空间是1个page，用于一个目的

当要进行一次临时映射的时候，需要指定映射的目的，根据映射目的，可以找到对应的小空间，然后把这个空间的地址作为映射地址。这意味着一次临时映射会导致以前的映射被覆盖。通过 kmap_atomic() 可实现临时映射。

#### 永久映射(PKMAP_BASE~FIXADDR_START)

通过kmap，建立永久映射

#### 动态映射(VMALLOC_START~VMALLOC_END)

内核通过调用vmalloc这个区域获得内存

### 内存分配
#### 页分配
alloc_pages 分配连续物理页，返回一个指针，指向第一个页的page结构体

#### 字节分配
kmalloc 从低端内存分配；分配的物理地址连续；一般用于分配小块内存(一般不超过128k)；kmalloc分配方式基于slab

vmalloc 一般为高端内存，当内存不够才分配低端内存；分配的物理地址不连续；一般分配大内存(伙伴算法)

#### slab分配
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/slab.png?raw=true)
主要功能是作为一个通用数据结构缓存层，存储内核中经常分配并释放的对象

slab层把不同的对象划分为高速缓存组，每种对象类型对应一个高速缓存

高速缓存被划分为slab，slab由一个或多个物理上连续的页组成

每个高速缓存有三个slab链表，full，partial，empty

### 内核分配内存

#### 内核栈分配

[TODO](https://blog.csdn.net/yangkuanqaz85988/article/details/52403726)

#### 进程栈

#### 线程栈

#### 内核栈

#### 中断栈


#### 按CPU数据分配

## 虚拟文件系统
### 文件系统抽象层
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/vfs-architecture.png?raw=true)
虚拟文件系统(VFS)是linux内核和存储设备之间的抽象层，主要有以下好处。

1. 简化了应用程序的开发：应用通过统一的系统调用访问各种存储介质
2. 简化了新文件系统加入内核的过程：新文件系统只要实现VFS的各个接口即可，不需要修改内核部分

### UNIX文件
UNIX文件系统的一些概念

1. 安装点 文件系统被安装在安装点上，在全局层次结构中被称为命名空间
2. 文件 文件数据
3. 索引节点 文件元数据
4. 目录项 VFS把目录项当做文件一样看待
5. 超级块 存放文件系统信息

### VFS对象
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/vfs-obj.png?raw=true)
VFS主要有4个对象类型：

1. 超级块对象 代表一个具体的已安装文件系统
2. 索引节点对象 代表一个具体文件
3. 目录项对象 代表一个目录项，是路径组成部分
4. 文件对象 代表进程打开的文件

#### 超级块对象
主要存储文件系统相关的信息，包括inode与block的总量、使用量、剩余量等

通常对应存储在磁盘的特定扇区中的文件系统超级块或文件系统控制块，但是对于那些基于内存的文件系统(比如proc,sysfs)，超级块是在使用时创建在内存中的

#### 索引节点对象
一个索引节点代表文件系统中的一个文件，它可以是设备或管道这样的特殊文件

索引节点仅当被文件访问时，才在内存中创建

#### 目录项对象
目录项没有没有对应磁盘数据结构，VFS根据字符串形式的路径名现场创建它，由于目录项对象并非真正保存在磁盘上，所以目录项结构体没有是否被修改的标志

* 目录项状态
三种有效状态：

* 目录项缓存

#### 文件对象


![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/vfs-obj-mapping.png?raw=true)

### 文件系统结构

## 块I/O层

## 进程地址空间

## 页高速缓存和页回写

## 设备与模块

## 调试

## 可移植性

## 补丁、开发和社区