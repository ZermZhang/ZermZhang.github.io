---
title: SENet
tags: ['machine-learning','Squeeze-and-Excitation','attention']
date: 2022-05-23 19:09
categories: ['thesis']
math: true
---
# SENet

SENet是在文章《[Squeeze-and-Excitation Networks](https://arxiv.org/abs/1709.01507)》中提出来的，原本的作用是为了在对图片中不同channel的关系进行建模，从而实现提升神经网络表达的作用。这种过程在文中被称作是特征矫正，即通过学习全局信息，从而有选择的强化有效信息，打压无效信息。
![SEBlock结构](https://s2.loli.net/2022/05/23/ODs4HpBNIlF8Cog.png)
上图是SEBlock的基本结构，清晰的展示了SEBlock主要由Squeeze和Excitation两部分组成。

* Squeeze结构

  $$
  c = F_{sq}(u_c) = \frac{1}{H\times W}\sum_{i=1}^{H}\sum_{j=1}^{W}u_{c}(i, j)

  $$

  * 通过指定维度，对特征进行聚合，产生channel上的描述
  * 原文中的squeeze结构是通过一个平均函数生成的，计算每个channel上的embedding信息的平均值，作为当前channel的代表
* Excitation结构

  $$
  begin{align}
  & s = F_{ex}(z, W) = \sigma(g(z, W)) = \sigma(W_{2}\delta(W_{1}z))  \tag{1} \\\\
  & W_{1} \in\mathbb{R}^{\frac{C}{r}\times C} \quad and \quad W_{2} \in \mathbb{R}^{C \times \frac{C}{r}}  \tag{2}
  \end{align}
  $$

  * 通过一个简单的门机制来计算每个channel对应的权重信息
