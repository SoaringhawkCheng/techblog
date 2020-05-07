---
title: 「MySQL技术内幕·InnoDB存储引擎」学习笔记[DOING]
catalog: true
date: 2019-04-15 21:09:15
subtitle: 
header-img:
tags:
- database
categories:
- 工程
---
> 书籍豆瓣链接：[《MySQL技术内幕·InnoDB存储引擎》](https://book.douban.com/subject/24708143/)
> 
> 开始学习时间：6-6
> 
> 预计完成时间：6-
> 
> 实际完成时间：


# 体系结构

## 基础概念

* **数据库** 物理操作系统文件或其他形式文件类型的集合，是依照某种数据模型组织起来并存放于二级存储器的数据集合

* **实例** MySQL数据库由后台线程以及一个共享内存区组成。共享内存可以被运行的后台线程所共享，数据库实例才是真正用于操作数据库文件的。Mysql单进程多线程架构，实例就是一个进程。

## 存储引擎

Mysql数据区别于其他数据库的最重要的一个特点就是其插件式的表存储引擎。存储引擎是基于表的，而不是数据库的。

数据处理系统分成两类：

* **联机事务处理OLTP**(Online Transaction Processing)，传统型关系数据库的主要应用，作为数据管理的主要手段，主要用作操作处理。
* **联机分析处理OLAP**(Online Analytical Processing)，数据仓库的最主要的应用，支持复杂的分析操作，侧重决策支持。

## SQL语言分类

SQL语言分四类：数据查询语言DQL，数据操纵语言DML，数据定义语言DDL，数据控制语言DCL

## 数据库连接

连接MySQL操作是一个连接进程和MySQL数据库实例进行通信

# 存储引擎

## InnoDB体系架构

* innodb内存池

主要作用有：

	1. 维护所有进程/线程需要访问的多个内部数据结构
	2. 缓存磁盘上的数据用于快速读取，缓存改动用于修改
	3. 重做日志缓冲

* 后台线程

主要作用是负责刷新内存池中的数据，保证缓冲池中的内存缓存的是最近的数据

### 后台进程

* master线程

核心后台线程，负责将缓冲池中的数据异步刷新到磁盘

* io线程

innodb大量使用异步io来处理写io请求，io线程的作用主要是负责io请求的回调

* purge线程

事务提交后，undolog可能不再需要，需要purge线程回收已经使用并分配的undo页

* Page Cleaner线程

innodb1.2引入，将之前版本的脏页刷新操作放入到单独的线程中完成，目的是为了减轻master线程的工作和对用户查询线程的阻塞


### 内存

#### 缓冲池

由于CPU与磁盘速度之间的鸿沟，基于磁盘的数据库系统使用缓冲池技术来提高数据的整体性能

缓冲池中的数据类型有：

	索引页
	数据页
	undo页
	插入缓冲
	自适应哈希索引
	锁信息
	数据字典信息
	
#### 内存管理

* LRU list

如果新读取到的页被放在LRU列表的首部，那么非常可能将热点数据页从LRU列表中移除，下次读取时需要再次访问磁盘

解决方案

1. **innodb\_old\_blocks\_pct** 数据库的缓冲池通过LRU算法进行管理，加入了midpoint位置，新读取到的页并不直接放入到LRU列表的首部，而是放入到LRU列表的midpoint位置，由innodb_old_blocks_pct(通常3/8)控制

2. **innodb\_old\_blocks\_time** 用于表示页读取到mid位置后需要等待多久才能加入到LRU列表的热端，通过这个方法尽可能使LRU列表中热点数据不被刷出。
	
	**page made young**: 页从LRU列表的old部分加入到new部分
	
	**page not made young**: 因为block time的设置没有从old部分移到new部分

* free list

空闲页列表

* flush list

脏页列表

#### 重做日志缓冲

首先将重做日志信息放入到缓冲区，然后按一定的频率将其刷新到外部磁盘的重做日志文件

### Checkpoint技术

### MasterThread工作方式

### 关键特性

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/innodb-storage-engine/1.png?raw=true)

# 文件

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/innodb-storage-engine/2.png?raw=true)

# 表

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/innodb-storage-engine/3.png?raw=true)

# 事务

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/innodb-storage-engine/4.png?raw=true)

# 索引与算法

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/innodb-storage-engine/5.png?raw=true)

B+树索引并不能

# 锁

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/innodb-storage-engine/6.png?raw=true)