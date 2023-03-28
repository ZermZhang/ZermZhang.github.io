---
title: tensorflow常用计算逻辑
tags: ['tensorflow', 'method']
date: 2023-03-28 17:36
categories: ['tensorflow']
math: true
---
> 记录tensorflow代码开发过程中常用的utils方法

### 1. 计算向量的cosine相似度
```python

def cos_dis(tensor1, tensor2):
    """
    cosine相似度：是计算两个向量之间的相似度常用的方法，通过两个向量之间的夹角大小来判断相似度。夹角越小，相似度越高
    tensor1/tensor1: 维度一致的两个tensor，这里采用(n,)的标量作为介绍
    """

    # 求模长
    tensor1_norm = tf.sqrt(tf.reduce_sum(tf.square(tensor1)))
    tensor2_norm = tf.sqrt(tf.reduce_sum(tf.square(tensor2)))

    # 计算内积
    tensor1_tensor2 = tf.reduce_sum(tf.multiply(tensor1, tensor2))
    cosin = tensor1_tensor2 / (tensor1_norm * tensor2_norm)
    return cosin


def cos_dis_v2(tensor1, tensor2):
    """
    功能同上，这里先讲两个tensor进行归一化，再直接计算内积，可以省去计算模的过程
    """
    tensor1 = tf.nn.l2_normalize(tensor1)
    tensor2 = tf.nn.l2_normalize(tensor2)

    cosin = tf.reduce_sum(tf.multiply(tensor1, tensor2))
    return cosin
```