---
title: 对dataframe使用自定义函数
tags: ['python','dataframe','apply','custom function']
date: 2022-05-02
categories: ['python']
---
# 1. 背景
在日常的数据处理过程中，分类聚合操作是一种常见的操作类型，而在pandas中，这种操作常通过`groupby`完成。
但是在groupby过程中，经常会有一些需要进行额外的自定义操作需要进行处理。
本文主要是为了介绍groupby过程中如何使用自定义操作对groupby之后的数据集进行对应的操作处理。

<!--more-->

# 2. 处理过程
dataframe在进行了groupby之后，可以通过自定义的操作函数，对sub_df进行指定处理。

操作过程如下图：
![apply](https://s2.loli.net/2022/05/02/8KkMsmiAPcvRU1V.png)
![progress](https://s2.loli.net/2022/05/02/jc9dPvb5zLY6oNU.png)