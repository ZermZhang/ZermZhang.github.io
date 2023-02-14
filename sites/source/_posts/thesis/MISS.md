---
title: MISS
tags: ['machine-learning','Multi-Interest Self-Supervised']
date: 2022-06-30 11:32
categories: ['thesis']
math: true
---
# MISS
MISS是近期华为提出的一个应用在ctr预估任务上的多兴趣自监督学习插件，能简单的插入到现有的NN模型中，并有明显的效果提升（13.55%）。
文章见《[MISS: Multi-Interest Self-Supervised Learning Framework for Click-Through Rate Prediction](https://arxiv.org/abs/2111.15068)》
该论文通过$Sample-Level\ Data\ Argumentation,\ Multi-Interest\ Data\ Argumentation,\ Interest\ view\ encoder,\ Contrastive\ Loss$等几个模块实现了用户兴趣粒度上的特征embedding的增强。从而解决了特征稀疏和特征噪音可能造成的问题。

# 1. 背景

# 2. 模型框架
![overview-for-MISS](https://raw.githubusercontent.com/ZermZhang/pictures/main/20220718165421.png)
## 2.1 样本粒度的数据增强
> 数据增强是MISS框架的第一步

对来自于$\mathcal{B} = \{x_1, x_2, \dots, x_{|\mathcal{B}|}\}$的任意一个样本$x$, 都可以通过增强函数等到两个不同的视角:
$$
\begin{align}
    & <h^1, h^2> = Aug^{s}(x) \\\\
    & Aug^{s}(\cdot): the sample-level argumentation function \\\\
    & <h^1, h2>: the pair of generated view
\end{align}
$$

* 考虑到用户行为序列的多兴趣特征，MISS框架对每个样本，从特征和兴趣层面进行了自监督学习。

### 2.1.1 多兴趣数据增强

#### MIE
首先，抽取用户的多兴趣表示
- 直观方法：将用户行为序列通过商品类目进行直接划分
- 本文方法：基于CNN的多兴趣抽取器(multi-interest extractor network)

$\tau = MIE(x) = \{t_1, t_2, \dots, t_{|\tau|}\}$
* $MIE(\cdot)$是多兴趣抽取器网络
* $\tau$是用户兴趣表征的输出
* $t_k$是第k个从x中抽取出来的兴趣表征

MIE的主要作用是发现来自用户行为序列里的潜在兴趣。当前的MIE基于一个亲密性假设：产生自相同兴趣的用户行为在用户行为序列中更有可能处在相近的位置上。

##### 技术细节
1. 使用CNN来抽取隐藏的兴趣表征，因为CNN能够较好的捕捉局部联系。
    * RNN和self-attention也可以用，但是实际表现并不太好
2. 对所有的行为序列Embedding进行padding到相同的长度$L$上
$$
\begin{gathered}
C = \begin{bmatrix} 
    e_{1,1} & e_{1,2} & \dots & e_{1,L} \\
    e_{2,1} & e_{2,2} & \dots & e_{2,L} \\
    & \vdots \\
    e_{J,1} & e_{J,2} & \dots & e_{J,L}
    \end{bmatrix}
\end{gathered}
$$
    * $C \in \mathbb{R}^{J \times L \times K}$
    * $J$: 序列型特征的数量
    * $L$: 每个序列型特征padding之后的长度
    * $K$: 每个序列型特征内的单个特征$e_{j,l}$对应的Embdding的维度
3. 根据时间轴维度的水平卷积对隐藏兴趣进行抽取
![history-sequence](https://raw.githubusercontent.com/ZermZhang/pictures/main/20220719094128.png)

### 2.1.2 兴趣层面编码
## 2.2 对比loss