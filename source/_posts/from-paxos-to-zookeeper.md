---
title: 「从Paxos到Zookeeper：分布式一致性原理与实践」学习笔记[DOING]
catalog: true
date: 2019-08-27 17:00:38
subtitle:
header-img:
tags:
- gateway
categories:
- 工程
---
> 书籍豆瓣链接：[《从Paxos到Zookeeper》](https://book.douban.com/subject/26292004/)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

# 前言

## 建议阅读顺序 

I. -ZooKeeper相关: 1、5、6、7章(建议中间穿插第8章阅读). 

II. -分布式一致性协议相关: 2、3、4. 

# 第1章 分布式架构

## 1.1 从集中式到分布式

### 1.1.1 集中式特点

部署结构简单（因为基于底层性能卓越的大型主机，不需考虑对服务多个节点的部署，也就不用考虑多个节点之间分布式协调问题）

### 1.1.2 分布式特点

分布式系统是一个硬件或软件组件分布在不同的网络计算机上，彼此之间仅仅通过消息传递进行通信和协调的系统

分布式的特点：

1. 分布性：在空间随意分布
2. 对等性：没有主从之分，都是对等的
3. 并发性
4. 缺乏全局时钟：很难定义两个事件谁先谁后
5. 故障总是会发生
 
### 1.1.3 分布式环境的各种问题

1. 通信异常：主要是因为网络本身的不可靠性
2. 网络分区：当网络发生异常时，导致部分节点之间的网络延时不断增大，最终导致部分节点可以通信，而另一部分节点不能。
3. 三态（成功、失败与超时）
4. 节点故障：组成分布式系统的服务器节点出现宕机或“僵死”现象

### 1.1.4 分布式事务

分布式事务是指事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于分布式系统的不同节点之上。也可以被定义为一种嵌套型的事务，同时也具有了ACID事务特性。

## 1.2 一致性理论

### 1.2.1 ACID

### 1.2.2 CAP

一个分布式系统不可能同时满足一致性(Consistency)、可用性(Availability)和分区容错性(Parition tolerance)

#### 一致性

```
A read is guaranteed to return the most recent write for a given client
对某个指定客户端，读操作保证能够返回最新的写操作结果
```

数据在多个副本之间是否能够保持一致的特性。

#### 可用性

```
A non-failing node will return a reasonable response within a reasonable amount of time (no error or timeout)
非故障的节点在合理的时间内返回合理的响应(不是错误和超时的响应)
```

系统提供的服务必须一致处于可用的状态，对于用户的每一个操作请求总是能够在有限的时间内返回结果。

#### 分区容错性

```
The system will continue to function when network patitions occur
当发生网络分区后，系统能够继续“履行职责”
```

分布式系统在遇到任何网络分区故障的时候，仍然需要能够保证对外提供满足一致性和可用性的服务，除非是整个网络环境都发生了故障。

#### CAP的矛盾

一个分布式系统不可能同时满足一致性、可用性和分区容错性这三个需求

对于一个分布式系统，P是一个最基本的需求，只能在C和A之间权衡

**CA** 优先保证一致性和可用性，放弃分区容错。 这也意味着放弃系统的扩展性，系统不再是分布式的，有违设计的初衷。

**CP** 优先保证一致性和分区容错性，放弃可用性。在数据一致性要求比较高的场合(譬如:zookeeper,Hbase)是比较常见的做法，一旦发生网络故障或者消息丢失，就会牺牲用户体验，等恢复之后用户才逐渐能访问。

**AP** 优先保证可用性和分区容错性，放弃一致性。比如数据库的主从复制，Kafka的主从复制就是这种架构。跟CP一样，放弃一致性不是说一致性就不保证了，而是逐渐的变得一致

### 1.2.3 BASE

是Basically Available（基本可用）、Soft state（软状态）和Eventually consistent（最终一致性）三个短语的简写。核心思想是即使无法做到强一致性，但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性

**基本可用** 分布式系统在出现不可预知故障的时候，允许损失部分可用性。

**弱状态** 也称软状态，允许系统在不同节点的数据副本之间进行数据同步存在延时。

**最终一致性** 系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态。

# 第2章 一致性协议

## 2.1 2PC和3PC

协调者：用来统一调度所有分布式节点的执行逻辑

参与者：被调度的分布式节点

### 2.1.1 2PC
![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/2pc.png?raw=true)

#### 流程

* 阶段一：提交事务请求（投票阶段）

	协调者节点发送事务内容，询问是否可以执行事务提交操作
	
	参与者节点执行询问发起为止的所有事务操作，并将Undo信息和Redo信息写入日志
	
	各参与者节点响应协调者节点发起的询问。YES or NO

* 阶段二A：执行事务提交（执行阶段）

	所有询问响应都为“YES”，执行事物提交：
	
	协调者向参与者发送commit请求
	
	参与者节点接受到COMMIT请求后正式完成操作，并释放在整个事务期间内占用的资源
	
	参与者节点向协调者节点发送"ACK”消息
	
	协调者节点收到所有参与者节点反馈的"ACK”消息后，完成事务

* 阶段二B：中断事务（执行阶段）

	如果任一参与者节点在第一阶段返回的响应消息为“NO”，或者协调者节点在第一阶段的询问超时之前无法获取所有参与者节点的响应消息时,那么就会中断事务：
	
	协调者向参与者发送Rollback请求
	
	参与者节点利用之前写入的Undo信息执行回滚，并释放在整个事务期间内占用的资源
	
	参与者节点向协调者节点发送"ACK”消息
	
	协调者节点收到所有参与者节点反馈的"ACK”消息后，中断事务
	
#### 分析

* 优点

	原理简单，实现方便
	
* 缺点

	**同步阻塞问题** 事务阻塞型，锁定事务资源
	
	**单点问题** 阶段二commit没有执行，参与者会处在锁定事务资源的状态
	
	**脑裂** 分区两侧保持可用的状态称为脑裂，阶段二部分commit，会造成网络分区数据不一致
	
	**太过保守** 一个节点失败导致整个事务失败，没有容错机制

### 2.1.2 3PC

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/3pc.jpg?raw=true)

#### 流程

* 阶段一：CanCommit阶段

	协调者向参与者发送CanCommit请求

	参与者响应CanCommit请求 YES or NO？

* 阶段二A：PreCommit阶段 preCommit请求

　　发送预提交请求协调者向参与者发送PreCommit请求，并进入Prepared阶段
	
* 阶段二B：PreCommit阶段 abort请求

* 阶段三A：doCommit阶段 doCommit请求

* 阶段三B：doCommit阶段 abort请求

* 故障情况

	参与者没有收到来自协调者的doCommit或abort请求，会在等待超时后，继续进行事务提交

#### 分析

* 优点

	降低了参与者的阻塞范围，出单单点故障后继续打成一致

* 缺点

	网络分区，参与者依然会进行事务提交，出现数据不一致性

## 2.2 Paxos算法

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-architecture.png?raw=true)

**Paxos** 解决的什么问题呢，解决的就是保证每个节点执行相同的操作序列

**Instance** 不断重试2PC，直到最终各方达成一致，最终确定一个值，是Paxos执行的过程，也就是一个Paxos Instance

**Multi Paxos** 重复这个过程，确定一系列值，也就是日志的每一条

### 2.2.1 概念与术语

**Proposer** 提议发起者，处理客户端请求，将客户端的请求发送到集群中，以便决定这个值是否可以被批准

**Acceptor** 提议批准者，负责处理接收到的提议，他们的回复就是一次投票，会存储一些状态来决定是否接收一个值

**Replica** 节点或者副本，分布式系统中的一个server，一般是一台单独的物理机或者虚拟机，同时承担paxos中的提议者和接收者角色

**ProposalId** 每个提议都有一个编号，编号高的提议优先级高

**acceptedProposal** 在一个Paxos Instance内，已经接收过的提议

**acceptedValue** 在一个Paxos Instance内，已经接收过的提议对应的值

**minProposal** 在一个Paxos Instance内，当前接收的最小提议值，会不断更新

**WARO** write all and read one，更新时写所有副本，只有在所有副本中更新才算成功

**Quorum** 当更新操作wi 在N个节点中的W个节点上更新成功，称之为“成功提交的更新操作”

### 2.2.2 Basic Paxos

#### 单点问题

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-single-point.png?raw=true)

Q: 如果acceptor挂掉怎么办？

A: 多个acceptors，选中被大多数acceptors的值

#### 平分提案

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-vote.png?raw=true)

Q: Acceptor只接受第一个到达的值？

A: 如果同时提出提案，不会选中任何值，所以Acceptors必须可以选择多个不同的值

#### 选择冲突

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-conflict-1.png?raw=true)

Q: Acceptor接受所有到达的值？

A: 可能会导致选择多个值，所以一旦一个值被选中，以后的提案必须提出/选择一样的值

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-conflict-2.png?raw=true)

Q: 后提出的提案被现提出的提案覆盖

A: 因此提案必须有序，拒绝旧的提案

#### 提案编号

如何产生唯一的编号呢？在《Paxos made simple》中提到的是让所有的Proposer都从不相交的数据集合中进行选择，例如系统有5个Proposer，则可为每一个Proposer分配一个标识j(0~4)，则每一个proposer每次提出决议的编号可以为5*i+j(i可以用来表示提出议案的次数)

Q: 每个提案的值递增且唯一

A: 时间戳+IP

#### 算法流程

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-basic-paxos.png?raw=true)

* prepare阶段

	**prepare请求** 
	
	**promise响应** 找到已经被选中的值，通过minProposal阻止旧的提案

* accept阶段

	**accept请求** 请求acceptors接受值

	**accepted响应**

#### 提案覆盖

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-accept-value-1.png?raw=true)

旧的值被选中，新的proposer发现并使用它

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-accept-value-2.png?raw=true)

旧的值没有被钻中，但发现被接受了，新的proposer发现并使用它

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-accept-value-3.png?raw=true)

旧的值没有被选中，也没有被发现接受了，新的proposer使用自己的值

#### 算法缺陷

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-live-lock.png?raw=true)

**活锁问题** 旧的提案被接受未被选中时，新的提案已被接受，因此旧提案不能被接受，循环往复

**性能问题** 每选中一个值，至少需要两次RTT+两次写盘

**状态同步** 被选中的日志，状态如何同步给其他机器

### 2.2.3 Multi Paxos[没理解透]

[微信团队PhxPaxos项目的wiki](https://github.com/Tencent/phxpaxos/wiki)

**Multi Paxos** 需要运行多次Paxos Instance，使用logID标识每条log日志以及对应的Paxos Instance，logID全局唯一且自增

multi-paxos协议中有一个Leader。Leader是系统中唯一的Proposal，在lease租约周期内所有提案都有相同的ProposalId，可以跳过prepare阶段，议案只有accept过程

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-multi-paxos.png?raw=true)

#### 活锁问题

为了减少并发冲突，我们可以变多写为单写，也就是选出一个Leader，只让这个Leader充当Proposer。

#### 性能问题

Multi-paxos在选出Leader之后，可以把2PC优化成1PC，也就只需要1个RTT + 1次落盘

#### 状态同步

日志项只需要复制到大多数，我们需要复制到 全部

只有 proposer 知道某个日志项被选中了，我们需要所有服务器都知道哪些日志项被选中了

![](https://github.com/SoaringhawkCheng/blog/blob/master/source/_posts/from-paxos-to-zookeeper/paxos-status-sync.jpeg?raw=true)

## ZAB算法

# 第3章 Paxos的工程世界

# 第4章 Zookeeper与Paxos

# 第5章 使用Zookeeper

# 第6章 ZooKeeper的典型应用场景

# 第7章 ZooKeeper技术内幕

# 第8章 ZooKeeper运维