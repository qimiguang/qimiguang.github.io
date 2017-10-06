---
title: Evaluating a Learning Algorithm
date: 2017-10-06 15:10:00
categories:
- Machine Learning
tags:
- Machine Learning
---

在我们初步实现了机器学习算法后，可能会发现结果不尽如人意。训练集上运行很好的算法参数（cost function 很低），却在预测新数据（new examples）时误差很大（cost function 很高）。本篇博客主要关于评估学习算法的质量以及找出可能存在的原因。

# Evaluating a Learning Algorithm
为了评估学习算法的优劣，我们首先将数据集分成三部分：training set \ cross validation set \ test set。三者的比例一般是 60% : 20% : 20%。

## degree of polynomial
作为进行对比，不同的算法可以拥有不同的多项式次数。很多情况下，一次项不能很好的 match 数据，需要用到多项式，但是具体几项式最好，需要我们用 training set & cross validation set & test 来评估。具体如下：
![](/assets/images/ml/week6/polynomial.jpeg)

# Bias vs Variance
在之前的帖子中我们谈到过 bias & variance。bias or variance 是算法质量较差的不同极端，其中 high bias（高偏差）是指 underfitting，而 high variance（高方差）是指 overfitting。
随着多项式项数的增加，training set / cross validation 的cost function 呈不同的走势：
![](/assets/images/ml/week6/polynomial-degree.jpeg)
可以看到，当 polynome degree 位于某个**中间值**时，cross validation 的 cost function 最小。
![](/assets/images/ml/week6/bias-variance.jpeg)

# Regularization and Bias/Variance
当多项式的项数较高时，随之而来的是 overfit 现象，为了一定程度上消除 overfit，我们可以引入 regularization (正则化)。正则化中一个重要的参数是 lamda。该参数既不能过大（high bias/ underfit），也不能过小（high variance / overfit）。那么如何选择这个 lamda 呢？
![](/assets/images/ml/week6/choose-lamda.jpeg)


