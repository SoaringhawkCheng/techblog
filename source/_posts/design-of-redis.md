---
title: 「Redis设计与实现」学习笔记[DOING]
catalog: true
date: 2019-12-24 14:38:09
subtitle: 
tags: 
- cache
categories:
- 工程
---
> 书籍豆瓣链接：[《Redis设计与实现》](https://book.douban.com/subject/25900156/)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

# 数据结构与对象

## 简单动态字符串sds
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/1.png?raw=true)


```
1. 记录字符串长度len，常数时间将获得字符串长度
2. 杜绝缓冲区溢出，对sds修改时先检查空间是否满足修改所需的要求
3. 减少内存重分配次数
	空间预分配，空间扩展的时候，额外分配内存len(sds)>=1MB?1MB:len(sds)
	惰性空间释放，free属性记录空闲空间，提供api释放空间
4. 二进制安全，sds api以二进制的方式处理sds存放在buf数组里的数据
```
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/2.png?raw=true)
## 链表list

```
双端，prev和next指针 
无环，prev和next指针指向null
带表头表尾指针，head tail
带链表长度计数器，len
多态，通过void*指针来实现
```
## 字典dictht
hashtable

```
槽数量size，掩码sizemask=size-1，used槽使用数量,
负载因子 load_factor=used/size
哈希表ht包含两个hashtable，ht[1]用于rehash
```
rehash

```
时间
1. if BGSAVE和BGREWRITEAOF && 负载因子>=5
2. if not BGSAVE和BGREWRITEAOF && 负载因子>=1
3. 负载因子<=0.1
分配空间
1. 扩展操作，ht[1].size=第一个大于等于ht[0].used*2的2**n
2. 收缩操作，ht[1].size=第一个大于等于ht[0].used的2**n
```

渐进式rehash

```
步骤
1. 新分配的一个table ht[1]，维持一个索引计数器rehashidx，初始值为0，rehash工作正式开始
2. 每次对字典执行crud，将当前rehashidx索引上的所有键值对rehash，结束后rehashidx增一
3. rehash完成时，rehashidx属性的值-1
期间
1. create ht[1]
2. delete ht[0]和ht[1]都删除
3. retrieve 先ht[0]后ht[1]
4. update ht[0]和ht[1]都更新
```

### 
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/3.png?raw=true)

## 跳跃表skiplist

表数据结构

```
head tail
level 层数
length 跳跃表长度
头结点 长度为32的level数组
```

节点数据结构

```
level数组
	长度1-32随机
	前进指阵，跨度，跨度用于计算rank
backward指针
分值
成员
```

## 整数集合intset

```
contents 申明为int8数组
encoding 保存数组真正类型
contents item按从小到大排列
```

## 压缩列表ziplist

表数据结构

```

```

节点数据结构

```
previous_length 1 or 5字节
encoding 
content 
```

## 对象
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/4.png?raw=true)

对象数据结构

```
type
encoding
ptr 指向底层实现
```

字符串对象

```
int ptr指向long
raw
embstr object和sdshdr在连续空间，只读，如果要写，从embstr转换成raw
```

列表对象

```

```

对象共享

```
初始化服务器，创建0-9999，1万个对象
```

# 单机数据库
## 数据库
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/5.png?raw=true)

数据库

```
redisDb *db 维护一组数据库 select 下标选择
```

键空间

```
读写键空间的维护操作

读
更新键空间hit和miss次数
更新键的LRU时间
发现键过期，删除过期键

写
如果客户端用WATCH命令监视这个键，将键标记为dirty
修改键后dirty计数增1
```

过期删除策略

```
定时删除：设置键过期时间同时，创建定时器，定时删除
惰性删除：每次从键空间获取键，如果过期就删除，没过期就返回
定期删除：每隔一段时间，批量删除过期键
```

过期键处理

```
RDB
生成RDB文件，已过期的键不会被保存
载入RDB文件，主服务器

AOF


复制


```

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/6.png?raw=true)
## RDB持久化

```
BGSAVE派生出一个子进程，然后子进程负责创建RDB文件
服务器启动，AOF更新频率高，优先使用RDB文件还原数据库
serviceCon每个一段时间执行一次，根据dirty和lastsave是否满足条件，执行BGSAVE
```

## AOF持久化 

AOF持久化步骤

```
1. 命令追加
服务器实时将写命令追加到服务器状态的aof_buf缓冲区末尾

2. 文件写入
每次结束一个事件循环，会调用flushAppendOnlyFile，考虑是否将aof_buf写入到AOF文件

3. 文件同步
write只会更新页缓存，脏页的更新取决于os，系统提供fsync和fdatasync两个同步函数
appendfsync配置
	always 每个事件循环写入并同步
	everysec 每个事件循环写入，每个1s同步
	no 每个事件循环写入，同步由os
```

## 事件
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/7.png?raw=true)
## 服务器
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/8.png?raw=true)
# 多机数据库
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/9.png?raw=true)
## 复制
## 哨兵

## 集群
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/10.png?raw=true)

# 独立功能
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/design-of-redis/11.png?raw=true)
## 发布与订阅

## 事务

## 慢查询日志