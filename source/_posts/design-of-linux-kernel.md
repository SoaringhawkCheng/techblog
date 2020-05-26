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
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/mmu.png?raw=true)

IOMMU，IO管理单元，作用是连接DMA-capable I/O总线（Direct Memory Access-capable I/O Bus）和主存（main memory），IOMMU将设备访问的虚拟地址转化为物理地址

DMA(direct memory access)，直接存储器访问，传输将数据从一个地址空间复制到另外一个地址空间。当CPU初始化这个传输动作，传输动作本身是由DMA控制器来实行和完成

MMU，内存管理单元，是负责处理CPU的内存访问请求的计算机请求，功能包括虚拟地址到物理地址转换

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

task_list 任务队列 双向循环链表 节点类型是task_struct

task_struct 进程描述符 包含进程信息

#### 进程描述符
每个进程的thread_info结构在它内核栈的尾端分配，结构中的task域存放的是指向task struct的指针

task_struct包含的进程信息：打开的文件，进程地址空间，挂起的信号，进程的状态

task struct成员：

1. parent指针 指向父进程task_struct
2. state 进程状态

#### 描述符存放

内核使用唯一的值pid标志每个进程

内核中大部分处理进程的代码都是直接通过task_struct进行

通过current宏查找当前正在运行进程的task_struct

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

1. 创建内核栈，thread_info和task_struct，值和当前进程完全相同
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
4. 仍占用的内存：内核栈，thread_info和task_struct，目的是给父进程提供信息
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

CFS没有时间片的概念，需要维护进程运行的时间记账sched_entity，嵌入在task_struct中

sched_entity中的vruntime，存放进程的虚拟运行时间，vruntime并不是实际的运行时间，它是实际运行时间进行加权运算后的结果

#### 算法步骤

1. 计算每个进程的vruntime，通过update_curr()函数更新进程的vruntime。
2. 选择具有最小vruntime的进程投入运行。
3. 进程运行完后，更新进程的vruntime，转入步骤2)

#### 进程管理
可运行队列 使用红黑树，存放可执行进程，迅速找到最小的vruntime值的进程

等待队列 简单链表，存放休眠(被阻塞)进程

### 抢占和上下文切换
**上下文切换**即从一个可执行进程切换到另一个可执行进程，每个进程包含一个need_resched标志

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
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/kernel-stack.png?raw=true)
每个进程都有1-2页固定大小的内核栈

#### 中断栈
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/interrupt-stack.png?raw=true)
中断栈为每个进程提供了一个用于中断处理程序的栈，中断处理程序不用再和被中断进程共享一个内核栈，对每个进程仅仅耗费了一页

#### 按CPU数据分配
略

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

### VFS对象和数据结构
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/vfs-obj.png?raw=true)
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/vfs-obj-mapping.png?raw=true)
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
目录项没有没有对应磁盘数据结构，VFS根据字符串形式的路径名现场创建它，由于目录项对象并非真正保存在磁盘上，所以目录项结构体没有是否被修改的标志。路径/bin/vi中/,bin和vi都是目录项，前两个是目录，最后一个是普通文件

目录项状态

1. 被使用：对应一个有效的索引节点，并且该对象由一个或多个使用者
2. 未使用：对应一个有效的索引节点，但是VFS当前并没有使用这个目录项
3. 负状态：没有对应的有效索引节点（可能索引节点被删除或者路径不存在了）

目录项缓存

1. 已使用目录链表
2. 最近被使用目录双向链表，用于缓存
3. 散列表，快速定位

#### 文件对象
只存在于内存中，是已打开文件在内存中的表示，反过来指向目录项

#### 文件系统相关数据结构

file_system_type 描述文件系统类型

vfsmount 描述一个安装文件系统的实例

每种文件系统，不管有多少个实例安装到系统中，都只有一个file_system_type

#### 进程相关数据结构

file_struct 进程描述符中的files目录项指向，打开的文件及文件描述符

fs_struct 进程描述符中的fs域指向，文件系统和进程相关的信息

namespace 命名空间，所有进程共享同样的命名空间，只有在进行clone()操作时使用CLONE_NEWS标志，才会给进程一个唯一的命名空间结构体拷贝

### 文件系统结构
#### 磁盘结构
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/fs-structure.png?raw=true)
自举块(boot block)：占用1 block；主要存储分区的操作系统类型、分区起始地址；pc联盟规定的大小，和各个文件系统无关；每个分区只有一个；

超级块：占用1 block；存储分区操作系统版本、块大小、文件系统；

块组描述表(GDT)：占用3 block；存储一个块组的描述信息；

块位图(block bitmap)：占用1 block;每个比特位表示一个数据块是否被标记使用；1：已使用，0：未使用；

inode 位图(inode bitmap)：占用1 block;每一个比特位表示一个inode节点。1：已使用，0：未使用；

inode节点：占用128B；分为文件属性部分和数据指针部分；数据指针占用60B，长度为15的指针数组；每个数据块的大小是1 block，那么15个指针最多只能维护15*1K个文件大小，显然，文件大小的上限太小了

数据块：存储文件的内容；

#### 三级间接寻址
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/3-level-inode.png?raw=true)

文件系统通过三级间接寻址的方式来提高文件大小的上限。块大小若以1 block计算，文件大小的上限为16G，若以4 block为块大小，那文件大小的上限超过了1T；ext4文件系统中，数据指针有256B，那文件大小上限就非常大了；

#### 数据块存储内容
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/data-block-structure.png?raw=true)

数据块中存储的是一条一条的记录项，每条记录项都由文件名、inode编号、记录长度(该记录项首地址到下一条记录项的首地址的长度)。每一个记录项就是该目录下“ls -a”的结果

#### 普通文件结构
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/file-structure.png?raw=true)

#### 目录文件结构
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/entry-structure.png?raw=true)

## 块I/O层

### 块设备简介

I/O设备主要有2类:

1. 字符设备 只能顺序读写设备中的内容，比如 串口设备，键盘
2. 块设备 能够随机读写设备中的内容，比如 硬盘，U盘

#### 扇区

物理上的最小寻址单元

**原子性问题**操作系统与磁盘的数据交换单位是扇区。一次操作，是写一个扇区大小(512b)的数据。如果写入数据小于一个扇区，只需要无缓冲的写入到磁盘。磁盘特性保证，这种情况可以原子地写入磁盘

#### 块

逻辑上的最小寻址单元，内核执行的所有磁盘操作都是按照块进行的

块的大小一般是扇区整数倍，并且小于等于页的大小

### 内核访问块设备方法
内核通过文件系统访问块设备时，需要先把块读入到内存中。所以文件系统为了管理块设备，必须管理块和内存页之间的映射。

内核中有2种方法来管理块和内存页之间的映射。

1. 缓冲区和缓冲区头
2. bio

#### 缓冲区(2.4.10不存在)
每个块都是一个缓冲区，同时对每个块都定义一个缓冲区头来描述它。

由于块的大小是小于内存页的大小的，所以每个内存页会包含一个或者多个块

在2.6之前的内核中，主要就是通过缓冲区头来管理块和内存之间的映射的。

用缓冲区头来管理内核的 I/O 操作主要存在以下2个弊端，所以在2.6开始的内核中，缓冲区头的作用大大降低了

缓冲区头管理

1. 对内核而言，操作内存页是最为简便和高效的。而且每个块对应一个缓冲区头，导致内存的利用率降低
2. 每个缓冲区头只能表示一个块，所以内核在处理大数据时，会分解为对一个个小的块的操作，造成不必要的负担和空间浪费

#### bio
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/bio.png?raw=true)

目前，块IO操作的基本容器由bio结构体表示

bio结构体的出现就是为了改善上面缓冲区头的2个弊端，它表示了一次 I/O 操作所涉及到的所有内存页。

bio相当于在缓冲区上又封装了一层，使得内核在 I/O操作时只要针对一个或多个内存页即可，不用再去管理磁盘块的部分。

使用bio结构体还有以下好处：

1. bio结构体很容易处理高端内存，因为它处理的是内存页而不是直接指针
2. bio结构体既可以代表普通页I/O，也可以代表直接I/O(指不通过**页高速缓存**的IO操作)
3. bio结构体便于执行分散-集中（矢量化的）块I/O操作，操作中的数据可以取自多个物理页面

### IO调度
块设备将它们挂起的快IO请求保存在请求队列reques_queue中，队列中的reques结构体包含多个bio

## 进程地址空间
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/process-space.jpg?raw=true)

### 内存描述符
内核使用内存描述符结构体mm_struct表示进程的地址空间

进程描述符task_struct结构体中的mm域存放着该进程使用的内存描述符

#### 分配内存描述符
子进程使用allocate_mm从slab缓存中分配得到mm_struct

线程创建，调用clone设置CLONE_VM标志，会共享mm_struct

#### 撤销内存描述符
mm_struct中的mm_users计数降到零，将调用mmdrop函数

#### 内核线程
内核线程没有进程地址空间，不需要自己的mm_struct

但是内核仍然需要页表，来访问内核空间(高端内存)

故内核的mm为NULL，active_mm为前一个进程的mm_struct

### 虚拟内存区域
vm_area_struct描述了指定地址空间内连续区间上的一个独立内存范围，比如堆和栈就是一个内存区域

内存描述符的mmap域，使用单链表连接所有的内存区域对象vm_area_struct，内存描述符的mm_rb使用红黑树组织所有vm_area_struct

链表适合遍历，红黑树适合用于查找定位

### 内存映射
内核使用do_mmap创建一个新的线性区间

匿名映射

文件映射 指定了文件名和偏移量

### 地址空间与页表
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/pte.png?raw=true)

地址空间中的地址都是虚拟内存中的地址，而CPU需要操作的是物理内存，所以需要一个将虚拟地址映射到物理地址的机制。

这个机制就是页表，linux中使用3级页面来完成虚拟地址到物理地址的转换。

1. PGD - 全局页目录，包含一个 pgd_t 类型数组，多数体系结构中 pgd_t 类型就是一个无符号长整型
2. PMD - 中间页目录，它是个 pmd_t 类型数组
3. PTE - 简称页表，包含一个 pte_t 类型的页表项，该页表项指向物理页面

**TLB块表**是一个缓存虚拟地址到物理地址映射的硬件

## 页高速缓存和页回写
### 缓存简介
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/read-disk.png?raw=true)

内核开始一个读操作，检查需要的数据是否在页高速缓存中

1. 缓存未命中，调度块IO操作从磁盘读取，内核将读来的数据放入页缓存中，以满足用户态进程的读请求
2. 缓存命中，则放弃访问磁盘，直接从内存中读取

注意区分页高速缓存和用户态进程的缓冲区

#### 写缓存策略

不缓存(nowrite) 也就是不缓存写操作，当对缓存中的数据进行写操作时，直接写入磁盘，同时使此数据的缓存失效

写透缓存(write-through) 写数据时同时更新磁盘和缓存

回写(copy-write or write-behind) 写数据时直接写到缓存，由另外的进程(回写进程)在合适的时候将数据同步到磁盘

#### 缓存回收策略
略

### 缓存数据结构
#### 页高速缓存，块和页描述符的关系
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/page-block.png?raw=true)
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/page-cache-bio.jpg?raw=true)
#### address_space对象
address_space结构体是虚拟内存区域vm_area_struct的物理对等体

address_space和vm_area_struct的关系是一对多的关系

host域指向所属inode

page_tree域是一个包含所有page的radix树

指定文件偏移量去page_tree中搜，如果不存在，分配一个page，用于从用户空间拷贝数据，或者从磁盘读入数据

### 页回写


## 设备与模块

## 调试

## 可移植性

## 补丁、开发和社区