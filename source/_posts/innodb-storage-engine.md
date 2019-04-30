---
title: 「MySQL技术内幕·InnoDB存储引擎」学习笔记
catalog: true
date: 2019-04-15 21:09:15
subtitle: 
header-img:
tags:
- sql
categories:
- 编程
---
> 书籍豆瓣链接：[《MySQL技术内幕·InnoDB存储引擎》](https://book.douban.com/subject/24708143/)
> 
> 开始学习时间：4-15
> 
> 预计完成时间：5-4
> 
> 实际完成时间：


# 体系结构

## 数据处理系统

数据处理分成两类：联机事务处理OLTP(Online Transaction Processing)、联机分析处理OLAP(Online Analytical Processing)。

### 联机事务处理

### 联机分析处理

## SQL语言分类

SQL语言分四类：数据查询语言DQL，数据操纵语言DML，数据定义语言DDL，数据控制语言DCL

# 存储引擎

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/innodb-storage-engine/1.png?raw=true)

# 文件

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/innodb-storage-engine/2.png?raw=true)

# 表

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/innodb-storage-engine/3.png?raw=true)

# 事务

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/innodb-storage-engine/4.png?raw=true)

# 索引与算法

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/innodb-storage-engine/5.png?raw=true)

# 锁

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/innodb-storage-engine/6.png?raw=true)