---
title: Support Vector Machine 简介
date: 2017-10-11 23:50:00
categories:
- Machine Learning
tags:
- Machine Learning
- Support Vector Machine
---

之前我们已经介绍了逻辑回归、神经网络等机器学习算法，今天来聊聊另一种非常常用的算法：Support Vector Machine（支持向量机）。
首先我们来回顾下逻辑回归问题。对于逻辑回归，预测函数如下，当z >> 0时，h(x)≈1，当 z << 0时，h(x) ≈ 0.
![](/assets/images/ml/week7/hx.jpeg)
对于单个 example 的 cost function 如下：
![](/assets/images/ml/week7/1examplecostfunction.jpeg)

这里有点值得注意，在 SVM 算法中，单个 example 的 cost function 从原先的曲线变成了直线，这样可以带来更高的效率.
普通的逻辑回归算法的 cost function 为：
![](/assets/images/ml/week7/lrcostfunction.jpeg)
SVM 算法的 cost function 为：
![](/assets/images/ml/week7/svmcostfunction.jpeg)
需要注意：
* 由于 m （example count）是常量，所以在 SVM 中习惯删掉
* SVM 中单个 example 的 cost function 为 cost1、cost2，即前面提到的两条相交的直线
* SVM 中 C = 1 / λ

# Large Margin Classifier
有些人习惯将 SVM 称作 Large Margin Classifier（大间距分类器），为什么这么说呢，因为在普通的逻辑回归算法中，当 y ＝ 1 时，我们仅仅要求 θ'X ≥ 0，当 y ＝ 0 时，要求 θ'X ≤ 0。但是 SVM 要求θ'X ≥ 1 ／θ'X ≤ －1，即更大程度的区分度：
![](/assets/images/ml/week7/svmmargin.jpeg)
![](/assets/images/ml/week7/svmlc.jpeg)

这里的 C 是个关键因子，考虑如下图所示，如果数据集中出现一个异常点，当 C 设置很大时(λ 很小)，会发生 overfit 现象，所以我们需要谨慎的设置 C：
![](/assets/images/ml/week7/svmexp.jpeg)

## Mathematics Behind Large Margin Classification
在数学上，两个向量的内积公式如下：u'v = v'u = p⋅||u|| = u1⋅v1 + u2⋅v2。其中 p = length of projection of v onto the vector u。需要记住：If the angle between the lines for v and u is greater than 90 degrees, then the projection p will be negative.

现在再来看 SVM 的 cost function：
![](/assets/images/ml/week7/svmcf.jpeg)
最右侧的正则化公式可以变形为：
![](/assets/images/ml/week7/theta2.jpeg)
为了降低 cost function 的值，左侧的公式可变形为：
![](/assets/images/ml/week7/thetax.jpeg)

> 我们反过来看，为了降低 full cost function，右侧的正则化参数不能太大，即** Θ 不能太大**，θ 较小，但是 p . ||Θ|| 又需要大，则 p 只能较大，即** X 映射到 Θ 上的长度较大**，所以 decision boundaries 只能尽可能的原理正负样本集，这也正式 SVM 叫做 Large Margin Classifier 的原因。
![](/assets/images/ml/week7/margin.jpeg)

# Kernels


![](/assets/images/ml/week7/)

# logistic regression

