---
title: Machine Learning Neural Network入门
date: 2017-09-14 22:10:00
categories:
- Machine Learning
tags:
- Machine Learning
- Neural Network
---

本周主要介绍 Logistic Nonlinear Classification & Neural Network。

![](week4-logistic-classifier.jpeg)
之前处理 Nonlinear Classifier 时，我们一般通过引入特征的多项式。

当特征个数较少时，多项式这种做法还可行。但是许多机器学习问题涉及很多特征（如图片识别： n * m pixel(n * m features)），对众多的特征应用二项式甚至多项式时，会产生极多的项数，造成：
- 计算量过大
- 容易过拟合

神经网络在解决复杂的非线性分类问题上被证明是一种好得多的算法 即使特征空间或输入的特征维数n很大也能轻松搞定。

## Neural Network
神经网络是模仿人类大脑中的神经元网络的工作原理。神经元（neuron）包含 cell body、input wires(树突，dendrite)、output wires(轴突，axon)，信息在多个神经元之间的传递构成了整个通信网络。

神经网络首先分层，信息从一层经过**计算**后传递到下一层，直到最终输出层。每一层中的每个元素叫做 unit。
![](/assets/images/ml/week4-nn.jpeg)
![](/assets/images/ml/week4-Q.jpeg)


## Multi-class Classificationi
上周提到两个输出值的分类器。当 y = 1 if h(x) > = 0.5, y = 0 if h(x) < 0.5。
在多类别分类问题中，实际上是求解一对多神经网络算法。我们需要对每一种类别训练 classifier。当训练某个单独 classifier 时，将等于当前类别的 y 设置为 1，所有不等于当前类别的全部设置为 0。

![](/assets/images/ml/week4-multi-out-nn.jpeg)

