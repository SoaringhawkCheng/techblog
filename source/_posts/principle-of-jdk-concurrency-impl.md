---
title: 「Java并发实现原理：JDK源码剖析」学习笔记
catalog: true
date: 2020-12-23 15:40:23
subtitle:
header-img:
tags:
- language
categories:
- 工程
---
> 豆瓣书籍链接：
> [《Java并发实现原理：JDK源码剖析》](https://book.douban.com/subject/35013531/)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

## 多线程基础

cpu 缓存一致性协议

重排序

## 

程序次序规则

单线程happen-before原则：在同一个线程中，书写在前面的操作happen-before后面的操作。

锁的happen-before原则：同一个锁的unlock操作happen-before此锁的lock操作。

volatile的happen-before原则：对一个volatile变量的写操作happen-before对此变量的任意操作(当然也包括写操作了)。

happen-before的传递性原则：如果A操作 happen-before B操作，B操作happen-before C操作，那么A操作happen-before C操作。

线程启动的happen-before原则：同一个线程的start方法happen-before此线程的其它方法。

线程中断的happen-before原则：对线程interrupt方法的调用happen-before被中断线程的检测到中断发送的代码。

线程终结的happen-before原则：线程中的所有操作都happen-before线程的终止检测。

对象创建的happen-before原则：一个对象的初始化完成先于他的finalize方法调用。
