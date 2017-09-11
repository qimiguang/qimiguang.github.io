---
title: Machine-Learning-by-ANg-week3-笔记
date: 2017-09-11 10:30:04
categories:
- Machine Learning
tags:
- Machine Learning
---

本周主要介绍逻辑回归的问题

##
有监督学习中分为两类问题：regression problem & classification problem。上周聊过了 regression problem，这周来总结下 classification problem。
所谓 Classification Problem（分类问题），与Regression Problem的不同是，它的output是有限个数的、离散的值,这里的有限个数不局限在｛0，1｝两个值，是**可数**范围内的多个，如：
- email 是否为垃圾邮件
- 肿瘤是良性／恶性

分类问题不可以用Linear Regression（线性回归算法）来解决，需要用Logistic Regression（逻辑回归算法）。逻辑函数（Logistic function）也叫S型函数（Sigmoid function）。
我们先来考虑2个输出值的情况，h(x)的的输出表示：结果为1的可能性：
![](/assets/images/hxtoy.jpeg)

![](/assets/images/Logistic-Function.jpg)

![](/assets/images/week3-hx.jpg)
可以看到，当theta' * X >= 0时，h(x) >= 0.5，theta' * X = 0的这条线，也称为Decision Boundary（决策边界）
![](/assets/images/.jpg)
