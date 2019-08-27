---
title: 机器学习笔记
catalog: true
date: 2019-08-18 19:09:06
subtitle:
header-img:
tags:
- ml
categories:
- 编程
---

# SVM

[什么是P问题、NP问题和NPC问题](http://www.matrix67.com/blog/archives/105)

[一文详解凸函数和凸优化，干货满满](https://blog.csdn.net/feilong_csdn/article/details/83476277)

一元和多元函数，可导，可微，可积，连续之间的关系还没搞清楚

[如何理解拉格朗日乘子法](https://www.matongxue.com/madocs/939.html)

[如何理解KKT条件](https://www.zhihu.com/question/23311674/answer/235256926)


1. 
2. 满足所有约束条件的n维空间成为**可行域**，函数是局部最大值意味着在可行域内的可行方向上，目标函数不能增大
3. 等式约束，可行方向是平面。不等式约束，当约束函数小于0时，可行方向是全空间，当约束函数等于0时，可行方向是切面和负梯度方向
4. 满足多个约束的点的可行方向是每个约束的可行方向的交集
5. KKT条件等价于函数增大的方向不在可行方向上
