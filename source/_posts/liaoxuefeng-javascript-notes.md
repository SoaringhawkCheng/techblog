---
title: 「廖雪峰javascript教程」笔记学习[DOING]
catalog: true
date: 2019-06-06 20:41:47
subtitle:
header-img:
tags:
- fe
categories:
- 工程
---
> 文档地址[《廖雪峰JavaScript教程》](https://www.liaoxuefeng.com/wiki/1022910821149312)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：

# 快速入门

# 函数

# 标准对象

# 面向对象编程

# 浏览器

## AJAX

	Web的运作原理：一次HTTP请求对应一个页面
	如果要让用户留在当前页面中，同时发出新的HTTP请求，就必须用JavaScript发送这个新请求。
	接收到数据后，再用JavaScript更新页面。
	这样一来，用户就感觉自己仍然停留在当前页面，但是数据却可以不断地更新。

* 同源策略

		JavaScript在发送AJAX请求时，URL的域名必须和当前页面完全一致
		
		解决方法：
			1. 通过Flash插件发送HTTP请求
			2. 同源域名下假设代理服务器转发，javascript负责把请求发到代理服务器
			e.g. /proxy?url=http://www.sina.com.cn
			3. JSONP



# jQuery

# 错误处理

# underscore

# Node.js

# React