---
title: Neural Network Backpropagation 入门
date: 2017-09-23 17:20:00
categories:
- Machine Learning
tags:
- Machine Learning
- Neural Network
---

上期从总体上聊了下 neural network，本篇 post 将深化学习 neural network。

## neural network cost function
当 neural network 的 output layer 有k (>2)个输出值时，h(x) 不再是一个 real number，而是一个 k 维向量，对应的 cost function 如下：
![](/assets/images/ml/week5/cost-function.jpeg)
其中正则化表达式中的 theta，包含本层的偏置单元（bias units），但不包含下层的。


![](/assets/images/ml/week5/forward-propagation.jpeg)
