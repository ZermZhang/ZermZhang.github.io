---
title: tf2中subclassing自定义模型
tags: ['tensorflow','subclassing','save model']
date: 2022-04-25
categories: ['tensorflow']
---
# 1. tf2中的subclassing模型
tf2中常见的定义模型的方法分成三种：functional、sequenctial、subclassing

<!--more-->

其中，subclassing方法就是自己实现一个类来继承`tf.keras.Model`，来构建一个Model类的子类，在构建过程中，主要需要实现下面两个方法：
1. `__init__()`
这个方法是子类的构造器，主要是用来初始化参数（比如layers之类的）。
`super`主要用来初始化父构造器。
2. `call()`
该函数用于执行在`__init__`中定义的layers操作
同时，该函数可以使我们定义的模型直接作为函数使用，该功能主要依赖于python的`__call__`实现。

# 2. subclassing模型的例子
```python
import tensorflow as tf


class MLP(tf.keras.Model):
    def __init__(self):
        super(MLP, self).__init__()
        self.flatten = tf.keras.layers.Flatten()
        self.dense1 = tf.keras.layers.Dense(units=100, activation=tf.nn.relu)
        self.dense2 = tf.keras.layers.Dense(units=10)
    
    def call(self, inputs):
        x = self.flatten(inputs)
        x = self.dense1(x)
        x = self.dense2(x)
        output = tf.nn.softmax(x)
        return output
```

# 3. 使用summary/plot_model观察模型结构
上述的模型定义无法通过summary观察模型结构，主要是因为在定义模型的时候没有指定input_shape，只需要增加一个定义input_shape的函数就可。
```python
    def build_graph(self, input_shape):
        input_ = tf.keras.Input(shape=input_shape)
        return tf.keras.models.Model(inputs=[input_], outputs=self.call(input_))
```
增加该函数后，即可以在定义input_shape之后通过summary和plot_model直观的观察模型结构情况。
```python
# 定义模型
test_model = MLP()
```

## 3.1 summary输出
```python
# 通过summary观察模型结构
test_model.build_graph(input_shape=(16,)).summary()
```
![summary-output](../images/tensorflow/subclassing-introduce/summary-output.png)

## 3.2 plot_model输出
```python
# 通过plot_model观察模型结构
tf.keras.utils.plot_model(test_model.build_graph(input_shape=(16,)), to_file='./test_model.png', show_shapes=True)
```
![plot_model-output](../images/tensorflow/subclassing-introduce/plot_model-output.png)
