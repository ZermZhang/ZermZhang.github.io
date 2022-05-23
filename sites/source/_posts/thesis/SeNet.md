---
title: SeNet
tags: ['machine-learning','Squeeze-and-Excitation','attention']
date: 2022-05-23 19:09
categories: ['thesis']
---
# SENet
SENet是在文章《[Squeeze-and-Excitation Networks](https://arxiv.org/abs/1709.01507)》中提出来的，原本的作用是为了在对图片中不同channel的关系进行建模，从而实现提升神经网络表达的作用。这种过程在文中被称作是特征矫正，即通过学习全局信息，从而有选择的强化有效信息，打压无效信息。
![SEBlock结构](https://s2.loli.net/2022/05/23/ODs4HpBNIlF8Cog.png)
上图是SEBlock的基本结构，清晰的展示了SEBlock主要由Squeeze和Excitation两部分组成。
* Squeeze结构
    $$z_c = F_{sq}(u_c) = \frac{1}{H\times W}\sum^{H}_{i=1}\sum^{W}_{j=1}u_{c}(i, j)$$
* Excitation结构
    $$s = F_{ex}(z, W) = \sigma(g(z, W)) = \sigma(W_{z}\delta(W_{1}z))$$