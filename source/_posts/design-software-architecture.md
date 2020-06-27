---
title: 「软件架构设计」学习笔记[TODO]
catalog: true
date: 2020-04-27 21:06:15
subtitle:
header-img:
tags:
- architecture
categories:
- 工程
---
> 书籍豆瓣链接：[《软件架构设计》](https://book.douban.com/subject/30443578/)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

[分布式共识博客](http://blog.kongfy.com/2016/05/%E5%88%86%E5%B8%83%E5%BC%8F%E5%85%B1%E8%AF%86consensus%EF%BC%9Aviewstamped%E3%80%81raft%E5%8F%8Apaxos/)

[零拷贝](https://www.jianshu.com/p/193cae9cbf07)
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/%E5%88%86%E5%B8%83%E5%BC%8F.jpg?raw=true)

## 第4章 操作系统

### 零拷贝

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/zero-copy-0.png?raw=true)

上下文切换是CPU密集型的工作，数据拷贝是I/O密集型的工作。如果一次简单的传输就要像上面这样复杂的话，效率是相当低下的。零拷贝机制的终极目标，就是消除冗余的上下文切换和数据拷贝，提高效率。

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/zero-copy-1.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/zero-copy-2.png?raw=true)

## 高并发

## Raft算法

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-log-index.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-replicated-state-machine.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-state-machine-safety.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-state-transfer.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-term.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-two-disjoint-majorities.png?raw=true)