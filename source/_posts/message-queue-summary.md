---
title: message-queue-summary
catalog: true
date: 2020-06-28 19:57:28
subtitle:
header-img:
tags:
- mq
categories:
- 工程
---
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
[kafka面试问](https://zhuanlan.zhihu.com/p/147054382)

[RabbitMQ/RocketMQ/Kafka 消息顺序](https://blog.csdn.net/H_L_S/article/details/106582466)

顺序消息

1、当生产端是异步发送时，此时有消息发送失败，比如你异步发送了 1，2，3 消息，2 消息发送异常重试发送，这时候顺序就乱了；

2、当 Broker 宕机出现问题，此时生产端有可能会把顺序消息发送到不同的分区，这时会发生短暂消息顺序不一致的现象。