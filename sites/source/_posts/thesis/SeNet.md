---
title: SENet
tags: ['machine-learning','Squeeze-and-Excitation','attention']
date: 2022-05-23 19:09
categories: ['thesis']
math: true
---
# SENet
SENet是在文章《[Squeeze-and-Excitation Networks](https://arxiv.org/abs/1709.01507)》中提出来的，原本的作用是为了在对图片中不同channel的关系进行建模，从而实现提升神经网络表达的作用。这种过程在文中被称作是特征矫正，即通过学习全局信息，从而有选择的强化有效信息，打压无效信息。

# 1. SEBlock 结构

![SEBlock结构](https://s2.loli.net/2022/05/23/ODs4HpBNIlF8Cog.png)
上图是SEBlock的基本结构，清晰的展示了SEBlock主要由Squeeze和Excitation两部分组成。
* Squeeze结构
    $$z_c = F_{sq}(u_c) = \frac{1}{H\times W}\sum_{i=1}^{H}\sum_{j=1}^{W}u_{c}(i, j)$$
    * 通过指定维度，对特征进行聚合，产生channel上的描述
    * 原文中的squeeze结构是通过一个平均函数生成的，计算每个channel上的embedding信息的平均值，作为当前channel的代表

* Excitation结构
    $$
    \begin{align}
    & s = F_{ex}(z, W) = \sigma(g(z, W)) = \sigma(W_{2}\delta(W_{1}z))  \tag{1} \\\\
    & W_{1} \in\mathbb{R}^{\frac{C}{r}\times C} \quad and \quad W_{2} \in \mathbb{R}^{C \times \frac{C}{r}}  \tag{2} \\\\
    & \quad \\\\
    & x_c = F_{scale}(\textbf{u}_c, s_c) = s_c\textbf{u}_c \tag{3}
    \end{align}
    $$
    * 通过一个简单的门机制来计算每个channel对应的权重信息
    * 为了捕捉每个通道的依赖关系，学习的函数需要满足两个条件：
        1. 必须是可伸缩的，必须能够学习两个channel之间的非线性关系
        2. 不能是相互排斥的。需要能够同事对多个channel进行强化，而不能是简单的针对一个channel的单独强化
    
# 2. SEBlock 使用
![SEBlock使用](https://s2.loli.net/2022/05/26/7fjD5Y9QhMwHGsB.png)
SE block的可伸缩性意味着它可以直接的应用到各类映射上。
如上图，是两种典型的SE block的使用场景。在Inception网络中，可以直接对整个Inception模块应用SE Block。对于ResNet，可以将SE Block嵌入到每个残差结构的残差分支当中。

# 3. SEBlock在ctr任务中的使用
SE Block的原始作用是为了解决图片中不同channel的权重强化问题，同样，我们可以将这个逻辑应用到ctr任务中，用来出来对不同的特征域的强弱关注度的学习使用中。
![seblock-for-ctr](https://s2.loli.net/2022/05/26/WrLVdEncBS49JqI.png)
使用逻辑：
1. 针对每个输入features生成input_embedding（经过preprocessing之后的embedding信息）
2. 针对之前的输入进行SEBlock的处理，这里可以将原文中的channel概念替换为特征域进行Global pooling的处理
3. 通过SEBlock之后，会生成一个和特征域个数相同维度weighted向量
4. 将weighted向量分别加权到对应的特征域之后，将结果作为input信息输入到NN中。

# 4. 代码实现
```python
class SeBlockLayer(tf.keras.layers.Layer):
    """
    the SeBlock layer for features
    """
    def __init__(self, out_dim, ratio=4.0, *args, **kwargs):
        super(SeBlockLayer, self).__init__(*args, **kwargs)

        self.out_dim = out_dim
        self.mid_dim = tf.math.ceil(out_dim / ratio)
        self.ratio = ratio

        self.avg_weights = []

    def build(self, input_shape):
        super(SeBlockLayer, self).build(input_shape)

    def call(self, feature_embs):
        # 对每个特征域进行global pooling
        for feature_emb in feature_embs:
            self.avg_weights.append(tf.reduce_mean(feature_emb))
        se_inputs = tf.convert_to_tensor([self.avg_weights])
        # FC层
        outputs = tf.keras.layers.Dense(self.mid_dim)(se_inputs)
        # relu激活
        outputs = tf.nn.relu(outputs)
        # FC层
        outputs = tf.keras.layers.Dense(self.out_dim)(outputs)
        # sigmoid激活
        outputs = tf.nn.sigmoid(outputs)
        # scaled
        se_outputs = feature_embs * outputs

        return se_outputs
```