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

### 用户空间和内核空间

* 地址空间

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/address-space.png?raw=true)

内核地址空间存放的是内核代码和数据

用户空间存放的是用户程序的代码和数据

Linux使用两级保护机制: 0级供内核使用，3级供用户使用

* 操作系统内部结构

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-linux-kernel/os-architecture.png?raw=true)

* 进程执行状态

1. 用户态 进程执行用户程序代码

CPU在特权级别最低(3级)的用户代码中运行。当进程被中断程序中断时，中断处理程序使用当前进程的内核栈，进程可以象征性地称为处于进程的内核态

2. 内核态 进程执行系统调用陷入内核代码中执行

CPU在特权等级最高(0级)的内核代码中执行。当进程处于内核态时，执行的内核代码会使用当前进程的内核栈。每个进程都有自己的内核栈

### 上下文

* CPU状态

1. 运行于用户空间，执行用户线程
2. 运行于内核空间，处理进程上下文(系统调用，陷阱)
3. 运行于内核空间，处于中断上下文(同步中断&异步中断)

* 进程上下文

用户进程需要传递给内核的参数，以及内核需要保存的一整套变量和寄存器的值和当时的环境。进程的上下文可以分为三个部分：

1. 用户级上下文 代码，数据，用户堆栈以及共享存储区
2. 寄存器上下文 通用寄存器，程序寄存器IP，处理器状态寄存器EFLAGS，栈指针ESP
3. 系统级上下文 进程控制块task_struct，内存管理信息(mm_struct, vm_area_struct, pgd, pte)，内核栈

* 中断上下文 

硬件通过触发信号，导致内核调用中断处理程序，进入内核空间。这个过程中，硬件的 一些变量和参数也要传递给内核，内核通过这些参数进行中断处理。

可以看作硬件传递过来的参数以及内核需要保存的一些其他环境(当前)

### 特性

* 单内核微内核

1. 单内核

将内核整体上作为单独的大过程实现，运行在一个单独的地址空间

2. 微内核

划分为多个独立的过程，运行在各自的地址空间上，使用IPC通信

* Linux内核特性

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

* 进程任务队列

task list 任务队列 双向循环链表 节点类型是task struct

task struct 进程描述符 包含进程信息

* 进程描述符



### 进程创建

### 线程在Linux中的实现

### 进程终结

## 进程调度

## 系统调用

## 内核数据结构

## 中断和中断处理

## 中断下半部

## 内核同步介绍

## 内核同步方法

## 定时器和时间管理

## 内存管理

## 虚拟文件系统

## 块I/O层

## 进程地址空间

## 页高速缓存和页回写

## 设备与模块

## 调试

## 可移植性

## 补丁、开发和社区