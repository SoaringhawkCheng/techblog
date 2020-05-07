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
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

## Linux内核简介

CPU状态

```
运行于用户空间，执行用户线程
运行于内核空间，处理进程上下文
运行于内核空间，处于中断上下文
```

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