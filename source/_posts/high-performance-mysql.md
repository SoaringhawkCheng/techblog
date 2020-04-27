---
title: 「高性能MYSQL」学习笔记[DOING]
catalog: true
date: 2019-04-05 14:38:09
subtitle: 
tags: 
- database
categories:
- 工程
---
> 书籍豆瓣链接：[《高性能MySQL》](https://book.douban.com/subject/23008813/)
> 
> 开始学习时间：4-5
> 
> 预计完成时间：5-4
> 
> 实际完成时间：

# MySQL架构
![MySQL三层架构](https://github.com/SoaringhawkCheng/blog/blob/gh-pages/%E7%BC%96%E7%A8%8B/high-performance-mysql/1.png?raw=true)
## 服务层

负责链接管理、授权认证、安全。维护一个连接池，使用用户名和密码或SSL进行认证。

## 查询优化

负责解析查询（编译SQL），并进行优化。对于SELECT语句，先查询缓存。存储过程、触发器、视图都在这一层。

## 存储层

负责存储数据、提取数据、开启一个事务等等。存储引擎通过api与上层进行通信，api屏蔽了不同存储引擎之间的差异。

# 隔离性

针对隔离性遇到的问题如下：

1. 脏读：读到了别的事务回滚前的脏数据

2. 不可重复读：事务A前后两次读取数据不一致(事务B改变了数据，update和delete)

3. 幻读：事务A首先根据索引得到N条数据，再次搜索发现有N+M条数据了(事务B进行了insert

4. 丢失更新：同时更新数据，一方数据被覆盖

# 隔离级别

1. read uncommitted(未提交读)

解决了丢失更新问题

读取时是**不加锁**的；但在更新数据时加**行共享锁**

2. read committed(提交读)

解决了脏读的问题

给读的数据加**行级共享锁**，**读完就释放**；写数据加**行级排他锁**，**事务结束释放**

3. repeatable read(可重复读)

解决了不可重复读问题

给读的数据加行级共享锁，**事务结束释放**；写数据加**行级排他锁**，**事务结束释放**

4. serializable(序列化)

解决了幻读问题

读数据加**表级共享锁**;写数据加**表级排他锁**

# 乐观锁和悲观锁

## 乐观锁

获取数据不加锁

更新数据判断数据是否被修改，如果修改不更新

适合于多读的机制，可以提高吞吐量

一般使用：version方式和CAS方式

### version方式

提交版本必须大于记录当前版本才能执行更新，

### CAS方式

compare and swap或者compare and set，是一种有名无锁算法，不使用锁的情况下（线程不会被阻塞）实现多线程之间的变量同步。

涉及到三个操作数：内存值、预期值和新值

* 需要读写的内存值 V
* 进行比较的值 A
* 你写入的新值 B

CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。

```
int compare_and_swap (int* reg, int oldval, int newval) 
{
  ATOMIC(); 
  int old_reg_val = *reg;
  if (old_reg_val == oldval) 
     *reg = newval;
  END_ATOMIC();
  return old_reg_val;
}
```

## 悲观锁

每次获取数据都加锁

依赖数据库的锁机制，无法保证外部系统不会修改数据

# 脑图
[链接](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/high-performance-mysql/4.png?raw=true)
