---
title: 「统计学习方法」学习笔记[TODO]
catalog: true
date: 2019-08-18 19:09:06
subtitle:
header-img:
tags:
- ml
categories:
- 算法
---
> 书籍豆瓣链接：[《统计学习方法》](https://book.douban.com/subject/10590856/)
> 
> 开始学习日期：8-1
> 
> 预计完成时间：9-1
>
> 实际完成时间：

# SVM

[什么是P问题、NP问题和NPC问题](http://www.matrix67.com/blog/archives/105)

[一文详解凸函数和凸优化，干货满满](https://blog.csdn.net/feilong_csdn/article/details/83476277)

一元和多元函数，可导，可微，可积，连续之间的关系还没搞清楚

[如何理解拉格朗日乘子法](https://www.matongxue.com/madocs/939.html)

[如何理解KKT条件](https://www.zhihu.com/question/23311674/answer/235256926)

1. 满足所有约束条件的n维空间成为**可行域**，函数是局部最大值意味着在可行域内的可行方向上，目标函数不能增大
2. 等式约束，可行方向是平面。不等式约束，当约束函数小于0时，可行方向是全空间，当约束函数等于0时，可行方向是切面和负梯度方向
3. 满足多个约束的点的可行方向是每个约束的可行方向的交集
4. KKT条件等价于函数增大的方向不在可行方向上

对偶算法 《统计学习方法》P227的三个理论没看

