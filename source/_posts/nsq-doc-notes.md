---
title: 「NSQ文档」学习笔记[DOING]
catalog: true
date: 2019-07-14 16:03:38
subtitle:
header-img:
tags:
- mq
categories:
- 工程
---
> 书籍豆瓣链接：[NSQ 1.1.0文档](https://nsq.io/overview/design.html)
> 
> 开始学习日期：7-14
> 
> 预计完成时间：7-20
>
> 实际完成时间：

nsqd实例处理多个数据流，即topic，一个topic有多个channel

topic->channel是多播的，channel->consumers，每个consumer接受消息的一部分

consumer通过nsqlookupd查找topic所在的nsqd实例地址

每个nsq与nsqlookupd有一个TCP长连接，并定期将状态推送给nsqlookupd

nsqadmin提供一个web ui管理topic，channel，consumer的层次结构

nsq保证消息至少传送一次

收到的消息是无序的