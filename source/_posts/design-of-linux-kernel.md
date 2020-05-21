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

CPU状态

```
运行于用户空间，执行用户线程
运行于内核空间，处理进程上下文(系统调用，陷阱)
运行于内核空间，处于中断上下文(同步中断&异步中断)
```

上下文

```
进程上下文 用户进程需要传递给内核的参数，以及内核需要保存的一整套变量和寄存器的值和当时的环境

中断上下文 可以看作硬件传递过来的参数以及内核需要保存的一些其他环境(当前)

## 内核开发的准备

```
没有内存保护，不分页
内核栈小而固定
```

## 进程管理

进程描述符

```
task list 任务队列 双向循环链表
task struct 链表结点 包含进程信息
```

## 进程调度

```

```

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