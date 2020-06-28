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

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/zero-copy-context-0.png?raw=true)

上下文切换是CPU密集型的工作，数据拷贝是I/O密集型的工作。如果一次简单的传输就要像上面这样复杂的话，效率是相当低下的。零拷贝机制的终极目标，就是消除冗余的上下文切换和数据拷贝，提高效率

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/zero-context-1.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/zero-copy-1.png?raw=true)

可见，不仅拷贝的次数变成了3次，上下文切换的次数也减少到了2次，效率比传统方式高了很多。但是它还并非完美状态，下面看一看让它变得更优化的方法

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/zero-context-2.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/zero-copy-2.png?raw=true)

在“基础”零拷贝方式的时序图中，有一个“write data to target socket buffer”的回环，在框图中也有一个从“Read buffer”到“Socket buffer”的大箭头。这是因为在一般的Block DMA方式中，源物理地址和目标物理地址都得是连续的，所以一次只能传输物理上连续的一块数据，每传输一个块发起一次中断，直到传输完成，所以必须要在两个缓冲区之间拷贝数据。

而Scatter/Gather DMA方式则不同，会预先维护一个物理上不连续的块描述符的链表，描述符中包含有数据的起始地址和长度。传输时只需要遍历链表，按序传输数据，全部完成后发起一次中断即可，效率比Block DMA要高。也就是说，硬件可以通过Scatter/Gather DMA直接从内核缓冲区中取得全部数据，不需要再从内核缓冲区向Socket缓冲区拷贝数据。

## 高并发

## Raft算法

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-log-index.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-replicated-state-machine.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-state-machine-safety.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-state-transfer.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-term.png?raw=true)

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-software-architecture/raft-two-disjoint-majorities.png?raw=true)