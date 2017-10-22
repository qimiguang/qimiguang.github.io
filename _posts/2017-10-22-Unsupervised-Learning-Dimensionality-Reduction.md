---
title: Unsupervised Learning：Dimensionality Reduction
date: 2017-10-22 13:50:00
categories:
- Machine Learning
tags:
- Machine Learning
- Unsupervised Learning
---

上周我们谈到了无监督学习中的 <span style="color:red">Clustring</span> 话题，今天我们来聊聊无监督学习另一个话题： <span style="color:red">Dimensionality Reduction</span>（维度约减／降维）。

一般来说，降维的使用场景有：
1. Data Compression
2. Visualization

其中 Data Compression 可以：

* 更快的算法速度
* 更小的内存 & 硬盘占用

当向量超过 3 维时，我们已经不能很直观的观看数据了。所以，可以通过降维到 3 维以内来 Visualization。

下图是一个将二维特征降维到一维的例子：
![](/assets/images/ml/week8/data-compression.jpeg)

<!-- more -->
# Principal Componet Analysis (PCA)
目前最常用的降维算法是 <span style="color:red">PCA（主成分分析法）</span>。将一个 n 维特征降维到 k 维时，PCA 的目的是寻找到一个合适的 k 维向量(u<sup>1</sup>、u<sup>2</sup>...u<sup>k</sup>)，使得原始数据的 n 维向量投影到 k 维向量的平均 projection error（投影误差） 最小。

在执行 PCA 算法前，我们需要先进行 **data preprocessing**。所谓 data preprocessing，即 **mean normalization（均值归一化） & feature scaling（特征缩放）**：
![](/assets/images/ml/week8/mean-scale.jpeg)
其中 μ 表示均值，而 s 一般用方差来特征缩放。

接下来就是 PCA 的主要步骤了：
## 1. Compute covariance matrix
covariance matrix（协方差矩阵），数学符号中用 Σ（sigma）表示，当原始特征是 n 维时，Σ 是 n * n 维向量：
![](/assets/images/ml/week8/sigma.jpeg)

## 2. Compute "eigenvectors" of covariance matrix Σ
接下来要做的是计算 **Sigma 矩阵的特征向量 (eigenvectors)**。我们可以直接调用 svd function or eig function 来实现。

svd 表示奇异值分解 (singular value decomposition):
![](/assets/images/ml/week8/u-matrix.jpeg)
它输出的 U ∈ ℝ<sup>n*n</sup>

## 3. Take the first k columns of the U matrix and compute z
当我们想要将特征从 n 维降到 k 维时，从 U 中取前 k 列， 即一个 n*k 矩阵，并用 U<sub>reduce</sub> 来表示。

我们用 z<sup>(i)</sup> 来表示 x<sup>(i)</sup> 降维后的值，这个值可以通过如下计算得到：
**z<sup>(i)</sup> = U<sub>reduce</sub><sup>T</sup> ⋅ x<sup>(i)</sup>**

总结来看，整个过程如下：
![](/assets/images/ml/week8/pca.jpeg)

# Reconstruction from Compressed Representation
PCA 可以将 n 维数据压缩成 k 维，那么如何将 k 维还原成 n 维呢？
**x<sup>(i)</sup><sub>approx</sub> = U<sub>reduce</sub> ⋅ z<sup>(1)</sup>**
当然了，还原后的 x 只能近似于压缩前的 x，而做不到完全等于。

# Choosing the Number of Principal Components
理论上 k 值越小，压缩程度越高，size 越小，算法运行越快。但是随着 k 的变小，数据失真度会变高（x<sup>(i)</sup> - x<sup>(i)</sup><sub>approx</sub> 越大）。那么我们如何选择这个 k(the number of principal components) 值呢？

有一个常用的公式：**平均平方误差**，如下图表明平均平方误差 < 0.01，即 99% 的差异度被保留：
![](/assets/images/ml/week8/num-pca.jpeg)

所以我们的目标就是，找出一个最小的 k，使得平均平方误差 < 0.01：
1.	Try PCA with k=1,2,…
2.	Compute U<sub>reduce</sub>, z, x
3.	Check the formula given above that 99% of the variance is retained. If not, go to step one and increase k.

好消息是，svd 函数提供了一种更简单的方式求解这个最小 k 值。svd 返回的第二个值 S 是一个 n*n 的[对角矩阵](https://zh.wikipedia.org/wiki/%E5%B0%8D%E8%A7%92%E7%9F%A9%E9%99%A3)，我们可以通过下面的计算来求解 k：
![](/assets/images/ml/week8/svd-s.jpeg)


# Advice for Applying PCA
PCA 其实是一种非常有效的手段来提高 supervised learning 的算法速度。既然它的使用场景是 supervised learning，那为什么我们把它当成一种 unsupervised learning algorithm 呢？回顾下 unsupervised learning 的定义：
```Unsupervised machine learning is the machine learning task of inferring a function to describe hidden structure from "unlabeled" data.```

非监督学习区别与监督学习最大的特点在于 **unlabeled data**，PCA 在计算过程中是完全忽略 y 值的存在的，它是完全基于特征向量进行降维，只不过将降维后的特征值结合 y 来进行监督学习。

PCA 虽然有很多好处，但是我们也不应该滥用它，譬如拿它来避免 overfit。在开始一段 ML 算法时，我们应该首先按照正常流程来处理，当发现问题（存在大量相似特征、特征太大导致运算缓慢）时，再考虑使用 PCA！
 






