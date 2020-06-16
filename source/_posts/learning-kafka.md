---
title: 「深入理解Kafka&其他消息中间件」学习笔记[TODO]
catalog: true
date: 2020-05-20 12:08:36
subtitle:
header-img:
tags:
- mq
categories:
- 工程
---
> 书籍豆瓣链接：[《深入理解Kafka：核心设计与实践原理》](https://book.douban.com/subject/30437872/)
> 
> 相关资料笔记：
> [基础篇](https://blog.csdn.net/wk52525/article/details/96985534)
> [进阶篇](https://blog.csdn.net/wk52525/article/details/98070770)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

## 灵魂三问

1. 为什么使用消息队列？
2. 消息队列有什么优点和缺点？
3. Kafka、ActiveMQ、RabbitMQ、RocketMQ都有什么区别，以及适合哪些场景？

解耦、异步、削峰

[消息队列面试题](https://www.jianshu.com/p/4491cba335d1)

## 第1章 初始Kakfa

### Kafka体系结构

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/kafka-architecture.png?raw=true)

服务代理节点broker：独立的服务节点或kafka实例

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/partition.png?raw=true)

topic包含多个分区，partition包含的消息不同，**kafka引入分区解决机器IO性能瓶颈**

分区是kafka的最小并行操作单元，单个partition数据写入可以并行化，partition消费只能被一个consumer线程消费

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/broker.png?raw=true)

**kafka为分区引入多副本机制，提升容灾能力**

一个topic可以横跨多个broker，每个broker包含全部partition副本

producer和consumer只和leader副本进行交互，follower只负责同步，消息相对滞后

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/offset.png?raw=true)

分区中所有副本AR，同步中副本ISR，同步滞后副本OSR，AR=ISR+OSR

**LEO(Log End Offset)** 标识当前分区下一条待写入消息的offset

**HW(High Watermark)** 高水位，标识了一个特定的offset，消费者只能拉渠道这个offset之前的消息（不含HW）

只有ISR集合的副本才有资格被选为leader，ISR集合中最小的LEO即为分区的HW

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/offset-2.png?raw=true)

Kafka使用ISR有效权衡了数据可靠性和性能，既不是完全同步复制，也不是淡出异步复制

## 第2章 生产者

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/learning-kafka/producer-architecture.png?raw=true)

## 第3章 消费者


每个分区能被一个消费组的一个消费组消费

## 第4章 主题与分区



## 第5章 日志存储



## 第6章 深入服务端

## 第7章 深入客户端

## 第8章 可靠性研究

## 第9章 Kafka应用

## 第10章 Kafka监控

## 第11章 高级应用