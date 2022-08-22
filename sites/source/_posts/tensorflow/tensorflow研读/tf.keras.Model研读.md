---
title: tf.keras.Model研读
tags: ['tensorflow','Model','Module']
date: 2022-07-22 16:28
categories: ['tensorflow']
---
> Model模块可以将Layer组合成具有针对特征的训练的推演功能的对象。

# 1. 简单介绍
## 1.1 输入参数
|参数名称|参数说明|
|---|---|
|inputs|模型的输入，可以是单个的`tf.keras.Input`也可以是一个由`tf.keras.Input`组成的List|
|outputs|模型的输出|
|name|模型的名称，string类型|

## 1.2 使用方法

### 1.2.1 Functional API
> 将使用到的层按照模型前序的顺序构建好后，在最后从输入到输出构建需要的模型.

```python
import tensorflow as tf

inputs = tf.keras.Input(shape=(3,))

x = tf.keras.layers.Dense(4, activation=tf.nn.relu)(inputs)
outputs = tf.keras.laters.Dense(5, activation=tf.nn.softmax)(x)

model = tf.keras.Model(inputs=inputs, outputs=outputs)
```


### 1.2.2 SubClassing API
> 使用subclassing api的时候，可以在`__init__()`中定义将要使用到的layer，并在`call()`中定义模型的前馈逻辑.

```python
import tensorflow as tf


class MyModel(tf.keras.Model):
    def __init__(self):
        super().__init__()
        self.dense1 = tf.keras.layers.Dense(4, activation=tf.nn.relu)
        self.dense2 = tf.keras.layers.Dense(5, activation=tf.nn.softmax)
        self.dropout = tf.keras.layers.Dropout(0.5)

    def call(self, inputs, trainging=False):
        x = self.dense1(inputs)
        if training:        # 通过training参数对训练和interface过程中的不同逻辑进行控制
            x = self.dropout(x, training=training)
        return self.dense2(x)
    
model = MyModel()
```

# 2. 常用方法

## 2.1 call
```
call(inputs, training=None, mask=None)
```
|参数名称|参数说明|
|--|--|
|inputs|模型输入，可以是由tensor组成的dict/list/tuple|
|training|Boolean值，对网络是进行training或者是interface进行区分|
|mask|网络中可能会用到的mask|


## 2.2 compile
```
compile(optimizer='rmsprop',
        loss=None,
        metrics=None,
        loss_weights=None,
        weighted_metrics=None,
        run_eagerly=None,
        steps_per_execution=None,
        jit_compile=None,
        **kwargs)
```
|参数名称|参数说明|
|--|--|
|optimizer|优化器|
|loss|损失函数|
|metrics|评估指标|

### 作用过程
