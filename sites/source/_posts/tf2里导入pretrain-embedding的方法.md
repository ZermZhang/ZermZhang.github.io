---
title: tf2里导入pretrain-embedding的方法
tags: ['tensorflow','custom layers','preprocessing', 'pre-train', 'embedding']
date: 2022-04-29
categories: ['tensorflow']
---

> 在深度模型训练中，我们经常会使用到pre-train embedding，比如glove、w2v等模型的输出，尤其是在nlp和推广搜业务中。
> 面对多种多样的embdding形式，如何方便的利用就成为了一个问题，这里记录一下我用来reload embedding的方法。

<!--more-->

# 1. embedding存储方式
## 1.1 单层映射
pre-train embedding的存储方式多种多样，最常见的一种是直接存到在文件中，通过<key, vlaue>的方式进行存储。
```text
# file_name
id1\t0.1,0.2,0.3,0.4,0.5
id2\t0.3,0.4,0.5,0.3,0.5
id3\t0.2,0.4,0.2,0.3,0.4
```
该方式可以将key, value分别存储成个一个list，方便后续查找。

## 1.2 多层映射
还有多层映射的embedding存储，为了节省存储空间，可能会通过多个文件进行存储
```text
# file_name_1
id1\tattr1,attr2
id2\tattr2,attr3

# file_name_2
attr1\t0.1,0.2,0.3,0.4,0.5
attr2\t0.3,0.4,0.5,0.3,0.5
attr3\t0.2,0.4,0.2,0.3,0.4
```
这种情况下，需要存储多个list，分别是id的list，id对应的属性的list，属性对应的embedding的list。

如上两种embedding的存储方式，都可以通过tf2中已有方法进行快速的embeding查找。

# 2. 处理逻辑
> 主要处理逻辑分成两部分：
>   1. 获取需要的embedding在embedding list中的idx
>   2. 根据idx获取需要的embdding

## 2.1 获取idx
```python
class InputIdxLayer(tf.keras.layers.experimental.preprocessing.PreprocessingLayer):
    """
    获取输入特征对应的特征list中的idx信息，方便后续从embeddinglist中获取需要的embdding
    1. 直接映射
        item => idx for item in inputs_list
    2. 间接映射
        item => attr => idx for item's attr in target_list
    """
    def __init__(self, inputs_list: List, mapping_list: List = None,
                 target_list: List = None, **kwargs):
        super(InputIdxLayer, self).__init__(**kwargs)
        self.inputs_list = inputs_list
        self.mapping_list = mapping_list
        self.target_list = target_list
        print(self.inputs_list, self.mapping_list, self.target_list)
        if self.mapping_list is None:
            self.target_table = self.get_table(self.inputs_list,
                                               [i for i in range(len(self.inputs_list))])
        else:
            self.mapping_table = self.get_table(self.inputs_list, self.mapping_list,
                                                default_value='')
            self.target_table = self.get_table(self.target_list,
                                               [i for i in range(len(self.target_list))])

    @staticmethod
    def get_table(keys: List, vals: List, default_value: Any = -1):
        init = tf.lookup.KeyValueTensorInitializer(keys, vals)
        table = tf.lookup.StaticHashTable(init, default_value=default_value)
        return table

    def call(self, inputs):
        if self.mapping_table is None:
            idx_ = self.target_table.lookup(inputs)
            return idx_
        else:
            target_ = self.mapping_table.lookup(inputs)
            idx_ = self.target_table.lookup(target_)
            return idx_
```

## 2.2 获取embdding
```python
class LoadEmbeddingLayer(tf.keras.layers.Layer):
    """
    get the pre-trained embedding in embdding List
    inpust: the idx_ output from InputIdxLayer
    """
    def __init__(self, embedding, **kwargs):
        super(LoadEmbeddingLayer).__init__(**kwargs)
        self.embedding = tf.constant(embedding)

    def call(self, inputs):
        return tf.nn.embedding_lookup(self.embedding, inputs)
```