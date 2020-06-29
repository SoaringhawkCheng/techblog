---
title: 「RabbitMQ实战指南」学习笔记
catalog: true
date: 2020-06-18 15:43:55
subtitle:
header-img:
tags:
- mq
categories:
- 工程
---
> 书籍豆瓣链接：
> [《RabbitMQ实战指南》](https://book.douban.com/subject/27591386/)
> 
> 相关资料链接：
> 
> [《RabbitMQ中文文档》](http://rabbitmq.mr-ping.com/)
> 
> [学习笔记-1](https://blog.csdn.net/weixin_43064185/article/details/101908977)
> [学习笔记-2](https://blog.csdn.net/weixin_43064185/article/details/101981701)
> [学习笔记-3](https://blog.csdn.net/weixin_43064185/article/details/102064857)
> [学习笔记-4](https://blog.csdn.net/weixin_43064185/article/details/102261479)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

## RabbitMQ简介

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/rabbitmq-guide/01-RabbitMQ.png?raw=true)

### 消息中间件作用

消息中间件凭借其独到的特性，在不同的应用场景下可以展现不同的作用 。总的来说，消息中间件的作用可以概括如下。

解耦:在项目启动之初来预测将来会碰到什么需求是极其困难的。消息中间件在处理过程 中间插入了一个隐含的、基于数据的接口层，两边的处理过程都要实现这一接口，这允许你独 立地扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束即可 。

冗余〈存储) : 有些情况下，处理数据的过程会失败。消息中间件可以把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。在把一个消息从消息中间件中删 除之前，需要你的处理系统明确地指出该消息己经被处理完成，从而确保你的数据被安全地保 存直到你使用完毕。

扩展性:因为消息中间件解捐了应用的处理过程，所以提高消息入队和处理的效率是很容易的，只要另外增加处理过程即可，不需要改变代码，也不需要调节参数。

削峰: 在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常 见。如果以能处理这类峰值为标准而投入资源，无疑是巨大的浪费 。 使用消息中间件能够使关 键组件支撑突发访问压力，不会因为突发的超负荷请求而完全崩惯 。

可恢复性: 当系统一部分组件失效时，不会影响到整个系统 。 消息中间件降低了进程间的 稿合度，所以即使一个处理消息的进程挂掉，加入消息中间件中的消息仍然可以在系统恢复后 进行处理 。

顺序保证:在大多数使用场景下，数据处理的顺序很重要，大部分消息中间件支持一定程度上的顺序性。

缓冲: 在任何重要的系统中，都会存在需要不同处理时间的元素。消息中间件通过一个缓 冲层来帮助任务最高效率地执行，写入消息中间件的处理会尽可能快速 。 该缓冲层有助于控制 和优化数据流经过系统的速度。

异步通信: 在很多时候应用不想也不需要立即处理消息 。 消息中间件提供了异步处理机制， 允许应用把一些消息放入消息中间件中，但并不立即处理它，在之后需要的时候再慢慢处理 。

## RabbitMQ入门

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/rabbitmq-guide/02-RabbitMQ.png?raw=true)

## 客户端开发向导

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/rabbitmq-guide/03-RabbitMQ.png?raw=true)

## RabbitMQ进阶

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/rabbitmq-guide/04-RabbitMQ.png?raw=true)