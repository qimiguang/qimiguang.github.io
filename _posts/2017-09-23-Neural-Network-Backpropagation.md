---
title: Neural Network Back Propagation 入门
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

## back propagation
现在我们有了 cost function，接下来需要求解 cost function 的导数，今天介绍一种新方法：back propagation。
在介绍 back propagation 前，先来介绍一个新的概念 **delta**: "error" of node j in layer。对于输出层来说，delta ＝ a - y. (a: h(x), y: actual output)
![](/assets/images/ml/week5/deltaL.jpeg)

所谓向后传播，即在求解 delta 时，先求解最后一层（输出层）的 delta，然后以此 back propagation 求解每一层的 delta:
![](/assets/images/ml/week5/delta.jpeg)

## back propgation algorithm
![](/assets/images/ml/week5/back-propagation-algorithm.jpeg)
详情可参考：[](https://www.coursera.org/learn/machine-learning/supplement/pjdBA/backpropagation-algorithm)

