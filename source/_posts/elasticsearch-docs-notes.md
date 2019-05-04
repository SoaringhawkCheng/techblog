---
title: 「Elasticsearch文档」学习笔记
catalog: true
date: 2019-04-24 10:59:21
subtitle:
header-img:
tags:
- nosql
categories:
- 编程
---
> 书籍豆瓣链接：[ES 5.2 官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/5.2/index.html)
> 
> 开始学习日期：4-1
> 
> 预计完成时间：5-4
>
> 实际完成时间：

# 入门
## 基础概念
### 集群 cluster
集群是一个或多个节点（服务器）的集合，它们共同保存您的整个数据，并提供跨所有节点的联合索引和搜索功能
### 节点 node
节点是作为群集一部分的单个服务器，存储数据并参与群集的索引和搜索功能
### 索引 index
索引是具有某些类似特征的文档集合
### 类型 type
类型是索引的逻辑类别/分区，其语义完全取决于您。 通常，具有一组公共字段的文档定义类型
### 文档 document
文档是可以被索引的基础信息单元
### 分片 shards
每个分片本身都是一个功能齐全且独立的“索引”，可以托管在集群中的任何节点上

* 水平分割/缩放内容量
* 跨分片（可能在多个节点上）分布和并行化操作

### 副本 replicas
* 分片/节点出现故障时提供高可用性。 副本永远不会和主分片在相同的节点
* 允许您扩展搜索量/吞吐量，可以在所有副本上并行执行搜索

## 集群探索
### 健康
```
GET / _cat / health?v 
GET / _cat / nodes?v
```
#### 集群健康
> * 绿色
> 
> 集群功能齐全
> 
> * 黄色 
> 
> 所有数据可用，但尚未分配一些副本
> 
> * 红色
> 
> 部分数据不可用
> 

#### 三种通讯模式
> * 单播
>
> 主机之间“一对一”的通讯模式，网络中的交换机和路由器对数据只进行转发不进行复制
>
> * 广播
>
> 主机之间“一对所有”的通讯模式，网络对其中每一台主机发出的信号都进行无条件复制并转发，所有主机都可以接收到所有信息
>
> * 多播
> 
> 主机之间“一对一组”的通讯模式，也就是加入了同一个组的主机可以接受到此组内的所有数据，网络中的交换机和路由器只向有需求者复制并转发其所需数据

# 安装与配置
## 安装
### 使用docker安装
源文档中的方法

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:5.2.2
docker run -p 9200:9200 -e "http.host=0.0.0.0" -e "transport.host=127.0.0.1" docker.elastic.co/elasticsearch/elasticsearch:5.2.2
```

java路径

```
/usr/libexec/java_home -V

export JAVA_HOME=$(/usr/libexec/java_home)
export PATH=$JAVA_HOME/bin:$PATH
export CLASS_PATH=$JAVA_HOME/lib
```

找了一篇安装elk的博客

elk是三个开源软件的缩写，分别表示elasticsearch，logstash，kibana，都是开源软件。

elasticsearch是个开源分布式搜索引擎，提供搜集、分析、存储数据三大功能

logstash是用来搜集、分析、过滤日志的工具

kibana为logstash和elasticsearch提供日志分析友好的web界面

```
docker pull sebp/elk
docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -e -it --name elk sebp/elk  # 启动elk
sudo apt-get install net-tools
ip addr show(本机一般不需要)
0.0.0.0:5044->5044/tcp, 0.0.0.0:5601->5601/tcp, 0.0.0.0:9200->9200/tcp, 9300/tcp   elk
```

docker常用操作

```
docker ps  # 查看正在运行的容器
docker ps -a  # 查看所有的容器
docker logs -f con_name  # 动态查看容器日志
docker attach con_name  # 进入容器
docker rm con_name  # 删除容器
docker images # 查看镜像
docker start con_id  # 启动容器
docker stop con_id  # 停止容器
```

# 文档接口

# 查询接口
## 请求体搜索
根目录下字段
> query 定义一个使用dsl的搜索
> 
> from,size 查询分页 from+size不能大于index.max_result_window(默认10000)
> 
> sort 

# 索引接口

## 分析器
分析器分类：索引时分析和查询时分析器
分析器结构：字符过滤器、分词器、单词过滤器

### 测试分析器
```
GET index/_analyze # 获取index的分词器分词
{
	"analyzer": "",
	"text": ,
}
GET index/_analyze # 获取index的某个字段的分词器分词
{
	"field": "",
	"text": ,
}
POST _analyze  # 测试分词器分词 
```
### 分析器详解

#### 标准分词 

#### 简单分词(simple analyzer)
只要遇到不是字母的字符， simple分析器就会将文本分成多个术语

#### TODO

# 查询DSL

将查询DSL视为查询的AST，由两种类型的子句组成：`叶子查询语句`和`复合查询语句`

## 查询和过滤上下文

查询：匹配程度`score`

过滤：是否匹配，过滤掉不匹配的

## 全文查询

# 映射

* 多重字段

有时出于不同的目的，需要使用不同的方法对同一个字段进行索引，比如一个`string`字段可以作为一个`text`字段被索引用于全文搜索，作为一个`keyword`字段用于排序和聚合。`multi-field`通过`fields`参数支持一个字段多种索引。

* 动态映射

在使用之前不需要定义字段和映射类型，通过索引文档自动添加新的映射类型和新的字段名称(顶级字段和`object`和`nested`字段里的字段都可以)

* 更新现有映射

改变现有mapping意味着已有index的文档会失效，你可以使用新的mapping创建一个新的index，而不是

* 字段在映射类型间共享

`type`用于对字段进行分组，但每种`type`的字段不是彼此独立的

相同`index`下不同`type`中的同名字段，必须具有相同的映射

## 字段数据类型

* 基本类型

	* 字符串
	
		`keywords`通常用于过滤，排序和聚合，只能按确切值搜索，如果搜索全文内容，应该使用`text`字段。在5.x中创建的索引不支持`string`，从2.x导入的索引仅支持`string`，不支持`keywords`和`text`。为了简化从2.x的迁移，Elasticsearch会将从2.x导入的索印里的`keywords`和`text`映射降级为`string`。

	* 数值

	* 日期

		```
		"strict_date_optional_time || epoch_millis"
		```

* 复杂类型

	* 对象

		json本身是分层级的，在ES内部被索引为一个扁平的键值对
		
		```
		PUT my_index/my_type/1
		{ 
		  "region": "US",  
		  "manager": { 
		    "age":     30,  
		    "name": { 
		      "first": "John",
		      "last":  "Smith"
		    }
		  }
		}
		```
		转换为
		
		```
		{
		  "region":             "US",
		  "manager.age":        30,
		  "manager.name.first": "John",
		  "manager.name.last":  "Smith"  //层级结构被以 "." 来表示。
		}
		```

	* 数组类型

		数组类型，要求数组元素的数据类型必须一致，由第一个元素的数据类型决定
		
		```
		PUT my_index/my_type/1
		{
		  "group" : "fans",
		  "user" : [ 
		    {
		      "first" : "John",
		      "last" :  "Smith"
		    },
		    {
		      "first" : "Alice",
		      "last" :  "White"
		    }
		  ]
		}
		```
		转换为
		
		```
		{
  			"group" : "fans",
  			"user.first" : [ "alice", "john" ],
		  	"user.last" : [ "smith", "white" ]

		```

	* 对象数组
	
		```
		PUT my_index/my_type/1
		{ 
		  "region": "US",
		  "manager": { 
		    "age":     30,
		    "name": [
		    { 
		      "first": "John",
		      "last":  "Smith"
		    },
		    { 
		      "first": "Bob",
		      "last":  "Leo"
		    }
		    ]
		  }
		}
		```
		转换为
		
		```		
		{
		  "region":             "US",
		  "manager.age":        30,
		  "manager.name.first": "John Bob",
		  "manager.name.last": "Smith Leo" 
		}
		// 如果我们搜索：
		"bool": {
		      "must": [
		        { "match": { "manager.name.first": "John" }},   // John Smith
		        { "match": { "manager.name.last": "Leo"}}       // Bob Leo
		      ]
		}
		// 将会导致文档被命中，显然，John Smith 、Bob Leo 两组字段它们内在的关联性都丢失了
		```

	* 嵌套(nested)

		如果需要索引对象数组并保持数组中每个对象的独立性，则应使用`nested`数据类型而不是`object`数据类型。 在内部，嵌套对象将数组中的每个对象索引为单独的隐藏文档，这意味着可以使用嵌套查询独立于其他对象查询每个嵌套对象：

		```
		PUT my_index/my_type/1
		{ 
		  "region": "US",
		  "manager": { 
		    "age":     30,
		    "name": [
		    { 
		      "first": "John",
		      "last":  "Smith"
		    },
		    { 
		      "first": "Bob",
		      "last":  "Leo"
		    }
		    ]
		}
		```
		
		转换为：
		
		```
		{
		  "region":             "US",
		  "manager.age":        30,
		  {
		      "manager.name.first": "John",
		      "manager.name.last": "Smith"
		  },
		  {
		      "manager.name.first": "Bob",
		      "manager.name.last": "Leo" 
		  }
		}
		// 如果我们搜索：
		"bool": {
		      "must": [
		        { "match": { "manager.name.first": "John" }},   // John Smith
		        { "match": { "manager.name.last": "Leo"}}       // Bob Leo
		      ]
		}
		//这样的查询将不能命中文档！！！
		```

## 元字段


### 文档标识元字段

* _id
	
	略
	
### 原文档元字段

* _source字段

	`_source`包含在索引时间传递的原始JSON文档正文，`_source`字段本身没有编入索引，但它被存储，以便在执行获取请求时可以返回它
	
### 

### 索引元字段

* _all

	当你只想返回包含某个关键字的文档，但是不明确地搜某个字段的时候就需要使用`_all`字段，`_all`字段查询的值使用空格分开
	
* _field_names

### 路由元字段

* _

* _
	
### 

##  


